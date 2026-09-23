# PLAN: Medi-Mops Sensorintegration — Hardware-Schicht

**Status:** Konzept — parallel zum Rust-Kernel entwickelbar
**Ziel:** Vier Kanäle, eine Ampel. Keine eigene Hardware bauen.
         Bestehende Sensoren als Datenlieferanten, der Mops als Hirn.

---

## Prinzip: Der Mops baut keine Sensoren. Er liest sie.

```
┌─────────────────────────────────────────────────────────┐
│                    SENSORSCHICHT                         │
│                                                         │
│  🧠 Kopf     ⌚ Handgelenke    📱 Smartphone           │
│  EEG-Patch   2× Watch/Band    Kamera + Mikrofon        │
│                                                         │
│  Epilog      Apple Watch L+R   Vision / ML Kit          │
│  Ceribell    Neuralert-Bänder  SpeechRecognizer         │
│  Muse S      Fitbit/Garmin     HealthKit / Health Con.  │
└────────────┬──────────────┬──────────────┬──────────────┘
             │              │              │
             ▼              ▼              ▼
┌─────────────────────────────────────────────────────────┐
│              MEDI-MOPS-CORE (Rust)                      │
│                                                         │
│  brain_score ← EEG-Daten (Asymmetrie, Verlangsamung)   │
│  arm_score   ← Accelerometer-Delta links/rechts        │
│  face_score  ← Landmark-Symmetrie                      │
│  speech_score ← Sprach-Features                        │
│  vital_score ← HF, SpO2, Blutdruck                    │
│                                                         │
│  derive_health_state() → Ampel                          │
└─────────────────────────────────────────────────────────┘
```

---

## Die fünf Kanäle

### Kanal 1: BRAIN — EEG vom Kopf

**Was gemessen wird:**
- Alpha-Asymmetrie (links/rechts Hemisphäre)
- Delta-Verlangsamung (Schlaganfall-Marker)
- Spike-Erkennung (Anfälle)
- Schlafphasen-Veränderungen (Baseline-Shift über Nacht)

**Hardware-Optionen (keine eigene Entwicklung):**

| Gerät | Typ | Kanäle | Preis | BLE | SDK | Mops-Eignung |
|-------|-----|--------|-------|-----|-----|--------------|
| Epilog Sensor | Pflaster, 2.5cm | 1 pro Patch (4×) | klinisch | ✓ | Forschung | ★★★ ideal |
| Muse S Gen 2 | Stirnband Nacht | 4 EEG + PPG | ~350€ | ✓ | öffentlich | ★★☆ Consumer |
| Neuroelectrics Enobio | Haube | 8-32 EEG | ~3000€ | ✓ | Forschung | ★★★ klinisch |
| Ceribell | Haube Klinik | 10 EEG | klinisch | WiFi | klinisch | ★★★ Klinik only |
| IDUN Guardian | In-Ear | 1 EEG | ~250€ | ✓ | SDK offen | ★★☆ unauffällig |

**Empfehlung Phase 1:** Muse S (Consumer, SDK offen, sofort verfügbar)
**Empfehlung Klinik:** Ceribell oder Epilog (FDA-zugelassen)
**Traum:** IDUN Guardian In-Ear — EEG im Ohrstöpsel, unsichtbar

**Integration in den Rust-Kern:**

```rust
// src/brain_score.rs — NEU

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct BrainMeasurement {
    pub alpha_left: f64,          // Alpha-Power linke Hemisphäre (µV²)
    pub alpha_right: f64,         // Alpha-Power rechte Hemisphäre (µV²)
    pub delta_power: f64,         // Delta-Band Power (Verlangsamung)
    pub asymmetry_index: f64,     // (R-L)/(R+L), DAR-Index
    pub spike_count: u32,         // Spikes in letzten 60 Sekunden
    pub source: BrainSource,
    pub timestamp: DateTime<Utc>,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub enum BrainSource {
    MuseS,
    Epilog,
    Ceribell,
    IdunGuardian,
    Unknown(String),
}

/// Brain-Score: 0.0 = normal, 1.0 = schwer auffällig
pub fn compute_brain_score(m: &BrainMeasurement) -> f64 {
    // Alpha-Asymmetrie: normal < 0.1, Schlaganfall > 0.3
    let asym_score = (m.asymmetry_index.abs() / 0.5).min(1.0);

    // Delta-Verlangsamung: erhöhte Delta-Power = ischämisch
    let delta_score = (m.delta_power / 100.0).min(1.0);

    // Spike-Score: > 3 Spikes/min = auffällig
    let spike_score = (m.spike_count as f64 / 10.0).min(1.0);

    // Gewichtung: Asymmetrie am stärksten
    asym_score * 0.50 + delta_score * 0.35 + spike_score * 0.15
}
```

