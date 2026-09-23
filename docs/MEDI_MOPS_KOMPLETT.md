# ═══════════════════════════════════════════════════════════════════
# MEDI-MOPS — Komplette Projektdokumentation
# ═══════════════════════════════════════════════════════════════════
#
# Autor: Andreas Pelczer
# Stand: September 2026
# Inhalt:
#   Teil 1: Masterplan (Roadmap, Budget, Regulierung)
#   Teil 2: Integration (Phasen, Module, Kernel-Transfer)
#   Teil 3: Codi-Auftrag Rust-Kernel (12 Module, Tests, UniFFI)
#   Teil 4: Sensorintegration (5 Kanäle, Hardware, Setups)
#
# ═══════════════════════════════════════════════════════════════════


---

# TEIL 1: MASTERPLAN

# MASTERPLAN: Medi-Mops 🐕🏥

## Deterministische Gesundheitsüberwachung — ein Mops, alle Domänen

**Autor:** Andreas Pelczer
**Standort Entwicklung:** Brasilien, ab 2027
**Kernel:** iMOPS `deriveState()` / `EventChain` — portiert nach Rust
**Plattformen:** Android (Primär, größerer Markt) + iOS
**Philosophie:** Code lügt nicht, Fantasie schon.

---

## Die Idee in einem Satz

Derselbe Mops der dem Koch das Filet sperrt und dem Polier den Beton,
erkennt beim Patienten die Abweichung — und sagt: "Ruf jetzt an."

---

## Warum Brasilien

- 220 Millionen Menschen, >70% Android-Nutzer
- SUS (Sistema Único de Saúde): öffentliches Gesundheitssystem mit
  enormem Bedarf an digitaler Überwachung in ländlichen Gebieten
- Telemedizin-Gesetzgebung seit 2020 aktiv und wachsend
- Schlaganfall: zweithäufigste Todesursache in Brasilien
- Pflegedokumentation: ähnliche Papier-Probleme wie in Deutschland
- Portugiesisch als Sprache: SpeechRecognizer verfügbar (Apple + Google)
- Andreas vor Ort ab 2027

---

## Was existiert (Stand September 2026)

### Fertige Dokumente

| # | Datei | Inhalt | Größe |
|---|-------|--------|-------|
| 1 | PLAN_medi_mops_integration.md | Masterplan: 5 Phasen, Kernel-Transfer, Module, Dateien, Regulierung | 19 KB |
| 2 | CODI_AUFTRAG_medi_mops_core_rust.md | Codi-Auftrag: 12 Rust-Module, Orakel-Tests, UniFFI, 26h Aufwand | 25 KB |
| 3 | PLAN_medi_mops_sensorintegration.md | Hardware: 5 Kanäle, Sensor-Optionen, 4 Setups, Datenschutz | 21 KB |

### Bewiesener Kernel (iMOPS Construction Grid)

- `deriveState()` — Zustand ableiten, nie setzen
- `evaluate()` — Handlung prüfen bevor sie passiert
- `EventChain` — Append-Only, nichts wird gelöscht
- `SYSTEM_REFUSAL` — Das Nein als Datenpunkt
- Ampel: GRÜN / GELB / ROT
- YAML-Kataloge als Wissensbasis (deterministisch)

### Bewiesenes Buch

- "Der Mops kam in die Küche" — 12 Kapitel, 6 Branchen
- Küche → Pflege → Bau → Truppenküche → Notaufnahme → Eventcatering
- Schwester Maria, Sandra 02:40, Frau Bergmann, Tomasz Onboarding
- BourdainGuard: `visibility: .teamMember` — nie verhandelbar

---

## Die drei Schichten

```
┌──────────────────────────────────────────────────┐
│  SCHICHT 3: Apps (SwiftUI / Kotlin Compose)      │
│  UI, Sensor-Adapter, Benachrichtigungen          │
│  → plattformspezifisch                           │
├──────────────────────────────────────────────────┤
│  SCHICHT 2: Sensor-Integration                   │
│  Brain, Arms, Face, Speech, Vitals               │
│  → PLAN_medi_mops_sensorintegration.md           │
├──────────────────────────────────────────────────┤
│  SCHICHT 1: Rust-Kernel (medi-mops-core)         │
│  deriveState, evaluate, EventChain, Ampel        │
│  → CODI_AUFTRAG_medi_mops_core_rust.md           │
│  → PLAN_medi_mops_integration.md                 │
└──────────────────────────────────────────────────┘
```

---

## Roadmap

### Q4 2026 — Fundament (Deutschland, vor Umzug)

**Oktober-Dezember 2026**

- [ ] Bau-Mops Python-Port abschließen (mops-engine, Wochenende 19./20.09.)
- [ ] "Mops fass" in iMOPS integrieren und mit Raffi testen
- [ ] Rust lernen (Andreas): Ownership, Lifetimes, Traits, UniFFI
- [ ] Rust-Kernel Prototyp: models.rs + event_chain.rs + baseline.rs
- [ ] Muse S beschaffen und EEG-Rohdaten verstehen
- [ ] 2× Apple Watch bilateral testen: CoreMotion Symmetrie-Messung
- [ ] Buch "Der Mops kam in die Küche" fertigstellen / publizieren

**Deliverable:** Rust kompiliert, EventChain funktioniert, erster Face-Score
aus Vision Framework läuft auf dem iPhone.

### Q1 2027 — Kernel (Brasilien)

**Januar-März 2027**

- [ ] Rust-Kernel komplett: alle 12 Module nach Codi-Auftrag
- [ ] UniFFI-Bindings: Swift + Kotlin generiert und getestet
- [ ] Android-Prototyp: ML Kit Face + SpeechRecognizer → Rust-Kern
- [ ] Bilaterales Arm-Monitoring: 2× Watch Proof of Concept
- [ ] Orakel-Tests: Sandra-Szenario, Schlaganfall-Muster, Baseline < 7d
- [ ] YAML-Kataloge: medikamente_mops.yaml (50 häufigste Wirkstoffe)

**Deliverable:** `cargo test` grün. Android-App zeigt Morgen-Check Ampel.

### Q2 2027 — Sensor-Integration

**April-Juni 2027**

- [ ] Brain-Score: Muse S SDK → BrainMeasurement → Rust-Kern
- [ ] Passives Arm-Monitoring: Hintergrund-Messung alle 5 Min
- [ ] Nacht-Modus: EEG-Stirnband im Schlaf, Morgen-Report
- [ ] AFib-Detection: Watch-Daten → Threshold-Boost-Logik
- [ ] Graceful Degradation: 2, 3, 4, 5 Kanäle getestet
- [ ] Kontaktpersonen-Kaskade: Konfigurierbar wer wann alarmiert wird

**Deliverable:** 5-Kanal-Prototyp läuft mit echten Sensoren.

### Q3 2027 — Pflege-Module

**Juli-September 2027**

- [ ] Medikamentenplan: MedicationState + evaluate() + Wechselwirkungen
- [ ] Pflege-Textbausteine: pflege_mops_textbausteine.yaml (PT-BR + DE)
- [ ] Schichtübergabe: automatischer Tagesbericht
- [ ] BourdainGuard: Belastungsmonitoring, visibility: SelfOnly
- [ ] Onboarding: Tomasz-Logik (5 Regeln, rollenabhängig)
- [ ] Vitalwerte-Dashboard: Arzt-Schwellwerte als Firmenwerte

**Deliverable:** Komplette Pflege-App mit Doku + Medikation + Monitoring.

### Q4 2027 — Validierung + Launch

**Oktober-Dezember 2027**

- [ ] Pilottest: 10-20 Nutzer (Risikopatienten, freiwillig)
- [ ] Baseline-Validierung: Wie stabil sind die Scores über 14 Tage?
- [ ] False-Positive-Rate: < 2 Fehlalarme pro Tag (Neuralert-Standard)
- [ ] Arzt-Report PDF: 14-Tage-Verlauf, Auffälligkeiten markiert
- [ ] Regulierung klären: ANVISA (Brasilien) / MDR (EU) — Wellness vs. Medizinprodukt
- [ ] Lokalisierung: Portugiesisch (BR), Deutsch, Englisch
- [ ] Google Play Store: Beta-Release

