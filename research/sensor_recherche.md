# Sensor-Recherche: Schlaganfall-Früherkennung

**Stand:** September 2026
**Recherchiert für:** Medi-Mops Phase 1

---

## Kopf-Sensoren (EEG)

### Ceribell (Sunnyvale, CA — Nasdaq: CBLL)
- FDA Breakthrough Device Designation Januar 2026
- EEG-Haube mit Trockenelektroden, 5 Minuten angelegt
- KI-Algorithmus erkennt Large Vessel Occlusion (LVO) Schlaganfälle
- FDA-zugelassen seit 2017 für Delirium + Anfallserkennung
- Seit April 2025 auch für Kinder ab 1 Jahr
- **Eignung:** Klinik-Only, nicht Consumer

### Muse S Gen 2 (InteraXon, Toronto)
- Consumer-EEG-Stirnband, 4 EEG-Kanäle + PPG
- ~350€, BLE, SDK öffentlich
- Primär für Meditation/Schlaf, aber Rohdaten zugänglich
- **Eignung:** Consumer, Phase 2 geeignet, Nacht-EEG

### IDUN Guardian (Kopenhagen)
- EEG im Ohrstöpsel, 1 Kanal
- ~250€, BLE, SDK offen
- Unsichtbar tragbar, tagsüber nutzbar
- **Eignung:** Consumer, unauffällig, interessant für Langzeit

### Epilog (NINDS-Studie)
- Kabelloses EEG-Pflaster, 2.5 × 2.5 × 0.6 cm, 6.6 Gramm
- Knopfzellenbatterie, Hydrokolloid-Pflaster auf Kopfhaut
- 4 Sensoren (Stirn + hinter Ohren), Bluetooth
- **Eignung:** Forschung, ideal aber nicht Consumer-verfügbar

---

## Arm-Sensoren (Bewegungs-Asymmetrie)

### Neuralert (Philadelphia, UPenn Spin-off)
- FDA Breakthrough Device
- Zwei Armbänder (links + rechts), misst Arm-Asymmetrie
- Erkennt asymmetrische Bewegungen innerhalb 15 Minuten
- Filtert Alltags-Asymmetrie raus (Essen, Telefonieren)
- Automatischer Textalarm an Pflegepersonal
- **Eignung:** Klinik, Algorithmus-Vorbild für Apple Watch

### Apple Watch × 2 (unser Ansatz Phase 1)
- CoreMotion Accelerometer + Gyroskop
- Neuralert-Logik nachbauen: links vs. rechts Aktivität
- Proof of Concept ohne eigene Hardware
- **Eignung:** Consumer, Phase 1, 45€ gebraucht

---

## Herz-Sensoren (Vorhofflimmern → Schlaganfall-Ursache #1)

### Apple Watch Series 4+ (EKG)
- FDA-zugelassen, 98.3% Genauigkeit Vorhofflimmern
- 30-Sekunden-EKG über Digital Crown
- Hintergrund-Rhythmus-Prüfung (passiv, auch SE)
- HRV, kontinuierliche HF, SpO2 (ab Series 6)
- **Eignung:** Consumer, Phase 1, bestellt (Series 5, 45€)

### 14-Tage-ECG-Patch (Taiwan-Studie)
- Langzeit-EKG-Pflaster für paroxysmales Vorhofflimmern
- Wie Langzeit-EKG, nur 14 Tage statt 24h
- **Eignung:** Klinisch, Vorbild für Langzeit-Monitoring

---

## Gesicht + Sprache (Smartphone)

### Vision Framework (Apple)
- VNDetectFaceLandmarks: 76 Punkte
- Symmetrie-Score links/rechts berechenbar
- Frontkamera, kein Extra-Gerät
- **Eignung:** Phase 1, kostenlos, sofort

### ML Kit (Google Android)
- Face Detection: 468 Punkte
- Genauer als Apple Vision, aber Android
- **Eignung:** Phase 2 (Android/Rust)

### SpeechRecognizer (Apple/Google)
- WPM, Confidence, Pausen
- Sprechgeschwindigkeit + Artikulation
- **Eignung:** Phase 1, kostenlos, sofort

---

## Blutdruck

### Withings BPM Connect
- BLE Blutdruckmessgerät, ~100€
- HealthKit-Integration direkt
- **Eignung:** Phase 2, wichtig (Bluthochdruck = Risikofaktor #1)

---

## Zusammenfassung: Was Andreas für Phase 1 braucht

| Kanal | Hardware | Kosten | Status |
|-------|----------|--------|--------|
| Arms + Vitals | Apple Watch Series 5 | 45€ | ✅ bestellt |
| Face | iPhone Frontkamera | 0€ | ✅ vorhanden |
| Speech | iPhone Mikrofon | 0€ | ✅ vorhanden |
| Typing | Mac Tastatur | 0€ | ✅ vorhanden |
| Brain (EEG) | — | — | Phase 2 |
| Blutdruck | — | — | Phase 2 |
| **Gesamt Phase 1** | | **45€** | |
