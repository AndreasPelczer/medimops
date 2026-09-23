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