**Deliverable:** Medi-Mops v1.0 im Store. Erste echte Nutzer.

---

## Budget-Schätzung (Entwicklung)

| Posten | Kosten |
|--------|--------|
| 2× Apple Watch (Test) | ~500€ |
| Muse S Gen 2 (EEG) | ~350€ |
| BT-Blutdruckmessgerät | ~80€ |
| IDUN Guardian In-Ear (wenn verfügbar) | ~250€ |
| Android-Testgerät | ~300€ |
| Apple Developer Account | 99€/Jahr |
| Google Developer Account | 25€ einmalig |
| Server (keiner — alles lokal) | 0€ |
| **Gesamt Hardware** | **~1.600€** |

Software-Entwicklung: Andreas + Codi. Keine externen Entwickler.
Kein Cloud-Backend. Keine laufenden Kosten.

---

## Regulierung — der klare Weg

### Phase 1: Wellness-App (kein Medizinprodukt)

- "Gesundheits-Tagebuch mit Sensordaten"
- Zeigt Trends, keine Diagnosen
- "Abweichung erkannt" statt "Schlaganfall-Verdacht"
- Empfiehlt Arztbesuch, stellt keine Diagnose
- DSGVO / LGPD (Brasilien) konform: alle Daten lokal

### Phase 2: Medizinprodukt (wenn klinische Studie steht)

- Kooperation mit Klinik (Brasilien oder Deutschland)
- ANVISA-Zulassung (Brasilien) oder MDR Klasse IIa (EU)
- "Schlaganfall-Frühwarnsystem" als zugelassene Indikation
- Braucht: klinische Studie, 100+ Patienten, prospektiv
- Vorbild: Ceribell (FDA Breakthrough 2026), Neuralert (FDA Breakthrough 2021)

### Der Mops-Weg

Erst beweisen dass es funktioniert (Wellness-App mit echten Daten).
Dann zulassen. Nicht umgekehrt. Das System muss stehen bevor der
Papierkram anfängt — genau wie beim Bau: erst bauen, dann abrechnen.

---

## Abgrenzung — was der Medi-Mops NICHT ist

- ❌ Kein Ersatz für den Arzt
- ❌ Kein Diagnosesystem
- ❌ Kein Cloud-Service
- ❌ Kein Abo-Modell
- ❌ Kein Datenhandel
- ❌ Keine KI die "wahrscheinlich alles ok" sagt
- ✅ Ein lokales, deterministisches Frühwarnsystem
- ✅ Das dem Menschen gehört, nicht der Firma
- ✅ Das sagt was es weiß und schweigt wo es nicht weiß
- ✅ Das den Arzt ruft, nicht ersetzt

---

## Die Kette

```
2024  Küche      → iMOPS HACCP (Swift, iOS)
2025  Baustelle  → iMOPS Construction Grid (Swift, iOS)
2026  Motor      → mops-engine (Python) + Stammdaten (YAML)
2027  Pflege     → Medi-Mops (Rust → Android + iOS)
2028  ???        → Derselbe Kernel. Andere YAML. Andere Domäne.
```

Der Mops wächst. Aber er ändert sich nicht.
Er sagt immer noch: Nein.
Auf jedem Gerät. In jeder Sprache. Für jeden Menschen.

---

*Ein Mops kam in die Küche und stahl dem Koch ein Ei.*
*Da nahm der Koch sein Smartphone und schlug den Mops zu Brei.*
*Da kamen alle Möpse und gruben ihm ein Grab*
*und setzten ihm 'nen Grabstein, auf dem geschrieben stand:*

*Ein Mops kam in die Pflege — und hat kein Ei gestohlen.*
*Er hat nur leise gemessen, gerechnet, und gemeldet.*
*Und wenn er Nein gesagt hat, dann hatte er recht.*

---

**Dokumente:**
1. `PLAN_medi_mops_integration.md` — Phasen + Module + Architektur
2. `CODI_AUFTRAG_medi_mops_core_rust.md` — Rust-Kernel Bauauftrag
3. `PLAN_medi_mops_sensorintegration.md` — Hardware + Kanäle + Setups
4. `MASTERPLAN_medi_mops.md` — dieses Dokument (Roadmap + Übersicht)

---

# TEIL 2: INTEGRATION

# PLAN: Medi-Mops — Gesundheits-Monitoring auf dem iMOPS-Kernel

**Status:** Konzept — nach Construction Grid "Mops fass" und Python-Port
**Kernel:** iMOPS `deriveState()` + `evaluate()` + `EventChain` (Append-Only)
**Ziel:** Dasselbe Nein das dem Koch das Filet sperrt und dem Polier den Beton,
         erkennt beim Patienten die Abweichung — und sagt: "Ruf jetzt an."

---

## Der Kern-Transfer: Küche → Bau → Pflege

```
Küche:     Filet.Standzeit > max    → SYSTEM_REFUSAL → Neu zubereiten
Bau:       Bewehrung.Freigabe fehlt → SYSTEM_REFUSAL → Prüfing. anfordern
Pflege:    Sprache.Abweichung > Δ   → SYSTEM_ALERT   → 112 / Arzt anrufen
```

Gleicher Mechanismus. Gleicher `deriveState()`. Andere YAML, andere Schwellwerte,
anderes Ergebnis. Der Mops ändert sich nicht. Die Domäne ändert sich.

---

## Phase 1: Baseline-System (rein deterministisch — GRÜN)

### Was es kann
Täglicher "Guten Morgen Mops"-Check. Der Patient öffnet die App, macht 60 Sekunden
lang drei Tests. Das System vergleicht mit seiner persönlichen Baseline.

### Die drei FAST-Kanäle

#### 1. Face — Gesichtssymmetrie (Apple Vision Framework)

```swift
// MediMops/Modules/FaceCheck/FaceSymmetryAnalyzer.swift

import Vision

struct FaceSymmetryResult {
    let leftEyeHeight: CGFloat
    let rightEyeHeight: CGFloat
    let leftMouthCorner: CGPoint
    let rightMouthCorner: CGPoint
    let symmetryScore: Double      // 0.0 = perfekt symmetrisch, 1.0 = maximal asymmetrisch
    let timestamp: Date
}

// 68 Facial Landmarks → Symmetrie-Score ableiten
// Apple Vision VNDetectFaceLandmarksRequest liefert die Punkte
// Kein ML-Modell nötig — reine Geometrie:
//   Δ = |links.y - rechts.y| für Augen, Mundwinkel, Brauen
//   Score = gewichtete Summe der Δ-Werte
//   Baseline = Durchschnitt der letzten 14 Tage
//   Abweichung = heutiger Score vs. Baseline
```

**Technik:** `VNDetectFaceLandmarksRequest` (Apple Vision, kein Import nötig)
**Determinismus:** Ja — gleiche Landmarks = gleicher Score
**Genauigkeit laut FAST.AI-Studie:** ~97% für Gesichtsasymmetrie

#### 2. Arms — Armdrift (CoreMotion Accelerometer)

```swift
// MediMops/Modules/ArmCheck/ArmDriftAnalyzer.swift

import CoreMotion

struct ArmDriftResult {
    let holdDuration: TimeInterval  // Wie lange konnte gehalten werden
    let maxDrift: Double            // Maximale Abweichung in Grad
    let tremor: Double              // Frequenz/Amplitude des Zitterns
    let symmetry: Double            // Links vs. Rechts Differenz
    let timestamp: Date
}

// Ablauf:
// 1. "Strecke beide Arme vor dir aus" (Phone in einer Hand)
// 2. 10 Sekunden halten
// 3. Accelerometer + Gyroscope messen Drift und Tremor
// 4. Vergleich mit Baseline der letzten 14 Tage
//
// Apple Watch: CMMotionManager auf BEIDEN Handgelenken
// iPhone only: eine Hand, Tremor + Haltezeit
```

