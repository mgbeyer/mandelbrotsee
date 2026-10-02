# Fraktalsee: Eine fantastische Reise - Vom Code zur Mathematik

## Vom Fraktal zur C64-Landschaft

Dieser Primer erklärt die JavaScript/Canvas-Renderer Portierung des ursprünglichen C-64 Quellcodes. Und zwar nicht von der Mathematik aus, sondern **vom vorhandenen Quellcode rückwärts**:

> **Was bedeutet diese Codezeile mathematisch – und warum steht sie dort?**


---

# 0. Der Original Commodore 64 Basic Quellcode

```basic
1 XQ=49152:XW=49241:FOR I=0 TO 303:READ A:POKE XQ+I,A:NEXT 
2 PI=3.14159265
20 POKE 53281,6:POKE 53280,0:POKE 646,1:INPUT"X=";X1:INPUT"Y=";Y1 
50 Z=2:XC=X1:YC=Y1:XA=0:YA=0:GOSUB 270 
60 PRINT"HIER IST ES ";Z;"HOCH":INPUT"DEINE HOEHE";H:IF H<Z THEN RUN 
70 INPUT"BLICKRICHTUNG=";KU:KU=KU*PI/180:SI=SIN(KU):CO=COS(KU) 
80 SYS XQ:FOR V=1 TO 600 STEP 2:E=H/V*10:FOR HO=-80 TO 80 
130 FA=3:Z=2:HC=E*HO*.0187:XA=0:YA=0:XC=X1+HC*CO+E*SI:YC=Y1-HC*SI+E*CO 
160 GOSUB 270:Z=ZA*.7+Z*.3:IF Z<0 THEN FA=0:Z=-1E-4 
250 C=Z-H:C=C*V/H:ZA=Z:IF ABS(C)>98 THEN C=98*SGN(C) 
261 SYS 49385,81+HO,98-C,199,2:IF C<0 THEN SYS XW,80+HO,99-C,FA 
264 IF C>CA+.2 THEN SYS 49368,80+HO,99-C,99-CA,0 
265 SYS XW,80+HO,98-C,FA:CA=C:NEXT HO,V 
266 GOTO 266 
270 X2=ABS(XA):Y2=ABS(YA):XN=X2-Y2-XC:YA=XA*YA:YA=YA+YA-YC:XA=XN:Z=Z-.05 
271 IF Z<0 OR X2+Y2>400 THEN RETURN 
272 GOTO 270 
32000 DATA 173,17,208,9132,141,171208,173, 22,208,9,16,141,22,208,173,24,208,9 
32001 DATA 8,141,24,208,169,6,141,33,208,1 62,0,169,49,157,0,4,15710,5,157,016 
32002 DATA 157,0,7,169,5,157,0,216,157,0,2,17,157,0,218,157,0,219,202,208,225
32003 DATA 169,0,133,251,169,32,113,252,160,0,169,0,145,251,230,251,208,250,230 
32004 DATA 252,165,252,201,64,208,240,96,32,155,183,134,251,32,155,183,134,252 
32005 DATA 32,155,183,134,215,169,0,133,253,169,32,133,254,165,252,74,74,74,170 
32006 DATA 240,16,169,64,24,101,253,133,253,165,254,105,1,133,254,202,208,240
32007 DATA 165,251,74,74,10,10,10,144,3,230,254,24,101,253,133,253,169,0,101,254
32008 DATA 133,254,165,252,4117,24,101,253,133,253,169,0,101,254,133,254,165,251
32009 DATA 41,3,133,216,169,3,56,229,216,170,165,215,224,0,240,5,10,10,202,208
32010 DATA 251,133,12,166,216,16010,177,253,61,212,192,5,12,145,253,96,63,207
32011 DATA 243,252,32,21,193,32,104,192,165,252,197,2,240,4,230,252,208,243,96
32012 DATA 32,21,193,166,251,202,134,172,166,252,134,173,166,2,232,134,174,166
32013 DATA 215,202,134,175,32,219,192,165,172,133,251,165,173,133,252,165,174
32014 DATA 133,2,165,175,133,215,76,219,192,32,155,183,134,251,32,155,187,134
32015 DATA 252,32,155,183,134,2,32,155,183,134,215,96,104,240,48,208,16,80
```

Dieses Listing stammt aus der Ausgabe 6/87 der legendären deutschen Computerzeitschrift "Happy Computer". Man findet heute die Scans dieser alten Zeitschriften in Form von PDF Dateien im Internet (z.B. Archive.org oder Kultpower.de).

Die ganzen wilden `DATA`, `POKE` und `SYS` Befehle implementieren dabei Grafik Primitive (Setzen von Farben und Punkten) in Maschinensprache und müssen uns an dieser Stelle nicht wirklich interessieren, Canvas übernimmt in der modernen Portierung das Zeichnen. Das damalige schwache Basic v2 des Commodore 64 hatte solche Grafikbefehle soweit ich mich erinnere nicht mit an Bord, und wenn wäre es hoffnungslos zu langsam gewesen.

Interessant in diesem Zusammenhang ist die Aussage zur Laufzeit der Implementationen von damals. Der Artikel enthielt nicht nur die Implementierung für 8-bit Computer wie den Commodore 64, sondern auch u.a. Quellcodes für 16-bit Maschinen dieser Ära wie der Commodore Amiga oder dem Atari ST. Auf einem alten 8-bit Homecomputer brauchte die Berechnung eines einzigen Bildes mehrere Stunden. Eine 16-bit Maschine wie die Amiga erledigte das bereits in wenigen Minuten (und dies in reinem Basic). Im Artikel schrieb der Autor dann scherzhaft "Wer zufällig eine Cray besitzt, sollte unbedingt mit dem Joystick fliegen...". Nun, heute braucht es keinen teuren Supercomputer mehr, wir "fliegen" durch die Landschaft rund um den See in unserem Webbrowser, wie die vorliegende JavaScript Implementierung beweist :-)

Es ist übrigens auch absolut erstaunlich, wie viel hochgradig komprimierte und teils komplexe Mathematik in einem solchen alten Listing auf engstem Raum untergebracht ist und sich auf den ersten Blick für jemanden, der nicht im Thema ist, jeglicher Ergründung entzieht.

