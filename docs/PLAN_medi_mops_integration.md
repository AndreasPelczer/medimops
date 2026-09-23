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
