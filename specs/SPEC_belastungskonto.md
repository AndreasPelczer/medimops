# SPEC: Belastungskonto — Körperliche Belastung sichtbar machen

**Status:** Spezifikation — noch kein Code
**Einordnung:** Brücke zwischen Bau-Mops und Medi-Mops
**Prinzip:** Der Mops hat die Daten schon. Er muss sie nur anders lesen.

---

## Was es ist

Jede Position in einem LV hat einen Aufwandswert (Stunden pro Einheit) und eine
Kolonne (wer macht es). Was fehlt ist die dritte Dimension: **was macht es mit
dem Menschen?**

Das Belastungskonto rechnet aus dem was der Mops schon weiß — Positionen, Mengen,
Aufwandswerte, Kolonnen — die physische Belastung pro Arbeiter, pro Woche, pro
Jahr, pro Berufsleben.

Keine neuen Sensoren. Keine neue App. Nur eine neue Spalte in der YAML und eine
Aggregation über die Zeit.

---

## Die Belastungskategorien

Sechs Kategorien, angelehnt an das französische "compte professionnel de prévention"
und die arbeitsmedizinische Forschung zu Berufskrankheiten (BK-Liste, DGUV):

| Kategorie | Was gemeint ist | Messeinheit | Beispiel |
|-----------|----------------|-------------|----------|
| schwere_last | Heben, Tragen, Schieben > 15kg | Stunden | Steine setzen, Schalung tragen |
| haltung | Ergonomisch belastende Position | Stunden + Typ | kniend (Pflasterer), über_kopf (Decke), gebückt (Graben) |
| vibration | Hand-Arm oder Ganzkörper | Stunden | Rüttelplatte, Bohrhammer, Bagger |
| hitze_kaelte | Extreme Temperaturen | Stunden | Asphalt Sommer, Rohbau Winter, Küche |
| laerm | Dauerpegel > 80 dB(A) | Stunden | Abbruch, Sägen, Rammen |
| hoehe | Arbeiten > 2m Absturzhöhe | Stunden | Dach, Gerüst, Fassade |

Jede Kategorie hat drei Stufen: **niedrig / mittel / hoch.**
Die Stufe bestimmt den Multiplikator fürs Konto.

---

## Ergänzung der aufwandswerte_bau.yaml

Jeder bestehende Eintrag bekommt einen neuen Block `belastung:`.
Keine bestehenden Felder ändern sich. Nur ein Block dazu.

### Beispiele (alle 99 Einträge brauchen das, hier die wichtigsten)

**Erdarbeiten:**

| Key | Aufwandswert | Belastungsprofil |
|-----|-------------|------------------|
| baugrube_ausheben_bagger | 0.05 h/m³ | vibration: hoch, schwere_last: niedrig, haltung: sitzend, laerm: mittel |
| oberboden_abtragen | 0.008 h/m² | vibration: mittel, schwere_last: niedrig, haltung: sitzend |
| boden_verdichten | 0.020 h/m² | vibration: hoch (Rüttelplatte), schwere_last: mittel, laerm: mittel |
| graben_ausheben | 0.15 h/m | schwere_last: hoch (Handschachtung), haltung: gebückt, vibration: niedrig |
| frostschutzschicht | 0.012 h/m³ | vibration: mittel, schwere_last: niedrig, haltung: sitzend |
| boden_laden_transport | 0.05 h/m³ | vibration: hoch (LKW), schwere_last: niedrig, laerm: mittel |

**Beton- und Stahlbetonarbeiten:**

| Key | Aufwandswert | Belastungsprofil |
|-----|-------------|------------------|
| bodenplatte | 0.90 h/m² | schwere_last: hoch (Bewehrung), haltung: gebückt, vibration: mittel (Rüttler) |
| stahlbeton_wand | 1.20 h/m² | schwere_last: hoch (Schalung+Bewehrung), haltung: stehend, hoehe: niedrig |
| stahlbeton_decke | 1.50 h/m² | schwere_last: hoch, haltung: über_kopf, hoehe: mittel (Deckenschalung) |
| sauberkeitsschicht | 0.15 h/m² | schwere_last: mittel, haltung: gebückt |
| betonieren_allgemein | 0.40 h/m³ | schwere_last: mittel (Pumpe), vibration: mittel (Rüttler), laerm: mittel |
| bewehrung_verlegen | 8.0 h/t | schwere_last: hoch (Stahl biegen/tragen), haltung: gebückt+kniend |

**Mauerarbeiten:**

