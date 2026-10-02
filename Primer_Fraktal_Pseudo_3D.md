# Fraktalsee: Eine fantastische Reise - Von der Mandelbrot-artigen Iteration zur pseudo-3D-Fraktallandschaft

Dieser Primer erklärt die Mathematik hinter dem ursprünglichen C-64 Quellcode sowie der vorliegenden JavaScript/Canvas-Renderer Portierung.

Für eine einführende Erklärung was es mit dem ursprünglichen Basic Quellcode für den Commodore 64 aus dem Jahr 1987 auf sich hat und aus welchem Kontext er stammt, möchte ich auf den "anderen Primer" verweisen ("Vom Code zur Mathematik").

Übrigens gibt es zwischen beiden Primern eine gewisse redundante Schnittmenge. Sorry dafür, der Primer "vom Code zur Mathematik" ist erst nach diesem entstanden, um etwas tiefer in Details rund um den Quellcode zu gehen. Wer hier etwas nicht verstehen sollte, kriegt es im anderen Primer u.U. etwas anders oder verbessert erklärt, reinschauen lohnt also ;-)


## 0. Einleitung

Erklärt wird Schritt für Schritt, wie aus einer mathematischen Fraktal-Iteration eine scheinbare 3D-Landschaft entsteht.

Dabei werden drei Dinge miteinander verbunden:

1. eine mathematische Iteration,
2. eine daraus abgeleitete Höhenkarte,
3. eine einfache perspektivische Darstellung mit vertikalen Linien.

Das Ergebnis ist keine echte 3D-Geometrie. Es handelt sich um eine **pseudo-3D-Darstellung**, bei der für jeden sichtbaren Bildpunkt berechnet wird, welcher Punkt der Fraktal-Höhenkarte dort liegt und wie hoch dieser Punkt auf dem Bildschirm erscheinen muss.

---

## 1. Ausgangspunkt: eine komplexe Iteration

Das Mandelbrot-Fraktal basiert auf einer sehr einfachen Vorschrift.

Für jeden Punkt der komplexen Ebene wird eine Zahl `c` betrachtet:

$$
c = x + iy
$$

Dabei sind `x` und `y` die Koordinaten des Punktes und `i` die imaginäre Einheit mit

$$
i^2 = -1
$$

Für jeden Punkt beginnt die Iteration mit

$$
z_0 = 0
$$

und anschließend wird immer wieder

$$
z_{n+1} = z_n^2 + c
$$

berechnet.

Die entscheidende Frage lautet:

> Bleibt die Folge `z₀, z₁, z₂, ...` beschränkt oder wächst sie irgendwann über alle Grenzen?

Daraus entsteht das typische Mandelbrot-Muster.

---

## 2. Die komplexe Rechnung wird zu zwei normalen Zahlen

Für einen Computer ist es nicht notwendig, tatsächlich mit komplexen Zahlen zu rechnen.

Man kann `z` in einen Real- und einen Imaginärteil zerlegen:

$$
z = x + iy
$$

Das Quadrat lautet

$$
(x+iy)^2 = x^2-y^2+2xyi
$$

Damit erhält man zwei normale Zahlen:

$$
x' = x^2-y^2
$$

und

$$
y' = 2xy
$$

Wenn `c` ebenfalls in Real- und Imaginärteil zerlegt wird,

$$
c = a+ib
$$

ergibt sich aus

$$
z_{n+1}=z_n^2+c
$$

die beiden Berechnungsvorschriften

$$
x_{n+1}=x_n^2-y_n^2+a
$$

und

$$
y_{n+1}=2x_ny_n+b
$$

Damit ist die komplexe Fraktalberechnung letztlich nur noch eine Folge von Multiplikationen und Additionen.

---

## 3. Die verwendete Variante

Der hier betrachtete Renderer verwendet keine exakt klassische Mandelbrot-Iteration.

Vor jedem Iterationsschritt werden die Beträge der beiden Komponenten verwendet:

$$
x \rightarrow |x|
$$

$$
y \rightarrow |y|
$$

Damit entsteht eine Mandelbrot-artige Struktur, die der bekannten **Burning-Ship-Variante** ähnelt.

Die Iteration lautet dadurch:

$$
x_{n+1}=|x_n|^2-|y_n|^2+a
$$

$$
y_{n+1}=2|x_n||y_n|+b
$$