**Technik:** `CMMotionManager` (CoreMotion, auf iPhone + Apple Watch)
**Determinismus:** Ja — gleiche Sensordaten = gleicher Score
**Genauigkeit laut FAST.AI-Studie:** ~72% für Armschwäche

#### 3. Speech — Sprachanalyse (Speech Framework + Audio-Features)

```swift
// MediMops/Modules/SpeechCheck/SpeechPatternAnalyzer.swift

import Speech
import AVFoundation

struct SpeechAnalysisResult {
    let wordsPerMinute: Double          // Sprechgeschwindigkeit
    let articulationScore: Double       // Konsonanten-Klarheit
    let pausePattern: [TimeInterval]    // Pausen zwischen Wörtern
    let pitchVariance: Double           // Prosodie (monoton vs. lebendig)
    let wordFindingDelay: Double        // Zeit bis erstes Wort nach Prompt
    let timestamp: Date
}

// Ablauf:
// 1. Standardsatz anzeigen: "Die Sonne scheint und der Himmel ist blau"
// 2. Patient liest vor (SFSpeechRecognizer + AVAudioRecorder parallel)
// 3. SFSpeechRecognizer: Wörter + Zeitstempel → Geschwindigkeit, Pausen
// 4. AVAudioEngine: Pitch-Analyse (vDSP FFT) → Prosodie
// 5. Vergleich mit Baseline
//
// KEIN ML-Modell für Phase 1. Reine Signal-Analyse:
//   - Sprechrate: Wörter / Dauer
//   - Pausen: Standardabweichung der Wort-zu-Wort-Abstände
//   - Pitch: Varianz der Grundfrequenz (monoton = niedrig)
//   - Artikulation: Confidence-Score des SFSpeechRecognizer
//     (verwaschene Sprache → niedrigere Confidence)
```

**Technik:** `SFSpeechRecognizer` + `AVAudioEngine` (Apple Frameworks)
**Determinismus:** Annähernd — gleiche Aufnahme = gleicher Score (±Rauschen)
**Forschungsstand:** Pusan National University startet 2026 klinische Studie
zur KI-basierten Sprachverständlichkeitsbewertung bei Schlaganfall-Patienten

### Die Ableitung — deriveState() für den Medi-Mops

```swift
// MediMops/Core/HealthStateDerivation.swift

enum HealthCheckState {
    case normal(confidence: Double)
    case deviation(channels: [DeviationChannel], severity: Severity)
    case alert(channels: [DeviationChannel], action: RecommendedAction)
}

enum DeviationChannel {
    case face(score: Double, baselineDelta: Double)
    case arms(score: Double, baselineDelta: Double)
    case speech(score: Double, baselineDelta: Double)
}

enum Severity { case mild, moderate, severe }

enum RecommendedAction {
    case retest           // "Bitte nochmal in 30 Minuten"
    case callDoctor       // "Ruf deinen Hausarzt an"
    case callEmergency    // "Ruf 112 an. JETZT."
}

func deriveHealthState(
    face: FaceSymmetryResult,
    arms: ArmDriftResult,
    speech: SpeechAnalysisResult,
    baseline: PatientBaseline,
    at now: Date
) -> HealthCheckState {

    let fDelta = abs(face.symmetryScore - baseline.faceAvg) / baseline.faceStdDev
    let aDelta = abs(arms.maxDrift - baseline.armAvg) / baseline.armStdDev
    let sDelta = abs(speech.wordsPerMinute - baseline.speechRateAvg) / baseline.speechRateStdDev

    var channels: [DeviationChannel] = []

    // Abweichung > 2 Standardabweichungen = auffällig
    if fDelta > 2.0 { channels.append(.face(score: face.symmetryScore, baselineDelta: fDelta)) }
    if aDelta > 2.0 { channels.append(.arms(score: arms.maxDrift, baselineDelta: aDelta)) }
    if sDelta > 2.0 { channels.append(.speech(score: speech.wordsPerMinute, baselineDelta: sDelta)) }

    switch channels.count {
    case 0:
        return .normal(confidence: 1.0 - max(fDelta, aDelta, sDelta) / 10.0)
    case 1:
        // Ein Kanal auffällig → in 30 Min nochmal
        return .deviation(channels: channels, severity: .mild)
    case 2:
        // Zwei Kanäle → Arzt anrufen
        return .deviation(channels: channels, severity: .moderate)
    case 3:
        // ALLE DREI Kanäle → 112. Sofort.
        return .alert(channels: channels, action: .callEmergency)
    default:
        return .normal(confidence: 1.0)
    }

    // ZUSATZREGEL: Face + Speech gleichzeitig > 3σ → immer .alert
    // (klassisches Schlaganfall-Muster: hängendes Gesicht + verwaschene Sprache)
    if fDelta > 3.0 && sDelta > 3.0 {
        return .alert(
            channels: channels,
            action: .callEmergency
        )
    }
}
```

### Ampel — identisch zum Bau-Mops

```
🟢 GRÜN  — Alle Werte im Baseline-Korridor. "Guten Morgen, alles normal."
🟡 GELB  — Ein Kanal auffällig. "Bitte in 30 Minuten nochmal testen."
🟠 ORANGE — Zwei Kanäle. "Ruf deinen Hausarzt an. Heute."
🔴 ROT   — Drei Kanäle oder Face+Speech > 3σ.
           "Ruf 112. JETZT. Oder soll ich für dich anrufen?"
           [112 anrufen]  [Kontaktperson anrufen]
```

### Was der Mops NICHT tut

- ❌ Keine Diagnose. "Schlaganfall-Verdacht" sagt der ARZT, nicht der Mops.
- ❌ Kein "wahrscheinlich alles ok". Entweder im Korridor oder nicht.
- ❌ Kein Ersetzen des Notrufs. Der Mops ist der Auslöser, nicht die Behandlung.
- ❌ Kein nachträgliches Ändern der Messwerte. EventChain, Append-Only.

---

## Phase 2: Medikamentenplan (deterministisch — GRÜN)

Direkt aus dem Buch, Kapitel 2+3: Frau Bergmann und Sandra.

### MedicationState — exakt wie im Buch

```swift
// MediMops/Modules/Medication/MedicationState.swift
// 1:1 aus "Der Mops kam in die Küche", Kapitel 2

enum MedicationState {
    case scheduled(dueAt: Date)
    case prepared(at: Date, by: StaffID)
    case confirmed(at: Date, by: StaffID)
    case windowExpired(preparedAt: Date, expiredAt: Date)
    case refused(by: ResidentAction, at: Date)
}

// Kein medication.status = .done
// Der Zustand ergibt sich aus Ereignissen und Zeitfenstern.
```

### Datenquelle: medikamente_mops.yaml

```yaml
# Analog zu aufwandswerte_bau.yaml und stlb_mops_textbausteine.yaml
meta:
  version: "1.0.0"
  quelle: "ABDA-Interaktionsdatenbank (öffentliche Teile)"

medikamente:
  metformin_500:
    wirkstoff: "Metformin"
    dosierung: "500mg"
    einnahme: "morgens, zum Essen"
    wechselwirkungen: ["novalgin", "ibuprofen_hoch", "kontrastmittel"]
    kontraindikationen: ["niereninsuffizienz_schwer", "alkohol_akut"]
    zeitfenster_minuten: 60    # ±30 min um Sollzeit
    tags: ["Diabetes", "Blutzucker", "orale Antidiabetika"]

  levodopa_100:
    wirkstoff: "Levodopa"
    dosierung: "100mg"
    einnahme: "morgens nüchtern, 30 min vor dem Essen"
    wechselwirkungen: ["eisenpraeparat", "vitamin_b6_hoch"]
    zeitfenster_minuten: 30    # Parkinson: Timing ist kritisch
    tags: ["Parkinson", "Dopamin", "L-Dopa"]
```

### Autorisierungslogik — Sandras Nein (Kapitel 3)