| Key | Aufwandswert | Belastungsprofil |
|-----|-------------|------------------|
| mauerwerk_poroton | 0.60 h/m² | schwere_last: hoch (15-25kg/Stein), haltung: stehend+über_kopf |
| mauerwerk_ks_planstein | 0.55 h/m² | schwere_last: hoch (20-30kg/Stein), haltung: stehend |
| mauerwerk_porenbeton | 0.45 h/m² | schwere_last: mittel (leichter), haltung: stehend |
| mauerwerk_klinker | 0.90 h/m² | schwere_last: mittel, haltung: stehend+gebückt |
| sturz_einbauen | 0.30 h/St | schwere_last: hoch (Fertigteile 30-80kg), hoehe: niedrig |

**Pflasterarbeiten:**

| Key | Aufwandswert | Belastungsprofil |
|-----|-------------|------------------|
| verbundpflaster_verlegen | 0.45 h/m² | schwere_last: hoch, haltung: kniend, vibration: mittel (Rüttelplatte) |
| natursteinpflaster | 0.80 h/m² | schwere_last: hoch, haltung: kniend, hitze_kaelte: hoch (draußen) |
| bordstein_setzen | 0.30 h/m | schwere_last: hoch (Steine 30-50kg), haltung: gebückt |
| splittbett_herstellen | 0.08 h/m² | schwere_last: mittel, haltung: gebückt |
| fugen_einsanden | 0.03 h/m² | vibration: mittel, haltung: stehend |

**Dacharbeiten:**

| Key | Aufwandswert | Belastungsprofil |
|-----|-------------|------------------|
| dacheindeckung_tondachziegel | 0.35 h/m² | hoehe: hoch, schwere_last: mittel, hitze_kaelte: hoch |
| lattung_konterlattung | 0.12 h/m² | hoehe: hoch, schwere_last: mittel, haltung: gebückt |
| dach_abdecken | 0.15 h/m² | hoehe: hoch, schwere_last: mittel, laerm: niedrig |
| daemmung_zwischensparren | 0.25 h/m² | hoehe: mittel, haltung: über_kopf, schwere_last: niedrig |
| dachrinne_montieren | 0.30 h/m | hoehe: hoch, schwere_last: niedrig, haltung: stehend |

**Abbruch:**

| Key | Aufwandswert | Belastungsprofil |
|-----|-------------|------------------|
| mauerwerk_abbrechen | 0.50 h/m² | schwere_last: hoch, vibration: hoch (Bohrhammer), laerm: hoch |
| asphalt_aufnehmen | 0.02 h/m² | vibration: hoch (Fräse), laerm: hoch, schwere_last: niedrig |
| boden_aufnehmen | 0.10 h/m² | schwere_last: mittel, haltung: gebückt, vibration: mittel |

**Trockenbau:**

| Key | Aufwandswert | Belastungsprofil |
|-----|-------------|------------------|
| staenderwand_einfach | 0.40 h/m² | schwere_last: mittel (GK-Platten 25kg), haltung: stehend |
| staenderwand_doppelt | 0.60 h/m² | schwere_last: mittel, haltung: stehend |
| decke_abhaengen | 0.50 h/m² | haltung: über_kopf (den ganzen Tag!), schwere_last: mittel, hoehe: mittel |
| spachteln_q2 | 0.10 h/m² | haltung: stehend+über_kopf, schwere_last: niedrig |

**Putz:**

| Key | Aufwandswert | Belastungsprofil |
|-----|-------------|------------------|
| innenputz_maschinenputz | 0.20 h/m² | haltung: stehend+über_kopf, schwere_last: mittel, vibration: niedrig |
| aussenputz | 0.40 h/m² | haltung: stehend, hoehe: mittel (Gerüst), hitze_kaelte: mittel |

**Maler:**

| Key | Aufwandswert | Belastungsprofil |
|-----|-------------|------------------|
| waende_streichen | 0.08 h/m² | haltung: stehend, schwere_last: niedrig |
| decke_streichen | 0.12 h/m² | haltung: über_kopf (!), schwere_last: niedrig |
| fassade_streichen | 0.10 h/m² | hoehe: mittel (Gerüst), haltung: stehend |
| lackieren_tuer | 0.80 h/St | haltung: kniend+stehend, schwere_last: niedrig |

**Fliesen:**

| Key | Aufwandswert | Belastungsprofil |
|-----|-------------|------------------|
| wandfliesen | 0.50 h/m² | haltung: stehend+kniend, schwere_last: mittel |
| bodenfliesen | 0.45 h/m² | haltung: kniend (den ganzen Tag!), schwere_last: mittel |

**Sanitär:**

| Key | Aufwandswert | Belastungsprofil |
|-----|-------------|------------------|
| wasserleitung_verlegen | 0.35 h/m | haltung: gebückt+kniend (Schlitze), schwere_last: niedrig |
| abwasserleitung_verlegen | 0.30 h/m | haltung: gebückt, schwere_last: mittel |
| wc_montieren | 1.50 h/St | haltung: kniend, schwere_last: mittel |
| waschbecken_montieren | 1.00 h/St | haltung: stehend, schwere_last: niedrig |