Der wesentliche Unterschied zur klassischen Mandelbrot-Menge steckt also in den Beträgen.

Diese kleine Änderung der mathematischen Vorschrift verändert die entstehende Landschaft erheblich.

---

## 4. Die Iteration im JavaScript-Code

Die zentrale Funktion des Renderers sieht vereinfacht so aus:

```js
function iterate(xc, yc) {
  let xa = 0;
  let ya = 0;
  let z = 2;

  while (true) {
    const x2 = Math.abs(xa);
    const y2 = Math.abs(ya);

    const xn = x2 * x2 - y2 * y2 - xc;

    ya = xa * ya;
    ya = ya + ya - yc;

    xa = xn;

    z -= 0.05;

    if (z < 0 || x2 + y2 > 400)
      return { z };
  }
}
```

Die Variablen `xa` und `ya` entsprechen dabei den beiden Komponenten des aktuellen Fraktalwertes.

`xc` und `yc` sind die Koordinaten des Punktes, der untersucht wird.

---

### 5. Der Fluchttest

Bei Fraktalen wird normalerweise geprüft, ob die Iteration „entkommt“.

Im Code geschieht das hier:

```js
x2 + y2 > 400
```

Dabei stehen `x2` und `y2` in dieser Variante bereits für die Beträge der beiden Komponenten. Die Bedingung entspricht daher mathematisch:

$$
|x|+|y|>400
$$

Die Zahl `400` ist somit **keine Radiusangabe**. Sie bezeichnet die Escape-Schwelle der verwendeten Betragssumme.

Zum Vergleich: Bei der klassischen Mandelbrot-Menge wird häufig der Betrag der komplexen Zahl selbst betrachtet:

$$
|z|=\sqrt{x^2+y^2}
$$

und eine typische Escape-Bedingung lautet:

$$
|z|>2.
$$

Das ist geometrisch eine Kreisgrenze. Die im historischen Programm verwendete Bedingung

$$
|x|+|y|>400
$$

folgt dagegen einer anderen Geometrie und sollte deshalb nicht als „Escape-Radius 20“ bezeichnet werden.

Die ungewöhnlich hohe Escape-Schwelle bedeutet, dass die Iteration wesentlich länger laufen kann, bevor ein Punkt als entkommen gilt. Dadurch können sich auch Unterschiede in der Fluchtdauer über einen größeren Bereich der Iteration entwickeln.

Für die Darstellung als Landschaft ist das relevant: Die Fluchtdauer wird anschließend in einen Höhenwert übersetzt. Die konkrete Form der Höhenverteilung hängt dabei allerdings nicht allein von der Escape-Schwelle ab, sondern auch davon, wie die Fluchtdauer im weiteren Programm verarbeitet und auf die Landschaftshöhe abgebildet wird.

---

## 6. Aus der Iterationsdauer wird eine Höhe

Der interessante Schritt für die Landschaftsdarstellung ist, dass die Iteration nicht nur fragt:

> Entkommt der Punkt?

Stattdessen wird gleichzeitig gezählt, **wie lange** die Iteration läuft.

Im Code beginnt die Variable `z` bei

```js
z = 2;
```

und wird bei jeder Iteration um `0.05` verringert:

```js
z -= 0.05;
```

Nach `n` Iterationen gilt daher ungefähr

$$
z = 2-0.05n
$$

Je länger die Iteration läuft, desto kleiner wird `z`.

Damit entsteht aus der ursprünglichen Fraktalberechnung eine Art Höhenwert.

Man kann diesen Wert als Funktion

`H(x,y)`

auffassen:

$$
H(x,y)=2-0.05N(x,y)
$$

Dabei bezeichnet `N(x,y)` die Anzahl der ausgeführten Iterationen.

Damit wird aus

> „Wie schnell entkommt dieser Punkt?“

praktisch

> „Welche Höhe besitzt dieser Punkt?“

---

## 7. Warum daraus eine Landschaft entsteht

Eine gewöhnliche Fraktaldarstellung verwendet die Anzahl der Iterationen meistens nur zur Farbgebung.

Hier wird dieselbe Information stattdessen als **Höhe** interpretiert.

Man kann sich deshalb eine zweidimensionale Fläche vorstellen:

$$
H(x,y)
$$

Für jeden Punkt `(x,y)` existiert ein Höhenwert.

Das Ergebnis ist eine mathematische Höhenkarte.

