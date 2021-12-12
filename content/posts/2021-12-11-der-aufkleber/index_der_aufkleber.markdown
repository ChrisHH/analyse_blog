---
title: Der Aufkleber
author: Christian Jähnert
date: '2021-12-11'
slug: []
categories:
  - COVID
  - Falschinformation
tags: []
draft: yes
---


# Eine Entlarvung

## Hintergrund

Im Fahrstuhl meines Fitnessstudios ist eine Informationstafel angebracht, die über die aktuell geltende 2G-Regel informiert. Links unten fiel mir dort in dieser Woche dieser Aufkleber auf, der meiner Meinung nach die Pandemie verharmlosen soll. Insbesondere aber soll wohl verdeutlicht werden (durch die roten Pfeile), dass die Impfung nicht funktioniert.

<img src="/posts/2021-12-11-der-aufkleber/index_der_aufkleber_files/PHOTO-2021-12-11-19-13-52.jpg" alt="" width="50%"/>

Ich habe diesen unsinnigen Aufkleber entfernt und heute etwas Zeit, mich dem Inhalt zu widmen.

## Erkenntnis 1

Die Quellenangabe ist sehr dürftig. Nach etwas Recherche konnte ich die Zahlen dann aber finden und nachstellen. Leichte Abweichungen könnten auf nachträgliche Korrekturen seitens des Statistischen Bundesamtes zurückzuführen sein.  

[Hier](https://www.destatis.de/DE/Themen/Gesellschaft-Umwelt/Bevoelkerung/Sterbefaelle-Lebenserwartung/sterbefallzahlen.html) finden sich die Informationen zu Sonderauswertung der Sterbefallzahlen in Deutschland.

Fazit: Durch unpräzise Quellenangaben wird versucht zu verhindern bzw. erschwert, die Inhalte zu überprüfen.

## Erkenntnis 2

Ich musste etwas nachdenken, aber die mittlere Spalte ist wohl wie folgt zu verstehen:

* Monat 2019 – es gab noch kein COVID
* Monat 2020 – es gab COVID aber noch keine Impfung
* Monat 2021 – es gab COVID und es waren schon Menschen geimpft

Aufgrund des roten Pfeils soll man nun in den Glauben versetzt werden, dass im Zeitraum mit COVID und geimpften Menschen mehr oder annähernd genauso viele Menschen, wie in 2019 gestorben sind. Also COVID ist nicht so schlimm bzw. die Impfung bringt nichts.  

Auf der Seite des Statistischen Bundesamtes werden die Zahlen ausführlich kommentiert und eingeordnet. 

Die Ableitung auf dem Aufkleber ist extrem verkürzt und Stimmungsmache. Sie unterstellt in der Form, dass die Anzahl der Todesfälle allein von Monat, Jahr und der Gegebenheit, ob COVID bzw. ein Impfstoff dagegen existiert abhängt. 
Dabei ist in dem Text des Statistischen Bundesamts so einiges bereits kommentiert. So gab es Mitte Juni 2021 eine auffällige Erhöhung der Sterbefallzahlen in Zusammenhang mit einer Hitzewelle in Deutschland. 

Ich habe daher die Tabelle auf dem Aufkleber einmal um zusätzliche Faktoren ergänzt.


Es wird sehr schnell deutlich .... 

Fazit: Durch sehr selektive Angaben wird bewusst getäuscht.


## Erkenntnis 3

Ich habe mich gefragt: Warum hat die Person, die diesen Aufkleber erstellt hat, ausgerechnet die Monate Mai, Juni und Juli gewählt? Das sind Monate, in denen es in Deutschland schon wärmer wird. Und wir wissen, dass das Virus sich in wärmeren Monaten durch die höheren Temperaturen und UV Strahlung schlechter verbreitet. Deshalb habe ich recherchiert: das RKI berichtet auch die Sterbefälle im Zusammenhang mit COVID, nämlich [hier](https://www.rki.de/DE/Content/InfAZ/N/Neuartiges_Coronavirus/Projekte_RKI/COVID-19_Todesfaelle.html).

<img src="/posts/2021-12-11-der-aufkleber/index_der_aufkleber_files/figure-html/unnamed-chunk-2-1.png" width="2400" />

Hätte die Person die Monate Februar, März und April genutzt, dann hätte die Geschichte wohl nicht gut funktioniert.

Fazit: 



## Erkenntnis 4

Nun verbinde ich einmal die Daten des RKI mit denen des Statistischen Bundesamtes.
Mein Ziel ist ein Strukuraufriss der Sterbezahelen pro Monat nach COVID und Rest.












```
## # A tibble: 12 × 2
##    name      `sd(value)`
##    <chr>           <dbl>
##  1 april           3966.
##  2 august          3317.
##  3 dezember          NA 
##  4 februar         4903.
##  5 januar          9512.
##  6 juli            2304.
##  7 juni            2955.
##  8 mai             2302.
##  9 marz            9490.
## 10 november          NA 
## 11 oktober         3554.
## 12 september       3301.
```