**Kanalbau:**

| Key | Aufwandswert | Belastungsprofil |
|-----|-------------|------------------|
| kanalrohr_verlegen | 0.40 h/m | schwere_last: hoch (Rohre), haltung: gebückt (Graben), hitze_kaelte: mittel |
| schacht_setzen | 4.0 h/St | schwere_last: hoch (Fertigteile, Kran), vibration: mittel |

---

## Belastungspunkte — die Umrechnung

Jede Stunde in einer Belastungskategorie ergibt Punkte:

| Stufe | Multiplikator |
|-------|--------------|
| niedrig | 0.5 Punkte/Stunde |
| mittel | 1.0 Punkte/Stunde |
| hoch | 2.0 Punkte/Stunde |

Sonderfaktoren die den Multiplikator erhöhen:

| Faktor | Aufschlag |
|--------|-----------|
| kniend + schwere_last gleichzeitig | ×1.5 (Knie-Rücken-Kombi) |
| über_kopf + schwere_last gleichzeitig | ×1.5 (Schulter-Nacken) |
| vibration + sitzend > 6h/Tag | ×1.3 (Wirbelsäule Bagger) |
| hoehe + hitze_kaelte gleichzeitig | ×1.3 (Dach im Sommer/Winter) |

---

## Rechnung: Vom LV zum Belastungskonto

### Schritt 1: Wochenkonto (aus dem aktuellen LV)

Der Mops kalkuliert die Baustelle "Parkplatz Bestenheid":

| Position | Menge | h gesamt | Kolonne | h/Person | Belastung |
|----------|-------|----------|---------|----------|-----------|
| Verbundpflaster | 960 m² | 432h | 2 Pflasterer | 216h | schwere_last: hoch, haltung: kniend |
| Bordstein setzen | 180 m | 54h | 2 Tiefbauer | 27h | schwere_last: hoch, haltung: gebückt |
| Frostschutz | 480 m³ | 5.8h | 1 Baggerfahrer | 5.8h | vibration: mittel |
| Planum herstellen | 850 m³ | 17h | 1 Baggerfahrer | 17h | vibration: hoch |

Für Pflasterer A (8 Wochen Baustelle, 40h/Woche):
- schwere_last hoch: 216h → 216 × 2.0 = 432 Punkte
- haltung kniend: 216h → 216 × 2.0 = 432 Punkte
- Kombi kniend+schwere_last: × 1.5
- **Wochenkonto: (432 + 432) × 1.5 ÷ 8 Wochen = 162 Punkte/Woche**

Für Baggerfahrer:
- vibration hoch: 17h → 17 × 2.0 = 34 Punkte
- vibration mittel: 5.8h → 5.8 × 1.0 = 5.8 Punkte
- haltung sitzend + vibration > 6h/Tag: × 1.3
- **Wochenkonto: (34 + 5.8) × 1.3 ÷ 8 Wochen = 6.5 Punkte/Woche**

Der Pflasterer hat 25× mehr Belastungspunkte als der Baggerfahrer.
Das Rentensystem sieht beide gleich.

### Schritt 2: Jahreskonto (Summe aller Baustellen)

| Quartal | Baustelle | Wochen | Punkte/Woche | Punkte Quartal |
|---------|-----------|--------|-------------|----------------|
| Q1 | EFH Müller (Pflaster Hof) | 3 | 145 | 435 |
| Q2 | Parkplatz Bestenheid | 8 | 162 | 1.296 |
| Q3 | Gehweg Schule | 4 | 120 | 480 |
| Q4 | Kirchplatz Sanierung | 6 | 170 | 1.020 |
| **Jahr** | | **21 Wochen aktiv** | | **3.231 Punkte** |

Zum Vergleich — Büroangestellter:
52 Wochen × (haltung sitzend: 40h × 0.5) = 1.040 Punkte/Jahr.

Pflasterer: 3.231. Büroangestellter: 1.040. Faktor 3,1.

### Schritt 3: Lebenskonto (Aggregation über Jahre)

| Alter | Berufsjahr | Jahrespunkte | Kumuliert | Ampel |
|-------|-----------|-------------|-----------|-------|
| 18 | 1. Lehrjahr | 1.800 | 1.800 | 🟢 |
| 25 | 8 | 3.000 | 22.000 | 🟢 |
| 35 | 18 | 3.200 | 55.000 | 🟢 |
| 45 | 28 | 3.100 | 86.000 | 🟡 |
| 50 | 33 | 2.800 | 100.000 | 🟠 |
| 55 | 38 | 2.500 | 112.000 | 🔴 |