---

### Kanal 2: ARMS — Bilaterale Beschleunigungsmessung

**Was gemessen wird:**
- Bewegungssymmetrie links vs. rechts (Δ Amplitude)
- Drift bei Haltetest (Arm sinkt ab)
- Tremor-Frequenz und -Amplitude
- Aktivitätslevel über den Tag (plötzlicher Abfall = Alarmsignal)

**Hardware-Optionen:**

| Gerät | Typ | Preis | BLE | SDK | Mops-Eignung |
|-------|-----|-------|-----|-----|--------------|
| 2× Apple Watch | Smartwatch | 2×250€ | ✓ | CoreMotion | ★★★ ideal |
| Neuralert Bands | Medizin-Band | klinisch | ✓ | geschlossen | ★★★ Klinik |
| 2× Fitbit Sense 2 | Smartwatch | 2×200€ | ✓ | Health Con. | ★★☆ Android |
| 2× Garmin Venu 3S | Smartwatch | 2×300€ | ✓ | Connect IQ | ★★☆ |
| BioStamp nPoint | Pflaster | klinisch | ✓ | Forschung | ★★★ unsichtbar |

**Empfehlung Phase 1:** 2× Apple Watch (sofort, CoreMotion, HealthKit)
**Empfehlung Android:** 2× Fitbit oder Galaxy Watch (Health Connect)
**Die Neuralert-Logik nachbauen:**

```rust
// src/arm_score.rs — ERWEITERT um bilaterales Monitoring

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct BilateralArmMeasurement {
    // Aktiver Test (1× täglich, 10 Sekunden)
    pub active_test: Option<ArmMeasurement>,

    // Passives Monitoring (kontinuierlich, alle 5 Minuten)
    pub left_activity: f64,       // Bewegungs-Integral links (g·s)
    pub right_activity: f64,      // Bewegungs-Integral rechts (g·s)
    pub activity_asymmetry: f64,  // |L-R| / (L+R)
    pub window_minutes: u32,      // Messfenster (z.B. 5 min)
    pub timestamp: DateTime<Utc>,
}

/// Neuralert-Logik: Asymmetrie erkennen, Alltag rausfiltern
pub fn compute_bilateral_arm_score(
    m: &BilateralArmMeasurement,
    context: &ActivityContext,  // Essen? Telefonieren? Schlafen?
) -> f64 {
    let raw_asymmetry = m.activity_asymmetry;

    // Kontext-Filter (Neuralert-Patent-Logik):
    // Essen, Telefonieren, Schreiben = erwartete Asymmetrie → dämpfen
    let filter = match context.current_activity {
        Activity::Eating => 0.3,      // hohe Toleranz
        Activity::PhoneUse => 0.4,
        Activity::Sleeping => 0.9,    // im Schlaf: Asymmetrie = Alarm
        Activity::Resting => 0.8,
        Activity::Walking => 0.6,
        Activity::Unknown => 0.7,
    };

    let filtered_score = (raw_asymmetry * filter / 0.5).min(1.0);

    // Aktiver Test überschreibt wenn vorhanden
    if let Some(test) = &m.active_test {
        let active_score = compute_arm_score(test);
        return active_score.max(filtered_score);  // schlimmerer Wert zählt
    }

    filtered_score
}
```

**Passives Monitoring — der Unterschied zum täglichen Test:**

```
Täglicher Test:    "Strecke beide Arme aus, halte 10 Sekunden"
                   → 1× am Tag, bewusst, 60 Sekunden

Passiv (NEU):      Zwei Watches messen im Hintergrund alle 5 Min
                   → 24/7, unsichtbar, 288 Messpunkte pro Tag
                   → Trend-Erkennung: "Linker Arm heute 30% weniger aktiv"
                   → Nacht-Monitoring: Asymmetrie im Schlaf = Alarmsignal
```

---

### Kanal 3: FACE — Gesichtssymmetrie

**Was gemessen wird:**
- 68 Facial Landmarks (Augen, Mund, Brauen)
- Symmetrie-Score links/rechts
- Veränderung gegenüber Baseline

**Hardware:** Smartphone-Frontkamera (kein Extra-Gerät)

| Plattform | Framework | Landmarks | Genauigkeit |
|-----------|-----------|-----------|-------------|
| iOS | Vision VNDetectFaceLandmarks | 76 Punkte | ★★★ |
| Android | ML Kit Face Detection | 468 Punkte | ★★★ |
| Web | MediaPipe Face Mesh | 478 Punkte | ★★☆ |

