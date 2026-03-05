# Konfigurationsdokumentation

## Übersicht

Das Notebook liest Messdaten (CSV), Kanalbeschriftungen (Excel) und eine Spannungs-Temperatur-Kalibrierung (Excel) ein und stellt die Temperaturverläufe interaktiv dar.

---

## `CSV_PATH` — Messdaten-CSV

**Format:** Komma-separierte CSV-Datei, exportiert von einem mehrkanaligen Datenlogger.

**Aufbau:**

- **Zeilen 1–33** werden übersprungen (Header/Metadaten des Loggers) → `skiprows=33`
- Ab Zeile 34 folgen die Messdaten mit diesen Spalten (ohne Header-Zeile):

| Spalte | Inhalt | Beispiel |
|--------|--------|----------|
| 1 | Laufender Index (Messpunkt-Nr.) | `1` |
| 2 | Datum/Uhrzeit | `2025-03-04 17:19:42` |
| 3 | Intervall | `0:00:01` |
| 4–13 | CH1 bis CH10 — Messwerte | `+1.2345` |
| 14 | Alarm 1–10 | |
| 15 | Alarm 11–20 | |
| 16 | Alarm Out | |

**Hinweise:**

- Spannungswerte können ein führendes `+` und Leerzeichen enthalten (wird bereinigt).
- Dezimaltrennzeichen kann Komma oder Punkt sein.
- Die Anzahl der Kanäle wird über `MAX_CH` gesteuert (Standard: 10 → CH1 bis CH10).
- **Wichtig — Kanalzuordnung ist rein positionsbasiert:** Das Skript benennt die Datenspalten strikt der Reihe nach als CH1, CH2, CH3, … Es wird nicht ausgewertet, welche physischen Kanäle am Logger tatsächlich belegt sind. Wenn z.B. nur die Logger-Kanäle 10–17 benutzt werden, erscheinen diese im Skript trotzdem als CH1–CH8. Die Zuordnung in `LABELS_PATH` (und die Einstellungen `NTC_CHANNEL` / `DIODE_CHANNEL`) muss sich daher immer auf die **Position in der CSV** beziehen, nicht auf die Kanalnummer am Gerät.

---

## `LABELS_PATH` — Kanalbeschriftungen (Excel)

**Format:** `.xlsx`-Datei mit mindestens einem Sheet, dessen Name dem Konfigurationswert `sheet_name` entspricht (z.B. `ECK2`).

**Erforderliche Spalten:**

| Spalte | Inhalt | Beispiel |
|--------|--------|----------|
| `channel` | Kanalname | `CH1` |
| `label` | Anzeigename für den Plot | `Sensor oben links` |

**Beispiel-Sheet `ECK2`:**

```
channel    label
CH1        Luft Einlass
CH2        Luft Auslass
CH3        Oberfläche Mitte
...
CH10       Diode Gehäuse
```

---

## `UTOT_PATH` — Spannungs→Temperatur-Kalibrierung (Excel)

**Format:** `.xlsx`-Datei mit einem Sheet namens `sheet_name` (z.B. `ECK2`).

**Erforderliche Spalten** (Namenssuche erfolgt per Teilstring, Groß-/Kleinschreibung egal):

| Spalte | Erkennung durch Teilstring | Inhalt | Einheit |
|--------|---------------------------|--------|---------|
| Temperatur | `temp`, `°c`, `temperatur` | Referenztemperatur | °C |
| Spannung | `span`, `mv`, oder beginnt mit `u` | Gemessene Diodenspannung | **mV** |
| Position | `pos` | Messposition | `L` oder `H` |
| Phase | `phase` | Messphase | `U`, `V` oder `W` |

**Beispiel-Sheet `ECK2`:**

```
Temperatur [°C]    Spannung [mV]    Position    Phase
25                 520.3            L           U
25                 519.8            L           V
25                 521.1            L           W
25                 518.5            H           U
50                 480.2            L           U
50                 479.9            L           V
...
```

**Hinweise:**

- Temperaturwerte dürfen Lücken haben (werden per `ffill` aufgefüllt — d.h. der Wert gilt für alle nachfolgenden Zeilen bis zur nächsten Temperaturangabe).
- Dezimaltrennzeichen kann Komma oder Punkt sein.
- Der `UTOT_FILTER` bestimmt, welche Zeilen verwendet werden (z.B. `L_W` → nur Position `L` und Phase `W`). Danach wird pro Temperatur der Mittelwert der Spannung gebildet.

---

## Konfigurationsparameter

| Parameter | Beschreibung | Standard |
|-----------|-------------|----------|
| `MAX_CH` | Anzahl Messkanäle (CH1 bis CHn) | `10` |
| `sheet_name` | Excel-Sheet-Name in Labels- und UtoT-Datei | `'ECK2'` |
| `NTC_CHANNEL` | Kanal mit NTC-Spannungsteiler | `'CH9'` |
| `DIODE_CHANNEL` | Kanal mit Dioden-Sensor (nutzt UtoT-Kalibrierung) | `'CH10'` |
| `V_SUPPLY` | Versorgungsspannung des NTC-Spannungsteilers | `3.3` V |
| `R_FIX` | Festwiderstand im Spannungsteiler | `20000` Ω |
| `R0` | NTC-Nennwiderstand bei T0 | `47000` Ω |
| `B_CONST` | B-Konstante des NTC | `4050` |
| `T0_K` | Referenztemperatur des NTC | `298.15` K (25 °C) |
| `CH11_REGRESSION` | Regressionsmethode für Diode: `linear`, `poly2`, `poly3`, `spline` | `'linear'` |
| `UTOT_FILTER` | Filter für UtoT-Daten: `MW`, `L`, `H`, `L_U`, `L_V`, `L_W`, `H_U`, `H_V`, `H_W` | `'L_W'` |
| `HIDE_CHANNELS` | Liste auszublendender Kanäle, z.B. `['CH7']` | `['CH7']` |
