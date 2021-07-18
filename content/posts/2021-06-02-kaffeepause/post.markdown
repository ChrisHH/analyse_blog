---
title: Kaffeepause
author: Christian Jähnert
date: '2021-07-17'
slug: []
categories:
  - R
  - Tidyverse
  - Tidytuesday
  - Kaffee
tags: []
draft: no
---

Er ist wortwörtlich in aller Munde: Kaffee. Ob als Filterkaffee, Espresso, Cafe Creme oder Mokka. Zum Frühstück, nach dem Mittag oder am Nachmittag. Ob heiß gebrüht oder auf Eiswürfeln serviert. Mal kommt er schwarz daher, mal weiß, mal mit wenig, mal mit viel Zucker. Gelegentlich ist eine ordentliche Portion Milchschaum oben drauf. Wann und wie auch immer Kaffee genossen wird: Das wichtigste ist den Genießer*innen doch, dass der Kaffee gut schmeckt.

<img src="/posts/2021-06-02-kaffeepause/post_files/nathan-dumlao-6VhPY27jdps-unsplash.jpg" alt="source: unsplash.com / Nathan Dumlao" width="400px"/>

Das [Coffee Quality Institute](https://www.coffeeinstitute.org) hat im Januar 2018 einen Datensatz veröffentlicht, der die Ergebnisse von Kaffee-Degustationen von April 2014 bis September 2016 enthält. Dieser Datensatz ist Teil des Projektes [Tidytuesday](https://github.com/rfordatascience/tidytuesday) und wird heute verwendet, um gutem Kaffee auf die Spur zu kommen.

# Beurteilung des Kaffees

Der Kaffee wird nach einem detailierten Protokoll auf insgesamt zehn Dimensionen beurteilt:

* Aroma (aroma)
* Geschmack (flavor)
* Nachgeschmack (aftertaste)
* Säure (acidity)
* Körper (body)
* Ausgewogenheit (balance)
* Einheitlichkeit (uniformity)
* Keine Geschmacksfehler (clean_cup)
* Süße (sweetness)
* Gesamteindruck (cupper_points)

Das Rating funktioniert pro Dimension in der Regel auf einer Skala von 6 (gut) bis 10 (außerordentlich gut) mit jeweils Unterstufen (mehr Informationen vermittelt [dieser Leitfaden](https://www.scaa.org/PDF/resources/cupping-protocols.pdf)). Eigentlich beginnt die Skala bei 0, jedoch wird alles unter 6 als nicht "specialty grade" beurteilt.

Diese Werte werden aufsummiert und es ergeben sich die "Total Cup Points", also ein Gesamtscore. Dieser rangiert entsprechend im Bereich von 0-100; je höher dieser Wert ist, desto besser ist der Kaffee insgesamt.





# Sorten und Anbauländer

Der Datensatz enthält die Ergebnisse von 1311 Degustationen der Sorte "Arabica" und von 28 Degustationen der Sorte "Robusta".
Im folgenden wird nur die Sorte "Arabica" weiter untersucht.

Zunächst erfolgt ein Blick darauf, welche die häufigsten Anbauländer des degustierten Kaffees sind.

<table>
 <thead>
  <tr>
   <th style="text-align:left;"> Anbauland </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> Other </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Mexico </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Colombia </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Guatemala </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Brazil </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Taiwan </td>
  </tr>
  <tr>
   <td style="text-align:left;"> United States (Hawaii) </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Honduras </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Costa Rica </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Ethiopia </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Tanzania, United Republic Of </td>
  </tr>
</tbody>
</table>

Die Kaffees, die am häufigsten verkostet wurden, stammen aus Mexico (236 Proben) am unteren Ende befindet sich mit 40 Proben Kaffee aus Tansania. Weniger häufige Anbauländer wurden in der Position "Other" zusammengefasst. 

Während ich die meisten Länder auch mit Kaffeeanbau in Verbindung bringe, wusste ich noch nicht, dass in Taiwan und auf Hawaii auch Kaffee angebaut wird. 

# Kaffeebewertung nach Anbauland

Nun wird betrachtet, wie sich die Total Cup Points der Degustationen im Durchschnitt über die Anbauländer unterscheiden. Dazu wurden die Total Cup Points pro Land über alle Degustationen gemittelt.

<img src="/posts/2021-06-02-kaffeepause/post_files/figure-html/unnamed-chunk-4-1.png" width="2400" />

Das Land mit den höchsten durchschnittlichen Total Cup Points ist Äthiopien; das Schlusslicht bildet Kaffee aus Honduras. 

Das führt nun zu der Frage, woher es kommt, dass die Total Cup Points variieren. Zu diesem Zweck erfolgt die Aufsplittung der Total Cup Points in die zehn Unterdimensionen.

<img src="/posts/2021-06-02-kaffeepause/post_files/figure-html/unnamed-chunk-5-1.png" width="2400" />

Während die Dimensionen Einheitlichkeit (uniformity), Süße (sweetness) und Geschmack (flavor) über die Proben aller Länder mehr oder weniger gleich gut abschneiden, öffnet sich das Feld beispielsweise bei den Cupper Points. 

# Höhenlage und Kaffeebewertung

Eine weitere Variable, die in dem Datensatz vorhanden ist, ist die Höhenlage der Plantage des degustierten Kaffees. Die Frage ist: wie wirkt sich diese auf die Total Cup Points aus? Gibt es einen Zusammenhang? Die Antwort liefert die folgende Grafik, in der die Total Cup Points in Abhängigkeit von der Höhenlage dargestellt sind.

<img src="/posts/2021-06-02-kaffeepause/post_files/figure-html/unnamed-chunk-6-1.png" width="2400" />

Die Regressionsgerade in der Grafik deutet an, dass ein höher gelegenes Anbaugebiet zu höheren Total Cup Points führt. Wahrscheinlich wirkt sich die Höhenlage auf einen oder mehrere der o.g. Unterdimensionen aus, wodurch die Total Cup Points entsprechend variieren.

Um einzukreisen, welche Dimensionen dafür verantwortlich sind, werden nun noch einmal die einzelnen Unterdimensionen in Abhängigkeit von der Höhenlage betrachtet. 

<img src="/posts/2021-06-02-kaffeepause/post_files/figure-html/unnamed-chunk-7-1.png" width="2400" />

Während einige Dimensionen, wie z.B. Süße oder Einheitlichkeit offensichtlich unbeeinflusst sind von der Höhenlage der Plantage, sieht es bei anderen Dimensionen schon etwas anders aus (z.B. Säure: je höher die Plantage liegt, desto mehr steigt die Beurteilung auf dieser Dimension). Hier hat die Höhenlage einen erkennbaren Einfluss.

# Zwischenfazit

Die bisherige Analyse führt zu zwei Erkenntnissen:

1. Die Bewertung (Total Cup Points) des Kaffees unterscheidet sich je nach Anbauland. 
2. Die Höhenlage hat Einfluss auf die Bewertung des Kaffees.

Beides sind Informationen, die in der Regel auf der Kaffeepackung zu finden sind, wie das Beispiel auf dem folgenden Bild zeigt.

<img src="/posts/2021-06-02-kaffeepause/post_files/Folie1.png" alt="" width="700"/>

Somit werde ich jetzt einen Entscheidungsbaum berechnen, der beide Merkmale kombiniert und geeignet ist, die Total Cup Points zu prognostizieren - für eine kleine Hilfe beim nächsten Einkauf, um guten Kaffee zu kaufen.

# Entscheidungsbaum für guten Kaffee

Nachdem 1.600 Entscheidungsbäume berechnet wurden, ist dies die beste Lösung.

<img src="/posts/2021-06-02-kaffeepause/post_files/figure-html/unnamed-chunk-8-1.png" width="2400" />

Der Baum unterscheidet an oberster Stelle anhand des Anbaulandes; im weiteren nach der Höhenlage des Anbaugebiets.

Nehme ich einmal, den von mir gekauften Kaffee (Herkunftsland: Mexiko, Höhenlage: "in Lagen über 1.000 Metern"), dann kann ich nun den Pfad wie folgt durchschreiten: das Herkunftsland ist Mexiko, also geht es im Baum von der Wurzel den ersten Ast nach links ("Yes"). Am nächsten Knoten geht es wieder nach links, dann nach rechts, dann links, und nochmal links. Somit erzielt der Kaffee ein geschätztes Rating von 80. Das Modell schätzt im Mittel 3 Punkte ungenau. Also liegen die Total Cup Points irgendwo zwischen 77 und 83; und damit schon eher am unteren Rand.

Beim nächsten Einkauf weiß ich also, worauf ich achten kann, um in den Genuss von gutem Kaffee zu kommen!