Die Fraktalberechnung liefert also nicht direkt Berge und Täler.

Sie liefert zunächst nur eine Funktion, die jedem Punkt eine Zahl zuordnet.

Erst die anschließende Darstellung interpretiert diese Zahl als Höhe.

---

## 8. Wasser und Land

Der Renderer führt zusätzlich eine künstliche Meereshöhe ein.

Als Grenze wird

$$
H=0
$$

verwendet.

Damit gilt:

$$
H(x,y)<0 \quad \Rightarrow \quad Wasser
$$

und

$$
H(x,y)\geq0 \quad \Rightarrow \quad Land
$$

Im JavaScript-Code entspricht das sinngemäß:

```js
const isWater = raw < 0;
```

Die Fraktalmathematik selbst kennt also kein Wasser.

Wasser entsteht erst durch die Interpretation des berechneten Höhenwertes.

---

## 9. Das Wasser wird zusätzlich abgeflacht

Für Wasser wird der berechnete Höhenwert anschließend praktisch auf Meereshöhe gesetzt:

```js
z = -1e-4;
```

Mathematisch entspricht das ungefähr

$$
H_{\text{Wasser}}\approx0
$$

Dadurch liegt die Wasseroberfläche nahezu exakt auf einer Ebene.

Das ist wichtig für das Erscheinungsbild:

* Das Fraktal bestimmt weiterhin, wo Wasser und Land liegen.
* Die ursprünglichen negativen Höhen werden aber nicht als unterschiedlich tiefe Täler dargestellt.
* Stattdessen bildet das Wasser eine relativ flache Oberfläche.

Damit entsteht der Eindruck eines Sees oder Meeres in der Landschaft.

---

# 10. Jetzt kommt die Kamera ins Spiel

Bis hierhin existiert lediglich eine mathematische Höhenkarte.

Nun muss daraus ein Bild entstehen.

Die Kamera besitzt drei wesentliche Eigenschaften:

* eine Position,
* eine Blickrichtung,
* eine Augenhöhe.

Die Position kann als

$$
P_0=(x_1,y_1)
$$

geschrieben werden.

Die Blickrichtung wird durch den Winkel `θ` beschrieben.

Die Augenhöhe wird mit `E` bezeichnet.

---

## 11. Die Entfernung vom Betrachter

Der Renderer läuft für verschiedene Entfernungen durch die Landschaft.

Vereinfacht sieht die Schleife so aus:

```js
for (let v = 1; v <= 600; v += 2) {
    ...
}
```

`v` ist dabei keine echte metrische Entfernung.

Es ist ein Projektionsparameter, der gleichzeitig bestimmt:

* wie weit der betrachtete Punkt von der Kamera entfernt ist,
* wie groß seine perspektivische Darstellung wird.

Die horizontale Entfernung wird berechnet als

$$
e=\frac{10E}{v}
$$

Das bedeutet:

* kleines `v` → großer Abstand im Fraktalraum,
* großes `v` → kleiner Abstand.

Die Kamera tastet also die Landschaft entlang ihrer Blickrichtung ab.

---

## 12. Warum das perspektivisch wirkt

Der entscheidende Trick besteht darin, dass die sichtbare Breite mit zunehmender Entfernung kleiner wird.

Dazu wird

$$
e=\frac{10E}{v}
$$

verwendet.

Der Faktor `E` koppelt die Projektion an die Augenhöhe.

Je größer die Augenhöhe ist, desto weiter reicht die Abtastung der Landschaft.

Damit beeinflusst `E` nicht nur die Position der Kamera, sondern gleichzeitig die Perspektive.

---

# 13. Die Blickrichtung als Vektor

Die Blickrichtung lässt sich aus dem Winkel `θ` als Vektor darstellen:

$$
f=
\begin{pmatrix}
\sin(\theta)\\
\cos(\theta)
\end{pmatrix}
$$

Dieser Vektor zeigt in die Blickrichtung.

Für die horizontale Richtung quer zur Blickrichtung wird ein zweiter Vektor benötigt:

$$
s=
\begin{pmatrix}
\cos(\theta)\\
-\sin(\theta)
\end{pmatrix}
$$

`f` bedeutet also „vorwärts“ und `s` bedeutet „seitwärts“.

Diese beiden Vektoren bilden zusammen das Koordinatensystem der Kamera.

---

## 14. Die horizontale Bildposition