Das ist sicher auch dem Umstand geschuldet, dass die damaligen Entwickler solcher Programme versucht haben, jedes Byte aus den alten 8-Bit-Maschinen herauszuquetschen und an allen Ecken und Enden zu komprimieren, zu vereinfachen und auf Geschwindigkeit zu optimieren. Ganz davon abgesehen, dass in diesen alten BASIC-Dialekten prinzipiell gern nach dem Motto „Kraut und Rüben“ programmiert wurde. Dicht an dicht gepackte, völlig überfüllte Zeilen Programmcode und so ungemein sprechende Variablennamen wie X9 waren damals an der Tagesordnung. Hektisches Hin- und Herspringen zu irgendwelchen Zeilennummern, statt vernünftige Unterprogramme definieren zu können, gehörte ebenfalls zur Normalität.

Alte BASIC-Programme sind also teils wirklich schwer zu lesen.

Dann wurden mathematische Verfahren in wildester Weise in Quellcode umgesetzt, sodass sie am Ende bis zur Unkenntlichkeit komprimiert, modifiziert und sonst wie verhackstückt waren. Moderne KI ist ganz gut darin, solche Algorithmen einem Reverse Engineering zu unterziehen, sodass die ursprüngliche Grundidee eines solchen Verfahrens wieder zum Vorschein kommt.

Dann gibt es noch all die „historisch gewachsenen“ Modifikationen der eingesetzten mathematischen Verfahren: unzählige kleine Änderungen, die im Trial-and-error-Verfahren den Code optimiert und immer weiter in Richtung des gewünschten Ergebnisses gebogen haben. Beim vorliegenden Mandelbrotsee war das an vielen Stellen der Fall.

Und zu guter Letzt haben die damaligen Programmierer ihr ausgeprägtes mathematisches Wissen auch einfach auf eine Weise eingesetzt, die beim Anwender, der solche Listings abgetippt hat, gewisse mathematische (Grund-)Kenntnisse voraussetzte.

All dem versuche ich in diesem Primer gerecht zu werden und gezielt darauf einzugehen.

Für eine stärker von der Mathematik ausgehende Betrachtung habe ich dann noch einen separaten Primer gebaut: „Von der Mandelbrot-artigen Iteration zur pseudo-3D-Fraktallandschaft“. Gewisse Redundanzen zwischen beiden sind dabei natürlich nicht ganz auszuschließen.

---

**Das Rendering eines Bildes geschieht in mehreren Stufen:**

Der Renderer besteht aus drei Schichten:

1. **Fraktalberechnung** – aus `(x, y)` wird eine Höhe.
2. **Kamera/Projektion** – aus einem Punkt der Landschaft wird ein Sichtstrahl.
3. **Stäbchen-Rendering** – aus der berechneten Höhe wird eine vertikale Bildschirmzeile.


# 1. Die kleinste Einheit: `iterate(xc, yc)`

Der Einstiegspunkt ist:

```js
function iterate(xc, yc) {
  let xa = 0;
  let ya = 0;
  let z = 2;

  while (true) {
    const x2 = Math.abs(xa);
    const y2 = Math.abs(ya);

    const xn = x2 - y2 - xc;

    ya = xa * ya;
    ya = ya + ya - yc;

    xa = xn;

    z -= 0.05;

    if (z < 0 || x2 + y2 > 400)
      return { z };
  }
}
```

Die Funktion beantwortet letztlich nur eine Frage:

> **Wie lange bleibt der Punkt `(xc, yc)` innerhalb des Fraktals?**

Aus dieser Aufenthaltsdauer erzeugen wir später eine Landschaftshöhe.

---

# 2. Zwei Zahlen ersetzen eine komplexe Zahl

Ein klassisches Mandelbrot-Fraktal arbeitet mit komplexen Zahlen:

$$
z=x+iy
$$

und einem konstanten Punkt

$$
c=x_c+i y_c.
$$

Im Programm werden die beiden Bestandteile einfach getrennt gespeichert:

```js
let xa = 0;
let ya = 0;
```

entspricht

$$
z_n=x_n+i y_n.
$$

Der untersuchte Punkt ist:

```js
xc
yc
```

also

$$
c=x_c+i y_c.
$$

---

# 3. Die Grundidee der Iteration

Beim klassischen Mandelbrot wird wiederholt

$$
z_{n+1}=z_n^2+c
$$

berechnet.

Für

$$
z=x+iy
$$

gilt:

$$
z^2=(x^2-y^2)+i(2xy).
$$

Daraus entstehen zwei reelle Gleichungen:

$$
x_{n+1}=x_n^2-y_n^2+x_c
$$

$$
y_{n+1}=2x_ny_n+y_c.
$$

Genau deshalb tauchen im JavaScript zwei getrennte Rechnungen für `x` und `y` auf.

---

# 4. Unsere Variante: Beträge vor der Berechnung

Der aktuelle Code beginnt jede Iteration mit:

```js
const x2 = Math.abs(xa);
const y2 = Math.abs(ya);
```

Mathematisch:

$$
x' = |x|
$$

$$
y' = |y|.
$$

Damit werden die Vorzeichen der beiden Koordinaten zunächst entfernt.

Das ist ein wesentlicher Unterschied zum klassischen Mandelbrot und macht die Berechnung **Burning-Ship-artig**.

Danach:

```js
const xn = x2 - y2 - xc;
```

entspricht

$$
x_{n+1}=|x_n|-|y_n|-x_c.
$$

Die zweite Koordinate:

```js
ya = xa * ya;
ya = ya + ya - yc;
```

ist zusammengefasst:

$$
y_{n+1}=2x_ny_n-y_c.
$$

Das zweimalige Addieren ist einfach eine sehr direkte Schreibweise für Multiplikation mit zwei:

```js
ya = xa * ya;
ya = ya + ya;
ya = ya - yc;
```

also:

$$
y\leftarrow2xy-y_c.
$$

---

# 5. Warum `xa` erst ganz am Ende verändert wird

