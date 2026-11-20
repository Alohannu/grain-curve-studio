# Grain & Curve Studio

Ein einzelnes HTML-Dokument — kein Build, keine Abhängigkeiten, kein Netzwerkzugriff.
Es rendert weiche, gekörnte Duotone-Verläufe entlang einer frei biegbaren kubischen
Bézier-Kurve und exportiert sie als PNG oder WebP in Full HD, QHD oder 4K.

**English:** A single-file, dependency-free browser tool that renders grainy duotone
gradients along a freely editable cubic Bézier curve and exports them as PNG or WebP
up to 4K.

## Loslegen

`index.html` im Browser öffnen — fertig:

```
open index.html
```

Es gibt nichts zu installieren und nichts zu bauen. Wer es über einen Server laufen
lassen will (GitHub Pages, lokaler Test):

```
python3 -m http.server 8000
# danach http://127.0.0.1:8000/ öffnen
```

## Wofür das gut ist

Große Flächenverläufe auf 8-Bit-Displays zeigen Streifen (Banding). Ein feines,
deterministisches Filmkorn darüber kaschiert sie, ohne das Bild zu verwaschen — und
eine Kurve statt eines linearen Verlaufs macht den Übergang organisch. Das Werkzeug
ist genau dafür gebaut: Verlaufsbilder erzeugen, die als Übergang zwischen zwei
Seiten einer Website dienen (halbhoch beschnitten, verlustfrei gespeichert, mit
`image-rendering: pixelated` eingebunden).

## Bedienung

### Kurve (Maus oder Touch)

| Bedienelement | Wirkung |
| --- | --- |
| **Punkte ziehen** | Die vier Kontrollpunkte P0–P3 verschieben — oder direkt auf die Kurve greifen und sie organisch biegen |
| **Kurve zeichnen** | Freihand zeichnen; die Skizze wird auf eine kubische Bézier-Kurve zurückgerechnet |
| **Handles: An/Aus** | Kontrollpunkte, Tangenten und Hilfslinien ein- und ausblenden (werden nie mitexportiert) |
| **Hell / Dunkel** | Licht- und Schattenseite vertauschen |
| **Zufallskurve** | Neue zufällige Kurvenform |
| **Neues Korn** | Neuen Zufallsstartwert für das Korn (die Kurve bleibt) |

### Verlauf

| Regler | Bereich | Wirkung |
| --- | --- | --- |
| **Übergangsbreite (SDF)** | 12–100 % | Breite des weichen Bandes um die Kurve. Der Verlauf entsteht aus dem vorzeichenbehafteten Abstand (Signed Distance Function) zur Kurve, nicht aus einer Farbrampe |
| **Zweitschicht (Tiefe)** | 0–100 % | Eine zweite, breitere und weichere Welle parallel zur Kurve. Verhindert flache oder fleckige Übergänge |
| **Kontrast** | 0–100 % | S-Kurven-Schärfe und Anziehen von Schwarz- und Weißpunkt |

### Duotone

Sechs Voreinstellungen — **S/W**, **Mitternacht**, **Sepia**, **Cyanotypie**, **Sand**,
**Laut** — oder ein eigenes Paar aus Licht- und Schattenfarbe (Farbwähler oder
Hex-Eingabe). Die 256-stufige Farbtabelle wird in **Oklab** interpoliert, damit die
Rampe perzeptuell gleichmäßig läuft. Das neutrale Paar `#000000`/`#ffffff` wird
bewusst als Identität gebaut und ist damit exakt die Ausgabe eines reinen
Schwarzweiß-Renderers.

### Filmkorn

| Regler | Bereich | Wirkung |
| --- | --- | --- |
| **Kornstärke** | 0–100 % | Amplitude; folgt der Helligkeit — in extremen Schatten und Lichtern dezenter, in den Mitten am stärksten |
| **Korngröße** | 1,00–2,60 px | Blockgröße des Korns |

Das Korn ist deterministisch: ein 32-Bit-Integer-Hash pro Pixel, geglättet durch
Summation von drei unabhängigen Stichproben (Irwin-Hall-Näherung an eine
Glockenkurve). Deshalb ist es reproduzierbar, fleckenfrei und läuft nicht weg —
und deshalb sieht es wie analoges Filmkorn aus statt wie Blockrauschen.

### Vorschau

Während des Ziehens rechnet das Studio eine billige Entwurfsvorschau (rund 130 000
statt mehrerer Millionen Pixel) und skaliert sie weich hoch. **Scharf rechnen** startet
den vollen Durchlauf: streifenweise über `requestAnimationFrame` mit einem Zeitbudget
pro Frame, damit die Oberfläche bedienbar bleibt.

### Export

* **Auflösungen:** 1920 × 1080, 2560 × 1440, 3840 × 2160
* **Formate:** PNG oder WebP mit Qualitätsregler (0,10–1,00)
* **Dateiname:** `grain-curve-<Breite>x<Höhe>.<Endung>`
* Blockiert die Umgebung den automatischen Download (z. B. ein sandboxed iframe),
  erscheint ein Fallback-Link zum Öffnen und Speichern der Datei

### Persistenz

Der aktuelle Stand liegt im `localStorage` unter `grain-curve-studio:v1` und wird beim
nächsten Aufruf wiederhergestellt. **Reset** setzt auf die Standardwerte zurück und
löscht den gespeicherten Stand.

## Technik

* Reines Vanilla JavaScript und Canvas 2D — kein Framework, kein Bundler, keine
  externen Schriften oder Bibliotheken. Eine Datei, offline lauffähig.
* Verlauf über Signed Distance Function zur abgetasteten Bézier-Polylinie
  (seitenverhältnis-korrigiert, damit die Übergangsbreite in beide Richtungen gleich
  aussieht).
* Duotone über eine 256er-LUT in Oklab.
* Export und Vorschau teilen sich dieselbe Pixelmathematik — der Export ist also
  exakt das, was die Vorschau zeigt.
* Keine Netzwerkzugriffe, kein Tracking.

Getestet in aktuellen Versionen von Chrome, Edge und Safari. Der WebP-Export setzt
`canvas.toBlob` mit `image/webp` voraus; fehlt das, bleibt der PNG-Export.

## Lizenz

Noch keine Lizenz hinterlegt. Ohne Lizenz gilt „alle Rechte vorbehalten" — wenn das
Werkzeug weiterverwendet werden darf, empfiehlt sich eine explizite Lizenz (z. B. MIT).