```swift
// 1:1 aus dem Buch: Externe Kraft darf Bedarfsmedikation nicht freigeben

func evaluate(_ action: MedicationAction) -> ActionResult {
    var reasons: [RefusalReason] = []

    if action.medication.type == .onDemand
        && action.requestedBy.role == .external {
        reasons.append(.insufficientAuthorization(
            required: .permanentStaff,
            actual: .external
        ))
    }

    let interactions = checkInteractions(
        action.medication,
        against: action.resident.activeMedications(at: action.requestedAt)
    )
    if !interactions.isEmpty {
        reasons.append(.interactionCheckRequired(with: interactions))
    }

    return reasons.isEmpty
        ? .permitted(action: action)
        : .refused(reasons: reasons)
}
```

---

## Phase 3: Vitalwerte-Monitoring (deterministisch — GRÜN)

### Schwellwerte aus ärztlicher Verordnung (= Firmenwerte im Bau)

```yaml
# vitalwerte_schwellwerte.yaml
# Analog zu: Firmenwerte beim Bau-Mops (Mittellohn, BGK, W&G)
# Werden VOM ARZT gesetzt, nicht vom System.

patient_bergmann:
  blutdruck:
    systolisch: { gruen: [110, 140], gelb: [140, 160], rot: [160, 999] }
    diastolisch: { gruen: [65, 90], gelb: [90, 100], rot: [100, 999] }
  herzfrequenz: { gruen: [55, 90], gelb: [90, 110], rot: [110, 999] }
  blutzucker_nuechtern: { gruen: [70, 120], gelb: [120, 180], rot: [180, 999] }
  temperatur: { gruen: [36.0, 37.5], gelb: [37.5, 38.5], rot: [38.5, 999] }
  gewicht_delta_kg_woche: { gruen: [-0.5, 0.5], gelb: [0.5, 2.0], rot: [2.0, 999] }
```

### deriveVitalState() — gleiche Logik wie Baugrube EV2

```swift
func deriveVitalState(
    measurement: VitalMeasurement,
    thresholds: PatientThresholds,  // vom Arzt, wie Firmenwerte
    at now: Date
) -> VitalState {
    let range = thresholds.range(for: measurement.type)

    if range.gruen.contains(measurement.value) {
        return .normal(value: measurement.value, at: now)
    }
    if range.gelb.contains(measurement.value) {
        return .elevated(value: measurement.value, at: now,
                        action: .notifyNurse)
    }
    return .critical(value: measurement.value, at: now,
                    action: .notifyDoctor)
}
```

### Datenquellen
- **Manuell:** Patient tippt Werte ein (Blutdruck, Blutzucker)
- **Bluetooth:** Zugelassene Messgeräte (Withings, Omron → HealthKit)
- **Apple Watch:** Herzfrequenz, Schritte, SpO2 (HealthKit-Bridge)
- **KEIN Smartwatch-Blutdruck.** Nicht medizinisch validiert → GELB-Markierung

---

## Phase 4: BourdainGuard Pflege (aus dem Buch, Kapitel 9)

### Belastungsmonitoring für Pflegekräfte

```swift
// Exakt wie im Buch: visibility = .teamMember
// Chef sieht es NICHT. Nur die Person selbst.

struct WellbeingCheck {
    let staffMember: StaffMember
    let consecutiveShifts: Int
    let weeklyHours: Double
    let showWarning: Bool
    let supportResources: [SupportContact]
    let visibility: ViewScope = .teamMember  // NUR für dich sichtbar
}
```

---

## Phase 5: Pflegedokumentation + Onboarding (Kapitel 2 + 10)

### Pflege-Textbausteine (= STLB-Mops für Pflege)

```yaml
# pflege_mops_textbausteine.yaml
# Analog zu stlb_mops_textbausteine.yaml (92 Bau-Bausteine)

pflege_bausteine:
  GKW-001:
    kurztext: "Ganzkörperwäsche mit Unterstützung"
    langtext: >
      Ganzkörperwäsche im Bett durchgeführt. Bewohner/in wurde bei der
      Oberkörperwäsche unterstützt, Unterkörper selbständig. Hautzustand
      kontrolliert: {{hautzustand}}. Inkontinenzversorgung: {{inkontinenz}}.
      Bewohner/in war {{orientierung}} und {{kooperation}}.
    sis_bereich: "Körperpflege"
    tags: ["Waschen", "Grundpflege", "Körperpflege", "GKW"]

  MED-001:
    kurztext: "Medikamentengabe oral"
    langtext: >
      Medikamente laut Verordnung verabreicht: {{medikamente}}.
      Einnahme {{einnahme_status}}. Zeitpunkt: {{uhrzeit}}.
      Besonderheiten: {{besonderheiten}}.
    sis_bereich: "Medikation"
    tags: ["Medikamente", "Tabletten", "oral", "Einnahme"]
```

### Onboarding — 5 Regeln (Kapitel 10, Tomasz)

```swift
// Rollenabhängig, stationsabhängig, kontextabhängig
func onboardingView(
    for staff: StaffMember,
    on station: Station,
    shift: Shift,
    at now: Date
) -> OnboardingView {
    let residents = station.residents(needingAttention: shift)
    let permissions = staff.role.permissions(on: station)
    let emergency = station.emergencyContacts

    return OnboardingView(
        rules: [
            .residentOverview(residents),      // Wer liegt wo, wer braucht was
            .medicationTonight(permissions),    // Was darfst du, was nicht
            .accessAndKeys(station),            // Schlüssel, Notfallkoffer
            .yourBoundaries(permissions),       // Deine Grenzen
            .yourNetwork(emergency)             // Wen rufst du an
        ]
    )
}
```

---

## Dateistruktur im Repo

```
iMOPS-Haccp/               (oder eigenes Repo: iMOPS-MediMops/)
├── MediMops/
│   ├── Core/
│   │   ├── HealthStateDerivation.swift    ← deriveState() für Gesundheit
│   │   ├── EventChain.swift               ← Append-Only (aus Kernel)
│   │   └── BaselineManager.swift          ← 14-Tage-Baseline berechnen
│   ├── Modules/
│   │   ├── FaceCheck/
│   │   │   └── FaceSymmetryAnalyzer.swift ← Vision Framework, 68 Landmarks
│   │   ├── ArmCheck/
│   │   │   └── ArmDriftAnalyzer.swift     ← CoreMotion Sensoren
│   │   ├── SpeechCheck/
│   │   │   └── SpeechPatternAnalyzer.swift← SFSpeechRecognizer + vDSP
│   │   ├── Medication/
│   │   │   ├── MedicationState.swift      ← aus dem Buch, Kapitel 2
│   │   │   ├── MedicationEvaluator.swift  ← Sandras Nein, Kapitel 3
│   │   │   └── InteractionChecker.swift   ← ABDA-Datenbank Match
│   │   ├── Vitals/
│   │   │   ├── VitalStateDerivation.swift ← Schwellwert-Ampel
│   │   │   └── HealthKitBridge.swift      ← Apple Watch, BT-Geräte
│   │   ├── Documentation/
│   │   │   ├── CareTextBuilder.swift      ← Textbausteine → Pflegebericht
│   │   │   └── ShiftHandover.swift        ← Übergabebericht automatisch
│   │   └── BourdainGuard/
│   │       └── WellbeingCheck.swift       ← Kapitel 9, visibility: .teamMember
│   ├── Views/
│   │   ├── MorningCheckView.swift         ← "Guten Morgen Mops" 60-Sek-Check
│   │   ├── MedicationView.swift           ← Medikamentenplan + Ampel
│   │   ├── VitalsView.swift               ← Vitalwerte-Dashboard
│   │   ├── OnboardingView.swift           ← 5-Regeln-Ansicht (Tomasz)
│   │   └── AlertView.swift                ← ROT: "Ruf 112" + Emergency-Button
│   └── Data/
│       ├── medikamente_mops.yaml          ← Wirkstoff-Katalog
│       ├── pflege_mops_textbausteine.yaml ← Doku-Bausteine
│       └── vitalwerte_schwellwerte.yaml   ← Arzt-definierte Grenzen
├── Tests/
│   ├── FaceSymmetryTests.swift
│   ├── SpeechPatternTests.swift
│   ├── MedicationStateTests.swift         ← Orakel: Sandra-Szenario
│   └── BaselineDeviationTests.swift
└── README.md
```