**Integration:** Bleibt wie im Codi-Auftrag. App liefert Landmark-Koordinaten,
Rust-Kern berechnet Symmetrie-Score. Kein Extra-Gerät nötig.

---

### Kanal 4: SPEECH — Sprachanalyse

**Was gemessen wird:**
- Sprechgeschwindigkeit (WPM)
- Artikulationsgenauigkeit (Confidence)
- Pausen-Muster (Wortfindungsstörung)
- Prosodie (monoton vs. lebendig)
- Baseline-Abweichung

**Hardware:** Smartphone-Mikrofon oder Watch-Mikrofon

**Erweiterung — passives Sprach-Monitoring (GELB/Forschung):**

```rust
// src/speech_score.rs — ERWEITERT um passives Monitoring

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct PassiveSpeechMeasurement {
    // Während Telefonat (wenn Nutzer zustimmt)
    pub call_duration_secs: f64,
    pub avg_words_per_minute: f64,
    pub avg_confidence: f64,
    pub silence_ratio: f64,       // Anteil Stille am Gespräch
    pub pitch_baseline_delta: f64,
    pub timestamp: DateTime<Utc>,
    pub consent: ConsentLevel,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub enum ConsentLevel {
    FullAnalysis,           // Inhalt + Akustik
    AcousticOnly,           // Nur Tonmerkmale, kein Transkript
    Off,                    // Kein passives Monitoring
}
```

**Datenschutz-Regel:** Passives Sprach-Monitoring NUR mit explizitem Consent.
Acoustic-Only-Modus transkribiert NICHTS — nur Pitch, Rate, Pausen.
`ConsentLevel::Off` = Standard. Muss aktiv eingeschaltet werden.

---

### Kanal 5: VITALS — Körperwerte (bestehend, erweitert)

