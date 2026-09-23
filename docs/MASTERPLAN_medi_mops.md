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