Für jede Bildschirmspalte wird zusätzlich ein horizontaler Parameter `ho` verwendet.

Historisch liegt dieser ungefähr im Bereich

$$
-80\leq ho<80
$$

Der Wert `ho` bestimmt, wie weit links oder rechts von der Blickrichtung der betrachtete Punkt liegt.

Die tatsächliche seitliche Verschiebung lautet:

$$
hc=e\cdot ho\cdot0.0187
$$

Der Faktor `0.0187` bestimmt dabei die horizontale Blickfeldbreite.

Für `ho=80` ergibt sich

$$
80\cdot0.0187=1.496
$$

Der entsprechende halbe Blickwinkel beträgt ungefähr

$$
\arctan(1.496)\approx56.2^\circ
$$

Das gesamte horizontale Blickfeld liegt damit ungefähr bei

$$
112.4^\circ
$$

Das ist ein sehr weiter Blickwinkel.

---

## 15. Von Kamera zu Weltkoordinaten

Nun kann für jede Bildschirmposition berechnet werden, welcher Punkt der Fraktallandschaft dort betrachtet wird.

Der allgemeine Vektor lautet:

$$
P=P_0+e\,f+hc\,s
$$

Einsetzen der beiden Richtungsvektoren ergibt:

$$
x_c=x_1+hc\cos(\theta)+e\sin(\theta)
$$

$$
y_c=y_1-hc\sin(\theta)+e\cos(\theta)
$$

Genau diese Rechnung steckt im JavaScript:

```js
const xc = x1 + hc * co + e * si;
const yc = y1 - hc * si + e * co;
```

mit

```js
const angle = bearingDegrees * Math.PI / 180;
const si = Math.sin(angle);
const co = Math.cos(angle);
```

Damit wird aus einer Bildschirmposition und einer Entfernung ein konkreter Punkt `(xc,yc)` in der Fraktalwelt.

---

# 16. Die entscheidende Verbindung

Jetzt treffen die beiden bisher getrennten Teile aufeinander.

Die Kamera liefert einen Punkt:

$$
(x_c,y_c)
$$

Dieser Punkt wird wieder an die Fraktalberechnung übergeben:

```js
const raw = iterate(xc, yc).z;
```

Mathematisch bedeutet das:

$$
H=H(x_c,y_c)
$$

Damit beantwortet der Renderer für jeden betrachteten Punkt die Frage:

> Wie hoch ist die Fraktallandschaft genau dort, wo die Kamera hinschaut?

---

# 17. Die Höhe muss auf den Bildschirm projiziert werden

Jetzt kennen wir die Höhe `H` des betrachteten Landschaftspunktes.

Die Kamera befindet sich auf der Höhe `E`.

Relevant ist daher zunächst der Höhenunterschied zwischen Landschaft und Kamera:

$$
H-E
$$

Ein Berg oberhalb der Augenhöhe besitzt einen positiven Wert.

Eine Landschaft unterhalb der Augenhöhe besitzt einen negativen Wert.

Dieser Höhenunterschied wird anschließend mit der Entfernung `v` und der Augenhöhe `E` in eine **vertikale Projektionskoordinate** umgerechnet.

Im Code heißt diese Koordinate `c`:

```js
let c = (z - eyeHeight) * v / eyeHeight;
```

`c` bezeichnet dabei **nicht die Höhe selbst**. Die Höhe ist `H` beziehungsweise im Code zunächst `z`.

`c` ist vielmehr der aus der Höhe berechnete **vertikale Projektionswert**: Er bestimmt, wie weit der Landschaftspunkt in vertikaler Richtung auf dem Bildschirm von der gedachten Horizontlinie entfernt erscheint.

Mathematisch lautet die Berechnung:

$$
c=\frac{(H-E)v}{E}
$$

Man kann `c` daher als eine Art **vertikalen Bildschirmversatz** verstehen.

* `c > 0` → der Punkt liegt oberhalb der Augenhöhe und erscheint weiter oben im Bild.
* `c = 0` → der Punkt liegt genau auf Augenhöhe.
* `c < 0` → der Punkt liegt unterhalb der Augenhöhe und erscheint weiter unten im Bild.

Die eigentliche Bildschirmposition entsteht anschließend durch eine weitere Verschiebung. Im Renderer wird beispielsweise

```js
verticalLine(screenX, 98 - c, 199, COLOR.red);
```

verwendet.