---

## Reihenfolge der Umsetzung

```
Woche 1:  Core + Baseline (EventChain portieren, BaselineManager)
Woche 2:  FaceCheck (Vision Framework, Symmetrie-Score)
Woche 3:  SpeechCheck (SFSpeechRecognizer, Audio-Features)
Woche 4:  ArmCheck (CoreMotion) + HealthStateDerivation (Ampel)
Woche 5:  MorningCheckView + AlertView (UI)
Woche 6:  Tests (Orakel: bekannte Scores → erwartete States)
---
Woche 7:  Medication (MedicationState + Evaluator aus dem Buch)
Woche 8:  Vitals (HealthKit-Bridge + Schwellwerte-Ampel)
Woche 9:  Documentation (Pflege-Textbausteine)
Woche 10: BourdainGuard + Onboarding
```

---

## Regulierung — der Elefant im Raum

| Frage | Antwort |
|-------|---------|
| Ist das ein Medizinprodukt? | Wenn es Diagnosen stellt: JA → MDR Klasse IIa minimum |
| Wenn es nur erinnert und misst? | Wellness-App → KEIN Medizinprodukt |
| Wo ist die Grenze? | "Ruf 112" basierend auf Messwerten = medizinische Empfehlung |
| Der Mops-Weg? | Phase 1 als Wellness-App (Baseline-Tracking, keine Diagnose). "Abweichung erkannt" statt "Schlaganfall-Verdacht". Medizinprodukt-Zulassung erst wenn klinische Studie steht. |

**Die ehrliche Antwort:** Ein FAST-Check der "Ruf 112" sagt, bewegt sich
im MDR-Graubereich. Der sichere Weg: Kooperation mit einer Klinik für eine
Validierungsstudie. Pusan National University macht genau das gerade (2026).

---

## Was gleich bleibt (der Kernel)

- `deriveState()` — Zustand wird abgeleitet, nie gesetzt
- `evaluate()` — Handlung wird geprüft bevor sie passiert
- `EventChain` — Append-Only, nichts wird gelöscht
- `SYSTEM_REFUSAL` — Das Nein als Datenpunkt
- `FrictionPoint` — Reibung statt Schuld
- Ampel: GRÜN / GELB / ROT
- "Code lügt nicht, Fantasie schon."

---

## Was sich ändert (die Domäne)

| Bau-Mops | Medi-Mops |
|----------|-----------|
| aufwandswerte_bau.yaml | medikamente_mops.yaml |
| maschinenkatalog_bau.yaml | vitalwerte_schwellwerte.yaml |
| stlb_mops_textbausteine.yaml | pflege_mops_textbausteine.yaml |
| Firmenwerte (Mittellohn, W&G) | Arzt-Verordnung (Schwellwerte) |
| GAEB X83/X84 | SIS / Pflegebericht |
| Polier Krause | Schwester Sandra |
| "Bewehrungsfreigabe fehlt" | "Autorisierung fehlt" |
| "Beton gesperrt" | "Medikament gesperrt" |

---

*Der Mops kam in die Küche. Dann auf die Baustelle.*
*Jetzt geht er in die Pflege.*
*Er stiehlt immer noch kein Ei.*
*Er sagt immer noch: Nein.*

---

# TEIL 3: CODI-AUFTRAG RUST-KERNEL

# CODI-AUFTRAG: medi-mops-core (Rust)

## Plattformunabhängiger Gesundheits-Kernel für iOS + Android

**Auftraggeber:** Andreas Pelczer
**Datum:** September 2026
**Repo:** `AndreasPelczer/medi-mops-core` (neu anlegen)
**Sprache:** Rust (edition 2021, stable)
**Bindings:** UniFFI → Swift (iOS) + Kotlin (Android)
**Abhängigkeit auf Drittanbieter-Crates:** so wenig wie möglich.
**Prinzip:** Derselbe Kernel wie iMOPS. `deriveState()` statt Status-Setter.
             `EventChain` Append-Only. Das Nein als Datenpunkt. Code lügt nicht.

---

## Was dieser Kernel ist

Der Medi-Mops-Kern enthält die gesamte Gesundheitslogik — Zustandsableitung,
Medikamentenbewertung, Vitalwert-Ampel, Baseline-Berechnung, Sprachscore-Auswertung,
Ereigniskette. Er enthält KEIN UI, KEINEN Sensor-Zugriff, KEINE Plattform-APIs.

Die Apps (SwiftUI / Kotlin Compose) liefern Messwerte rein, der Kern liefert
abgeleitete Zustände und Empfehlungen raus. Der Kern ist die Wahrheit.

```
iOS App (SwiftUI)                Android App (Kotlin)
  │ Vision Framework               │ ML Kit Face Detection
  │ CoreMotion                      │ Sensor API
  │ SFSpeechRecognizer              │ SpeechRecognizer
  │ HealthKit                       │ Health Connect
  └──────────┬──────────────────────┘
             │
             ▼  UniFFI (FFI-Brücke)
  ┌──────────────────────────┐
  │   medi-mops-core (Rust)  │
  │                          │
  │   Reine Logik.           │
  │   Kein UI.               │
  │   Kein Netzwerk.         │
  │   Kein Sensor.           │
  │   Nur Mathe + Regeln.    │
  └──────────────────────────┘
```

---

## Paketstruktur

```
medi-mops-core/
├── Cargo.toml
├── uniffi.toml                     ← UniFFI-Konfiguration
├── src/
│   ├── lib.rs                      ← Modul-Deklarationen + UniFFI-Export
│   │
│   ├── event_chain.rs              ← Append-Only Ereigniskette (DER Kernel)
│   ├── models.rs                   ← Typen: Messwerte, Zustände, Schwellen
│   │
│   ├── baseline.rs                 ← 14-Tage-Baseline: Mittelwert + StdDev
│   ├── health_state.rs             ← deriveHealthState() — die Ampel
│   │
│   ├── face_score.rs               ← Symmetrie-Score aus Landmark-Koordinaten
│   ├── arm_score.rs                ← Drift-Score aus Accelerometer-Daten
│   ├── speech_score.rs             ← Sprach-Score aus Timing + Confidence
│   │
│   ├── medication.rs               ← MedicationState + evaluate() — Sandras Nein
│   ├── interactions.rs             ← Wechselwirkungsprüfung gegen Katalog
│   ├── vitals.rs                   ← Vitalwert-Ampel gegen Arzt-Schwellwerte
│   │
│   ├── bourdain_guard.rs           ← Belastungscheck, visibility: SelfOnly
│   ├── care_text.rs                ← Pflege-Textbausteine zusammenbauen
│   │
│   └── catalog/
│       ├── mod.rs                  ← Katalog-Loader (YAML → Structs)
│       ├── medikamente.yaml        ← Wirkstoff-Katalog
│       ├── wechselwirkungen.yaml   ← Interaktionsmatrix
│       ├── pflege_textbausteine.yaml ← Doku-Bausteine (SIS-Stil)
│       └── vitalwerte_normbereiche.yaml ← Default-Schwellwerte
│
├── tests/
│   ├── test_event_chain.rs
│   ├── test_health_state.rs        ← Orakel: bekannte Scores → erwartete States
│   ├── test_medication.rs          ← Orakel: Sandra-Szenario aus dem Buch
│   ├── test_baseline.rs
│   ├── test_face_score.rs
│   ├── test_speech_score.rs
│   ├── test_vitals.rs
│   └── test_interactions.rs
│
└── bindings/
    ├── swift/                      ← generierte Swift-Bindings (UniFFI)
    └── kotlin/                     ← generierte Kotlin-Bindings (UniFFI)
```

---

