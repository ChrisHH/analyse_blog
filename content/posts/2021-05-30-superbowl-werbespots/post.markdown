---
title: Superbowl Werbespots
author: Christian Jaehnert
date: '2021-06-01'
slug: []
categories:
  - Tidytuesday
  - Tidyverse
  - Superbowl
  - R
  - K-Means
  - Werbung
tags: []
draft: no
---

# Superbowl Werbespots

Heute lasse ich die weitere Analyse der Tagesthemen einmal ruhen und konzentriere mich auf einen Datensatz des Projektes "Tidytuesday". Jeden Dienstag wird auf der Webseite des Projektes (https://github.com/rfordatascience/tidytuesday) ein neuer Datensatz veröffentlicht, den man analysieren kann. Für meinen heutigen Post habe ich mir den Datensatz aus Kalenderwoche 10/2021 herausgesucht.
In diesem geht es um die Werbespots die in den Werbepausen des Superbowls der letzten 20 Jahre (2000 - 2020) ausgestrahlt wurden. 

Um die Lesbarkeit zu vereinfachen blende ich heute die Code Snippets aus (bei Interesse können diese im Markdown-Dokument in meinem GibHub Repository eingesehen werden).




## Überblick der Variablen und Kennzahlen

Zunächst verschaffe ich mir einen Überblick über die Variablen in dem Datensatz, genauer gesagt schaue ich auf statistische Kennzahlen und Lagemaße. Diese sind für alle Variablen in dem Datensatz in der folgenden Tabelle 1 zusammengefasst.





<table style='width: auto;'
        class='table table-condensed'>
<caption>Table 1: Data summary</caption>
 <thead>
  <tr>
   <th style="text-align:left;">   </th>
   <th style="text-align:left;">   </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> Name </td>
   <td style="text-align:left;"> ads </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Number of rows </td>
   <td style="text-align:left;"> 247 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Number of columns </td>
   <td style="text-align:left;"> 25 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> _______________________ </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Column type frequency: </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:left;"> character </td>
   <td style="text-align:left;"> 10 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> logical </td>
   <td style="text-align:left;"> 7 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> numeric </td>
   <td style="text-align:left;"> 7 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> POSIXct </td>
   <td style="text-align:left;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ________________________ </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Group variables </td>
   <td style="text-align:left;"> None </td>
  </tr>
</tbody>
</table>


**Variable type: character**

<table>
 <thead>
  <tr>
   <th style="text-align:left;"> skim_variable </th>
   <th style="text-align:right;"> n_missing </th>
   <th style="text-align:right;"> complete_rate </th>
   <th style="text-align:right;"> min </th>
   <th style="text-align:right;"> max </th>
   <th style="text-align:right;"> empty </th>
   <th style="text-align:right;"> n_unique </th>
   <th style="text-align:right;"> whitespace </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> brand </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1.00 </td>
   <td style="text-align:right;"> 3 </td>
   <td style="text-align:right;"> 9 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 10 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> superbowl_ads_dot_com_url </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1.00 </td>
   <td style="text-align:right;"> 34 </td>
   <td style="text-align:right;"> 120 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 244 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> youtube_url </td>
   <td style="text-align:right;"> 11 </td>
   <td style="text-align:right;"> 0.96 </td>
   <td style="text-align:right;"> 43 </td>
   <td style="text-align:right;"> 43 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 233 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> id </td>
   <td style="text-align:right;"> 11 </td>
   <td style="text-align:right;"> 0.96 </td>
   <td style="text-align:right;"> 11 </td>
   <td style="text-align:right;"> 11 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 233 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kind </td>
   <td style="text-align:right;"> 16 </td>
   <td style="text-align:right;"> 0.94 </td>
   <td style="text-align:right;"> 13 </td>
   <td style="text-align:right;"> 13 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> etag </td>
   <td style="text-align:right;"> 16 </td>
   <td style="text-align:right;"> 0.94 </td>
   <td style="text-align:right;"> 27 </td>
   <td style="text-align:right;"> 27 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 228 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> title </td>
   <td style="text-align:right;"> 16 </td>
   <td style="text-align:right;"> 0.94 </td>
   <td style="text-align:right;"> 6 </td>
   <td style="text-align:right;"> 99 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 228 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> description </td>
   <td style="text-align:right;"> 50 </td>
   <td style="text-align:right;"> 0.80 </td>
   <td style="text-align:right;"> 3 </td>
   <td style="text-align:right;"> 3527 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 194 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> thumbnail </td>
   <td style="text-align:right;"> 129 </td>
   <td style="text-align:right;"> 0.48 </td>
   <td style="text-align:right;"> 48 </td>
   <td style="text-align:right;"> 48 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 118 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> channel_title </td>
   <td style="text-align:right;"> 16 </td>
   <td style="text-align:right;"> 0.94 </td>
   <td style="text-align:right;"> 3 </td>
   <td style="text-align:right;"> 37 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 185 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
</tbody>
</table>


**Variable type: logical**

<table>
 <thead>
  <tr>
   <th style="text-align:left;"> skim_variable </th>
   <th style="text-align:right;"> n_missing </th>
   <th style="text-align:right;"> complete_rate </th>
   <th style="text-align:right;"> mean </th>
   <th style="text-align:left;"> count </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> funny </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 0.69 </td>
   <td style="text-align:left;"> TRU: 171, FAL: 76 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> show_product_quickly </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 0.68 </td>
   <td style="text-align:left;"> TRU: 169, FAL: 78 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> patriotic </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 0.17 </td>
   <td style="text-align:left;"> FAL: 206, TRU: 41 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> celebrity </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 0.29 </td>
   <td style="text-align:left;"> FAL: 176, TRU: 71 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> danger </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 0.30 </td>
   <td style="text-align:left;"> FAL: 172, TRU: 75 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> animals </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 0.37 </td>
   <td style="text-align:left;"> FAL: 155, TRU: 92 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> use_sex </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 0.27 </td>
   <td style="text-align:left;"> FAL: 181, TRU: 66 </td>
  </tr>
</tbody>
</table>


**Variable type: numeric**

<table>
 <thead>
  <tr>
   <th style="text-align:left;"> skim_variable </th>
   <th style="text-align:right;"> n_missing </th>
   <th style="text-align:right;"> complete_rate </th>
   <th style="text-align:right;"> mean </th>
   <th style="text-align:right;"> sd </th>
   <th style="text-align:right;"> p0 </th>
   <th style="text-align:right;"> p25 </th>
   <th style="text-align:right;"> p50 </th>
   <th style="text-align:right;"> p75 </th>
   <th style="text-align:right;"> p100 </th>
   <th style="text-align:left;"> hist </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> year </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1.00 </td>
   <td style="text-align:right;"> 2010.19 </td>
   <td style="text-align:right;"> 5.86 </td>
   <td style="text-align:right;"> 2000 </td>
   <td style="text-align:right;"> 2005 </td>
   <td style="text-align:right;"> 2010 </td>
   <td style="text-align:right;"> 2015.00 </td>
   <td style="text-align:right;"> 2020 </td>
   <td style="text-align:left;"> ▇▇▇▇▆ </td>
  </tr>
  <tr>
   <td style="text-align:left;"> view_count </td>
   <td style="text-align:right;"> 16 </td>
   <td style="text-align:right;"> 0.94 </td>
   <td style="text-align:right;"> 1407556.46 </td>
   <td style="text-align:right;"> 11971111.01 </td>
   <td style="text-align:right;"> 10 </td>
   <td style="text-align:right;"> 6431 </td>
   <td style="text-align:right;"> 41379 </td>
   <td style="text-align:right;"> 170015.50 </td>
   <td style="text-align:right;"> 176373378 </td>
   <td style="text-align:left;"> ▇▁▁▁▁ </td>
  </tr>
  <tr>
   <td style="text-align:left;"> like_count </td>
   <td style="text-align:right;"> 22 </td>
   <td style="text-align:right;"> 0.91 </td>
   <td style="text-align:right;"> 4146.03 </td>
   <td style="text-align:right;"> 23920.40 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 19 </td>
   <td style="text-align:right;"> 130 </td>
   <td style="text-align:right;"> 527.00 </td>
   <td style="text-align:right;"> 275362 </td>
   <td style="text-align:left;"> ▇▁▁▁▁ </td>
  </tr>
  <tr>
   <td style="text-align:left;"> dislike_count </td>
   <td style="text-align:right;"> 22 </td>
   <td style="text-align:right;"> 0.91 </td>
   <td style="text-align:right;"> 833.54 </td>
   <td style="text-align:right;"> 6948.52 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 7 </td>
   <td style="text-align:right;"> 24.00 </td>
   <td style="text-align:right;"> 92990 </td>
   <td style="text-align:left;"> ▇▁▁▁▁ </td>
  </tr>
  <tr>
   <td style="text-align:left;"> favorite_count </td>
   <td style="text-align:right;"> 16 </td>
   <td style="text-align:right;"> 0.94 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:left;"> ▁▁▇▁▁ </td>
  </tr>
  <tr>
   <td style="text-align:left;"> comment_count </td>
   <td style="text-align:right;"> 25 </td>
   <td style="text-align:right;"> 0.90 </td>
   <td style="text-align:right;"> 188.64 </td>
   <td style="text-align:right;"> 986.46 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 10 </td>
   <td style="text-align:right;"> 50.75 </td>
   <td style="text-align:right;"> 9190 </td>
   <td style="text-align:left;"> ▇▁▁▁▁ </td>
  </tr>
  <tr>
   <td style="text-align:left;"> category_id </td>
   <td style="text-align:right;"> 16 </td>
   <td style="text-align:right;"> 0.94 </td>
   <td style="text-align:right;"> 19.32 </td>
   <td style="text-align:right;"> 8.00 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 17 </td>
   <td style="text-align:right;"> 23 </td>
   <td style="text-align:right;"> 24.00 </td>
   <td style="text-align:right;"> 29 </td>
   <td style="text-align:left;"> ▃▁▂▆▇ </td>
  </tr>
</tbody>
</table>


**Variable type: POSIXct**

<table>
 <thead>
  <tr>
   <th style="text-align:left;"> skim_variable </th>
   <th style="text-align:right;"> n_missing </th>
   <th style="text-align:right;"> complete_rate </th>
   <th style="text-align:left;"> min </th>
   <th style="text-align:left;"> max </th>
   <th style="text-align:left;"> median </th>
   <th style="text-align:right;"> n_unique </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> published_at </td>
   <td style="text-align:right;"> 16 </td>
   <td style="text-align:right;"> 0.94 </td>
   <td style="text-align:left;"> 2006-02-06 10:02:36 </td>
   <td style="text-align:left;"> 2021-01-27 13:11:29 </td>
   <td style="text-align:left;"> 2013-01-31 09:13:55 </td>
   <td style="text-align:right;"> 227 </td>
  </tr>
</tbody>
</table>

Die Größe des Datensatzes ist mit 247 Fällen überschaubar. Die Variable "year" bestätigt, was schon in der Datensatzbeschreibung zu lesen war: der Datensatz enthält die Werbespots aus den Jahren 2000 (p0) bis 2020 (p100). Im Mittel gab es ungefähr 12 Werbespots pro Superbowl in den letzten 20 Jahren. Diese stammen von ingesamt 10 verschiedenen Marken (brand; n_unique = 10). Zudem sind ein paar YouTube Kennzahlen integriert: die Anzahl der Views, Likes, Dislikes und Kommentare (im Bereich der "numeric"-Variablen).

Interessant ist, dass die Werbespots klassifiziert wurden (logical-Variablen), z.B. nach "funny" oder "patriotic" - und die gute Nachricht ist, dass diese Klassifizierung vollständig gepflegt ist, denn es existieren keine "missings" und die Ausprägungen sind "TRUE" und "FALSE". 

Hier stellt sich die Frage, wie, das heißt, nach welchen Kriterien diese Klassifizierung vorgenommen wurde und auch von wem. Aber für heute nehme ich mal an, dass die Klassifizierung so in Ordnung ist. Anhand der Mittelwerte kann man es schon erkennen: etwa 69% aller Werbespots sind "funny" und 68% zeigen das Produkt kurz ("shop_product_quickly"). Ich habe einmal die TRUE-Werte addiert und durch die Anzahl der Werbespots geteilt: in dieser Mittelwertbetrachtung treffen 2.7 Attribute auf einen Werbespot zu und entsprechend 4.3 Attribute nicht zu. 

Mich interessiert nun: gibt es Gruppen von Werbespots, die entlang dieser sieben Kriterien ähnlich sind? Das ruft nach einer Clusteranalyse (einzelne Beobachtungen werden anhand ausgewählter Variablen zu einem Cluster "gebündelt") und diese habe ich durchgeführt.

## Ergebnisse der Clusteranalyse

Für die Clusteranalyse habe ich den k-Means Algorithmus verwendet. Mit Hilfe des Screeplots habe ich entschieden, fünf Cluster zu bilden. 

<img src="/posts/2021-05-30-superbowl-werbespots/post_files/figure-html/unnamed-chunk-3-1.png" width="2400" />




Nun prüfe ich, wie groß diese fünf Cluster sind, d.h. wieviele Werbespots pro Cluster enthalten sind.
Wünschenswert ist hier eine möglichst gleichmäßige Verteilung. Im Kontrast dazu wäre es z.B. ungünstig, wenn ein Cluster 200 Beobachtungen (Werbespots) enthält, und die restlichen 47 Werbespots verteilen sich auf die restlichen vier Cluster. In diesem Fall sollte man zumindest einmal stutzig werden und die Clusterung noch einmal anschauen. 

<img src="/posts/2021-05-30-superbowl-werbespots/post_files/figure-html/unnamed-chunk-5-1.png" width="2400" />

Im Ergebnis existieren zwei größere Cluster (3 und 4), ein mittelgroßes und zwei kleinere Cluster. 

## Charakterisierung & Benennung der Cluster

Die extrahierten Cluster haben bis hier sehr technische Namen, nämlich einfach eine Nummerierung.
In der Praxis ist diese Art der Benennung ungebräuchlich, da z.B. der Begriff "Cluster 1" kein Bild in den Köpfen der Nutzer*innen erzeugt. Also sollten Namen gefunden werden, die die Cluster möglichst eindeutig charakterisieren.

Wie kann das gelingen? Ich schaue mir zunächst auf den Variablen, die für die Clusterung verwendet wurden die Mittelwerte an. 

<img src="/posts/2021-05-30-superbowl-werbespots/post_files/figure-html/unnamed-chunk-6-1.png" width="2400" />

Im Cluster 4 sind 85% der Werbespots "funny". Es kennzeichnet sich zudem dadurch, dass die Werbespots nicht mit Gefahr spielen und sie frei von Sexualität (Erotik) sind. Gut einer von 4 Spots in dem Cluster zeigt Tiere.

Und wie sieht es im Cluster 3 aus? Alle Werbespots in diesem Cluster spielen mit Gefahr ("danger"). Dabei können meiner Meinung nach negative Emotionen entstehen. Diese werden mit Spass ausgeglichen, denn 97% der Werbespots sind auch als "funny" klassifiziert. 

Nun folgt das Cluster 5. Es ist das Cluster der Werbespots in denen Sexualität bzw. Erotik eine Rolle spielt; vermutlich mit einem Augenzwinkern konnotiert, da auch hier ein Großteil der Werbespots als "funny" klassifiziert ist.

Die Werbespots in Cluster 1 sind nicht besonders lustig; dafür treten in Ihnen zu 73% bekannte Persönlichkeiten ("celebrity") auf.

Im Cluster 2 sind die Werbespots dadurch gekennzeichnet, dass sie zu 90% Tiere als Protagonisten zeigen.

Ich entscheide mich an dieser Stelle für die folgende Namensgebung:

- Cluster 1:    Celebrity
- Cluster 2:    Animal & Patriotic
- Cluster 3:    Danger & Funny
- Cluster 4:    Funny 
- Cluster 5:    Erotic

Tipp: In der Praxis muss man mit der Auswahl von Clusternamen sehr sorgfältig umgehen; denn wenn sie einmal ausgesprochen werden, sind sie in den Köpfen und erzeugen ein Bild (und das kann möglicherweise auch fehlleitend sein). Unter Umständen sollte man die Clusternamen in einem größeren Kreis mit verschiedenen Personen und Blickwinkeln diskutieren.

## Vergleich der Cluster 

Nachdem die Cluster nun feststehen, können sie weitergehend analysiert werden. Hierzu blicke ich zunächst auf die zeitliche Entwicklung und dann im Folgenden auf die YouTube Metriken und vergleiche die Cluster. 

Wenn man sich (so wie gleich) prozentuale Verteilungen betrachtet, halte ich es für wichtig, auch die dahinter stehende absolute Größe zu kennen. Diese stelle ich zunächst dar. 

<img src="/posts/2021-05-30-superbowl-werbespots/post_files/figure-html/unnamed-chunk-7-1.png" width="2400" />

Man erkennt, dass die Anzahl von Werbespots pro Jahr in den Jahren 2000 - 2020 schwankt. Die Range liegt im Bereich von 5 bis 15 Spots. Man erkennt auch, dass es in mehr oder weniger regelmäßigen Abständen Einbrüche bei der Anzahl gibt; erklären kann ich mir das im Augenblick nicht. Es könnte damit zu tun haben, dass die verbleibenden Spots länger sind (und die Sendezeit ist begrenzt) oder das gleiche Spots während der Werbepausen des Superbowls mehrfach verwendet werden. Zuletzt ist die Anzahl der Spots rückläufig und lag bei neun. 

Und wie entwickeln sich nun die vorab bestimmten Cluster relativ über die Zeit?

<img src="/posts/2021-05-30-superbowl-werbespots/post_files/figure-html/unnamed-chunk-8-1.png" width="2400" />

Es fällt auf, dass in Relation zur Anzahl der Spots pro Jahr, die Spots mit bekannten Persönlichkeiten über die letzten Jahre zunehmen. Bis zum Jahr 2009 waren Spots aus diesem Cluster relativ selten, während in den letzten fünf Jahren der Anteil relativ groß war (mindestens ein Drittel). Gleichzeitig sind in den letzten Jahren die Werbespots mit erotischen Inhalten weniger geworden. Um das Jahr 2005 waren diese offensichtlich angesagter.

Nun möchte ich verstehen, wie die einzelnen Cluster auf den YouTube Metriken abschneiden:

- Anzahl der Likes
- Anzahl der Dislikes
- Anzahl Kommentare

Die Filme stammen teilweise schon aus dem Jahr 2000 (Jahr der Ausstrahlung während des Superbowls). Dennoch prüfe ich vorab, wann die Filme bei YouTube gepublished wurden.

<table>
 <thead>
  <tr>
   <th style="text-align:left;">   </th>
   <th style="text-align:right;"> 2006 </th>
   <th style="text-align:right;"> 2007 </th>
   <th style="text-align:right;"> 2008 </th>
   <th style="text-align:right;"> 2009 </th>
   <th style="text-align:right;"> 2010 </th>
   <th style="text-align:right;"> 2011 </th>
   <th style="text-align:right;"> 2012 </th>
   <th style="text-align:right;"> 2013 </th>
   <th style="text-align:right;"> 2014 </th>
   <th style="text-align:right;"> 2015 </th>
   <th style="text-align:right;"> 2016 </th>
   <th style="text-align:right;"> 2017 </th>
   <th style="text-align:right;"> 2018 </th>
   <th style="text-align:right;"> 2019 </th>
   <th style="text-align:right;"> 2020 </th>
   <th style="text-align:right;"> 2021 </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> 2000 </td>
   <td style="text-align:right;"> 2 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 2 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2001 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 2 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 4 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2002 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 3 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2003 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 4 </td>
   <td style="text-align:right;"> 2 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2004 </td>
   <td style="text-align:right;"> 3 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 2 </td>
   <td style="text-align:right;"> 2 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2005 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 4 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2006 </td>
   <td style="text-align:right;"> 4 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2007 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 9 </td>
   <td style="text-align:right;"> 2 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2008 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 12 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2009 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 15 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2010 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 12 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2011 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 4 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2012 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 8 </td>
   <td style="text-align:right;"> 5 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2013 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 13 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2014 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 9 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2015 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 2 </td>
   <td style="text-align:right;"> 6 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2016 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 7 </td>
   <td style="text-align:right;"> 3 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2017 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 5 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2018 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 11 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2019 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 13 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2020 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 8 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
</tbody>
</table>

Die Plattform YouTube existiert erst seit dem Jahr 2005. Entsprechend konnten die Werbespots früherer Jahre natürlich nicht vorher eingestellt werden. Da jedoch einige Filme schon länger, und einige Film kürzer auf YouTube vorhanden sind, ist es meiner Meinung nach unfair, absolute Größen o.g. Metriken miteinander zu vergleichen. Daher nehme ich eine Normierung vor, undzwar auf die Anzahl der Views. Dadurch ergeben sich Anteile. So gibt zum Beispiel der Wert "rel_like_count" wieder, wieviele Likes anteilig auf die Anzahl der Views fallen.

<img src="/posts/2021-05-30-superbowl-werbespots/post_files/figure-html/unnamed-chunk-10-1.png" width="2400" />

Die erste Erkenntnis: in Relation zur Anzahl der Views wird wesentlich mehr geliked, als disliked und kommentiert (da dies nicht clusterspezifisch ist, sieht das nach einem allgemeinem YouTube-Effekt aus). 

Auf Basis der sehr geringen Anteilswerte beim mittleren Anteil der Likes (mean_like_share) zeigt sich meiner Meinung ein Trend dahingehend, dass die Spots aus dem Cluster "celebrity" häufiger kommentiert werden, als z.B. Werbespots aus dem Cluster "funny". Mein Gedanke: Spielt hier der Vampireffekt ein Rolle? Damit ist gemeint, dass Werbung mit Celebrities ein zweischneidiges Schwert sein kann, da die bekannte Persönlichkeit von der eigentlichen Werbebotschaft ablenken kann (als Einstieg empfiehlt sich dieses Paper: https://core.ac.uk/download/pdf/25590419.pdf). Es wäre also durchaus interessant, für diese Werbespots die Kommentare auf YouTube zu analysieren, um zu verstehen, was dort kommentiert wird; tauchen dort häufiger Namen der bekannten Persönlichkeit(en) aus dem Werbespot auf? (Aber das vertiefe ich heute nicht; hierzu müssten zunächst einmal die Kommentare über die YouTube API heruntergeladen werden.) 

## Cluster & Marken

Zuguterletzt möchte ich für heute untersuchen, in welchen Clustern die einzelnen Marken Ihre Schwerpunkte haben. Zu dem Zweck analysiere ich zunächst wieder, wieviele Werbespots eine Marke in den Jahren 2000-2020 ausgestrahlt hat und dann wie sich diese über die Cluster verteilen.

<img src="/posts/2021-05-30-superbowl-werbespots/post_files/figure-html/unnamed-chunk-11-1.png" width="2400" />

Das ist eindeutig: mehr als 100 Werbespots während des Superbowls in den Jahren 2000-2020 stammen von der Brauerei Anheuser Busch (Hersteller der Marken Bud Light und Budweiser). Dann folgt Pepsico mit Spots für Doritos und Pepsi sowie weitere Marken. 

Und wo liegen nun die Schwerpunkte der einzelnen Marken?

<img src="/posts/2021-05-30-superbowl-werbespots/post_files/figure-html/unnamed-chunk-12-1.png" width="2400" />

Aufgrund der vielen Fälle für Bud Light und Budweiser, halte ich die Anteile schon für gut interpretierbar und das ist wirklich spannend: Bud Light hat einen klaren Schwerpunkt auf Werbespots aus dem Cluster "Danger / Funny" und "Funny". Die Marke kommt also in 70% der Spots "lustig" daher; und sie wirbt komplett ohne bekannte Personen (keine Spots aus dem Cluster "Celebrity").

Bei Budweiser ist das Bild anders: hier stammt mehr als ein Drittel der Werbespots aus dem Cluster "Animal / Patriotic". Der zweite Schwerpunkt liegt auch hier bei "funny". 

Ich vermute, die unterschiedlichen Schwerpunkte der Werbespots haben mit den unterschiedlichen Marketingzielgruppen der beiden Marken zu tun. Bud Light scheint (auch mit Blick auf die Webseite) eher abteuerlustige Personen anzusprechen. Während bei Budweiser vermutlich eher Personen anspricht, die auch patriotisch sind. (Zur Erinnerung: die Ausstrahlung der Spots findet in den USA statt.)

Zum Abschluss noch ein Blick auf die zwei Konkurrenten Coca-Cola und Pepsi. Während Pepsi bei 40% der Werbespots auf Erotik setzt, verzichtet Coca-Cola fast ganz darauf und ist vor allem schwerpunktmäßig patriotisch unterwegs.
