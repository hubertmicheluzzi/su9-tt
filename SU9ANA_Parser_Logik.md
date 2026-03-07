# SU9ANA · Parser-Logik

## 1. Datenquelle

XTTV-Seiten werden direkt per Browser (Claude in Chrome) geladen:

| Liga | URL |
|------|-----|
| WTTV W1 | `https://oettv.xttv.at/ed/index.php?tid=52678&do=spiele` |
| WTTV W2 | `https://oettv.xttv.at/ed/index.php?tid=52679&do=spiele` |
| VÖB V1  | `https://voeb.xttv.at/ed/index.php?tid=3090&do=spiele`  |
| VÖB V2  | `https://voeb.xttv.at/ed/index.php?tid=3091&do=spiele`  |

Extraktion via `get_page_text` — der Plaintext enthält alle Spiele, Spieler und Ergebnisse in einem kompakten Format.

---

## 2. Struktur des XTTV-Plaintexts

### 2a. Spielzeile (Kopf)

```
Dg. R. Spieldatum  Mannschaften           Erg.   Spielort   Anmerkungen
1   3  Do. 02.10.2025 19:00  SU91 - SSH3  6:3   Vereinsheim  ...
```

**Felder:**
- `Dg.` = Durchgang (1=RAH=Herbst, 2=RAF=Frühjahr)
- `R.` = Runde (1–9 bei WTTV, 1–11 bei VÖB)
- Datum/Zeit
- Heim - Gast
- Ergebnis (z.B. `6:3`) — aus Heim-Perspektive
- Spielort

**Sonderfall:** `Spiel noch nicht eingegeben.` → Status = ausstehend

---

### 2b. Kreuzmatrix-Spielfolge

Nach dem Spielkopf folgen die Einzelergebnisse in fixer Reihenfolge:

**WTTV/VÖB (3 Spieler je Seite = 10 Partien):**

| Pos | Heim | Gast |
|-----|------|------|
| I    | A | 1 |
| II   | B | 2 |
| III  | C | 3 |
| IIIA | A+C (Doppel Heim) | 2+3 (Doppel Gast) |
| IV   | A | 2 |
| V    | B | 3 |
| VI   | C | 1 |
| VII  | A | 3 |
| VIII | B | 1 |
| IX   | C | 2 |

**Achsenlogik:**
- Heim-Spieler = senkrecht (1, 2, 3) → **Ziffern**
- Gast-Spieler = waagrecht (A, B, C) → **Buchstaben**
- SU9 spielt als **Heim** → SU9-Spieler sind 1/2/3
- SU9 spielt als **Gast** → SU9-Spieler sind A/B/C

**Achtung:** Das Ergebnis steht immer aus **Heim-Sicht** (z.B. `3:1` = Heim gewinnt 3:1).

---

### 2c. Aufstellung parsen

Die Spielernamen im Plaintext stehen paarweise:
```
[Heim-Spieler] [Gast-Spieler] [Ergebnis]
```

Beispiel (SU9 als Heim, Pos I = Heim-A vs. Gast-1):
```
Micheluzzi  SSH-Spieler  3:1
```

**SU9-Aufstellung ableiten:**
- Position I/II/III = Heim A/B/C (wenn SU9 Heim)
- Doppel IIIA: erster Name = Heim-Spieler 1 des Doppels

**Gastspielererkennung:** Spieler der nicht in der Stamm-Bindungsliste der Liga steht → Gastspieler, in Analyse als `(Gast)` markieren.

---

## 3. Ergebnis-Normierung

- Ergebnis immer **ohne Space**: `6:3` nicht `6 : 3`
- Aus Heim-Sicht: `6:3` = Heim gewinnt
- Für SU9-Perspektive umrechnen wenn SU9 = Gast: Ergebnis umdrehen → `3:6` für SU9
- Klassifikation:
  - `win` = SU9 ≥ 6 Mannschaftspunkte (oder Gegnerpunkte < SU9)
  - `loss` = SU9 < Gegner
  - `draw` = 5:5
  - `special` = Strafpunkte (0:7 oder 7:0 bei Nichtantreten)

---

## 4. Sonderfälle

| Muster im Plaintext | Bedeutung |
|---------------------|-----------|
| `Spiel noch nicht eingegeben.` | Ausstehend → `pending`-Button in archiv.html |
| `Spielfrei` / kein Gegner | Spielfrei → leerer `free`-Button |
| `w.o.` bei Spielernennung | Walkover — Gegner tritt nicht an → 3:0 gewertet |
| `Heimmannschaft nicht angetreten` | Strafverifizierung → Ergebnis 0:7 oder 7:0 |
| `(PT)` im Spielplan | Platztausch oder Terminänderung — Ort im Bericht beachten |
| `Von Gastmannschaft bestätigt` | Normaler Status — keine Sonderbehandlung |

---

## 5. Spielerstatistik-Auswertung

