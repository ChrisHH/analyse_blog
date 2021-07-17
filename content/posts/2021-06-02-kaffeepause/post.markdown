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

<img src="/posts/2021-06-02-kaffeepause/post_files/janko-ferlic-h9Iq22JJlGk-unsplash.jpg" alt="Bildquelle: www.unsplash.com" width="400px"/>

Das Coffee Quality Institute (https://www.coffeeinstitute.org) hat im Januar 2018 einen Datensatz veröffentlicht, der die Ergebnisse von Kaffee-Degustationen von April 2014 bis September 2016 enthält. Dieser Datensatz ist Teil des Projektes Tidytuesday (https://github.com/rfordatascience/tidytuesday) und wird heute verwendet, um den verschiedenen Facetten von Kaffee auf die Spur zu kommen.

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

Das Rating funktioniert pro Dimension in der Regel auf einer Skala von 6 (gut) bis 10 (außerordentlich gut) mit jeweils Unterstufen (sh. auch hier: https://www.scaa.org/PDF/resources/cupping-protocols.pdf). Eigentlich beginnt die Skala bei 0, jedoch wird alles unter 6 als nicht "specialty grade" beurteilt. 

Diese Werte werden aufsummiert und es ergeben sich die "Total Cup Points", also eine Art Gesamtscore. Dieser rangiert entsprechend im Bereich von 0-100.





# Betrachtete Sorten und Anbauländer

Der Datensatz enthält 1311 Proben der Sorte "Arabica" und nur 28 Proben der Sorte "Robusta".
Im folgenden werde nur die Sorte "Arabica" weiter untersuchen.

Nachfolgend sind die Anbauländer der Kaffees aufgezählt, die am häufigsten degustiert wurden.

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

Die Kaffees, die am häufigsten getestet wurden, stammen aus Mexico (236 Proben) am unteren Ende befindet sich mit 40 Proben Kaffee aus Tansania.
Weniger häufige Anbauländer wurden in der Position "Other" zusammengefasst. 

Während ich die meisten Länder auch mit Kaffeeanbau in Verbindung bringe, wusste ich noch nicht, dass in Taiwan und auf Hawaii auch Kaffee angebaut wird. 

Und wie unterschiedlich ist nun die Bewertung der Kaffees?

# Kaffeebewertung nach Anbauland

Zunächst wird betrachtet, wie sich die Total Cup Points der Degustationen im Durchschnitt über die Anbauländer darstellen.

<img src="/posts/2021-06-02-kaffeepause/post_files/figure-html/unnamed-chunk-4-1.png" width="2400" />
Das Land mit den höchsten Total Cup Points ist Äthiopien; das Schlusslicht bildet Kaffee aus Honduras. 

Das führt nun zu der Frage, woher es kommt, dass die Total Cup Points variieren. Zu diesem Zweck erfolgt nun die Aufsplittung der Total Cup Points in die zehn Unterdimensionen.

<img src="/posts/2021-06-02-kaffeepause/post_files/figure-html/unnamed-chunk-5-1.png" width="2400" />
Während die Dimensionen Einheitlichkeit (uniformity), Süße (sweetness) und Geschmack (flavor) über die Proben aller Länder mehr oder weniger gleich gut abschneiden, öffnet sich das Feld bei den Cupper Points.


Jetzt gehe ich kurz der Frage nach, ob es einen Zusammenhang zwischen der Höhenlage der Plantage und den Total Cup Points gibt.

# Höhenlage und Kaffeebewertung

<img src="/posts/2021-06-02-kaffeepause/post_files/figure-html/unnamed-chunk-6-1.png" width="2400" />
Ich musste ein paar Werte herausfiltern, die offensichtlich falsch waren; das trifft vor allem auf die Höhenlage zu. Hier gab es ein paar wenige Extremwerte im Bereich von 3.000 - 100.000 Metern, was unrealistisch ist. Danach stellt sich dieser Scatterplot dar. Durch diesen habe ich eine einfache Regressionsgerade gelegt; ein perfekter fit ist das  nicht, aber es deutet sich an, dass ein höher gelegenes Anbaugebiet zu höheren Total Cup Points führt. Vermutlich wirkt sich die Höhenlage auf einen oder mehrere der o.g. Dimensionen aus, wodurch die Total Cup Points entsprechend steigen.

Um einzukreisen, welche Dimensionen dafür verantwortlich sind, blicke ich noch einmal auf den Zusammenhang zwischen den einzelnen Dimensionen und der Höhenlage. 

<img src="/posts/2021-06-02-kaffeepause/post_files/figure-html/unnamed-chunk-7-1.png" width="2400" />
Während einige Dimensionen, wie z.B. Süße oder Einheitlichkeit offensichtlich unbeeinflusst sind von der Höhenlage der Plantage, sieht es bei anderen Dimensionen schon etwas anders aus (z.B. Säure: je höher die Plantage liegt, desto mehr steigt die Beurteilung auf dieser Dimension).

# Zwischenfazit

Die bisherige Analyse führt zu zwei Erkenntnissen:

1. Die Bewertung des Kaffees unterscheidet sich je nach Anbauland. 
2. Die Höhenlage hat ebenfalls Einfluss auf die Bewertung des Kaffees. 

Beides sind Informationen, die in der Regel auf der Kaffeepackung zu finden sind, wie das Beispiel auf dem folgenden Bild zeigt.

<img src="/posts/2021-06-02-kaffeepause/post_files/Folie1.png" alt="" width="700"/>

Somit werde ich jetzt ein Modell berechnen (Entscheidungsbaum), dass beide Merkmale kombiniert und die Total Cup Points prognostiziert; für eine kleine Hilfe beim nächsten Einkauf, um auf die Qualität zu schließen. 

Eine kurze Bemerkung zu Entscheidungsbäumen. Sie neigen zum Overfitting, d.h. sie lernen die präsentierten Daten auswendig und sind daher nicht in der Lage, die eigentlich zu lernenden Zusammenhänge in neuen Daten zu erkennen. Dies wird dadurch verhindert, dass der Entscheidungsbaum getunt wird; im hier vorliegenden Fall werden insgesamt 1.600 Modelle berechnet und nun folgt das Ergebnis un dann die beste Lösung ausgewählt. Und das Ergebnis ist im wahrsten Sinne des Wortes ein Baum. 

<img src="/posts/2021-06-02-kaffeepause/post_files/figure-html/unnamed-chunk-8-1.png" width="2400" />
Der Baum unterscheidet an oberster Stelle anhand des Anbaulandes; im weiteren nach der Höhenlage des Anbaugebiets.

Nehme ich einmal, den von mir gekauften Kaffee (Herkunftsland: Mexiko, Höhenlage: "in Lagen über 1.000 Metern"), dann kann ich nun den Pfad wie folgt durchschreiten. Das Herkunftsland ist Mexiko, also geht es im Baum von der Wurzel den ersten Ast nach links. Am nächsten Knoten geht es wieder nach links, dann nach rechts, dann links, und nochmal links. Somit landet der Kaffe bei einem Rating von 80. Das Modell schätzt im Mittel 3 Punkte ungenau. Also liegen die Total Cup Points irgendwo zwischen 77 und 83; und damit schon eher am unteren Rand. 
Hätte ich das mal vorher gewusst!

Viel Spass damit beim nächsten Einkauf von Kaffee!
