---
title: Von Bären, Glöckchen und Pfefferspray
author: Christian Jähnert
date: '2021-08-22'
slug: []
categories:
  - Canada
  - Wildleben
  - Natur
  - Bären
  - Open Data
tags: []
draft: no
---


Ich beabsichtige, bald nach Kanada zu reisen und habe mich deshalb schon durch einen Reiseführer gelesen, einen Reisevortrag angeschaut und natürlich auf verschiedenen Webseiten recherchiert.
Dabei bin ich immer wieder darauf gestoßen, dass Kanada ein "Bärenland" ist und man in der Natur an bestimmten Stellen damit rechnen müsse, Bären anzutreffen. In einer anderen Dokumentation über die in den Alpen angesiedelten Trentino-Bären sagte ein Naturschützer: Wenn es zu einer Begegnung kommt, sollte man sicher eher glücklich schätzen, eine auserwählte Person zu sein.

![](/posts/2021-08-15-wilde-tiere-in-kanada/post_files/geoff-brooks-K01-S2NVJCg-unsplash.jpg)

Immerhin weiß ich durch meine Recherche nun schon, das Bären eigentlich kaum interessiert daran sind, Menschen zu begegnen. Verhindern kann man das wohl in dem man lautstark singt oder sich unterhält und indem man Bärenglöckchen am Rucksack befestigt, die ständig klimpern und den Bären schon von Weitem signalisieren, dass Menschen im Anmarsch sind. So ein Glöckchen habe ich mir schon angeschafft. 

<img src="/posts/2021-08-15-wilde-tiere-in-kanada/post_files/Bearbell.png" alt="" width="40%"/>

Kommt die Begegnung unverhofft oder hat der Bär schlechte Laune, dann kann sich in letzter Konsequenz Bärenspray (eine große Dose Pfefferspray) als lebensrettend erweisen.