## Module — Reihenfolge und Spezifikation

### Modul 1: models.rs — Typen

Alle Datentypen. Keine Logik. Nur Strukturen.

```rust
use chrono::{DateTime, Utc};
use rust_decimal::Decimal;
use serde::{Deserialize, Serialize};

// ═══════════════════════════════════════
// Messwerte (kommen von der App rein)
// ═══════════════════════════════════════

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct FaceMeasurement {
    pub left_eye_height: f64,
    pub right_eye_height: f64,
    pub left_mouth_y: f64,
    pub right_mouth_y: f64,
    pub left_brow_y: f64,
    pub right_brow_y: f64,
    pub timestamp: DateTime<Utc>,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct ArmMeasurement {
    pub hold_duration_secs: f64,
    pub max_drift_degrees: f64,
    pub tremor_amplitude: f64,
    pub timestamp: DateTime<Utc>,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct SpeechMeasurement {
    pub words_per_minute: f64,
    pub recognition_confidence: f64,  // 0.0-1.0, von der Plattform
    pub pause_stddev_ms: f64,         // Standardabweichung der Pausen
    pub pitch_variance: f64,          // Prosodie
    pub word_finding_delay_ms: f64,   // Zeit bis erstes Wort
    pub timestamp: DateTime<Utc>,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct VitalMeasurement {
    pub vital_type: VitalType,
    pub value: f64,
    pub source: MeasurementSource,
    pub timestamp: DateTime<Utc>,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub enum VitalType {
    SystolicBP,
    DiastolicBP,
    HeartRate,
    BloodGlucose,
    Temperature,
    SpO2,
    Weight,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub enum MeasurementSource {
    Manual,
    BluetoothDevice(String),
    Smartwatch,
}

// ═══════════════════════════════════════
// Abgeleitete Zustände (gehen zur App raus)
// ═══════════════════════════════════════

#[derive(Debug, Clone, Serialize)]
pub enum HealthCheckState {
    Normal { confidence: f64 },
    Deviation {
        channels: Vec<DeviationChannel>,
        severity: Severity,
        action: RecommendedAction,
    },
    Alert {
        channels: Vec<DeviationChannel>,
        action: RecommendedAction,
    },
}

#[derive(Debug, Clone, Serialize)]
pub enum DeviationChannel {
    Face { score: f64, baseline_delta_sigma: f64 },
    Arms { score: f64, baseline_delta_sigma: f64 },
    Speech { score: f64, baseline_delta_sigma: f64 },
}

#[derive(Debug, Clone, Serialize)]
pub enum Severity { Mild, Moderate, Severe }

#[derive(Debug, Clone, Serialize)]
pub enum RecommendedAction {
    AllClear,
    RetestIn { minutes: u32 },
    CallDoctor,
    CallEmergency,
}

// ═══════════════════════════════════════
// Medikation (aus dem Buch, Kapitel 2+3)
// ═══════════════════════════════════════

#[derive(Debug, Clone, Serialize)]
pub enum MedicationState {
    Scheduled { due_at: DateTime<Utc> },
    Prepared { at: DateTime<Utc>, by: StaffId },
    Confirmed { at: DateTime<Utc>, by: StaffId },
    WindowExpired { prepared_at: DateTime<Utc>, expired_at: DateTime<Utc> },
    Refused { reason: RefusalReason, at: DateTime<Utc> },
}

#[derive(Debug, Clone, Serialize)]
pub enum RefusalReason {
    InsufficientAuthorization { required: Role, actual: Role },
    InteractionRisk { with_medication: String, severity: String },
    WindowClosed { was_due: DateTime<Utc>, now: DateTime<Utc> },
    ContraindicationActive(String),
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub enum ActionResult {
    Permitted,
    Refused { reasons: Vec<RefusalReason> },
}

#[derive(Debug, Clone, Serialize, Deserialize, PartialEq)]
pub enum Role {
    PermanentNurse,
    ExternalNurse,
    ShiftLead,
    Doctor,
    Trainee,
}

pub type StaffId = String;
pub type PatientId = String;

// ═══════════════════════════════════════
// Ereigniskette
// ═══════════════════════════════════════

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct SystemEvent {
    pub id: String,             // UUID als String (plattformunabhängig)
    pub event_type: EventType,
    pub timestamp: DateTime<Utc>,
    pub actor: Option<StaffId>,
    pub patient: Option<PatientId>,
    pub data: serde_json::Value,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub enum EventType {
    // Morgen-Check
    FaceCheckCompleted,
    ArmCheckCompleted,
    SpeechCheckCompleted,
    HealthStatesDerived,
    // Medikation
    MedicationScheduled,
    MedicationPrepared,
    MedicationConfirmed,
    MedicationWindowExpired,
    MedicationRefused,
    // Vitalwerte
    VitalRecorded,
    VitalThresholdExceeded,
    // System
    SystemRefusal,
    SystemAlert,
    // Pflege
    CareActionDocumented,
    ShiftHandover,
}
```

**Crates:** `chrono`, `rust_decimal`, `serde`, `serde_json`
**Test:** Alle Typen müssen serde round-trip (serialize → deserialize = identisch)

---

### Modul 2: event_chain.rs — DER Kernel

```rust
/// Append-Only Ereigniskette. Kein update. Kein delete.
/// Genau wie im Buch: "Was geschehen ist, ist geschehen."
pub struct EventChain {
    events: Vec<SystemEvent>,
}

impl EventChain {
    pub fn new() -> Self { ... }

    /// Einzige Mutation: anhängen.
    pub fn append(&mut self, event: SystemEvent) { ... }

    /// Lesen: alle Events, gefiltert, nach Patient, nach Typ
    pub fn all(&self) -> &[SystemEvent] { ... }
    pub fn for_patient(&self, id: &PatientId) -> Vec<&SystemEvent> { ... }
    pub fn of_type(&self, t: EventType) -> Vec<&SystemEvent> { ... }
    pub fn since(&self, since: DateTime<Utc>) -> Vec<&SystemEvent> { ... }

    /// Serialisierung: JSON-Export für Persistenz (App speichert)
    pub fn to_json(&self) -> String { ... }
    pub fn from_json(json: &str) -> Result<Self, ...> { ... }

    pub fn len(&self) -> usize { ... }
}
```

**Regel:** Kein `remove()`. Kein `update()`. Kein `clear()`. Append-Only.
**Test:** append → len steigt. from_json(to_json()) = identisch. Kein API für Löschen.

---

### Modul 3: baseline.rs — Persönliche Baseline

```rust
/// 14-Tage gleitender Durchschnitt + Standardabweichung
/// Aus den täglichen Check-Ergebnissen.

pub struct Baseline {
    pub face_avg: f64,
    pub face_std: f64,
    pub arm_avg: f64,
    pub arm_std: f64,
    pub speech_rate_avg: f64,
    pub speech_rate_std: f64,
    pub speech_confidence_avg: f64,
    pub speech_confidence_std: f64,
    pub sample_count: usize,
}

/// Berechnet Baseline aus den letzten N Tagen
pub fn compute_baseline(
    face_scores: &[f64],
    arm_scores: &[f64],
    speech_rates: &[f64],
    speech_confidences: &[f64],
) -> Baseline { ... }

/// Standardabweichungs-Delta: (wert - avg) / std
/// Wenn std == 0 (alle Werte gleich): delta = 0 wenn wert == avg, sonst 10.0
pub fn sigma_delta(value: f64, avg: f64, std: f64) -> f64 { ... }
```

**Minimum:** 7 Tage Daten bevor Baseline gültig ist (davor: kein Alarm, nur Sammeln)
**Test-Orakel:**
- 14× gleicher Score (z.B. 0.05) → avg=0.05, std≈0 → jede Abweichung = groß
- 14× Werte [0.03, 0.04, 0.05, 0.06, 0.07, ...] → avg≈0.05, std≈berechenbar
- sigma_delta(0.05, 0.05, 0.01) = 0.0
- sigma_delta(0.08, 0.05, 0.01) = 3.0