**Was gemessen wird:**
- Herzfrequenz + HRV (Vorhofflimmern → Schlaganfall-Ursache #1)
- SpO2 (Sauerstoffsättigung)
- Blutdruck (validiertes Gerät)
- Temperatur
- Schlafphasen

**Hardware:** Apple Watch / Galaxy Watch + BT-Blutdruckmessgerät

**NEU — Vorhofflimmern als Schlaganfall-Vorwarnung:**

```rust
// src/vitals.rs — ERWEITERT

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct HeartRhythmMeasurement {
    pub heart_rate: f64,
    pub hrv_sdnn: f64,             // HRV Standardabweichung
    pub irregular_rhythm: bool,     // Watch meldet "unregelmäßig"
    pub afib_detected: bool,        // Watch meldet Vorhofflimmern
    pub source: VitalSource,
    pub timestamp: DateTime<Utc>,
}

/// Vorhofflimmern ist die häufigste Ursache für Schlaganfall.
/// Wenn die Watch AFib meldet → erhöhte Wachsamkeit auf allen Kanälen.
pub fn derive_rhythm_state(
    rhythm: &HeartRhythmMeasurement,
) -> RhythmState {
    if rhythm.afib_detected {
        RhythmState::AFibAlert {
            action: RecommendedAction::CallDoctor,
            context: "Vorhofflimmern erkannt. Arzt kontaktieren — \
                      Schlaganfall-Risiko erhöht.".to_string(),
        }
    } else if rhythm.irregular_rhythm {
        RhythmState::IrregularRhythm {
            action: RecommendedAction::RetestIn { minutes: 30 },
        }
    } else {
        RhythmState::Normal
    }
}
```

---

## Die erweiterte Ampel — 5 Kanäle statt 3

```rust
// src/health_state.rs — ERWEITERT

pub fn derive_health_state_full(
    brain: Option<&BrainMeasurement>,
    arms: &BilateralArmMeasurement,
    face: &FaceMeasurement,
    speech: &SpeechMeasurement,
    rhythm: Option<&HeartRhythmMeasurement>,
    baseline: &FullBaseline,
) -> HealthCheckState {

    let mut channels: Vec<DeviationChannel> = Vec::new();
    let mut emergency_pairs = false;

    // Brain (wenn Sensor vorhanden)
    if let Some(b) = brain {
        let b_score = compute_brain_score(b);
        let b_delta = sigma_delta(b_score, baseline.brain_avg, baseline.brain_std);
        if b_delta > 2.0 {
            channels.push(DeviationChannel::Brain {
                score: b_score, baseline_delta_sigma: b_delta
            });
        }
        // SONDERREGEL: Brain + Face > 3σ → sofort Notfall
        let f_score = compute_face_symmetry(face);
        let f_delta = sigma_delta(f_score, baseline.face_avg, baseline.face_std);
        if b_delta > 3.0 && f_delta > 3.0 {
            emergency_pairs = true;
        }
    }

    // ... Face, Arms, Speech wie bisher ...

    // Rhythm-Boost: bei AFib alle Schwellen senken (1.5σ statt 2σ)
    let threshold = if rhythm.map_or(false, |r| r.afib_detected) {
        1.5  // verschärft
    } else {
        2.0  // normal
    };

    // ... Bewertung mit dynamischem Threshold ...
}
```

**Logik:** Wenn die Watch Vorhofflimmern meldet, werden die Schwellen für
alle anderen Kanäle von 2σ auf 1,5σ gesenkt. Der Mops wird wachsamer,
weil das Grundrisiko erhöht ist. Deterministisch.

---

## Sensor-Adapter-Architektur

```
┌────────────────────────────────────────────────────────────┐
│                    iOS App (SwiftUI)                        │
│                                                            │
│  SensorAdapter Protocol                                    │
│  ├── MuseSAdapter      → BrainMeasurement                  │
│  ├── WatchPairAdapter  → BilateralArmMeasurement           │
│  │   (WatchConnectivity L+R)                               │
│  ├── VisionAdapter     → FaceMeasurement                   │
│  ├── SpeechAdapter     → SpeechMeasurement                 │
│  ├── HealthKitAdapter  → HeartRhythmMeasurement            │
│  └── BluetoothBPAdapter → VitalMeasurement (Blutdruck)     │
│                                                            │
│  Jeder Adapter: Hardware → Mops-Struct → Rust-Kern (UniFFI)│
└────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────┐
│                Android App (Kotlin Compose)                 │
│                                                            │
│  SensorAdapter Interface                                   │
│  ├── MuseSAdapter      → BrainMeasurement                  │
│  ├── WearOSPairAdapter → BilateralArmMeasurement           │
│  │   (MessageClient L+R)                                   │
│  ├── MLKitFaceAdapter  → FaceMeasurement                   │
│  ├── SpeechAdapter     → SpeechMeasurement                 │
│  ├── HealthConnectAdapter → HeartRhythmMeasurement         │
│  └── BluetoothBPAdapter   → VitalMeasurement               │
│                                                            │
│  Gleiche Mops-Structs, gleicher Rust-Kern, gleiche Ampel.  │
└────────────────────────────────────────────────────────────┘
```

**Regel:** Kein Sensor ist Pflicht. Der Mops arbeitet mit dem was da ist.

```rust
// Graceful Degradation — der Mops passt sich an

pub fn derive_health_state_adaptive(
    brain: Option<&BrainMeasurement>,    // Muse S da? Wenn nicht: None
    arms: Option<&BilateralArmMeasurement>, // 2 Watches? Wenn nicht: 1 oder 0
    face: Option<&FaceMeasurement>,      // Kamera-Check gemacht? Wenn nicht: None
    speech: Option<&SpeechMeasurement>,  // Sprachtest gemacht? Wenn nicht: None
    rhythm: Option<&HeartRhythmMeasurement>,
    baseline: &AdaptiveBaseline,
) -> HealthCheckState {
    let available_channels = [
        brain.is_some(),
        arms.is_some(),
        face.is_some(),
        speech.is_some(),
    ].iter().filter(|&&x| x).count();

    // Mindestens 2 Kanäle für eine Bewertung
    if available_channels < 2 {
        return HealthCheckState::InsufficientData {
            available: available_channels,
            required: 2,
            message: "Zu wenige Sensoren für eine Bewertung.".to_string(),
        };
    }

    // Confidence sinkt mit weniger Kanälen
    let max_confidence = available_channels as f64 / 5.0;

    // ... Bewertung nur mit vorhandenen Kanälen ...
}
```

---

## Sensor-Setups: Von Minimal bis Klinik

### Setup 1: Minimal (0€ Extra)
```
Smartphone only
├── Kamera → Face-Score
├── Mikrofon → Speech-Score
└── (falls vorhanden) 1× Watch → Arm-Score einseitig + HF
```
2-3 Kanäle. Täglicher aktiver Test. Kein passives Monitoring.

### Setup 2: Home (500-600€)
```
Smartphone
├── Kamera → Face-Score
├── Mikrofon → Speech-Score
├── 2× Apple Watch → bilateraler Arm-Score + HF + SpO2 + AFib
└── BT-Blutdruckmessgerät (z.B. Omron Evolv) → Blutdruck
```
4 Kanäle. Täglicher Test + passives Arm-Monitoring 24/7.
**Das ist das Ziel-Setup für den Medi-Mops.**

### Setup 3: Premium (900-1200€)
```
Setup 2 plus:
└── Muse S Headband (nachts) → Brain-Score im Schlaf
    oder IDUN Guardian (tagsüber) → Brain-Score unsichtbar
```
5 Kanäle. Nacht-EEG erkennt Veränderungen die am Tag noch nicht da sind.

### Setup 4: Klinik / Pflegeheim
```
Ceribell EEG-Haube → Brain-Score kontinuierlich
Neuralert Bands → Arm-Score 24/7
Kamera am Bett → Face-Score periodisch
Nurse-Call → Speech-Score bei Interaktion
Monitored Vitals → HF, SpO2, RR kontinuierlich
14-Tage-ECG-Patch → AFib-Langzeit
```
Alle 5 Kanäle, kontinuierlich, medizinisch validiert.

---

## Datenschicht — was wo gespeichert wird

```rust
// ALLES lokal. Keine Cloud. Genau wie iMOPS.

pub struct MediMopsStore {
    // Roh-Messwerte: letzte 14 Tage (für Baseline)
    pub raw_measurements: RollingBuffer<SystemEvent>,

    // Baseline: 14-Tage-Durchschnitt (wird täglich neu berechnet)
    pub baseline: AdaptiveBaseline,

    // Event-Chain: Append-Only, alle Bewertungen, alle Alarme
    pub event_chain: EventChain,

    // Keine Cloud. Keine API. Offline-first.
    // Export nur auf expliziten Wunsch (PDF für den Arzt).
}
```

**Regel:** Gesundheitsdaten verlassen das Gerät NICHT.
Kein Cloud-Sync. Kein Analytics-Backend. Kein "wir verbessern unseren Service".
Die Daten gehören dem Menschen. Der Mops läuft lokal.

---

## Erweiterung des Codi-Auftrags

Zum bestehenden Rust-Kernel kommen 3 neue Module:

```
src/
├── brain_score.rs          ← NEU: EEG-Auswertung
├── bilateral_arm.rs        ← NEU: Neuralert-Logik (2× Watch)
├── rhythm.rs               ← NEU: AFib-Erkennung + Threshold-Boost
├── sensor_config.rs        ← NEU: Welche Sensoren sind da?
│
├── arm_score.rs            ← ERWEITERT: aktiv + passiv
├── health_state.rs         ← ERWEITERT: 5 Kanäle, adaptive Schwellen
├── models.rs               ← ERWEITERT: neue Measurement-Typen
└── baseline.rs             ← ERWEITERT: pro Kanal, adaptive Fenster
```

**Zusätzlicher Aufwand:** ~12h (1,5 Arbeitstage) auf den bestehenden 26h drauf.
**Neues Gesamt:** ~38h = 5 Arbeitstage für den vollständigen Rust-Kernel.

---

## Forschungsfragen (für dich, Andreas)

Während du dir Gedanken machst wie die Sensoren in den Alltag eingewebt werden:

1. **Nacht vs. Tag:** Das Muse S Stirnband im Schlaf ist unsichtbar und
   erfasst 8 Stunden EEG. Schlaganfälle passieren häufig nachts oder
   früh morgens. Ist der Schlaf der bessere Zeitpunkt zum Messen?

2. **Zwei Watches:** Wer trägt freiwillig zwei Uhren? Könnte die zweite
   ein leichtes Band sein (Fitbit Inspire, 20g) statt einer Watch?

3. **Der Mops-Anruf:** Du hattest die Idee "der Mops ruft an und fragt
   ob alles ok ist". Das ist gleichzeitig Speech-Check: Stimme
   analysieren während man antwortet. Ein Check verkleidet als Fürsorge.

4. **Kontaktpersonen-Kaskade:** Wer wird wann alarmiert? Patient selbst →
   Ehepartner → Kind → Hausarzt → 112? Konfigurierbar, wie die
   Firmenwerte beim Bau-Mops.

5. **Arzt-Report:** Ein PDF das der Mops generiert: 14-Tage-Verlauf
   aller Kanäle, Auffälligkeiten markiert. Der Arzt bekommt Daten statt
   "mir geht's eigentlich ganz gut". Wie der Preisspiegel beim Bau.

---

*Vier Sensoren. Fünf Kanäle. Eine Ampel.*
*Kein Sensor ist Pflicht. Jeder macht den Mops besser.*
*Die Daten bleiben auf dem Gerät.*
*Der Mops ruft den Arzt. Nicht die Cloud.*