Damit wird die obere Position des Stäbchens zu

$$
y_{\text{top}}=98-c
$$

Ein positives `c` macht `y_top` kleiner und verschiebt den Punkt damit **nach oben** auf dem Bildschirm. Ein negatives `c` macht `y_top` größer und verschiebt ihn **nach unten**.

Hier steckt der perspektivische Effekt:

Je größer `v` ist, desto stärker wird derselbe Höhenunterschied durch die Formel

$$
c=\frac{(H-E)v}{E}
$$

auf die vertikale Bildschirmposition abgebildet.

Das klingt zunächst umgekehrt zur normalen Perspektivvorstellung, ergibt sich aber aus der speziellen Konstruktion dieser historischen Stäbchenprojektion.

Wichtig ist deshalb die Unterscheidung:

* `H` → **Höhe des Landschaftspunktes**
* `E` → **Augenhöhe der Kamera**
* `H-E` → **Höhenunterschied zur Kamera**
* `v` → **Projektions-/Entfernungsparameter**
* `c` → **vertikaler Projektionswert**
* `y_top = 98-c` → **tatsächliche vertikale Bildschirmposition**

Die horizontale und vertikale Projektion verwenden dabei unterschiedliche, historisch gewachsene Skalierungen.

---

# 18. Die Bedeutung der Augenhöhe

Die Augenhöhe `E` taucht an zwei wichtigen Stellen der Projektion auf. Dabei entstehen zwei unterschiedliche Größen: `e` und `c`.

Sie haben nichts miteinander zu tun, obwohl beide von `E` abhängen.

## `e`: horizontale Entfernung der Abtastposition

Zunächst wird aus der Augenhöhe `E` und dem Projektionsparameter `v` die Größe `e` berechnet:

$$
e=\frac{10E}{v}
$$

`e` bezeichnet die **Entfernung des betrachteten Punktes von der Kamera in Blickrichtung**.

Je größer `e` ist, desto weiter liegt der berechnete Punkt vor der Kamera.

Da `e` umgekehrt proportional zu `v` ist, gilt:

* kleines `v` → großes `e` → weit entfernter Landschaftsbereich
* großes `v` → kleines `e` → näherer Landschaftsbereich

`e` bestimmt also, **wo in der Landschaft** die Kamera gerade abtastet.

Zusammen mit der Blickrichtung `f` entsteht daraus der Vorwärtsanteil der Weltkoordinate:

$$
P=P_0+e\,f
$$

Die Größe `e` gehört damit zur **horizontalen beziehungsweise räumlichen Positionierung** des betrachteten Landschaftspunktes.

---

## `c`: vertikaler Projektionswert

Aus der Höhe des Landschaftspunktes `H`, der Augenhöhe `E` und dem Projektionsparameter `v` wird dagegen `c` berechnet:

$$
c=\frac{(H-E)v}{E}
$$

`c` bezeichnet den **vertikalen Projektionswert**.

Er sagt nicht, wo der Punkt in der Landschaft liegt, sondern wie stark seine Höhe in der vertikalen Bildschirmrichtung sichtbar wird.

Dazu wird zunächst der Höhenunterschied zur Kamera gebildet:

$$
H-E
$$

Anschließend wird dieser Höhenunterschied mit `v` skaliert und durch die Augenhöhe `E` geteilt.

Damit gilt:

* `H > E` → `c > 0` → der Landschaftspunkt liegt über Augenhöhe
* `H = E` → `c = 0` → der Landschaftspunkt liegt auf Augenhöhe
* `H < E` → `c < 0` → der Landschaftspunkt liegt unter Augenhöhe

`c` gehört damit zur **vertikalen Projektion auf den Bildschirm**.

---

## Zwei verschiedene Aufgaben

Die beiden Größen lassen sich deshalb so unterscheiden:

| Größe | Bedeutung                                   | Aufgabe                                               |
| ----- | ------------------------------------------- | ----------------------------------------------------- |
| `e`   | Entfernung des Abtastpunktes von der Kamera | bestimmt **wo** in der Landschaft gesucht wird        |
| `c`   | vertikaler Projektionswert                  | bestimmt **wo** die Höhe auf dem Bildschirm erscheint |

Man kann sich den Ablauf deshalb so vorstellen:

```text
                 Kamera
                   ●
                  /|
                 / |
                /  |  E = Augenhöhe
               /   |
              /    |
             ●     |
       Landschaftspunkt
          ↑
          │ H = Höhe
          │
          └── H - E = Höhenunterschied
```

`e` beantwortet dabei die Frage:

> Wie weit vor der Kamera liegt der Punkt?

`c` beantwortet dagegen die Frage:

> Wie weit oberhalb oder unterhalb der Horizontlinie muss dieser Punkt auf dem Bildschirm erscheinen?

---

## Warum `E` beide Berechnungen beeinflusst

Die Augenhöhe `E` bestimmt damit gleichzeitig zwei Eigenschaften der Darstellung.

Für die Position in der Landschaft:

$$
e = \frac{E}{v} \cdot 10 = \frac{10E}{v}
$$

Für die vertikale Projektion:

$$
c=\frac{(H-E)v}{E}
$$

Eine Veränderung von `E` verändert also sowohl den Bereich der Landschaft, den die Kamera abtastet, als auch die Darstellung der Höhenunterschiede.

Die Augenhöhe ist deshalb nicht einfach nur eine Verschiebung der Kamera nach oben. Sie ist gleichzeitig Bestandteil der **geometrischen Skalierung der Projektion**.

Eine nützliche gedankliche Trennung ist daher:

```text
E = Augenhöhe der Kamera

       │
       ├──→ e = Entfernung des betrachteten Landschaftspunktes
       │
       └──→ c = vertikale Projektion seiner Höhe
```

Damit wird auch verständlich, warum `e` und `c` trotz ihrer ähnlichen Rolle im Code nicht dasselbe bedeuten: `e` arbeitet zunächst im **Welt-/Landschaftsraum**, während `c` bereits Teil der **Bildschirmprojektion** ist.

---

# 19. Aus der Höhe wird eine vertikale Linie

Jetzt kommt der entscheidende grafische Trick.

Der Renderer zeichnet keine Polygone.

Er baut auch kein echtes 3D-Mesh.

Stattdessen wird für jede sichtbare Position eine **vertikale Linie** gezeichnet.

Vereinfacht:

```js
verticalLine(screenX, 98 - c, 199, COLOR.red);
```

Die obere Position ist damit

$$
y_{\text{top}}=98-c
$$

Die Unterkante liegt ungefähr bei

$$
y_{\text{bottom}}=199
$$

Jeder Landschaftspunkt wird dadurch zu einem senkrechten „Stäbchen“.

---

# 20. Warum daraus trotzdem eine Landschaft entsteht

Eine einzelne vertikale Linie sieht natürlich noch nicht wie ein Berg aus.

Entscheidend ist die große Anzahl dieser Linien.

Die Kamera tastet viele Entfernungen ab.

Für jede Entfernung werden viele horizontale Positionen berechnet.

Jede Position liefert einen Höhenwert.

Jeder Höhenwert wird als vertikales Stäbchen gezeichnet.

Dadurch entsteht eine scheinbare Oberfläche.

Das Verfahren lässt sich ungefähr so darstellen:

```text
Fraktalpunkt
     │
     ▼
Höhenwert H
     │
     ▼
Perspektivische Projektion
     │
     ▼
vertikales Stäbchen
     │
     ▼
Pixel
```

Aus sehr vielen solchen Stäbchen entsteht die Landschaft.

---

# 21. Warum von hinten nach vorne gezeichnet wird

Der Renderer verwendet eine klassische Technik aus der Zeit, in der echte 3D-Hardware nicht selbstverständlich war.

Die Landschaft wird von weit nach nah gezeichnet.

Also ungefähr:

```text
weit entfernt
     ↓
     ↓
     ↓
   Kamera
```

Die hinteren Landschaftsteile werden zuerst gezeichnet.

Die näheren Bereiche kommen später darüber.

Dadurch können die vorderen Stäbchen die hinteren überdecken.

Ein echter Z-Buffer ist dafür nicht erforderlich.

---

# 22. Die einfache Verdeckung

Der Renderer merkt sich dazu die zuvor gezeichnete Höhe:

```js
previousC
```

Wenn ein neuer Punkt höher liegt als der vorherige, wird ein schwarzer Bereich zwischen den beiden Höhen gezeichnet:

```js
if (c > previousC + 0.2)
  verticalLine(
    screenX,
    99 - c,
    99 - previousC,
    COLOR.black
  );
```

Die mathematische Idee ist:

