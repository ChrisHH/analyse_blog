---
title: Wohin fliegt Hamburg?
author: R package build
date: '2021-08-14'
slug: []
categories: []
tags:
  - Flüge
  - Flughafen
  - HAM
  - U-Bahn
  - Taktung
  - Flugziele
draft: no
---





[GovData](https://www.govdata.de) ist das Portal für öffentliche Daten in Deutschland. Und dort bin ich auf den Datensatz gestoßen, den ich für den heutigen Post verwende. Der Datensatz beinhaltet die monatliche Anzahl von Abflügen von deutschen Flughäfen für die Jahre 2011 bis 2021. Da ich mich auch für Fliegerei und Flugzeuge interessiere (meine Faszination gilt hier den physikalischen Prinzipien die Fliegen überhaupt möglich machen und der Konstruktion der komplexen Fluggeräte), habe ich die Daten genutzt, um einmal festzustellen, wie verbunden Hamburg flugtechnisch mit der Welt ist. 

![](/posts/2021-08-14-wohin-fliegt-hamburg/post_files/mika-baumeister-DHlZenOMjJI-unsplash.jpg)
<center>
Hamburger Flughafen; Foto von Mika Baumeister auf https://unsplash.com
</center>

Das das Jahr 2020 ein Tiefpunkt der Flugbranche war, ist hinreichend bekannt. Insofern geht es im Folgenden um die Jahre 2011 bis 2019.

## Entwicklung der Anzahl von Abflügen pro Jahr

Zunächst folgt ein Blick darauf, wie sich die Anzahl der Abflüge über die Jahre entwickelt.

<img src="/posts/2021-08-14-wohin-fliegt-hamburg/post_files/figure-html/unnamed-chunk-3-1.png" width="2400" />

Da kommen etliche Flüge zusammen. Auffällig ist ein Einbruch der Zahlen im Jahr 2013, der sich im Jahr 2012 schon andeutete. Zur Erinnerung: das war die Zeit der Euro-Krise (und hier möglicherweise die Ursache für den Rückgang). Insgesamt betrachtet, schwankt die jährliche Zahl der Abflüge zwischen 64.708 und 72.710. Nimmt man mal 70.000 Flüge pro Jahr als jährlichen Mittelwert über diese Zeit, dann sind das rechnerisch 192 Abflüge pro Tag. Der Hamburger Flughafen ist in der Regel von 6:00 - 23:00 Uhr geöffnet. In dieser Zeit sind das also etwa 10 Abflüge pro Stunde oder einer alle 6 Minuten. Das ist nicht mehr weit entfernt von der [Taktung der Hamburger U-Bahn](https://www.hamburg.de/hvv/9806056/hvv-ausweitung-takt/)!

## Verteilung von Inlands- zu Auslandsflügen

Jetzt geht es weiter mit der Betrachtung in Richtung der Flugziele. Zunächst ein Blick auf die Verteilung von Inlands- und Auslandsflügen über die Jahre.

<img src="/posts/2021-08-14-wohin-fliegt-hamburg/post_files/figure-html/unnamed-chunk-4-1.png" width="2400" />

In den Jahren vor der Corona-Krise hatte sich bis zum Jahr 2017 der Anteil der Abflüge nach Deutschland (Inlandsverbindungen) auf ein Minimum von 33% entwickelt, ist dann zuletzt etwas angestiegen. Mit Blick auf die absoluten Zahlen weiter oben, dürfte das Minimum des Anteils für Inlandsflüge im Jahr 2017 jedoch eher darauf zurückzuführen sein, dass der Höchststand von Abflügen im Jahr 2017 durch eine wachsende Anzahl von Auslandsflügen erreicht wurde. Das bestätigt die nachfolgende Grafik.

<img src="/posts/2021-08-14-wohin-fliegt-hamburg/post_files/figure-html/unnamed-chunk-5-1.png" width="2400" />

Bei aller Faszination für das Thema Fliegen glaube ich, dass ein Großteil der Inlandsverbindungen (wenn nicht sogar alle) für mehr Klimaschutz eingespart werden kann. Mit Hilfe dieser offiziellen Zahlen kann dies für die Zukunft zweifelsfrei nachvollzogen werden.

## Welche Saisonalität gibt es?

Wie oben gesehen, pendelt sich im Mittel die Anzahl von Abflügen pro Jahr ab Hamburg bei ca. 70.000 ein. Mit den vorliegenden Daten lässt sich aber auch gut eine Saisonalität über das Jahr analysieren. Hierzu folgt nun die Veranschaulichung der mittleren Anzahl von Abflügen über die Jahre 2011-2019 nach Monaten.

<img src="/posts/2021-08-14-wohin-fliegt-hamburg/post_files/figure-html/unnamed-chunk-6-1.png" width="2400" />

Das Bild war fast zu erwarten. Im Frühling zieht die Anzahl der Abflüge an, erreicht dann in den Monaten Mai bis Oktober ein Plateau und flacht dann wieder ab. Aus praktischer Sicht bedeutet dies unter anderem auch, dass man in den Sommermonaten die besseren Voraussetzungen zum Beobachten von Flugzeugen am Flughafen hat (z.B. Planespotting), da die Flugbewegung größer ist. 

## Wohin gehen die Abflüge, außerhalb von Deutschland?

Um diese Frage zu beantworten, habe ich im Folgenden die Inlandsflüge aus dem Datensatz entfernt und für das Jahr 2011 - 2019 die TOP 20 Zielländer in der folgenden Grafik animiert.

![](post_files/figure-html/unnamed-chunk-7-1.gif)<!-- -->

![](/posts/2021-08-14-wohin-fliegt-hamburg/post_files/unnamed-chunk-7-1.gif)

In diesem Racing Bar Chart (in 15 Sekunden wechselt das Ranking der Jahre 2011-2019), sieht man: Hauptziele ab Hamburg (Direktverbindungen ins Ausland) sind seit Jahren Großbritannien und die Schweiz. Bis zum Jahr 2016 (Jahr des Brexit-Referendums) steigt die Anzahl der Flüge nach Großbritannien kontinuierlich und nimmt dann wieder ab. Die Türkei, Östereich und Frankreich sind relativ stabil auf den Plätzen 3-5 vertreten.