Aus den Einzel-Ergebnissen wird je Spieler berechnet:
- Siege / Niederlagen (nur echte Einzel, w.o. separat vermerkt)
- Siegquote in % → Farbe: ≥60% grün, 40–59% gelb, <40% rot
- Doppel separat ausgewiesen
- Gastspieler mit `(Gast)` im Namen

**Analyseregel:** Spielerentwicklung immer nach **Datum der Austragung** sortieren, nicht nach Rundennummer. Nachholspiele können chronologisch nach späteren Runden liegen.

---

## 6. Gegner-Kurzbezeichnungen

Immer Kurzform in Tabellen und result-bar. Langname nur im Analysetext.

**WTTV W1 · 3. Klasse B:**
`HAN1 · TT143 · SSH3 · LENZ4 · NFS7 · LSV6 · EDEN4 · SKFL10 · KONT7`

**WTTV W2 · Gruppe Ib:**
`DOEB3 · POSW5 · NFS10 · UNO2 · SKFL12 · HAK4 · LENZ6 · KORN5 · SOK2`

**VÖB V1 · 2. Klasse:**
`HGI2 · POST2 · WILI5 · SKH3 · BRZ2 · FSF1 · BTTC2 · FJB4 · HÄRM1`

**VÖB V2 · 3. Klasse:**
`SKH4 · HGI3 · POST3 · OLY3 · BBSV1 · SKH5 · WILI7 · WILI6 · LENZ2`

---

## 7. Datei-Output-Schema

### AR-Files (Archiv)

```
AR/JAHR/VERBAND/Liga/DG/Rx/SU9Ana_Xx.html
```

Beispiel: `AR/2025/WTTV/W1/RAF/R3/SU9Ana_W1.html`

- `RAH` = DG1 = Herbst
- `RAF` = DG2 = Frühjahr
- Dateiname: `SU9Ana_W1.html`, `SU9Ana_W2.html`, `SU9Ana_V1.html`, `SU9Ana_V2.html`

### LIVE-Files

```
LIVE/SU9Ana_W1.html  (wird bei SU9ANA_update überschrieben)
```

Vor dem Überschreiben: aktuelles LIVE-File → AR-Ordner kopieren.

---

## 8. HTML-Ausgabe-Struktur

### result-bar

```html
<div class="result-bar">
  <span class="rb-team">SU9 1</span>
  <span class="rb-score" style="color:var(--green)">6:3</span>
  <span class="rb-team">SSH3</span>
  <span class="badge win">SIEG</span>
</div>
```

- `rb-team`: IBM Plex Mono, 15px, `var(--txt2)`
- `rb-score`: IBM Plex Mono, 32px, bold, farbig je Ergebnis
- `badge`: win=grün, loss=rot, draw=gelb, special=lila

### Sektionen

| Nr | Inhalt |
|----|--------|
| 01 | Spielanalyse (result-bar + Kreuzmatrix-Tabelle) |
| 01a | Detailanalyse (nur bei Mehrfach-Runden im LIVE-File) |
| 01b | Kurznotiz einfache Spiele |
| 02 | Spielerleistung (player-cards mit Siegquote) |
| 03 | Gegneranalyse / Gastspieler-Analyse |
| 04 | Kreuzmatrix |
| 05 | Gesamtanalyse & Ausblick |
| 06 | Nächstes Spiel |
| 07 | Tabelle |
| 08 | Spielerstatistik (kumuliert) |
| 09 | Doppelstatistik |
| 10 | Alle Ergebnisse |
| 11 | Ausstehende Spiele |

---

## 9. Design-System

```css
--bg:     #0d0f14   /* Haupthintergrund */
--bg2:    #141720   /* Sektionen */
--bg3:    #1c2030   /* Cards */
--txt:    #e2e8f0
--txt2:   #94a3b8
--border: #2a3045
--green:  #4ade80
--yellow: #fbbf24
--red:    #f87171

/* Liga-Akzentfarben */
--W1: #4a9eff   /* WTTV1 blau */
--W2: #a78bfa   /* WTTV2 violett */
--V1: #34d399   /* VÖB1 grün */
--V2: #fb923c   /* VÖB2 orange */
```

Font: **IBM Plex Mono** (Code, Scores, Labels) + **IBM Plex Sans** (Fließtext)

---

## 10. archiv.html — Button-Zustände

| Zustand | CSS-Klasse | Beschreibung |
|---------|------------|--------------|
| gespielt | `.btn.wttv` / `.btn.voeb` | Farbiger Link, klickbar → öffnet AR-File |
| spielfrei | `.btn.free` | Leerer Button, gedimmt (opacity 0.25), kein Text |
| ausstehend | `.btn.pending` | Gegner-Name, hellgraue Schrift `#6b7280`, nicht klickbar |
| Label | `.btn-label` | Weißer Text, kräftigerer Hintergrund, kein Link |

**Workflow:** Sobald Ergebnis vorliegt → `SU9ANA_update` → AR-File erstellen → Button-Status: `pending` → `ok` + `href` setzen.
