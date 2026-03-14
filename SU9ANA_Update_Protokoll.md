# SU9ANA_update — Protokoll & Checkliste

## Auslöser
Neues Spielergebnis liegt vor. Befehl: `SU9ANA_update`

---

## Schritt 1: Ergebnis holen
XTTV-Seiten aufrufen (Browser):
- W1: `https://oettv.xttv.at/ed/index.php?tid=52678&do=spiele`
- W2: `https://oettv.xttv.at/ed/index.php?tid=52679&do=spiele`
- V1: `https://voeb.xttv.at/ed/index.php?tid=3090&do=spiele`
- V2: `https://voeb.xttv.at/ed/index.php?tid=3091&do=spiele`

Notieren:
- DG + Runde
- Datum, H/A
- Score (aus SU9-Sicht: SU9:Gegner)
- Aufstellung SU9 (A/B/C)
- Aufstellung Gegner
- Alle 10 Einzelergebnisse (IIIA, I–IX)

---

## Schritt 2: LIVE-File updaten

### Pflichtfelder pro Sektion:

| Sek | Inhalt | Update |
|-----|--------|--------|
| **01** | big-score | `team su9` = immer **SU9 1/2**, Score, Gegner-Team |
| **01** | card-title | DG + Runde + Datum + H/A |
| **01** | meta links | "Sieg/Niederlage/Remis · X Punkte" |
| **01** | meta mitte | "Heim/Auswärts · Rang X" |
| **01** | stat-chips | Mannschaftspunkte, SU9 Gesamtpunkte, SU9 Rang |
| **01** | analysis | Kurze Spielbeschreibung |
| **02** | section-label | "DG2 RX vs GEGNER" updaten |
| **02** | Spieler-Subtexte | "DG2 RX · Einzel Y/Z · Doppel ..." |
| **02** | Spieler-Details | Einzelergebnisse gegen Gegner-Spieler |
| **02** | Fazit-Text | Auf aktuelles Spiel anpassen |
| **03** | section-label | "Der Gegner · GEGNER" updaten |
| **03** | stat-chips | Rang, Punkte, S-U-N, Sp-V des Gegners |
| **03** | tbody | Gegner-Spieler mit S/N-Bilanz gegen SU9 |
| **03** | Analyse-Text | Stärken/Schwächen des Gegners |
| **04** | section-label | "Kreuzmatrix DG2 RX" updaten |
| **04** | card-title | "SU9 X vs GEGNER · Endstand Y:Z" |
| **04** | Aufstellungszeile | A/B/C und Gegner X/Y/Z |
| **04** | games-grid | Alle 10 Spielzellen (IIIA + I–IX) |
| **05** | Fazit-Text | "Fazit DG2 RX:" aktualisieren |
| **06** | card-title | "Vorschau: SU9 vs NÄCHSTER" |
| **06** | Oberer Block | Datum, H/A, nächster Gegner |
| **06** | stat-chips | Rang/Punkte des nächsten Gegners |
| **07** | card-title | "Stand DD.MM.YYYY" |
| **07** | stat-chips | SU9 Rang, Punkte, Gespielt, Offen |
| **08** | Spieler-Einzel | +Einsätze, +Siege, +Niederlagen pro Spieler |
| **09** | Doppel-Tabelle | +Einsätze, +Siege/Niederlagen pro Paar |
| **10** | card-title | "X offene Spiele" (–1) |
| **10** | Ergebnis-Zeile | Prognose → "Y:Z ✓/✗", tr class hl entfernen |
| **11** | card-title | "X offene Spiele" (–1) |
| **11** | Zeile entfernen | Gespielte Runde aus Tabelle löschen |
| **11** | Nächste Zeile | class="hl" auf nächstes Spiel setzen |

### ⚠ Kritische Fehlerquellen:
1. **big-score**: `team su9` muss IMMER "SU9 1" oder "SU9 2" enthalten — NIE den Gegner-Namen
2. **Score-Richtung**: Score immer aus SU9-Sicht (SU9:Gegner), auch bei Auswärtsspielen
3. **section-label**: Ist ein eigenes Element — NICHT der card-title. Beide updaten!
4. **Spieler-Subtexte**: Stehen direkt in den player-card divs, nicht in der Tabelle

---

## Schritt 3: AR-File erstellen

Neues AR-File in `AR/2025/WTTV/Wx/RAF/Rx/` oder `AR/2025/VÖB/Vx/RAF/Rx/`

Basis: **vorheriges AR-File** kopieren, dann updaten:

| Sek | Update |
|-----|--------|
| `<title>` | "DG2 · RX · TT.MM.JJJJ" |
| Header-Bar | "Stand nach Runde X" |
| **01** | Kompletter Spielbericht (Score, Datum, Aufstellung, Kreuzmatrix) |
| **02** | Spielerleistung (Quote, S/N, Details) |
| **03** | Gegner-Spieler mit Bilanz gegen SU9 in diesem Spiel |
| **04** | Kreuzmatrix dieses Spiels |
| **05** | Gesamtanalyse |
| **06** | Nächstes Spiel zum damaligen Zeitpunkt |
| **07** | Ligatabelle aus `su9_tabellen_v2.json` für diesen DG+Runde |
| **08** | Kumulierte Einzelstatistiken bis diese Runde |
| **09** | Kumulierte Doppelstatistiken bis diese Runde |
| **10** | Alle Ergebnisse bis diese Runde (+neues am Ende) |
| **11** | Ausstehende Spiele (–1 gespieltes, Ergebnis in Klammern) |

---

## Schritt 4: archiv.html updaten

```
<span class="btn wttv pending">GEGNER</span>
→
<a class="btn wttv" href="https://hubertmicheluzzi.github.io/su9-tt/AR/2025/WTTV/Wx/RAF/Rx/SU9Ana_Wx.html" target="_blank">GEGNER</a>
```
VÖB analog mit `class="btn voeb"`.

---

## Schritt 5: Git Push

```bash
git add LIVE/SU9Ana_Wx.html
git add AR/2025/.../Rx/SU9Ana_Wx.html
git add archiv.html
git commit -m "SU9ANA_update: W1 DG2 RX GEGNER Y:Z"
git push ...
```

GitHub Pages: ~40 Sekunden bis live.

---

## Sektion 07 — Tabellendaten

Tabellendaten liegen in `su9_tabellen_v2.json` (lokal + uploaded).
Key-Schema: `data['W1']['DG2_R5']` → Liste mit allen 10 Teams.
Wenn Runde fehlt (0 Teams): Fallback auf vorherige Runde.
VÖB-Teams: `\uFFFD` Encoding-Artefakt zwischen Buchstabe und Zahl → beim Parsen entfernen.

---

*Erstellt: 14.03.2026*