$$
c>c_{\text{previous}}+0.2
$$

Dann entsteht eine sichtbare Kante.

Diese Kante verstärkt den Eindruck, dass ein Höhenunterschied vorliegt und der vordere Bereich den hinteren verdeckt.

---

# 23. Begrenzung der vertikalen Projektion

Die Projektion kann theoretisch sehr große Werte erzeugen.

Auf einem 200 Pixel hohen Bild sind solche Werte jedoch nicht sinnvoll.

Deshalb wird `c` begrenzt:

```js
if (Math.abs(c) > 98)
  c = 98 * Math.sign(c);
```

Damit gilt:

$$
-98\leq c\leq98
$$

Das verhindert, dass die Berechnung extreme Werte erzeugt, die weit außerhalb des sichtbaren Bildes liegen.

---

# 24. Glättung der Höhenwerte

Die Fraktalberechnung kann relativ harte Übergänge erzeugen.

Deshalb wird der aktuelle Wert nicht immer direkt verwendet.

Stattdessen wird der vorherige Wert mit dem neuen Wert gemischt:

```js
z = previousZ * 0.7 + raw * 0.3;
```

Mathematisch:

$$
H_{\text{neu}}
=
0.7H_{\text{alt}}
+
0.3H_{\text{frisch}}
$$

Der neue Wert besteht also aus:

* `70 %` des vorherigen Wertes,
* `30 %` des frisch berechneten Wertes.

Das wirkt wie eine einfache Tiefpassfilterung.

Schnelle Änderungen werden abgeschwächt.

Die Landschaft erscheint dadurch weniger zackig.

---

# 25. Die Farben gehören nicht zur Fraktalmathematik

Die Fraktalberechnung liefert zunächst nur Zahlen.

Die Farben werden später festgelegt.

Vereinfacht:

```js
surfaceColor = isWater
  ? COLOR.blue
  : COLOR.green;
```

Die Farbe ist deshalb eine **Darstellungsebene**.

Die Mathematik entscheidet:

> Wie hoch ist dieser Punkt?

Die Darstellung entscheidet:

> Welche Farbe soll dieser Punkt bekommen?

Diese Trennung ist wichtig.

Man könnte dieselbe Höhenkarte beispielsweise auch als:

* Schwarzweißbild,
* Höhenkarte,
* farbige Topografie,
* Mondlandschaft,
* Wasserlandschaft

darstellen, ohne die zugrunde liegende Fraktalberechnung zu verändern.

---

# 26. Die drei Ebenen des gesamten Programms

Der Renderer lässt sich deshalb sehr gut in drei konzeptionelle Ebenen zerlegen.

## Ebene 1: Fraktal

Eingabe:

$$
(x,y)
$$

Ausgabe:

$$
H(x,y)
$$

Diese Ebene weiß nichts von Kamera, Bildschirm oder Farben.

---

## Ebene 2: Kamera und Projektion

Eingabe:

* Bildschirmspalte,
* Entfernung,
* Kameraposition,
* Blickrichtung,
* Augenhöhe.

Ausgabe:

$$
(x_c,y_c)
$$

Diese Ebene beantwortet:

> Welcher Punkt der mathematischen Landschaft wird von dieser Bildschirmposition aus betrachtet?

---

## Ebene 3: Rendering

Eingabe:

$$
(x_c,y_c,H)
$$

Ausgabe:

* Bildschirmposition,
* vertikale Linie,
* Farbe,
* Verdeckung.

Diese Ebene macht aus den mathematischen Daten tatsächlich Pixel.

---

# 27. Die komplette Pipeline

Das gesamte Verfahren lässt sich als Kette darstellen:

```text
Mandelbrot-artige Iteration
          │
          ▼
     Escape Time
          │
          ▼
       H(x,y)
          │
          ▼
    Höhenlandschaft
          │
          ├──── Wasser / Land
          │
          ▼
        Kamera
          │
          ├──── Position
          ├──── Richtung
          └──── Augenhöhe
          │
          ▼
 Perspektivische Projektion
          │
          ▼
      Weltpunkt (x,y)
          │
          ▼
       H(x,y)
          │
          ▼
 Vertikale Projektion
          │
          ▼
   Vertikales Stäbchen
          │
          ▼
         Pixel
```

Der entscheidende Gedankensprung ist dabei:

> Die Kamera rendert nicht eine fertige 3D-Welt. Sie fragt für jede sichtbare Position die mathematische Höhenfunktion ab.