Der Code macht:

```js
const xn = x2 - y2 - xc;

ya = xa * ya;
ya = ya + ya - yc;

xa = xn;
```

Das ist wichtig.

`xn` wird mit dem **alten** `xa` und `ya` berechnet.

Ebenso wird `ya` mit dem alten Zustand berechnet.

Erst danach:

```js
xa = xn;
```

wird der neue X-Wert übernommen.

Mathematisch bedeutet das:

$$
(x_n,y_n)
\longrightarrow
(x_{n+1},y_{n+1}).
$$

Würde `xa` schon vor der Berechnung von `ya` verändert, würde die zweite Gleichung plötzlich mit `x_{n+1}` statt mit `x_n` arbeiten. Das wäre eine andere Iterationsvorschrift.

---

# 6. Die Variable `z` ist keine komplexe Zahl

Ein wichtiger Punkt beim Lesen des Codes:

```js
let z = 2;
```

hat **nichts** mit der komplexen Zahl `z = x + iy` aus der mathematischen Beschreibung zu tun.

Die Variable `z` im JavaScript ist hier lediglich ein **Höhenwert**, der aus der Anzahl der durchlaufenen Iterationen abgeleitet wird.

Bei jeder Iteration wird dieser Wert verringert:

```js
z -= 0.05;
```

Damit stellt sich allerdings sofort die Frage:

> Warum ausgerechnet `0.05`?

Die Antwort ist wichtig: **`0.05` ist keine mathematische Konstante des Mandelbrot-Fraktals.**

Die Fraktalberechnung liefert zunächst nur die Anzahl der Iterationen. Diese Zahl muss anschließend in einen für die Landschaft geeigneten Höhenbereich übersetzt werden.

Der Renderer verwendet dafür eine einfache lineare Skalierung:

$$
H=2-0.05N
$$

Dabei ist `N` die Anzahl der ausgeführten Iterationen.

Die `2` legt den Ausgangswert fest.

Der Faktor `0.05` bestimmt dagegen, **wie stark jede zusätzliche Iteration die Höhe verändert**.

Man kann `0.05` deshalb als **Höhenschritt pro Iteration** verstehen.

---

## Warum gerade `0.05`?

Der Wert `0.05` ist letztlich eine Darstellungsentscheidung.

Man hätte genauso gut beispielsweise

```js
z -= 0.1;
```

verwenden können.

Dann würde jede zusätzliche Iteration die Höhe doppelt so stark verändern:

$$
H=2-0.1N
$$

Oder man könnte

```js
z -= 0.02;
```

verwenden:

$$
H=2-0.02N
$$

Dann würden die Höhenunterschiede deutlich flacher ausfallen.

Der Faktor bestimmt also die **vertikale Skalierung der aus dem Fraktal gewonnenen Landschaft**.

Er verändert nicht die zugrunde liegende Fraktalstruktur.

---

## Was bewirkt `0.05` konkret?

Mit dem im Renderer verwendeten Wert gilt:

$$
H=2-0.05N
$$

Nach 10 Iterationen:

$$
H=2-10\cdot0.05=1.5
$$

Nach 20 Iterationen:

$$
H=2-20\cdot0.05=1
$$

Nach 40 Iterationen:

$$
H=2-40\cdot0.05=0
$$

Nach 50 Iterationen:

$$
H=2-50\cdot0.05=-0.5
$$

Damit liegt bei `N=40` genau die künstlich definierte Meereshöhe `H=0`.

Das ist für die spätere Darstellung besonders praktisch:

* weniger als 40 Iterationen → `H > 0` → Land
* genau 40 Iterationen → `H = 0` → Meereshöhe
* mehr als 40 Iterationen → `H < 0` → Wasser

---

## Der eigentliche Zusammenhang

Die Fraktaliteration liefert also zunächst:

$$
N(x,y)=\text{Anzahl der Iterationen}
$$

Erst danach wird diese Zahl in eine Höhe umgerechnet:

$$
H(x,y)=2-0.05N(x,y)
$$

Das sind zwei unterschiedliche Schritte.

Die Fraktalmathematik bestimmt:

> Wie viele Iterationen benötigt dieser Punkt?

Die Landschaftsdarstellung bestimmt anschließend:

> Welche Höhe soll dieser Iterationszahl entsprechen?

Der Faktor `0.05` gehört ausschließlich zum zweiten Schritt.

---

## Warum ist das wichtig?

Dadurch wird auch verständlich, warum eine Änderung von `0.05` die Form der Landschaft verändern kann, ohne das zugrunde liegende Fraktal selbst zu verändern.

Die Positionen, an denen die Iteration schnell oder langsam entkommt, bleiben gleich.

Nur ihre Interpretation als Höhe ändert sich.

Man könnte es als Maßstab betrachten:

```text
Fraktal
  │
  │ liefert
  ▼
Iterationszahl N
  │
  │ Skalierung mit 0.05
  ▼
Höhe H
  │
  ▼
3D-Landschaft
```

`0.05` ist somit kein Bestandteil der Mandelbrot-artigen Gleichung, sondern ein **Parameter der Höhenabbildung**.

Die Wahl dieses Wertes entscheidet darüber, wie stark die Unterschiede in der Iterationsdauer als Höhenunterschiede der Landschaft sichtbar werden.


---

# 7. Der Fluchtradius

Die zweite Abbruchbedingung lautet:

```js
x2 + y2 > 400
```

Da

$$
x2=|x|
$$

und

$$
y2=|y|,
$$

ist

$$
x2+y2=|x|+|y|.
$$

**Wichtig:** Das ist in diesem konkreten Code nicht exakt dasselbe wie die klassische euklidische Betragsprüfung

$$
x^2+y^2>400.
$$

Der aktuelle Renderer verwendet stattdessen die Summe der Beträge als Escape-Kriterium:

$$
|x|+|y|>400.
$$

und beschreibt im x/y-Koordinatensystem eine Raute (Manhattan-Distanz) und keinen Kreis.

klassische Betragsprüfung:

```text
        y
        ↑
      ╱───╲
    ╱       ╲
   │    ●    │     x² + y² > R²
    ╲       ╱
      ╲───╱
        └────→ x


Burning-Ship-artig / Betrags-Summe:

        y
        ↑
        /\
       /  \
      /    \
     <  ●   >      |x| + |y| > R
      \    /
       \  /
        \/
        └────→ x
  ```      
        
Der Schwellenwert 400 dient als Escape-Grenze für die Summe der Beträge von Real- und Imaginärteil. Überschreitet |x| + |y| diesen Wert, wird die Iteration als divergent betrachtet.

Der Schwellenwert ist damit deutlich größer als der übliche mathematische Escape-Radius von 2 beim klassischen Mandelbrot-Set.

## Warum die Escape-Schwelle 400 wichtig ist

Eine kleinere Schwelle, wie ein klassischer Radius, beispielsweise:

R = 2

würde die Iteration wesentlich früher abbrechen lassen.

Dadurch werden Punkte bereits als Landpunkte klassifiziert, sobald sie die wesentlich kleinere Schwelle überschreiten.

Beispiel – zur Veranschaulichung mit einem klassischen Escape-Radius

Die folgende vereinfachte Darstellung bezieht sich nicht auf die konkrete Escape-Bedingung dieses Programms, sondern dient lediglich dazu, den Einfluss einer sehr großen Escape-Grenze anschaulich zu machen. Bei der klassischen Mandelbrot-Menge wird dafür der Betrag |z| mit einem vorgegebenen Radius verglichen.

Escape-Radius 20:

Z = 2.00
Z = 1.95
Z = 1.90
...
Z = 0.50
Z = 0.45
    ↑
    Punkt entkommt relativ spät

Ein vergleichbarer Punkt könnte bei einem deutlich kleineren Radius von 2 dagegen bereits wesentlich früher entkommen:

Escape-Radius 2:

Z = 2.00
Z = 1.95
Z = 1.90
...
Z = 1.65
    ↑
    Punkt entkommt bereits

Da "Z" als Höhe verwendet wird, erhält der zweite Punkt eine wesentlich größere Höhe.

Das Beispiel soll lediglich verdeutlichen: Je größer die Escape-Grenze gewählt wird, desto länger kann eine Iteration laufen, bevor sie als entkommen betrachtet wird.

Die tatsächliche Escape-Bedingung des hier untersuchten historischen Programms ist davon zu unterscheiden.


## Auswirkung auf die Grenze zwischen Wasser und Land

Das ist besonders wichtig.

Betrachtet man benachbarte Punkte nahe einer Wasser-/Landgrenze, kann der Escape-Radius beeinflussen, wie stark sich die Höhenwerte unterscheiden.

Mit einer großen Escape-Schwelle wie 400 bekommt das System mehr Raum, um Unterschiede in der Fluchtdauer auszunutzen. Die Landschaft kann dadurch vergleichsweise sanft aus dem Wasser heraus ansteigen.

Bei einer deutlich kleineren Escape-Schwelle kann die Klassifizierung früher erfolgen. Dadurch entstehen leichter größere Höhenunterschiede zwischen benachbarten Punkten. Das Gelände wird dadurch tendentiell:

- schroffer
- zerklüfteter
- stärker von einzelnen Spitzen geprägt
- weniger „hügelig“ und stärker „gebirgig“

Die Escape-Schwelle 400 ist deshalb eine grafische Entscheidung.

## Zusammenhang mit dem gewünschten Landschaftseffekt

Das erklärt auch die ursprüngliche Beobachtung beim JavaScript-Prototypen:

«„Alle Nicht-Wasser-Bereiche sind gleich lange vertikale Striche.“»

Wenn die Höhenberechnung zu früh oder zu hart gekappt wird, entstehen zwar Land-/Wasserbereiche, aber noch keine überzeugende Topographie.

Für eine Hügellandschaft braucht man dagegen:

Fraktalposition
      ↓
Iterationsverhalten
      ↓
Fluchtdauer
      ↓
      Z
      ↓
Geländehöhe
      ↓
Perspektivische Projektion
      ↓
Hügellandschaft

Der Escape-Radius ist dabei ein wichtiger Parameter, weil er bestimmt, wie lange die Iteration Unterschiede zwischen den Punkten überhaupt messen kann.

---

# 8. Zwei verschiedene Wege können die Iteration beenden

Die Schleife endet bei:

```js
if (z < 0 || x2 + y2 > 400)
  return { z };
```

Es gibt also zwei Fälle.

### Fall A: `z < 0`

Der Punkt hat lange genug iteriert:

$$
z=2-0.05n<0.
$$

Damit wurde eine maximale Iteration erreicht.

### Fall B: Escape

Der Zustand ist außerhalb des zulässigen Bereichs:

$$
|x|+|y|>400.
$$

Der Punkt gilt als „entkommen“.

In beiden Fällen wird nur `z` zurückgegeben:

```js
return { z };
```

Für den Renderer ist also nicht entscheidend, **warum** die Iteration endete, sondern welcher Höhenwert daraus entstanden ist.

---

# 9. Aus Iterationen wird Landschaft

Darüber liegt:

```js
function minimumHeight(x, y) {
  return iterate(x, y).z;
}
```

Damit wird die Fraktalfunktion mathematisch zu einer Höhenfunktion:

$$
H(x,y)=\operatorname{iterate}(x,y).z.
$$

Das ist der entscheidende Übergang.

Wir betrachten das Fraktal jetzt nicht mehr nur als zweidimensionales Bild.

Stattdessen bekommt jeder Punkt eine dritte Koordinate:

$$
(x,y,H(x,y)).
$$

Damit entsteht ein **Höhenfeld**:

```text
             Höhe H
               ↑
               |
               |       /\      /\
               |   ___/  \____/  \
               |__/               \__
              /
             +--------------------------→ x
            /
           y
```

Das ist die eigentliche Grundlage der Landschaft.

---

# 10. Wasser ist eine Konvention des Renderers

Im aktuellen Code:

```js
const raw = iterate(xc, yc).z;
const isWater = raw < 0;
```

wird definiert:

$$
H(x,y)<0
\quad\Rightarrow\quad
\text{Wasser}.
$$

Das ist keine zusätzliche mathematische Eigenschaft des Fraktals.

Wir haben zuvor lediglich beschlossen, dass negative Werte als Wasser interpretiert werden.

Anschließend:

```js
if (isWater)
  z = -1e-4;
```

wird das Wasser praktisch auf eine gemeinsame Oberfläche gelegt:

$$
H_\text{Wasser}\approx0.
$$

Dadurch entsteht im Rendering ein See bzw. eine Ebene anstelle einer Landschaft, die unterhalb des Wassers weiter nach unten reicht.

---

# 11. Jetzt kommt die Kamera

Die Funktion `render()` erhält:

```js
render(x1, y1, eyeHeight, bearingDegrees)
```

Diese vier Werte bedeuten:

```text
x1             Kameraposition X
y1             Kameraposition Y
eyeHeight      Augenhöhe
bearingDegrees Blickrichtung
```

Die Kamera befindet sich also bei

$$
P_0=(x_1,y_1).
$$

Ihre Höhe ist

$$
E=\text{eyeHeight}.
$$

---

# 12. Blickrichtung in Radianten

JavaScript arbeitet bei `sin()` und `cos()` mit Radianten:

```js
const angle = bearingDegrees * Math.PI / 180;
```

Mathematisch:

$$
\theta=
\text{bearingDegrees}\cdot\frac{\pi}{180}.
$$

Danach:

```js
const si = Math.sin(angle);
const co = Math.cos(angle);
```

also

$$
s=\sin(\theta)
$$

und

$$
c=\cos(\theta).
$$

---

# 13. Ein wichtiger Trick: Vorwärts- und Seitenrichtung

Der Renderer verwendet:

```js
const si = Math.sin(angle);
const co = Math.cos(angle);
```

und daraus:

```js
xc = x1 + hc * co + e * si;
yc = y1 - hc * si + e * co;
```

Das lässt sich wesentlich einfacher als Vektoren verstehen.

Die Blickrichtung (Forward) ist:

$$
F=(\sin\theta,\cos\theta).
$$

Der zur Blickrichtung senkrechte Seitenvektor (Side) ist:

$$
S=(\cos\theta,-\sin\theta).
$$

Seine konkrete Orientierung ergibt sich aus unserer Koordinatenkonvention. In unserem System gilt bei 90°: F = (1,0) (rechts) und S = (0,-1) (oben).


**Beispiele:**

Für unser mathematisches Weltkoordinatensystem gilt:

```text
       -Y
        ↑
        |
-X ←--- + ---→ +X
        |
        ↓
       +Y
```

Denn unser Renderer definiert:

```text
0°   → +Y
90°  → +X
180° → -Y
270° → -X
```

Damit ist bei 90°:

```text
F = (1, 0)       → nach rechts / +X
S = (0, -1)      → nach oben / -Y
```

F sagt: In welche Richtung schaut die Kamera?

Bei 0°:
```js
sin(0°) = 0
cos(0°) = 1

F = (0, 1)
```

Die Kamera schaut also in positive Y-Richtung.

Bei 90°:
```js
F = (1, 0)
```

Sie schaut nach rechts, also in positive X-Richtung.

S zeigt quer zur Blickrichtung.

Bei 0°:
```js
S = (1, 0)
```

also nach rechts.

Bei 90°:
```js
S = (0, -1)
```

also nach oben bzw. negative Y-Richtung.

Ein Punkt vor der Kamera kann deshalb geschrieben werden als:

$$
P=P_0+dF+hS.
$$

und bedeutet einfach:
Starte bei der Kamera P₀, gehe d Einheiten nach vorne und anschließend h Einheiten zur Seite.

Dabei ist:

* \(d\) = Abstand nach vorne
* \(h\) = Abstand zur Seite.

Das ist normale zweidimensionale Vektorrechnung.

Den Code:

```js
const hc = e * ho * 0.0187;

const xc = x1 + hc * co + e * si;
const yc = y1 - hc * si + e * co;
```

kann man nämlich umsortiert schreiben als:

```js
P₀ = (x1, y1)

F = (si, co)
S = (co, -si)

d = e
h = hc
```

und damit:

```js
P = P₀ + eF + hcS
```

Das ist exakt derselbe Code, nur als Vektorgleichung geschrieben.

---

# 14. Perspektive: `e`

Im Code:

```js
const e = eyeHeight / v * 10;
```

Mathematisch:

$$
e = \frac{E}{v} \cdot 10 = \frac{10E}{v}
$$

Dabei ist `v` der aktuelle Renderdurchlauf und beschreibt dabei die Position entlang der Blickrichtung – von der Kamera hinaus in Richtung Landschaft. Es ist jedoch keine reale Entfernung, sondern ein mathematischer Perspektivparameter des jeweiligen Renderdurchlaufs.

```text
                 Kamera
                    ●
                   /|\      v: 1, ←---- e ----→
                  / | \
                 /  |  \
                /   |   \
               /    |    \
              /     |     \     v: 300, ←-- e --→
             /      |      \
            /       |       \
           /        |        \
          /         |         \     v: 600, ← e →
         ─────────────────────── Landschaft
```

Das bedeutet:

$$
v\uparrow
\quad\Rightarrow\quad
e\downarrow.
$$

Je weiter v entlang der Blickrichtung anwächst, desto kleiner wird e.

Die Kamera tastet also die Landschaft mit einer perspektivischen Skalierung ab. Das ist genau der perspektivische Effekt: nahe Bereiche werden mit größeren Schritten abgetastet, entfernte Bereiche mit kleineren.

---

# 15. Warum `e` gleichzeitig Entfernung und Maßstab ist

Im aktuellen historischen Ansatz ist `e` nicht sauber in

* Entfernung
* Brennweite
* Projektionsmaßstab

aufgeteilt.

Stattdessen steckt alles in dieser einfachen Größe:

$$
e=\frac{10E}{v}.
$$

Das ist typisch für einen sehr einfachen Software-Renderer.

Für die Rekonstruktion ist das sogar hilfreich:

> Wir müssen keine moderne 3D-Kamera simulieren. Wir müssen genau diese Projektionsregel reproduzieren.

---

# 16. Der horizontale Bildschirmkoordinat

Der Code benutzt:

```js
const hc = e * ho * 0.0187;
```

Dabei ist `ho` die horizontale Position relativ zur Bildschirmmitte.

Mathematisch:

$$
h=e\cdot ho\cdot0.0187.
$$

Der Faktor

$$
0.0187
$$

ist ein **Kamera-/Projektionsparameter**.

Er gehört nicht zum Fraktal.

Er bestimmt, wie weit die Sichtstrahlen horizontal auseinanderliegen.

---

# 17. Was `0.0187` geometrisch bewirkt

Für `ho = 80` gilt:

$$
80\cdot0.0187=1.496.
$$

Der seitliche Abstand beträgt also:

$$
h=1.496e.
$$

Wenn man das als einfache Kamera-Geometrie interpretiert, ergibt sich ungefähr:

$$
\alpha=\arctan(1.496)
\approx56.2^\circ.
$$

Damit ergibt sich ungefähr:

$$
FOV\approx112.4^\circ.
$$

Der Faktor `0.0187` bestimmt also maßgeblich das **horizontale Sichtfeld**.

---

# 18. Warum `ho = -160 ... +159` nicht einfach die richtige Lösung ist

Ursprünglich:

```js
for (let ho = -80; ho <= 80; ho++)
```

liefert ungefähr 161 horizontale Abtastpunkte.

Bei einem 320 Pixel breiten Bildschirm ist das zu wenig.

Eine naheliegende Änderung wäre:

```js
for (let ho = -160; ho < 160; ho++)
```

Aber dadurch wird gleichzeitig das Sichtfeld verdoppelt.

Denn:

$$
160\cdot0.0187=2.992.
$$

Damit wird der halbe Öffnungswinkel ungefähr:

$$
\arctan(2.992)\approx71.5^\circ.
$$

Das gesamte Sichtfeld wäre dann etwa:

$$
143^\circ.
$$

Das Bild würde also nicht einfach detaillierter, sondern die Kamera würde deutlich weiter nach links und rechts sehen.

---

# 19. Die richtige Trennung: Bildschirmauflösung ≠ Sichtfeld

Wenn die historische Kamera ein Sichtfeld von `ho=-80...+80` besitzt, sollte dieses Sichtfeld erhalten bleiben.

Wir können es trotzdem auf 320 Pixel abbilden.

Dazu:

```js
for (let screenX = 0; screenX < WIDTH; screenX++) {
  const ho =
    (screenX - (WIDTH - 1) / 2) *
    80 / ((WIDTH - 1) / 2);
```

Damit gilt ungefähr:

```text
screenX = 0     → ho = -80
screenX = 159.5 → ho = 0
screenX = 319   → ho = +80
```

Die Kamera sieht also weiterhin denselben Ausschnitt.

Nur wird dieser Ausschnitt feiner abgetastet.

---

# 20. Vom Kamerapunkt zum Landschaftspunkt

Jetzt kommt alles zusammen.

Der Code:

```js
const hc = e * ho * 0.0187;

const xc =
  x1 + hc * co + e * si;

const yc =
  y1 - hc * si + e * co;
```

entspricht mathematisch:

$$
h=e\cdot ho\cdot0.0187
$$

und

$$
P=P_0+hS+eF.
$$

Ausgeschrieben:

$$
x_c=x_1+h\cos\theta+e\sin\theta
$$

$$
y_c=y_1-h\sin\theta+e\cos\theta.
$$

Dieser Punkt ist der eigentliche Übergang zwischen Kamera und Fraktal.

---

# 21. Genau hier treffen sich die beiden Welten

Bis hierhin war die Kamera-Geometrie unabhängig vom Fraktal.

Jetzt:

```js
const raw = iterate(xc, yc).z;
```

Die Kamera sagt:

> „Ich möchte wissen, wie hoch die Landschaft an diesem Punkt ist.“

Die Fraktalfunktion antwortet:

$$
H(x_c,y_c).
$$

Damit ist die komplette Verbindung hergestellt:

$$
\boxed{
\text{Kamera}
\rightarrow
(x_c,y_c)
\rightarrow
\text{Fraktal}
\rightarrow
H
}
$$

Das Fraktal weiß nichts von der Kamera.

Die Kamera weiß zunächst nichts vom Fraktal.

Erst der Render-Schritt verbindet beide.

---

# 22. Die Höhenmischung

Der Code enthält:

```js
let z = previousZ * 0.7 + raw * 0.3;
```

Mathematisch:

$$
H_\text{neu}
=
0.7H_\text{alt}
+
0.3H_\text{aktuell}.
$$

Das ist eine einfache Glättung.

Der neue Wert besteht aus:

* 70 % vorherigem Wert
* 30 % aktuellem Fraktalwert.

Dadurch reagieren benachbarte Stäbchen weniger stark auf kleine Sprünge.

Der Effekt ist eine Art **Low-Pass-Filter entlang der Render-Reihenfolge**.

---

# 23. Jetzt wird die Höhe auf den Bildschirm projiziert

Der Code:

```js
let c = (z - eyeHeight) * v / eyeHeight;
```

entspricht:

$$
c=
\frac{(H-E)v}{E}.
$$

Dabei ist:

* \(H\) = Landschaftshöhe
* \(E\) = Augenhöhe
* \(v\) = perspektivischer Skalierungsfaktor
* \(c\) = vertikale Bildschirmposition.

Wichtig ist vor allem die Differenz:

$$
H-E.
$$

Die Kamera interessiert sich nicht für die absolute Höhe allein.

Sie interessiert sich dafür:

> **Wie hoch liegt die Landschaft relativ zu meinen Augen?**

---

# 24. Warum die Augenhöhe entscheidend ist

Angenommen:

$$
E=1
$$

und:

$$
H=2.
$$

Dann:

$$
H-E=1.
$$

Der Punkt liegt oberhalb der Augenhöhe.

Bei:

$$
H=0
$$

gilt:

