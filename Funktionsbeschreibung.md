# Funktionsbeschreibung — Temperatur-Analyse-Notebook

## Zweck

Das Notebook liest Messdaten eines mehrkanaligen Datenloggers ein, rechnet Rohspannungen in Temperaturen um und stellt die Ergebnisse als interaktiven Plot dar. Es unterstützt zwei unterschiedliche Sensortypen (NTC und Diode) und bietet umfangreiche Filter- und Darstellungsoptionen.

---

## Ablauf

### 1. Kalibrierung laden und prüfen

Aus der `UTOT_PATH`-Datei wird eine Zuordnung **Spannung → Temperatur** für den Dioden-Sensor aufgebaut. Dabei wird zunächst nach Position und Phase gefiltert (`UTOT_FILTER`), anschließend pro Temperatur der Mittelwert gebildet. Auf Basis dieser Stützpunkte wird eine Regressionsfunktion erstellt. Vor der eigentlichen Auswertung wird die Regression grafisch dargestellt, damit man die Qualität visuell prüfen kann.

### 2. Kanalbeschriftungen laden

Die Excel-Datei unter `LABELS_PATH` liefert sprechende Namen für jeden Kanal (z.B. „Luft Einlass" statt „CH1"). Diese werden im Plot als Legendeneinträge verwendet.

### 3. CSV einlesen und umrechnen

Die Logger-CSV wird eingelesen. Für jeden Messpunkt werden die Rohspannungen bereinigt (Vorzeichen, Leerzeichen, Dezimaltrennzeichen). Danach werden zwei Kanäle automatisch in Temperaturen umgerechnet:

- **NTC-Kanal** (`NTC_CHANNEL`): Umrechnung über die Steinhart-Hart-Gleichung (B-Parameter-Variante) aus der gemessenen Spannung am Spannungsteiler.
- **Dioden-Kanal** (`DIODE_CHANNEL`): Umrechnung über die zuvor erstellte Regressionsfunktion aus der UtoT-Kalibrierung.

Alle übrigen Kanäle bleiben als Rohwerte (Spannung) erhalten.

### 4. Interaktiver Plot

Die Temperaturdaten aller aktiven Kanäle werden in einem interaktiven Matplotlib-Plot dargestellt.

---

## Funktionen im Detail

### NTC-Umrechnung (`ntc_voltage_to_celsius`)

Berechnet aus der gemessenen Spannung am NTC-Spannungsteiler die Temperatur in °C. Der Rechenweg:

1. Widerstand des NTC aus Spannungsteiler-Formel bestimmen
2. Temperatur über die B-Parameter-Gleichung berechnen
3. Ergebnis von Kelvin in Celsius umrechnen

### UtoT-Kalibrierung (`build_ch11_converter`)

Erstellt eine Konvertierungsfunktion Spannung → Temperatur für den Dioden-Sensor. Vier Methoden stehen zur Verfügung:

| Methode | Beschreibung | Geeignet für |
|---------|-------------|--------------|
| `linear` | Polynom Grad 1 | Annähernd linearer Zusammenhang |
| `poly2` | Polynom Grad 2 | Leichte Krümmung |
| `poly3` | Polynom Grad 3 | Stärkere Krümmung |
| `spline` | Kubischer Spline | Exakte Interpolation zwischen Stützpunkten (keine Extrapolation) |

Bei Polynomen wird zusätzlich der R²-Wert ausgegeben. Bei zu wenigen Datenpunkten für einen Spline fällt die Methode automatisch auf `linear` zurück.

### UtoT-Filterung (`load_utot`)

Filtert die Kalibrierdaten vor der Regression nach Position und Phase. Die Filterwerte im Überblick:

| Filter | Wirkung |
|--------|---------|
| `MW` | Kein Filter — Mittelwert über alle Messungen |
| `L` / `H` | Nur Position Low bzw. High |
| `L_U`, `L_V`, `L_W` | Position Low, jeweils Phase U, V oder W |
| `H_U`, `H_V`, `H_W` | Position High, jeweils Phase U, V oder W |

### Regressionsplot (`plot_ch11_regression`)

Zeigt die Kalibrierpunkte (Mittelwerte) als Scatter-Plot zusammen mit der gewählten Regressionskurve. Dient zur visuellen Kontrolle, ob die gewählte Methode gut zu den Daten passt.

### CSV-Import (`read_and_clean`)

Liest die Logger-CSV ein und führt folgende Schritte durch:

- Überspringt die ersten 33 Header-Zeilen
- Weist den Spalten Namen zu (Index, Datetime, Intervall, CH1–CHn, Alarme)
- Entfernt komplett leere Spalten
- Bereinigt Spannungswerte (Vorzeichen, Leerzeichen, Dezimaltrennzeichen)
- Rechnet `NTC_CHANNEL` und `DIODE_CHANNEL` automatisch in °C um

### Interaktiver Plot (`plot_channels_interactive`)

Erstellt einen Matplotlib-Plot mit folgenden interaktiven Bedienelementen:

| Widget | Funktion |
|--------|----------|
| **Zeitbereich-Slider** | Auswahl des dargestellten Messpunktbereichs per Schieberegler |
| **Von / Bis Textfelder** | Exakte numerische Eingabe des Zeitbereichs |
| **Manuelle Y-Achse** | Umschalten zwischen automatischer und manueller Y-Skalierung |
| **Y min / Y max** | Manuelle Grenzen der Y-Achse (nur bei aktivierter manueller Skalierung) |
| **< 20°C ausblenden** | Blendet alle Werte unter 20 °C aus (nützlich zum Filtern kalter Sensoren) |
| **Kelvin anzeigen** | Wechselt die Darstellung von °C auf Kelvin |
| **Peak Diode zoom** | Zoomt automatisch auf den höchsten Temperaturpeak des Dioden-Kanals (±1 Messpunkt) |

Zusätzlich können über `HIDE_CHANNELS` einzelne Kanäle dauerhaft aus dem Plot ausgeblendet werden.

---

## Zusammenfassung der Fähigkeiten

- Einlesen von Logger-CSVs mit beliebiger Kanalanzahl
- Automatische NTC-Temperaturberechnung (Steinhart-Hart / B-Parameter)
- Dioden-Temperaturberechnung über externe Kalibrierungstabelle
- Flexible Filterung der Kalibrierungsdaten nach Position und Phase
- Vier Regressionsmethoden mit visueller Qualitätsprüfung
- Benutzerdefinierte Kanalbeschriftungen aus Excel
- Interaktiver Plot mit Zoom, manueller Skalierung, Kelvin-Umschaltung und Peak-Zoom
- Kanäle ein-/ausblenden