---

# 28. Warum die Methode trotz ihrer Einfachheit funktioniert

Es gibt keine echten 3D-Punkte.

Es gibt keine Dreiecke.

Es gibt keine Flächennormalen.

Es gibt keinen Z-Buffer.

Es gibt lediglich sehr viele Abfragen der Form

$$
H(x,y)
$$

und eine Regel dafür, wo der jeweilige Höhenwert auf dem Bildschirm erscheint.

Die Illusion einer dreidimensionalen Landschaft entsteht allein durch:

1. Perspektive,
2. Höhenunterschiede,
3. Verdeckung,
4. viele vertikale Linien,
5. geeignete Farbgebung.

Gerade diese Einfachheit macht die Methode historisch interessant.

Sie benötigt nur vergleichsweise einfache Mathematik und kann trotzdem eine überzeugende Landschaft erzeugen.

---

# 29. Warum die Berechnung so teuer ist

Der Aufwand steckt vor allem in der Fraktalberechnung.

Nehmen wir beispielsweise ungefähr:

* `320` Bildschirmspalten,
* `300` Entfernungsstufen.

Dann entstehen bereits ungefähr

$$
320\cdot300=96000
$$

Fraktalabfragen.

Und jede einzelne dieser Abfragen besteht wiederum aus mehreren Iterationen.

Die tatsächliche Zahl der arithmetischen Operationen ist daher um ein Vielfaches größer.

Das erklärt, warum eine solche Darstellung auf einem C64 sehr langsam war.

Die eigentliche Landschaft war nicht das Problem.

Das Problem war, dass für praktisch jeden sichtbaren Punkt die Fraktalgleichung erneut berechnet werden musste.

---

# 30. Der entscheidende Unterschied zu einem normalen 3D-Spiel

Ein moderner 3D-Renderer arbeitet normalerweise mit einer Geometrie.

Zum Beispiel:

```text
Vertex → Vertex → Vertex
   \       |
    \      |
     Triangle
```

Die Fraktallandschaft besitzt dagegen zunächst überhaupt keine Geometrie.

Sie ist nur eine Funktion:

$$
H(x,y)
$$

Die Geometrie entsteht erst während der Darstellung.

Man könnte deshalb sagen:

> Die Landschaft wird nicht gespeichert. Sie wird bei Bedarf berechnet.

Das ist ein fundamentaler Unterschied zwischen einer klassischen 3D-Szene und dieser Fraktalprojektion.

---

# 31. Das eigentliche mathematische Gesamtbild

Am Anfang steht eine Iterationsvorschrift:

$$
z_{n+1}=F(z_n,c)
$$

Aus der Anzahl der Iterationen entsteht eine Höhe:

$$
H(x,y)=f(N(x,y))
$$

Die Kamera berechnet daraus einen Weltpunkt:

$$
(x_c,y_c)=P(x_{\text{screen}},v,\text{Kamera})
$$

Die Höhe dieses Punktes wird erneut bestimmt:

$$
H=H(x_c,y_c)
$$

Anschließend wird die Höhe perspektivisch auf eine Bildschirmkoordinate abgebildet:

$$
c=\frac{(H-E)v}{E}
$$

Und schließlich entsteht daraus ein Pixel beziehungsweise ein vertikales Liniensegment.

---

# 32. Der wichtigste Gedanke zum Mitnehmen

Die vielleicht wichtigste Erkenntnis des gesamten Verfahrens ist:

> Eine Fraktallandschaft muss nicht als 3D-Modell existieren, um wie eine 3D-Landschaft auszusehen.

Es genügt, für jeden Punkt `(x,y)` eine Höhe `H(x,y)` zu kennen.

Die Kamera entscheidet anschließend, welche dieser Punkte sichtbar sind und wie ihre Höhen auf dem Bildschirm erscheinen.

Damit wird aus einer zweidimensionalen mathematischen Funktion

$$
H(x,y)
$$

eine scheinbar dreidimensionale Welt.

Oder noch kürzer:

```text
Fraktal
   ↓
Höhenfunktion
   ↓
Kamera
   ↓
Projektion
   ↓
Stäbchen
   ↓
3D-Illusion
```

Genau an dieser Stelle treffen Fraktalmathematik, Perspektivprojektion und die historische „Stäbchen-Technik“ aufeinander.