$$
H-E=-1.
$$

Der Punkt liegt unterhalb der Augenhöhe.

Damit kann dieselbe Landschaft bei unterschiedlicher Kamerahöhe völlig unterschiedlich aussehen.

---

# 25. Der Bildschirmmittelpunkt

Im Renderer wird ungefähr um die Bildschirmmitte herum gezeichnet:

```js
verticalLine(screenX, 98 - c, 199, COLOR.red);
```

Der Wert `98` fungiert dabei als ungefährer Horizont-/Bildschirmmittelpunkt.

Damit gilt vereinfacht:

$$
y_\text{screen}=98-c.
$$

Ein größeres `c` bedeutet also:

$$
y_\text{screen}\downarrow.
$$

Der Punkt wandert auf dem Bildschirm nach oben.

Genau das wollen wir für einen höheren Berg.

---

# 26. Warum Berge nach oben wandern

Für einen hohen Punkt:

$$
H>E.
$$

Damit:

$$
H-E>0
$$

und deshalb:

$$
c>0.
$$

Also:

$$
y_\text{screen}=98-c<98.
$$

Der Punkt wird oberhalb des Horizonts gezeichnet.

Für einen tiefen Punkt:

$$
H<E.
$$

Dann:

$$
c<0
$$

und:

$$
y_\text{screen}>98.
$$

Der Punkt liegt weiter unten.

Die Landschaft entsteht somit allein aus einer einfachen Höhenprojektion.

---

# 27. Die Stäbchen-Technik

Statt Polygone zu zeichnen, verwendet der Renderer vertikale Linien.

Vereinfacht:

```js
verticalLine(
  screenX,
  98 - c,
  199,
  COLOR.red
);
```

Das bedeutet:

> Zeichne an der horizontalen Position `screenX` eine Linie von der Landschaft bis zum unteren Bildschirmrand.

Jede Bildschirmspalte wird damit zu einem „Stäbchen“.

```text
        /\                 /\
       /  \      /\       /  \
      /    \____/  \_____/    \
─────┴──────────────────────────
████████████████████████████████
```

Viele solcher Stäbchen nebeneinander erzeugen die scheinbare Landschaft.

---

# 28. Warum die Reihenfolge wichtig ist

Die Schleife läuft:

```js
for (let v = 1; v <= 600; v += 2)
```

Die einzelnen Landschaftsstreifen werden in einer bestimmten Reihenfolge abgearbeitet.

Dabei wird gespeichert:

```js
let previousZ = 0;
let previousC = 0;
```

Der Renderer weiß dadurch, wo die vorherige Oberfläche lag.

Das ermöglicht eine sehr primitive Form von **Verdeckung**.

---

# 29. Relief-Optik durch schwarze Stäbchen

Der Code:

```js
if (c > previousC + 0.2)
  verticalLine(
    screenX,
    99 - c,
    99 - previousC,
    COLOR.black
  );
```

Die Linie liegt genau in dem Bereich zwischen der Oberkante des vorherigen Stabes und der Oberkante des aktuellen Stabes. Wenn der aktuelle Stab höher herausragt, wird dieser Bereich dunkel dargestellt. Dadurch entsteht eine Art Schattierungs-/Reliefkante.

Mathematisch wird geprüft:

$$
c_\text{aktuell}>c_\text{vorher}+0.2.
$$

Der Wert `0.2` verhindert, dass winzige numerische Unterschiede sofort als neue Verdeckung behandelt werden.

---

# 30. Das gesamte Verfahren als Datenfluss

Jetzt kann der komplette Renderer als Kette geschrieben werden:

$$
\boxed{
\text{Pixelposition}
}
$$

↓

$$
\boxed{
\text{Kamerakoordinate}
}
$$

↓

$$
\boxed{
(x_c,y_c)
}
$$

↓

$$
\boxed{
H(x_c,y_c)
}
$$

↓

$$
\boxed{
\text{Höhenmischung}
}
$$

↓

$$
\boxed{
\frac{(H-E)v}{E}
}
$$

↓

$$
\boxed{
\text{Bildschirm-Y}
}
$$

↓

$$
\boxed{
\text{vertikales Stäbchen}
}
$$

Das ist der gesamte Trick.

---

# 31. Die drei mathematischen Ebenen des Programms

Man kann den Quellcode deshalb in drei vollkommen unterschiedliche mathematische Bereiche zerlegen.

## Ebene 1 – Fraktal

```js
iterate(xc, yc)
```

berechnet:

$$
(x_c,y_c)
\rightarrow H.
$$

Hier steckt die Fraktalmathematik.

---

## Ebene 2 – Kamera

```js
const e = eyeHeight / v * 10;

const hc = e * ho * 0.0187;

const xc = ...
const yc = ...
```

berechnet:

$$
(\text{Kamera},\text{Blickrichtung},\text{Pixel})
\rightarrow(x_c,y_c).
$$

Hier steckt die Projektionsgeometrie.

---

## Ebene 3 – Rendering

```js
let c = (z - eyeHeight) * v / eyeHeight;
```

und

```js
verticalLine(...)
```

machen daraus:

$$
H
\rightarrow
\text{Bildschirmposition}.
$$

Hier steckt die eigentliche Darstellung.

---

# 32. Ein konkretes Beispiel

Nehmen wir vereinfacht:

```text
Kamera:
x1 = 0
y1 = 0
eyeHeight = 1
bearing = 0°
```

Bei `0°` schaut die Kamera in positive Y-Richtung.

Damit:

$$
\sin(0)=0
$$

und

$$
\cos(0)=1.
$$

Also:

$$
F=(0,1)
$$

und

$$
S=(1,0).
$$

Die Kamera schaut gerade nach unten auf der Y-Achse.

---

## Ein Punkt in der Bildmitte

Für:

$$
ho=0
$$

gilt:

$$
h=0.
$$

Der Punkt liegt also genau vor der Kamera.

Wenn beispielsweise:

$$
e=2,
$$

dann:

$$
x_c=0
$$

und

$$
y_c=2.
$$

Der Fraktalwert wird berechnet:

```js
raw = iterate(0, 2).z;
```

