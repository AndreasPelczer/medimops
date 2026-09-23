# 🐕 Medi-Mops

**Deterministische Gesundheitsüberwachung — ein Mops, alle Domänen**

Derselbe Kernel der dem Koch das Filet sperrt und dem Polier den Beton,
erkennt beim Patienten die Abweichung — und sagt: "Ruf jetzt an."

---

## Was ist das?

Medi-Mops ist ein Schlaganfall-Frühwarnsystem das auf dem [iMOPS](https://github.com/AndreasPelczer/iMOPS-Construction-Grid-Baustellen-Management.) Kernel aufbaut.
Keine KI die rät. Keine Cloud die speichert. Keine App die Abo will.

Stattdessen: `deriveState()`. Der Zustand wird nicht gesetzt — er wird abgeleitet.
Aus dem was gemessen wird, verglichen mit deiner persönlichen Baseline.

## Prinzip

```
5 Kanäle messen → Baseline vergleichen → Ampel ableiten → handeln

🧠 Brain    EEG (Muse S / IDUN Guardian)     → optional
⌚ Arms     Apple Watch (Arm-Asymmetrie)      → Phase 1
📱 Face     iPhone Kamera (Gesichtssymmetrie) → Phase 1
🎤 Speech   iPhone Mikrofon (Sprachmuster)    → Phase 1
❤️ Vitals   Apple Watch (HF, HRV, EKG, SpO2) → Phase 1
```

Kein Sensor ist Pflicht. Jeder macht den Mops besser.
Minimum: iPhone + Apple Watch = 4 Kanäle.

## Die Ampel

```
🟢 GRÜN    Alles im Korridor. Nichts tun.
🟡 GELB    2+ Kanäle > 2σ Abweichung. Nochmal messen.
🔴 ROT     3σ+ oder 3+ Kanäle auffällig. Alarm-Kaskade.
```

## Architektur

```
┌─────────────────────────────────────────┐
│  Sensoren (Watch, iPhone, optional EEG) │
├─────────────────────────────────────────┤
│  Adapter (Swift: HealthKit, Vision,     │
│           SpeechRecognizer, CoreMotion) │
├─────────────────────────────────────────┤
│  Medi-Mops Kernel                       │
│  deriveState() · EventChain · Baseline  │
│  SYSTEM_REFUSAL · Ampel · Alarm        │
├─────────────────────────────────────────┤
│  Alles lokal. Keine Cloud. Deine Daten. │
└─────────────────────────────────────────┘
```

## Status

| Phase | Was | Wann |
|-------|-----|------|
| ✅ | Spezifikation komplett (4 Dokumente, 85 KB) | Sept 2026 |
| ✅ | Hardware bestellt (Apple Watch Series 5, 45€) | Sept 2026 |
| 🟡 | Phase 1: Swift-App für iOS (persönlicher Prototyp) | Q4 2026 |
| ⬜ | Phase 2: Rust-Kernel (Cross-Platform) | 2027 |
| ⬜ | Phase 3: Android (Brasilien) | 2027 |

## Dokumentation

```
docs/
├── MASTERPLAN_medi_mops.md          — Roadmap, Budget, Regulierung
├── PLAN_medi_mops_integration.md    — 5 Phasen, Kernel-Transfer
├── PLAN_medi_mops_sensorintegration.md — 5 Kanäle, Hardware, 4 Setups
└── MEDI_MOPS_KOMPLETT.md            — Alles in einer Datei

specs/
├── CODI_AUFTRAG_medi_mops_core_rust.md — 12 Rust-Module, Orakel-Tests
└── SPEC_belastungskonto.md          — Körperliche Belastung sichtbar machen

research/
└── sensor_recherche.md              — Ceribell, Neuralert, Muse S, IDUN Guardian
```

## Die Kette

```
2024  Küche      → iMOPS HACCP (Swift, iOS)
2025  Baustelle  → iMOPS Construction Grid (Swift, iOS)
2026  Motor      → mops-engine (Python) + Stammdaten (YAML)
2027  Pflege     → Medi-Mops (Rust → Android + iOS)
```

Der Mops wächst. Aber er ändert sich nicht.
Er sagt immer noch: Nein.

## Philosophie

- **Offline-First.** Deine Gesundheitsdaten verlassen dein Gerät nicht.
- **Deterministisch.** Gleiche Messung = gleiche Bewertung. Kein Raten.
- **SYSTEM_REFUSAL.** Der Mops sagt Nein wenn Daten fehlen.
- **BourdainGuard.** `visibility: .selfOnly` — nur du siehst deine Daten.
- **Graceful Degradation.** 2 Kanäle reichen. 5 sind besser.

---

*Code lügt nicht, Fantasie schon.*

**Autor:** Andreas Pelczer · Wertheim · 2026