Schwellwerte (aus arbeitsmedizinischer Forschung, kalibrierbar):
- 🟢 < 80.000: Normaler Verschleiß
- 🟡 80.000 - 100.000: Erhöhtes Risiko, Prävention empfohlen
- 🟠 100.000 - 120.000: Hohes Risiko, Umsetzung prüfen
- 🔴 > 120.000: Kritisch, Erwerbsminderung wahrscheinlich

---

## Die Ampel — Drei Richtungen

### Für den Arbeiter selbst (BourdainGuard — visibility: selfOnly)

> "Dein Belastungskonto steht bei 86.000 Punkten (GELB).
> Diese Baustelle bringt 1.300 Punkte dazu.
> Bei deinem aktuellen Tempo erreichst du ORANGE mit 51.
> Willst du mit dem Polier über den Baggereinsatz reden?"

Keine Anweisung. Eine Information. Der Mensch entscheidet.

### Für den Polier / Bauleiter (visibility: teamView, anonymisiert)

> "Kolonne Pflaster: 2 Arbeiter, Durchschnitts-Belastung 162 P/Wo.
> Empfehlung: Rotation mit Kolonne Tiefbau (38 P/Wo) nach 4 Wochen."

Kein Name. Keine persönlichen Daten. Nur die Kolonne-Belastung als Durchschnitt.

### Für den Rentenantrag (visibility: export, nur auf Wunsch des Arbeiters)

> "Belastungsprofil Andreas Pelczer, 36 Berufsjahre:
> - 18.400 Stunden schwere Last
> - 12.600 Stunden Hitze > 35°C (Küche)
> - 8.200 Stunden Stehen > 8h/Tag
> - 4.100 Stunden Nachtarbeit
> - Kumuliertes Belastungskonto: 128.000 Punkte
> - Vergleichswert Büroangestellter gleichen Alters: 38.000 Punkte
> - Faktor: 3,4×"

Das ist das PDF das kein Arzt heute ausstellen kann.
Weil kein Arzt 36 Jahre lang Daten gesammelt hat.

---

## Was sich im System NICHT ändert

- EventChain bleibt Append-Only
- deriveState() leitet ab, setzt nicht
- Das Nein bleibt ein Datenpunkt
- YAML-Kataloge bleiben die Wissensbasis
- Der Kern (Rust) bleibt plattformunabhängig
- Keine Cloud. Daten bleiben lokal.

## Was dazukommt

- 1 neue Spalte `belastung:` in aufwandswerte_bau.yaml (6 Kategorien × 3 Stufen)
- 1 neues Modul `belastungskonto.rs` im Rust-Kern (Aggregation + Ampel)
- 1 neue View `BelastungsView` (persönliches Konto, Wochen/Jahr/Leben)
- 1 Export-Funktion: PDF Belastungsprofil
- Verbindung zum Medi-Mops: Grip-Strength, Beweglichkeit, Schlafqualität
  als Gegenprobe zu den berechneten Belastungspunkten

---

## Was das politisch bedeutet

Wenn 10.000 Handwerker ihre anonymisierten Belastungsprofile teilen:

> "Dachdecker erreichen ORANGE im Schnitt mit 48 Jahren.
> Büroangestellte erreichen ORANGE nie.
> Das Renteneintrittsalter ist für beide 67."

Das ist kein Meinungsartikel. Das sind Daten.
Und Daten die aus echten Baustellen kommen — nicht aus einer Studie,
nicht aus einer Befragung, sondern aus dem Werkzeug das die Arbeit selbst
dokumentiert — die kann kein Politiker wegdiskutieren.

Der Mops misst nicht die Meinung. Er misst die Arbeit.
Und rechnet sie hoch. Und runter. Und über ein Leben.

---

**Zusammenfassung:**

| Was | Wo | Status |
|-----|----|--------|
| Aufwandswerte mit Belastungsprofil | aufwandswerte_bau.yaml | Spalte ergänzen |
| Belastungspunkte-Rechner | belastungskonto.rs (Rust-Kern) | Modul spezifiziert |
| Wochen/Jahr/Lebenskonto | BelastungsView | UI spezifiziert |
| BourdainGuard-Integration | visibility: selfOnly | Architektur steht |
| PDF Belastungsprofil | Export-Funktion | Format spezifiziert |
| Verbindung Medi-Mops | Grip-Strength vs. Belastung | Schnittstelle offen |
| Politische Aggregation | Anonymisiert, opt-in | Konzept steht |

---

*Der Mops hat die Daten. Er musste sie nur anders lesen.*
*Nicht: "Was kostet diese Wand?"*
*Sondern: "Was kostet diese Wand den Menschen der sie baut?"*