---

### Modul 4: face_score.rs — Gesichtssymmetrie

```rust
/// Berechnet Symmetrie-Score aus Landmark-Koordinaten.
/// Die App liefert die 68 (oder relevanten 6) Punkte,
/// der Kern berechnet die Asymmetrie.
///
/// Score 0.0 = perfekt symmetrisch
/// Score 1.0 = maximal asymmetrisch

pub fn compute_face_symmetry(measurement: &FaceMeasurement) -> f64 {
    let eye_delta = (measurement.left_eye_height - measurement.right_eye_height).abs();
    let mouth_delta = (measurement.left_mouth_y - measurement.right_mouth_y).abs();
    let brow_delta = (measurement.left_brow_y - measurement.right_brow_y).abs();

    // Gewichtung: Mund am stärksten (Schlaganfall-typisch)
    let weighted = eye_delta * 0.25 + mouth_delta * 0.50 + brow_delta * 0.25;

    // Normalisieren auf 0.0-1.0 (Gesichtsproportionen ca. 0-0.3 im Normalbereich)
    (weighted / 0.3).min(1.0)
}
```

**Test-Orakel:**
- Perfekt symmetrisch (alle Deltas 0) → Score 0.0
- Leichte Asymmetrie (Mund-Delta 0.02) → Score < 0.1
- Starke Asymmetrie (Mund-Delta 0.15) → Score > 0.5

---

### Modul 5: arm_score.rs — Armdrift

```rust
/// Normalisierter Drift-Score aus Sensor-Daten.
/// Score 0.0 = stabil gehalten
/// Score 1.0 = Arm sofort abgesunken

pub fn compute_arm_score(measurement: &ArmMeasurement) -> f64 {
    let drift_norm = (measurement.max_drift_degrees / 45.0).min(1.0);
    let hold_norm = 1.0 - (measurement.hold_duration_secs / 10.0).min(1.0);
    let tremor_norm = (measurement.tremor_amplitude / 5.0).min(1.0);

    drift_norm * 0.50 + hold_norm * 0.30 + tremor_norm * 0.20
}
```

**Test-Orakel:**
- 10s gehalten, 0° Drift, 0 Tremor → Score 0.0
- 3s gehalten, 20° Drift, mittel Tremor → Score > 0.4

---

### Modul 6: speech_score.rs — Sprachauswertung

```rust
/// Sprach-Score aus Timing + Confidence.
/// Die App liefert die Messwerte, der Kern bewertet.
/// Score 0.0 = normale Sprache
/// Score 1.0 = schwer beeinträchtigt

pub fn compute_speech_score(measurement: &SpeechMeasurement) -> f64 {
    // Confidence invertieren (niedrige Confidence = schlechte Artikulation)
    let conf_score = 1.0 - measurement.recognition_confidence;

    // Sprechrate: zu langsam (< 80 wpm) oder zu schnell (> 200 wpm) = auffällig
    let rate_score = if measurement.words_per_minute < 80.0 {
        1.0 - (measurement.words_per_minute / 80.0)
    } else if measurement.words_per_minute > 200.0 {
        (measurement.words_per_minute - 200.0) / 100.0
    } else {
        0.0
    }.min(1.0);

    // Pausen-Unregelmäßigkeit
    let pause_score = (measurement.pause_stddev_ms / 500.0).min(1.0);

    // Prosodie: niedrige Varianz = monoton
    let prosody_score = 1.0 - (measurement.pitch_variance / 50.0).min(1.0);

    // Wortfindung: > 2000ms = auffällig
    let word_finding = (measurement.word_finding_delay_ms / 3000.0).min(1.0);

    // Gewichtung
    conf_score * 0.30
        + rate_score * 0.20
        + pause_score * 0.15
        + prosody_score * 0.15
        + word_finding * 0.20
}
```

**Test-Orakel:**
- 120 wpm, confidence 0.95, geringe Pausen-StdDev, lebhafte Prosodie → Score < 0.1
- 60 wpm, confidence 0.50, hohe Pausen-StdDev, monoton → Score > 0.5

---

### Modul 7: health_state.rs — DIE AMPEL

```rust
/// DAS HERZSTÜCK. Leitet den Gesundheitszustand ab.
/// Gleiche Logik wie deriveState() auf der Baustelle.

pub fn derive_health_state(
    face: &FaceMeasurement,
    arms: &ArmMeasurement,
    speech: &SpeechMeasurement,
    baseline: &Baseline,
) -> HealthCheckState {

    // Baseline muss gültig sein (mindestens 7 Tage)
    if baseline.sample_count < 7 {
        return HealthCheckState::Normal {
            confidence: 0.0, // "Ich sammle noch Daten"
        };
    }

    let f_score = compute_face_symmetry(face);
    let a_score = compute_arm_score(arms);
    let s_score = compute_speech_score(speech);

    let f_delta = sigma_delta(f_score, baseline.face_avg, baseline.face_std);
    let a_delta = sigma_delta(a_score, baseline.arm_avg, baseline.arm_std);
    let s_delta = sigma_delta(s_score, baseline.speech_rate_avg, baseline.speech_rate_std);

    let mut channels = Vec::new();

    if f_delta > 2.0 {
        channels.push(DeviationChannel::Face {
            score: f_score, baseline_delta_sigma: f_delta
        });
    }
    if a_delta > 2.0 {
        channels.push(DeviationChannel::Arms {
            score: a_score, baseline_delta_sigma: a_delta
        });
    }
    if s_delta > 2.0 {
        channels.push(DeviationChannel::Speech {
            score: s_score, baseline_delta_sigma: s_delta
        });
    }

    // SONDERREGEL: Face + Speech beide > 3σ → immer Notfall
    // (klassisches Schlaganfall-Muster)
    if f_delta > 3.0 && s_delta > 3.0 {
        return HealthCheckState::Alert {
            channels,
            action: RecommendedAction::CallEmergency,
        };
    }

    match channels.len() {
        0 => HealthCheckState::Normal {
            confidence: 1.0 - f_delta.max(a_delta).max(s_delta) / 10.0
        },
        1 => HealthCheckState::Deviation {
            channels,
            severity: Severity::Mild,
            action: RecommendedAction::RetestIn { minutes: 30 },
        },
        2 => HealthCheckState::Deviation {
            channels,
            severity: Severity::Moderate,
            action: RecommendedAction::CallDoctor,
        },
        _ => HealthCheckState::Alert {
            channels,
            action: RecommendedAction::CallEmergency,
        },
    }
}
```

**Test-Orakel (kritisch — das sind die Abnahme-Tests):**

| Szenario | Face σ | Arms σ | Speech σ | Erwartung |
|----------|--------|--------|----------|-----------|
| Normaler Morgen | 0.3 | 0.5 | 0.2 | Normal |
| Müde (1 Kanal) | 0.8 | 2.5 | 0.4 | Deviation Mild, RetestIn 30 |
| Grippe (2 Kanäle) | 0.5 | 2.3 | 2.8 | Deviation Moderate, CallDoctor |
| Schlaganfall (3 Kanäle) | 3.5 | 4.1 | 3.8 | Alert, CallEmergency |
| Schlaganfall klassisch | 3.5 | 1.0 | 3.5 | Alert (Face+Speech Sonderregel) |
| Baseline < 7 Tage | egal | egal | egal | Normal { confidence: 0.0 } |

---

### Modul 8: medication.rs — Sandras Nein