Angenommen:

$$
H=1.5.
$$

Dann:

$$
c=\frac{(1.5-1)\cdot v}{1}.
$$

Für beispielsweise:

$$
v=10
$$

ergibt sich:

$$
c=5.
$$

Die Bildschirmposition ist:

$$
y=98-5=93.
$$

Das Landschaftsstück erscheint also fünf Pixel oberhalb des Horizontwertes.

---

# 33. Was sich ändert, wenn die Kamera dreht

Nehmen wir dieselbe Kamera, aber:

$$
\theta=90^\circ.
$$

Dann:

$$
\sin(90^\circ)=1
$$

und

$$
\cos(90^\circ)=0.
$$

Die Blickrichtung wird:

$$
F=(1,0).
$$

Die Kamera schaut jetzt in positive X-Richtung.

Die Seitenrichtung wird:

$$
S=(0,-1).
$$

Alle vom Renderer erzeugten Punkte werden dadurch entsprechend gedreht.

Das Fraktal selbst wird **nicht verändert**.

Nur die Punkte, die aus ihm abgefragt werden, ändern sich.

---

# 34. Genau deshalb ist die Navigation so einfach

Die Karte zeigt die Kamera im selben Koordinatensystem wie das Fraktal.

Wenn die Position verändert wird:

```js
x1
y1
```

ändert sich nur:

$$
P_0.
$$

Wenn die Blickrichtung verändert wird:

```js
bearingDegrees
```

ändert sich nur:

$$
F
$$

und

$$
S.
$$

Wenn die Höhe verändert wird:

```js
eyeHeight
```

ändert sich die Projektion.

Die Fraktalfunktion bleibt unverändert.

Das ist eine saubere Trennung der Zuständigkeiten.

---

# 35. Das gesamte aktuelle Modell in einer Formel

Der Renderer berechnet für jeden Bildschirmwert `screenX` und jeden Renderdurchlauf `v` zunächst:

$$
e=\frac{10E}{v}
$$

$$
h=e\cdot ho\cdot0.0187
$$

und daraus:

$$
x_c=x_1+h\cos\theta+e\sin\theta
$$

$$
y_c=y_1-h\sin\theta+e\cos\theta.
$$

Dann:

$$
H=\operatorname{iterate}(x_c,y_c).z.
$$

Anschließend gegebenenfalls:

$$
H\leftarrow0.7H_\text{vorher}+0.3H.
$$

Dann:

$$
c=\frac{(H-E)v}{E}.
$$

Und schließlich:

$$
y_\text{screen}=98-c.
$$

Diese eine Kette beschreibt praktisch den gesamten Landschaftsrenderer.

---

# 36. Was das Programm ausdrücklich NICHT macht

Obwohl das Ergebnis wie eine 3D-Landschaft aussieht, berechnet der Renderer kein normales 3D-Modell.

Es gibt:

* keine Dreiecke,
* keine Polygone,
* keine Vertex-Buffer,
* keine echte 3D-Matrix,
* keinen Z-Buffer,
* keine Perspektivmatrix,
* keine Oberflächenbeleuchtung.

Stattdessen gibt es nur:

1. eine zweidimensionale Fraktalfunktion,
2. eine künstliche Höhenfunktion,
3. eine sehr einfache Kameraformel,
4. vertikale Linien.

Das verblüffende Ergebnis entsteht durch das Zusammenspiel dieser einfachen Komponenten.

---

# 37. Warum das Verfahren für einen C64 so interessant war

Die Methode vermeidet praktisch alles, was ein damaliger Rechner nur schwer leisten konnte.

Ein moderner 3D-Renderer würde eine Landschaft als Geometrie erzeugen.

Hier genügt:

```text
Fraktalpunkt berechnen
        ↓
Höhe bestimmen
        ↓
einen vertikalen Strich zeichnen
```

Die „Geometrie“ entsteht erst während des Zeichnens.

Das passt hervorragend zu einer sehr einfachen Rastergrafik und erklärt auch die historische „Stäbchen-Technik“.

---

# 38. Die wichtigste mentale Vorstellung

Beim Lesen des Programms sollte man sich nicht vorstellen:

> „Das Programm baut zuerst ein 3D-Gebirge.“

Es macht etwas viel Einfacheres:

> **Es fragt das Fraktal aus der Perspektive einer Kamera immer wieder nach Höhenwerten und zeichnet diese Höhenwerte als vertikale Striche.**

Oder noch kürzer:

$$
\boxed{
\text{Fraktal}
\rightarrow
\text{Höhenfeld}
\rightarrow
\text{Kamera}
\rightarrow
\text{Stäbchen}
}
$$

Das ist die gesamte Idee hinter dem Renderer.

---

# 39. Die wichtigste Trennung beim Weiterentwickeln

Für spätere Änderungen sollte zwischen drei Arten von Parametern unterschieden werden.

### Fraktalparameter

Sie verändern die Landschaft selbst:

```js
Math.abs(xa)
Math.abs(ya)
xc
yc
0.05
400
```

---

### Kameraparameter

Sie verändern den Blick auf dieselbe Landschaft:

```js
x1
y1
eyeHeight
bearingDegrees
0.0187
```

---

### Darstellungsparameter

Sie verändern nur die Darstellung:

```js
0.7
0.3
98
199
0.2
COLOR.red
COLOR.green
COLOR.blue
```

Diese Unterscheidung ist besonders wichtig.

Wenn beispielsweise das Landschaftsprofil stimmt, aber das Bild zu weitwinklig aussieht, sollte nicht die Fraktalberechnung verändert werden.

Dann ist der relevante Parameter sehr wahrscheinlich:

```js
0.0187
```

Wenn dagegen die Berge an den falschen Stellen stehen, liegt das Problem eher bei:

```js
iterate(xc, yc)
```

Wenn die Berge an der richtigen Stelle liegen, aber zu hoch oder zu tief erscheinen, ist eher die Projektion betroffen:

```js
let c = (z - eyeHeight) * v / eyeHeight;
```

Damit wird aus dem Quellcode ein relativ gut zerlegbares mathematisches System.