Lange Rede kurzer Sinn: man liest und hört viel. Aber am Ende des Tages sind es doch Fakten, die überzeugen. Und deshalb kommt mir der offizielle Datensatz über die Co-Existenz von Mensch und Wildtieren der kanadischen Regierung sehr gelegen. Diesen habe ich auf dem [Open Data Portal von Kanada entdeckt](https://open.canada.ca/data/en/dataset/743a0b4a-9e33-4b12-981a-9f9fd3dd1680) und möchte heute einmal analysieren, wie häufig Begegnungen mit Bären in der kanadischen Wildnis dokumentiert sind und wie sich die Bären dabei verhielten.

# Struktur des Datensatzes

Zunächst erfolgt ein Blick auf die Dimensionen und Variablen, die in dem Datensatz enthalten sind. Der Datensatz umfasst 15 Variablen und 33559 Beobachtungen; damit ist er schon sehr reichhaltig.

Im Einzelnen kommen die folgenden Variablen im Datensatz vor:

- incident_number
- incident_date
- local_time
- field_unit
- protected_heritage_area
- incident_type
- species_common_name
- sum_of_number_of_animals
- animal_health_status
- cause_of_animal_health_status
- animal_behaviour
- reason_for_animal_behaviour
- animal_attractant
- deterrents_used
- animal_response_to_deterrents

Und wie sieht es aus mit der Vollständigkeit?

<table>
 <thead>
  <tr>
   <th style="text-align:left;"> Variable </th>
   <th style="text-align:left;"> Datentyp </th>
   <th style="text-align:right;"> Vollständigkeit </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> incident_number </td>
   <td style="text-align:left;"> character </td>
   <td style="text-align:right;"> 1.0000000 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> field_unit </td>
   <td style="text-align:left;"> character </td>
   <td style="text-align:right;"> 1.0000000 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> protected_heritage_area </td>
   <td style="text-align:left;"> character </td>
   <td style="text-align:right;"> 1.0000000 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> incident_type </td>
   <td style="text-align:left;"> character </td>
   <td style="text-align:right;"> 1.0000000 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> species_common_name </td>
   <td style="text-align:left;"> character </td>
   <td style="text-align:right;"> 1.0000000 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> animal_health_status </td>
   <td style="text-align:left;"> character </td>
   <td style="text-align:right;"> 0.4141661 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> cause_of_animal_health_status </td>
   <td style="text-align:left;"> character </td>
   <td style="text-align:right;"> 0.1379064 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> animal_behaviour </td>
   <td style="text-align:left;"> character </td>
   <td style="text-align:right;"> 0.5677464 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> reason_for_animal_behaviour </td>
   <td style="text-align:left;"> character </td>
   <td style="text-align:right;"> 0.3187223 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> animal_attractant </td>
   <td style="text-align:left;"> character </td>
   <td style="text-align:right;"> 0.3010519 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deterrents_used </td>
   <td style="text-align:left;"> character </td>
   <td style="text-align:right;"> 0.2528979 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> animal_response_to_deterrents </td>
   <td style="text-align:left;"> character </td>
   <td style="text-align:right;"> 0.2255431 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> incident_date </td>
   <td style="text-align:left;"> Date </td>
   <td style="text-align:right;"> 1.0000000 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> local_time </td>
   <td style="text-align:left;"> difftime </td>
   <td style="text-align:right;"> 1.0000000 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sum_of_number_of_animals </td>
   <td style="text-align:left;"> numeric </td>
   <td style="text-align:right;"> 1.0000000 </td>
  </tr>
</tbody>
</table>

Das sieht gut aus, denn das Datum, die Uhrzeit und die Spezies ist zu 100% vollständig. Auch das Verhalten des Tieres weist einen hohen Füllgrad auf (fast 60%). 

# Beteiligte Tiere / Spezies

Nun ist es an der Zeit einen Überblick zu gewinnen, welche Tiere an den gemeldeten Vorkommnissen (so wird das Wort "Incident" im Folgenden übersetzt) beteiligt waren. 
Zunächst erfolgt ein kurzer Blick darauf, wieviele Vorkommnisse pro Jahr in dem Datensatz enthalten sind. 

<table>
 <thead>
  <tr>
   <th style="text-align:right;"> Jahr </th>
   <th style="text-align:right;"> Anzahl Vorkommnisse </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:right;"> 2017 </td>
   <td style="text-align:right;"> 6207 </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 2018 </td>
   <td style="text-align:right;"> 6984 </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 2019 </td>
   <td style="text-align:right;"> 10582 </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 2020 </td>
   <td style="text-align:right;"> 9786 </td>
  </tr>
</tbody>
</table>

Die Vorkommnisse wachsen seit 2017. Den Ansprung um mehr als 3.000 Meldungen im Jahr 2019 habe ich versucht mit Hilfe des Datensatzes zu erklären, blieb dabei aber erfolglos.

Die folgende Grafik zeigt nun die Top 10 Spezies die in den Vorkommnissen dokumentiert wurden.

<img src="/posts/2021-08-15-wilde-tiere-in-kanada/post_files/figure-html/unnamed-chunk-5-1.png" width="2400" />

Um die ungleiche Anzahl der Summen pro Jahr auszugleichen und einen besseren Vergleich zu ermöglichen folgt ein relativierter Blick, mit den Spezies sortiert nach dem Jahr 2020.

<img src="/posts/2021-08-15-wilde-tiere-in-kanada/post_files/figure-html/unnamed-chunk-6-1.png" width="2400" />

Die erfassten Vorkommnisse konzentrieren sich in den Top 10 von "Bighorn Sheep" bis "Wolf". Andere Spezien sind unter "Others zusammengefasst. Fasst man die Spezies "Black Bear" und "Grizzly Bear"  zusammen, dann kommt schon ein nennenswerter Anteil zusammen, nämlich circa 30% der Vorkommnisse im Jahr 2020. In den Jahren davor waren die Vorkommnisse mit Bären sogar noch ausgeprägter. Der Großteil der Vorkommnisse entfiel jedoch im Jahr 2020 mit fast 50% auf Elche. 

Doch worum ging es bei diesen Vorkommnissen? Dazu konzentriere ich die weitere Analyse für diesen Beitrag auf die Bären (Schwarzbären und Grizzlybären), da sie im Fokus stehen (sorry liebe Elche, falls ich Euch an dieser Stelle unterschätze).

# Verhalten der Bären

Die folgende Grafik gibt einen Überblick: über die "Art" des Verhaltens sowie die Häufigkeit.

<img src="/posts/2021-08-15-wilde-tiere-in-kanada/post_files/figure-html/unnamed-chunk-7-1.png" width="2400" />

Am häufigsten wurde bei Bären das Verhalten "Presence - Wildlife Exclusion Zones" erfasst. Damit ist gemeint, dass sich die Bären in Zonen aufhalten, die stark von Menschen genutzt werden, also z.B. wenn ein Bär plötzlich im Garten steht oder in der Mülltonne nach Futter sucht. Als nächstes folgt das Verhalten "Indiffernt to People/Vehicles", welches sich am besten mit dem Begriff "ignorierend" beschreiben lässt; d.h. der Bär reagiert nicht auf die menschliche Begegnung. Der Grund "Avoidance", also Vermeidung steht an dritter Stelle. Dieses Verhalten ist dadurch charakterisiert, dass sich der Bär zurückzieht. 

Interessant ist das Verhalten "Bluff Charge" - ein Scheinangriff bei dem der Bär auf eine Person zuläuft, um dann 2-3 Meter vor ihr wieder abzudrehen. Dies wurde in den Jahren 2017-2020 131 gemeldet. In 20 Fällen wurde das Verhalten "Chase" also Verfolgung gemeldet und schlussendlich zum Glück in nur 4 Fällen ein "Predatory Approach". Dies ist ein Angriff, bei dem der Bär den Menschen als Beute sieht - rechnerisch ist das 1 Fall pro Jahr.

Blicke ich auf die Vorkommnisse und filtere die "unangenehmeren" heraus, sind es zum Glück gar nicht so viele (nochmal: die Daten in der Grafik beziehen sich auf die Jahre 2017-2020). Dennoch werde ich diese Begegnungen bzw. Vorkommnisse im folgenden nochmal genauer betrachten. Dazu zähle ich ab hier:

- Curious Approach
- Bluff Charge
- Physical or Aggressive Display
- Escort (Follow-Flank)
- Chase
- Contact-People
- Pretadorty Approach

# Ursachen für die Vorkommnisse

Die Frage ist, was der Auslöser für dieses Verhalten gewesen ist? Die folgende Grafik versucht das zu beantworten, indem sie für jedes Verhalten die Häufigkeit der Ursachen darstellt.

<img src="/posts/2021-08-15-wilde-tiere-in-kanada/post_files/figure-html/unnamed-chunk-8-1.png" width="2400" />
Der Blick auf die verschiedenen Gründe der einzelnen Verhaltensweisen zeigt, dass es den Bären häufig um Verteidigung geht, undzwar vorallem der Bärenjungen.
Ein weiterer Grund ist die Verteidigung des Lebensraumes aber teilweise auch von Futter. In den Gründen taucht auch immer wieder der Grund "Food Conditioned" auf. 

Die Daten besätigen eigentlich nur das, was man in zahlreichen Ratgebern nachlesen kann:

1. Bei einer Bärensichtung sollte man sofort das Umfeld ins Auge nehmen um festzustellen ob es Bärenjunge gibt; dann ist man gut beraten niemals zwischen Bärin und Bärenjunges zu geraten.
2. Man sollte Bären, auch wenn sie die Gegenwart eines Menschen akzeptieren, nicht nähern. Im Gegenteil, man sollte versuchen, sich zurückzuziehen. 
3. Man sollte sie niemals füttern. 

Insofern war das eine schöne Übung, um die Ratschläge mit Daten und Fakten zu untermauern.

# Tageszeiten der Vorfälle

Mich interessiert nun, wann, d.h. zu welchen Tageszeiten diese Vorfälle stattfinden. Da die Uhrzeiten ebenfalls dokumentiert sind, wird für die Vorkommnisse mit den Verhaltensweisen des vorigen Abschnittes die Tageszeit (volle Stunde) ermittelt und dann pro Zeitstunde gezählt, wie viele Vorkommnisse dokumentiert wurden.

<img src="/posts/2021-08-15-wilde-tiere-in-kanada/post_files/figure-html/unnamed-chunk-9-1.png" width="2400" />
Die Interpretation ist schwierig, denn eine Begegnung bedarf Bär und Mensch. Insofern sieht man an dieser Kurve wahrscheinlich , dass Menschen vorallem tagsüber zu bestimmten Zeiten unterwegs sind (z.B. beim Wandern) und dann überrascht es nicht, dass die meisten Vorkommnisse in der Zeit von 11-16 Uhr dokumentiert sind. Ohne eine Normierung fällt es schwer, diese Daten zu interpretieren.

# Abwehr im Falle einer unangenehmen Begegnung mit einem Bären

Für die im vorigen Abschnitt untersuchten Verhaltensweisen schaue ich schlussendlich noch, wie diese abgewehrt wurden und was das Ergebnis war. 

<img src="/posts/2021-08-15-wilde-tiere-in-kanada/post_files/figure-html/unnamed-chunk-10-1.png" width="2400" />

Eine gute Nachricht! Der Blick auf die Y-Achse verrät, dass sich viele Vorkommnisse in den Zeilen "Retreat" häufen und das ist die Flucht des Bären. Das heißt: im Fall einer unangenehmen Begegnung mit einem Bären kann man ihn dazu bringen wegzulaufen. 

Nahezu jede Antwort (ein Wurf) mit einem "Chalkball", also einem kleinen Sack, gefüllt mit Kreidepuder (den man wohl zum Klettern benötigt, um ausreichend Grip in den Händen zu haben), führte dazu, dass der Bär weglief. Auffällig ist auch, dass "Noise - Voice" dazu führt, dass die Bären weggehen/-laufen (ich denke hier auch an das Bärenglöckchen). das Bärenspray führt erfüllt seinen Zweck und der Bär läuft überwiegend davon. In keinem dokumentierten Fall wird der Bär derart wütend, dass er zum Angriff übergeht. Wenngleich das Bärenspray dem Bären keine bleibenden Beeinträchtigungen zufügt, sollte man daran denken, dass auch andere Mittel geeignet sind, einen Bären zu vertreiben. 

Mein Fazit: ich bin mit der Bärenglocke schon gut gerüstet. Über die Anschaffung eines "Chalkballs" denke ich noch nach! Das Bärenspray kann man in Kanada wohl auch örtlich ausleihen ("If you spray, you pay!"). 