```rust
/// Exakt wie im Buch, Kapitel 2+3.
/// Kein medication.status = .done.
/// Der Zustand ergibt sich aus Ereignissen und Zeitfenstern.

pub fn derive_medication_state(
    order: &MedicationOrder,
    events: &[SystemEvent],
    now: DateTime<Utc>,
) -> MedicationState { ... }

/// Sandras Nein: evaluate bevor die Handlung passiert.
pub fn evaluate_medication_action(
    medication: &Medication,
    requested_by: &StaffMember,
    patient: &Patient,
    catalog: &MedicationCatalog,
    now: DateTime<Utc>,
) -> ActionResult {
    let mut reasons = Vec::new();

    // Regel 1: Autorisierung (Sandra-Regel)
    if medication.med_type == MedType::OnDemand
        && requested_by.role == Role::ExternalNurse {
        reasons.push(RefusalReason::InsufficientAuthorization {
            required: Role::PermanentNurse,
            actual: Role::ExternalNurse,
        });
    }

    // Regel 2: Wechselwirkungen
    let interactions = check_interactions(
        &medication.substance,
        &patient.active_medications,
        catalog,
    );
    for interaction in interactions {
        reasons.push(RefusalReason::InteractionRisk {
            with_medication: interaction.other_med,
            severity: interaction.severity,
        });
    }

    // Regel 3: Zeitfenster
    if let Some(order) = &medication.current_order {
        if now > order.window_end {
            reasons.push(RefusalReason::WindowClosed {
                was_due: order.scheduled_time,
                now,
            });
        }
    }

    if reasons.is_empty() {
        ActionResult::Permitted
    } else {
        ActionResult::Refused { reasons }
    }
}
```

**Test-Orakel (Sandra-Szenario aus dem Buch):**
- Externe Pflegekraft + Bedarfsmedikation → Refused (InsufficientAuthorization)
- Externe + Bedarfsmed + Metformin aktiv → Refused (2 Gründe)
- Stammkraft + reguläres Medikament + kein Konflikt → Permitted
- Stammkraft + regulär + nach Zeitfenster → Refused (WindowClosed)

---

### Modul 9: interactions.rs — Wechselwirkungsprüfung

```rust
/// Prüft Wirkstoff gegen aktive Medikamente aus dem Katalog.
/// Deterministisch: gleicher Wirkstoff + gleiche aktive Liste = gleiches Ergebnis.

pub fn check_interactions(
    substance: &str,
    active_meds: &[ActiveMedication],
    catalog: &MedicationCatalog,
) -> Vec<Interaction> { ... }
```

**Datenquelle:** `catalog/wechselwirkungen.yaml`
**Test:** Metformin + Novalgin → Interaction gefunden. Metformin + Levodopa → leer.

---

### Modul 10: vitals.rs — Vitalwert-Ampel

```rust
/// Schwellwerte kommen vom Arzt (= Firmenwerte im Bau-Mops).
/// Der Kern rechnet nur: Wert vs. Schwelle → Farbe.

pub fn derive_vital_state(
    measurement: &VitalMeasurement,
    thresholds: &PatientThresholds,
) -> VitalState { ... }

pub enum VitalState {
    Normal { value: f64 },
    Elevated { value: f64, action: RecommendedAction },
    Critical { value: f64, action: RecommendedAction },
}
```

**Test-Orakel:**
- Systolisch 125 bei Schwelle [110-140] → Normal
- Systolisch 155 bei Schwelle gelb [140-160] → Elevated
- Systolisch 185 bei Schwelle rot [160+] → Critical

---

### Modul 11: bourdain_guard.rs — BourdainGuard

```rust
/// Aus dem Buch, Kapitel 9.
/// Sichtbarkeit: NUR für die Person selbst.
/// Der Chef sieht das NICHT.

#[derive(Debug, Clone, Serialize)]
pub struct WellbeingCheck {
    pub consecutive_shifts: u32,
    pub weekly_hours: f64,
    pub show_warning: bool,
    pub visibility: Visibility,
}

#[derive(Debug, Clone, Serialize, PartialEq)]
pub enum Visibility {
    SelfOnly,     // NUR die Person selbst
    ShiftLead,    // + Schichtleitung
    Management,   // + PDL
}

pub fn check_wellbeing(
    shifts: &[ShiftRecord],
    now: DateTime<Utc>,
) -> WellbeingCheck {
    // ... Berechnung ...
    WellbeingCheck {
        // ...
        visibility: Visibility::SelfOnly,  // IMMER. NICHT VERHANDELBAR.
    }
}
```

**Regel:** `visibility` ist HARDCODED auf `SelfOnly`. Kein Parameter. Kein Override.
**Test:** Egal welcher Input → visibility == SelfOnly. Immer.

---

### Modul 12: care_text.rs — Pflege-Textbausteine

```rust
/// Analog zum STLB-Mops: Textbausteine mit Platzhaltern.
/// "Ganzkörperwäsche mit Unterstützung, Bewohner war {{orientierung}}"

pub fn build_care_text(
    template_id: &str,
    values: &HashMap<String, String>,
    catalog: &CareTextCatalog,
) -> Result<String, CareTextError> { ... }
```

**Datenquelle:** `catalog/pflege_textbausteine.yaml`
**Test:** Template "GKW-001" + {"orientierung": "orientiert"} → vollständiger Text

---

## Crates (Abhängigkeiten)

```toml
[dependencies]
chrono = { version = "0.4", features = ["serde"] }
rust_decimal = { version = "1", features = ["serde"] }
serde = { version = "1", features = ["derive"] }
serde_json = "1"
serde_yaml = "0.9"
uniffi = "0.28"

[dev-dependencies]
approx = "0.5"  # float-Vergleiche in Tests

[build-dependencies]
uniffi = { version = "0.28", features = ["build"] }
```

**Keine ML-Crates. Keine Netzwerk-Crates. Keine UI-Crates.**
Nur Serialisierung, Dezimalzahlen, Zeit und die FFI-Brücke.

---

## UniFFI-Export (lib.rs Auszug)

```rust
uniffi::setup_scaffolding!();

// Alle Typen die über die Brücke gehen:
#[uniffi::export]
pub fn derive_health_state(...) -> HealthCheckState { ... }

#[uniffi::export]
pub fn evaluate_medication_action(...) -> ActionResult { ... }

#[uniffi::export]
pub fn derive_vital_state(...) -> VitalState { ... }

#[uniffi::export]
pub fn compute_baseline(...) -> Baseline { ... }
```

UniFFI generiert automatisch:
- `MediMopsCore.swift` → in iOS-App einbinden
- `MediMopsCore.kt` → in Android-App einbinden

---

## Abnahmekriterium

```
cargo test          → ALLE Tests grün
cargo build         → kompiliert ohne Warnungen
cargo clippy        → keine Warnungen
uniffi-bindgen      → Swift + Kotlin Bindings generiert fehlerfrei
```

**Orakel-Tests sind Pflicht:**
- Sandra-Szenario → Refused mit 2 Gründen
- Normaler Morgen → Normal
- Schlaganfall-Muster (Face+Speech 3σ) → Alert CallEmergency
- BourdainGuard → visibility == SelfOnly, IMMER

---

## Geschätzter Aufwand

| Modul | Aufwand |
|-------|---------|
| models.rs | 2h |
| event_chain.rs | 2h |
| baseline.rs | 2h |
| face/arm/speech_score.rs | 3h |
| health_state.rs + Tests | 4h |
| medication.rs + interactions.rs | 4h |
| vitals.rs | 2h |
| bourdain_guard.rs | 1h |
| care_text.rs + YAML-Kataloge | 3h |
| UniFFI-Setup + Bindings | 3h |
| **Gesamt** | **~26h = 3 Arbeitstage** |

---

## Was danach kommt

1. iOS-App: SwiftUI-Shell die den Rust-Kern über UniFFI aufruft
2. Android-App: Kotlin Compose-Shell mit denselben Bindings
3. Sensor-Module: plattformspezifisch (Vision/ARKit vs. ML Kit, CoreMotion vs. SensorManager)
4. Validierungsstudie: Kooperation mit Klinik (wie Pusan National University)
5. MDR-Bewertung: Wellness-App vs. Medizinprodukt-Grenze klären

---

*Der Mops kam in die Küche. Dann auf die Baustelle.*
*Jetzt wird er in Rust geschrieben.*
*Damit er überall Nein sagen kann.*
*Auf jedem Gerät. In jeder Sprache. Für jeden Menschen.*

---

# TEIL 4: SENSORINTEGRATION

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
