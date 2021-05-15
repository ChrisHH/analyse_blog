---
title: "Tagesthemen - Textanalyse"
author: "Christian"
date: '2021-05-18'
slug: []
categories:
- Natural Language Processing
- Textanalytics
tags:
- NLP
- Tidytext
- Tidymodels
draft: no
---



## Einleitung

Ich habe jetzt schon die eine oder andere deskriptive Analyse der Tagesthemen durchgeführt.
Bisher noch völlig ausser acht gelassen ist die Themenvielfalt der Sendungen. Und darum geht es im heutigen Post
Wie schon beschrieben, 

Zunächst werden die bis zu diesem Punkt verfeinerten Daten eingelesen.


```r
tagesthemen <- read_csv("https://funalytics.netlify.app/files/data_clean.csv")
```

## Ableitung von Features aus den Themen der Sendung

Aus den Themen der Sendung lassen sich bestimmt noch Features ableiten. Um dies zu realisieren kommt das R-Paket "tidytext" von Julia Silge und David Robinson zum Einsatz. Ich kann das zugehörige Buch "Text Mining with R (O'Reilly Verlag) sehr empfehlen. Es existiert aber auch als open source Version unter diesem Link: xxx

Im ersten Schritt müssen die Themen der Sendung vereinzelt werden. Nach kurzer Betrachtung des Strings stellte ich fest, dass die Theman durch ein Komma getrennt sind. Es bietet sich also ein String-Split mit Hilfe der separate Funktion an. Diese verlangt jedoch die Vorgabe einer Anzahl von Variablen, in die gesplittet werden soll. Ich habe die Anzahl einmal auf 20 gesetzt und prüfe im Nachgang ob das ausreichend war. Außerdem ist es bei Textanalysen immer sehr sinnvoll, Leerzeichen zu "strippen" und den gesamten String in Groß- oder Kleinschreibung umzuwandeln, wobei sich im NLP Umfeld die Kleinschreibung durchgesetzt hat. 

Wenn dies erfolgt ist, vertikalisiere ich den Datensatz gleich noch, um die weitere Feature-Ableitung einfacher zu halten. 


```r
library(tidytext)

tagesthemen_text <- tagesthemen %>%
  select(-contains("standbild"), -contains("url")) %>%
  mutate(
    themen = str_remove(themen, "Themen der Sendung:"),
    themen = trimws(themen)
  ) %>%
  separate(themen, sep = ",", into = paste("thema", 1:20, sep = ".")) %>%
  pivot_longer(cols = contains("thema"), names_to = "nummer_thema", values_to = "thema") %>%
  mutate(
    nummer_thema = str_remove(nummer_thema, "thema."),
    nummer_thema = as.numeric(nummer_thema),
    thema = trimws(thema),
    thema = tolower(thema)
  )
```

Nun prüfe ich zunächst, ob meine Annahme korrekt war, mit maximal 20 Themenvariablen auszukommen.
In der letzten Variable, also "thema.20" bzw. der Variable "nummer_thema" mit der Ausprägung 20, müssten alle themen ein NA besitzen.
Das heißt die Anzahl der NA's muss der Anzahl aller Fälle entsprechen.


```r
# Check ob Anzahl von 20 Variablen in separate Anweisung ausreicht

tagesthemen_text %>%
  filter(nummer_thema == 20) %>%
  count(thema)
```

```
## # A tibble: 1 x 2
##   thema     n
##   <chr> <int>
## 1 <NA>    813
```

Das sieht gut aus!

Nun verschaffe ich mir einen Überblick über die Häufigkeit einzelner Themen. 


```r
tagesthemen_text %>%
  count(thema, sort = TRUE) %>%
  kable()
```

<table>
 <thead>
  <tr>
   <th style="text-align:left;"> thema </th>
   <th style="text-align:right;"> n </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 10341 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weitere meldungen im überblick </td>
   <td style="text-align:right;"> 697 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter </td>
   <td style="text-align:right;"> 451 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> der kommentar </td>
   <td style="text-align:right;"> 362 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die meinung </td>
   <td style="text-align:right;"> 127 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> der sport </td>
   <td style="text-align:right;"> 93 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weitere nachrichten im überblick </td>
   <td style="text-align:right;"> 58 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    diese sendung wurde nachträglich bearbeitet </td>
   <td style="text-align:right;"> 31 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die lottozahlen </td>
   <td style="text-align:right;"> 28 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weitere meldungen im überblick ii </td>
   <td style="text-align:right;"> 16 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> der sport am sonntag </td>
   <td style="text-align:right;"> 14 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weitere meldungen im überblick i </td>
   <td style="text-align:right;"> 13 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> meldungen aus dem sport </td>
   <td style="text-align:right;"> 12 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zur &quot;fußball-bundesliga&quot; dürfen auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 9 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sport </td>
   <td style="text-align:right;"> 9 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fußball-bundesliga </td>
   <td style="text-align:right;"> 8 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sportblock </td>
   <td style="text-align:right;"> 7 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zur &quot;fußball-bundesliga&quot; dürfen aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 6 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema &quot;fußball-bundesliga&quot; darf aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 5 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zur fußball-bundesliga dürfen aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 5 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die sendung wurde nachträglich bearbeitet </td>
   <td style="text-align:right;"> 5 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutschlandtrend </td>
   <td style="text-align:right;"> 5 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema &quot;fußball-bundesliga&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 4 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zum thema &quot;fußball-bundesliga&quot; dürfen auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 4 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ard-deutschlandtrend </td>
   <td style="text-align:right;"> 3 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema &quot;fußball bundesliga&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 3 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema bundesliga darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 3 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die börse </td>
   <td style="text-align:right;"> 3 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ergebnisse der fußball-bundesliga </td>
   <td style="text-align:right;"> 3 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> helden des alltags </td>
   <td style="text-align:right;"> 3 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> (die beiträge zum thema &quot;fußball-bundesliga&quot; dürfen auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden.)
                                
                                    hinweis:
                                    die beiträge zum thema &quot;fußball-bundesliga&quot; dürfen auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 2 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-ticker </td>
   <td style="text-align:right;"> 2 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> da sie bildmaterial enthielt </td>
   <td style="text-align:right;"> 2 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden durfte. </td>
   <td style="text-align:right;"> 2 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema &quot;europa league&quot; darf aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 2 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema &quot;fußball-bundesliga&quot; darf aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 2 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema &quot;uefa champions league&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 2 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema fußball bundesliga darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 2 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zu den themen &quot;fußball bundesliga&quot; und &quot;formel 1&quot; dürfen aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 2 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zu den themen &quot;fußball-bundesliga&quot; und &quot;formel 1&quot; dürfen aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 2 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zum thema &quot;fußballbundesliga&quot; dürfen auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 2 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zur fußball-bundesliga dürfen auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 2 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zur fußball-bundesliga und zur formel 1 dürfen aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 2 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die bilder zur fußball-bundesliga dürfen aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 2 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    diese sendung wurde redaktionell bearbeitet </td>
   <td style="text-align:right;"> 2 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter 
                                
                                    hinweis:
                                    diese sendung wurde nachträglich bearbeitet </td>
   <td style="text-align:right;"> 2 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> dürfen aus rechtlichen gründen nicht auf tageschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 2 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> einigung bei eu-gipfel: von der leyen soll eu-kommissionschefin werden </td>
   <td style="text-align:right;"> 2 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erste runde dfb-pokal </td>
   <td style="text-align:right;"> 2 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu-sondergipfel: staats- und regierungschefs entscheiden über besetzung der spitzenposten </td>
   <td style="text-align:right;"> 2 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> haiti: 10 jahre nach dem erdbeben </td>
   <td style="text-align:right;"> 2 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> landtagswahlen in baden-württemberg und rheinland-pfalz </td>
   <td style="text-align:right;"> 2 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> linkspartei gewinnt landtagswahl in thüringen </td>
   <td style="text-align:right;"> 2 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> machtkampf in der cdu </td>
   <td style="text-align:right;"> 2 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weiter meldungen im überblick </td>
   <td style="text-align:right;"> 2 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weitere nachrichten im überblick i </td>
   <td style="text-align:right;"> 2 </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;7 deaths of maria callas&quot;: abramovic-premiere in der bayerischen staatsoper </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;aus worten werden taten&quot;: ein zwischenruf von esther bejarano </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;brecht&quot; doku-drama wird auf berlinale uraufgeführt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;calexit&quot; - kaliforniens ausstiegspläne und die anti-trump-bewegung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;der grüne knopf&quot; als neues gütesiegel für nachhaltige textilien vorgestellt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;edison - ein leben voller licht&quot; im kino </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;einheitsexpo&quot; in potsdam feiert 30 jahre deutsche einheit </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;es bleiben uns die worte&quot;: zwiegespräch zweier väter über den terroranschlag im konzertsaal &quot;bataclan&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;feindeslisten&quot; im internet: kritik an der informationspolitik der behörden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;fest der freiheit&quot; in prag erinnert an ausreise von ddr-flüchtlingen vor 30 jahren </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;frauenfußball weltmeisterschaft&quot; und &quot;formel 1&quot; dürfen auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;fridays for future&quot;-aktivisten besuchen un-klimakonferenz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;fußball-bundesliga&quot; sowie &quot;extrem-wetter in spanien&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;gärten des grauens&quot;: steinwüsten in deutschlands vorgärten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;geordnete-rückkehr-gesetz&quot;: opposition kritisiert seehofers pläne für abschiebepflichtige </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;german wunderkind&quot; dirk nowitzki beendet basketball-karriere </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;gottes influencer&quot; - kirche und die neuen medien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;heldin&quot; des alltags: labormitarbeiterin </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;heldinnen und helden des alltags&quot;: was ist aus geworden? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;in real life&quot; in der tate modern: olafur eliasson verwandelt naturphänomene in kunst </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;joker&quot; zwischen kritik und rekord </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;leid und herrlichkeit&quot; - ein neuer film von pedro almodóvar </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;lösungsfinder&quot;: hilfe für bedürftige durch sachspenden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;morals &amp; machines&quot; - künstliche-intelligenz-treffen in dresden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;mudlarks&quot;: schatzsuche in der themse </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;nacht des lichts&quot; - veranstaltungsbranche will auf katastrophale wirtschaftliche situation aufmerksam machen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;neue seidenstraße&quot;: wie china den balkan vereinnahmt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;new york times&quot;-bericht: us-präsident trump zahlte offenbar jahrelang kaum einkommensteuer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;no-covid&quot; auf der insel neuwerk </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;only human&quot;: fotoausstellung von martin parr in der national portrait gallery in london </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;paneuropäisches picknick&quot;: bewegendes wiedersehen am ort der ungarischen grenzöffnung von 1989 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;projekt pegasus&quot;: forschungen zum autonomen fahren </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;qanon&quot; in deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;querdenken&quot;-demo gegen die corona-auflagen: die rechtsradikale inszenierung vor dem reichstag </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;querdenken&quot;-demo in leipzig: wer übernimmt die verantwortung für das fiasko? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;rettet die bienen!&quot;: erfolgreiches volksbegehren </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;rettet die dörfer&quot; - protestspaziergang am tagebau garzweiler </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;rocketman&quot;: das wilde leben von elton john als musikfilm </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;schicksals&quot;-symphonie zum jubiläumskonzert von beethovens 250. geburtstag </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;secret walls gallery&quot;: graffiti-künstler im stuttgarter bahnhof </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;sparsame vier&quot; legen gegenentwurf zu eu-wiederaufbauplan vor </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;stronger still&quot;: kunstprojekt mit can dündar im maxim gorki </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;tag des freibades&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;tagesthemen&quot;mittendrin: umzug der schwartau ins alte bett </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;tour de france und &quot;em-qualifikation der frauen&quot; dürfen aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;umweltsau&quot;-satire des wdr löst hitzige debatte aus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;völkerschauen&quot; bei hagenbeck: wie umgehen mit kolonialer/rassistischer geschichte? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;was deutschland bewegt&quot;: leben im priesterseminar </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> &quot;werkstattgespräch&quot;: cdu diskutiert über flüchtlingspolitik </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> „extinction rebellion“: aufstand für das klima </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> „feinde“-drehbuchautor ferdinand von schirach im interview </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> „gott“ von ferdinand von schirach beleuchtet die frage um leben und tod </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> (der beitrag zum thema &quot;fußball bundesliga&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden.)

                                
                                    hinweis:
                                    der beitrag zum thema &quot;fußball bundesliga&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> (der beitrag zum thema &quot;fußball-bundesliga&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden.)

                                
                                    hinweis:
                                    der beitrag zum thema &quot;fußball-bundesliga&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> (die beiträge zum thema &quot;fußball-bundesliga&quot; dürfen auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden.)
                                
                                    hinweis:
                                     die beiträge zum thema &quot;fußball-bundesliga&quot; dürfen auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> (die moderation wurde aufgrund eines fehlers in der grafik nachträglich bearbeitet) 
                                
                                    hinweis:
                                    die moderation wurde aufgrund eines fehlers in der grafik nachträglich bearbeitet. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> (diese sendung wurde nachträglich bearbeitet. in der fernsehausstrahlung um 22:00 uhr wurde im beitrag zum klimacamp die augsburger oberbürgermeisterin eva weber irrtümlich in der unterzeile als oberbürgermeisterin bayerns bezeichnet.)
                                
                                    hinweis:
                                    diese sendung wurde nachträglich bearbeitet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #fridaysforfuture: in china demonstriert eine einzelne aktivistin für die umwelt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #kurzerklärt: erdüberlastungstag </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #kurzerklärt: nachhaltig reisen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #lösungsfinder </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #lösungsfinder: ein mut machendes hebammen-modell aus hamburg </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #lösungsfinder: erfolgsmodell familienklassen für schüler mit schwierigkeiten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #lösungsfinder: mobbing - das unterschätzte problem </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #lösungsfinder: neue modelle für flexibles arbeiten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #lösungsfinder: wie ein dorf seine kneipe rettet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #lösungsfinder: wie lässt sich der wohnungsmangel beheben? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #lösungsfinder: wie man den umgang mit wasser ändern könnte </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #lösungsfinder: wie syndikate wohnen bezahlbar machen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #lösungsfinder: wie wir weniger lebensmittel verschwenden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin aus graben: ehemalige heimkinder betreuen heimkinder </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin aus wuppertal: verödete innenstädte und neue utopien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin im allgäu: streit um bergwelt des grünten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin im ehemaligen kohleviertel dortmund-hörde entstehen klimafreundliche produkte </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin im ersten corona-hotspot: heinsberg </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin in deutschland: abriss der hochstraßen über ludwigshafen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin in deutschland: abzug der us-soldaten rund um den truppenübungsplatz grafenwöhr </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin in deutschland: streitschlichter unterwegs in der mannheimer nachtszene </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: &quot;wangerooger inselbote&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: ard-themenwoche &quot;wie wollen wir leben?&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: bauern protestieren gegen preisverfall durch preiskämpfe bei discountern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: baupannen bei einer brücke zwischen köln und leverkusen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: besuch im forschungs- und erlebniszentrum paläon im niedersächsischen schöningen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: bollstedt ein jahr nach corona </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: camping - urlaubstrend 2020 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: corona-weihnachten im pflegeheim </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: das höllental zwischen umwelt- und klimaschutz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: das saarland und das katzenproblem </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: der &quot;medibus&quot; unterwegs in gegenden ohne medizinische versorgung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: der frust einer feuerwerksfirma in bielefeld </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: der kampf um wustrow </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: die hessische bergbaustadt heringen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: die homberger - netzwerk macht landleben zukunftsträchtig </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: die not der obdachlosen in corona-zeiten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: die wende aus zwei blickwinkeln </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: digitaler semesterstart für lehramtsstudierende in landau </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: duisburgs hafen - hoffnung im strukturwandel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: ein dorf nimmt sein schicksal in die eigene hand </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: ein jahr corona-pandemie - mittendrin im supermarkt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: ein jahr nach halle - wie sich jüdisches leben in sachsen-anhalt gestaltet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: ein jahr nach hanau-attentat </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: ein kinderheim in corona-zeiten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: eine kirche für zwei konfessionen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: einkaufshilfe bei kiel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: formel 1 kehrt zum nürburgring zurück </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: freiburg und seine sinti-familien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: gemeinsam handeln magdeburg </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: gesegnete traktoren -pater mittendrin in seiner gemeinde </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: grenzenloses engagement - wie sich junge leute um deutsch-deutsche geschichte kümmern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: haribo schließt werk in ostdeutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: integration von geflüchteten im thüringischen gera </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: jüdische gemeinde in lörrach in baden-württemberg </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: kampf gegen windräder im münsterland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: kein land in sicht - helgoland und der corona-november </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: klimacamp in augsburg </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: krankenhaus statt theaterbühne </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: leben im tinyhouse und das problem mit der stellplatzsuche </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: leben in duisburg-marxloh </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: lernen fürs abitur im internat </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: lockdown im kaufhaus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: lüneburg - wenn der schimmelpilz zur untermiete wohnt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: mit den waldpädagogen im hambacher forst </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: musterhafte klimagemeinde wathlingen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: neuer kulturanlauf in bremen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: nur wenig straßen in magdeburg sind nach frauen benannt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: pödelwitz - ein dorf im tagebau sieht neue zukunftschancen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: schutz der alten – tübingen macht es vor </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: seiffen bangt ums weihnachtsgeschäft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: smart village remmesweiler </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: sommer in zeiten von corona </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: sportler bereiten sich auf olympia vor </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: staatstheater mainz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: straßenkarneval - im autokino </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: streit um sendemast in sacrow </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: übergangswohnen für obdachlose in bamberg </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: warum im sächsischen lommatzsch kein mann im rathaus sitzt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: was die fahrradstadt münster bewegt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: wie bremerhavens traum vom geschäft mit der windenergie platzte </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: wie die berliner abschied vom flughafen tegel nehmen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: wie hotelfach-auszubildende in pirna die corona-krise nutzen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #mittendrin: zirkus im lockdown </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> #niemalsverstummen: 15 jahre holocaust denkmal </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 02. oktober 1990: der letzte tag der ddr </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 04 showdown in birmingham -  johnson und hunt treffen auf basis </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 06 klimaproteste + fm </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 07 schlittenhunde </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 09 überleitung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 1. mai-demonstrationen in corona-zeiten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 100 jahre bauhaus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 100 jahre bauhaus: virtuelle ausstellung in erfurt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 100 jahre bergwacht </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 100 jahre politischer aschermittwoch </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 100 jahre waldorfschule </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 100 tage annegret kramp-karrenbauer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 100. salzburger festspiele im kleineren rahmen eröffnet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 12 überleitung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 12. spieltag der fußball-bundesliga </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 13 übergabe sport </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 150 jahre alpenverein: e-bikes verstopfen die wanderwege </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 16 verabschiedung
                                
                                    hinweis:
                                    der beitrag zum thema &quot;frauen-fußball wm&quot; darf aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 1700 jahre jüdisches leben in deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 176 tote nach flugzeugabsturz bei teheran </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 18 jahre nach 9/11: ersthelfer leiden bis heute unter den spätfolgen des terroranschlags </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 18. spieltag der fußball-bundesliga </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 19 übergabe nachrichten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 20 jahre iss </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2019 ist ein rekordjahr für deutsche rüstungsexporte </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 222 jahre bollenhut </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 25 jahre dayton-abkommen: ein bericht aus mostar </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 25 jahre ende der apartheid: landenteignungen weißer farmer in südafrika </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 25 jahre nach dem genozid in ruanda </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 25. jahrestag der estonia-katastrophe: angehörige fordern neue untersuchungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 250 jahre beethoven </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 3 milliarden menschen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 3 prozent auf alles: zwischenbilanz der mehrwertsteuersenkung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 3. spieltag fußball bundesliga </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 30 jahre deutsche einheit: diskussion um unterschiede zwischen west- und ostdeutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 30 jahre einheit: städtepartnerschaft zwischen gelsenkirchen und cottbus in der lausitz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 30 jahre mauerfall </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 30 jahre mauerfall - das schwere erbe der  kinder der friedlichen revolution </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 30 jahre mauerfall: die grenze zwischen zicherie und böckwitz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 30 jahre mauerfall: nach wie vor hartnäckige vorurteile zwischen &quot;wessis&quot; und &quot;ossis&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 30 jahre mauerfall: wie die tagesthemen den 9. november miterlebten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 30 jahre wiedervereinigung: ringen um die einheit </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 40 zentimeter vom balkon: autobahnzubringer in kairo </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 50 jahre &quot;perlflasche&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 50 jahre davos: hinter den kulissen des weltwirtschaftsforums </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 50 jahre deutsche entwicklungshelfer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 50 jahre nach missglückter mondmission der &quot;apollo 13&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 50 jahre nationalpark bayerischer wald </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 50 jahre robotron </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 50 jahre stonewall-razzien: von der new yorker christopher street geht 1969 ein signal um die welt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 50 jahre tatort </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 50 jahre woodstock </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 50. jahrestag: kniefall von willy brandt in warschau </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 500 jahre havanna: zwischen lebensmittelrationierungen und unternehmergeist </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 500. todestag von leonardo da vinci </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 500. todestag: raffael-ausstellung in rom </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 58. biennale in venedig: wenn kunst politisch ist </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 5g-entscheidung: bundesnetzagentur diskutiert umgang mit huawei </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 5g-versteigerung beendet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 70 jahre bundeswehr - die staatsbürger in uniform </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 70 jahre grundgesetz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 70 jahre nato: feierlichkeiten in washington trotz krise </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 70 jahre zentralrat der juden in deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 70. berlinale eröffnet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 70. jahre korea-krieg </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 70. jubiläum des sudetendeutschen tages </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 75 jahre &quot;chinesenaktion&quot; der gestapo </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 75 jahre cdu: machtkampf um die spitze </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 75 jahre nach der befreiung: erinnerung und mahnung in auschwitz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 75. jahrestag des atombombenabwurfs über hiroshima </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 75. jahrestag des sieges der früheren sowjetrepubliken über hitler-deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 9. spieltag der fußball-bundesliga </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 90. geburtstag von anne frank </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> abbau eines denkmal: geschichtsumdeutung in ungarn </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> abendliche maskenpflicht in touristengebieten im bundesland kärnten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> aberkennung der gemeinnützigkeit: wie politisch dürfen organisationen sein? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> abgesagte messen und kongresse: die auswirkungen von corona auf den wirtschaftsstandort deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> abgewiesener grönland-deal: trump sagt dänemark-besuch ab </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> abholzung der regenwälder: verantwortung deutscher firmen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> abitur in zeiten von corona </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> abschiebungen aus der türkei: was geschieht mit den rückkehrern? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> abschied vom händeschütteln? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> abschied von der &quot;lindenstraße&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> abschied von ennio morricone: der meister der filmmusik ist tot </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> abschluss des cdu-parteitags in leipzig </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> abschluss des g7-gipfels </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> abschlussbericht zu den boeing-737-max-abstürzen: zu große triebwerke beeinträchtigten die balance </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> abschuss von wölfen soll erleichtert werden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> absolute kontrolle: kreml will &quot;souveränes&quot; internet in russland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> abstand und anstand: wie sich der umgang mit corona und miteinander gewandelt hat </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> abstandsregelung für windräder: spd will &quot;windbürgergeld&quot; für anwohner </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> abstimmung im britischen unterhaus zum binnenmarktgesetz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> abstimmung im us-repräsentantenhaus über amtsenthebungsverfahren für trump </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> abstimmung über brexit-verschiebung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> abstimmung über mögliche neue verfassung in chile </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> abstrakter expressionismus: ausstellung &quot;lee krasner&quot; in schirn kunsthalle frankfurt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> achtjähriger in frankfurt getötet: mann stößt kind und dessen mutter vor ice </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> afd &quot;flügel&quot; wird aufgelöst </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> afd darf vorerst nicht als rechtsextremer verdachtsfall eingestuft werden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> afd debattiert auf parteitag über eu-ausstieg </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> afd im osten vorn - fragen an brandenburgs ministerpräsidenten woidke </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> afd klagt gegen verfassungsschutz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> afd scheitert erneut mit bundestagsvizepräsidentenwahl </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> afd vs. linke: warum die alte kümmererpartei ausgedient hat </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> afd will präsenz-parteitag in kalkar ausrichten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> afd wird prüffall </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> afd-bundesvorstand beschließt parteiausschluss von brandenburgs landes- und fraktionschef andreas kalbitz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> afd-flügel unter beobachtung: verfassungsschutz geht gegen rechtsextremistische gruppierung vor </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> afd-kandidat gescheitert: görlitz bekommt cdu-oberbürgermeister </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> afd-parteitag: chrupalla und meuthen zum neuen führungsduo gewählt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> afd-parteitag: diskussionen über rede von meuthen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> afd-parteitag: nachfolge von parteichef gauland gesucht </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> afd-politiker brandner als vorsitzender des rechtsausschusses abgewählt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> afd-vorsitzender meuthen will sich vom rechtsextremen &quot;flügel&quot; distanzieren </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> afd: die neue normalität? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> afghanistan: 30 jahre nach dem abzug der russen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> afghanistan: kaum aussicht auf frieden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> afrika gilt als besonderes risikogebiet für das coronavirus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> afrikanische schweinepest in brandenburg nachgewiesen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> afrikanische staaten hoffen auf wirtschaftlichen aufschwung durch größte freihandelszone der welt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> agrarminister-konferenz: mehrere bundesländer fordern bessere kontrollen bei tiertransporten in nicht-eu-länder </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> airbus : aus für passagierflugzeug a380 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> airbus will produktion von flugzeugen um 40 prozent drosseln </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> akademie der wissenschaften leopoldina liefert vorschläge für lockerung der corona-maßnahmen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> aktienboom geht an deutschen sparern vorbei </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> aktionstag für nawalny: in ganz russland demonstrieren anhänger für seine freilassung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> aktivisten der demokratiebewegung in hongkong zu haftstrafen verurteilt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> aktuelle ereignisse zum coronavirus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> aktuelle hochrechnungen zur wahl in brandenburg </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> aktuelle hochrechnungen zur wahl in sachsen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> aktuelle lage im syrischen grenzgebiet idlib </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> aktuelle lage in beirut </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> aktuelle stunde im bundestag: debatte über störungen durch afd-gäste </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> aktuelle zahlen der bürgerschaftswahl in bremen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> aktuelle zahlen zur us-wahl </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> aktueller stand von impfstoffentwicklung und verbreitung von mutierten coronaviren </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> alba berlin ist deutscher basketballmeister </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> alexander westermann berichtet live aus einem pub </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> alfons blum: aus verzweiflung auf der corona-demo </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> alle jahre wieder: weltmeisterschaft im weihnachtsbaumwerfen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> alltag in deutschland während der coronavirus-krise </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> alltag in putins russland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> alltäglicher rassismus in deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> alltagserfahrungen mit dem &quot;wechselmodell&quot; des neuen familienrechts </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> alte ansprüche:  hohenzollern streiten sich mit dem staat um kunst und schlösser </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> altenpflege steht wegen corona-pandemie vor dem kollaps </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> alternative antriebe: der lkw der zukunft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> altersarmut in deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ältestes auto der welt erhält zulassung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> am internationalen kindertag stehen die sorgen und nöte der jüngsten im mittelpunkt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> am limit wegen corona: wenn krankenhäuser patienten verlegen müssen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> amazonas in flammen: zahl der brände so hoch wie nie </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> amerikanische gesundheitsbehörde fda veranlasst notfall-zulassung des corona-impfstoffs von biontech und pfizer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> amri-untersuchungsausschuss: die mühsame suche der sicherheitsbehörden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> amsterdam will touristen umleiten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> amthor kandidiert nicht für cdu-landesvorsitz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> amtierender sächsischer ministerpräsident michael kretschmer im gespräch </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> amtsenthebungsverfahren der us-demokraten gegen ehemaligen präsidenten trump scheitert </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> amtsenthebungsverfahren gegen abgewählten us-präsident trump eingeleitet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> amtsenthebungsverfahren gegen ehemaligen us-präsidenten trump endet mit freispruch </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> amtsenthebungsverfahren gegen us-präsident trump vor dem aus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> amtsenthebungsverfahren: prozess im senat beginnt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> amy coney barrett als richterin am obersten us-gericht vereidigt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> analysen und hintergründe zum jahrestag des mordes an walter lübcke </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> andere länder - andere regeln: wie die einzelnen bundesländer den impfstart koordinieren </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> andrea nahles tritt als spd-partei- und fraktionschefin zurück </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> angekündigter wagenknecht-rückzug: was das für die zukunft der linken bedeutet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> angesichts steigender corona-infektionen in china wappnet sich europa gegen das virus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> angespannte lage in chile </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> angespannte lage nach unwetter in österreich </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> angespannte situation: kurden und türken in deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> angespannte wirtschaftslage: ist die konjunkturkrise hausgemacht? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> angriff auf  die &quot;schwarze null&quot; - wie die spd um wählerstimmen kämpft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> angriff auf afd-bundestagsabgeordneten frank magnitz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> angriff von hooligans auf erste lgbt-parade in polen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> angriffe gegen staatsdiener in deutschland häufen sich </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> angst vor der einmischung aus dem ausland in die us-wahl </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> angst vor neuer militäroffensive auf letzte syrische rebellenhochburg idlib </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> angst vor unruhen anlässlich der us-wahl </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> anhaltende aufstände trotz neuwahl-versprechen in beirut </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> anhaltende demonstrationen in hongkong </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> anhaltende unruhen gegen polizeigewalt in den usa </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> anhänger von us-präsident trump protestieren in washington wegen angeblichem wahlbetrug </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> anhörung zur reform des bundesnachrichtendienstgesetzes </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> anklage gegen ex-vw-chef martin winterkorn </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ankunft der ersten teilnehmer vor g20-gipfel in osaka </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> annegret kramp-karrenbauer wird verteidigungsministerin </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> anpfiff oder nicht: streit um fußball-bundesliga </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> anschlag von hanau überschattet 11. integrationsgipfel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> anschläge in den usa: kühler empfang für trump in dayton und el paso </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> anstehende verhandlungen mit großbritannien: premierminister johnson lehnt bedingungen der eu ab </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> antarktis: deutsche forschungsstation neumayer iii feiert jubiläum </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> anti-corona-demos: die politische nacharbeitung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> anti-missbrauchskonferenz im vatikan: papst verspricht konsequenteres vorgehen gegen sexuellen missbrauch </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> antisemitismus: der milliardär george soros wird zum feindbild </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> anwerbung außerhalb europas: was inder über deutschland denken </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> apfelernte zwischen hoffen und bangen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> apple muss keine 13 milliarden euro steuern an irland nachzahlen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> arbeit statt schule: kinder im irak </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> arbeiten im home-office </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> arbeitsmarktzahlen: ausbildungsplatz &quot;last minute&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> arbeitsminister hubertus heil schlägt grundrente von 900€ vor </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> archäologen öffnen 1000 jahre alten sarkophag in mainz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ard stellt deutschen song für esc vor </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ard-deutschlandtrend i </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ard-deutschlandtrend ii </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ard-deutschlandtrend teil i </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ard-deutschlandtrend teil ii </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ard-deutschlandtrend: aktuelle umfrage zur europawahl </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ard-deutschlandtrend: corona-krise </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ard-deutschlandtrend: die grünen bundesweit erstmals vor cdu </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ard-deutschlandtrend: mehrheit  will groko fortsetzen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ard-deutschlandtrend: mehrheit sieht grundrente positiv </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ard-deutschlandtrend: us-wahl </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> argentinien - ein land das nicht aus der krise kommt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> argentinischer fußball-star maradona verstorben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> argentinischer zeichner mordillo verstorben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> arm und reich: wie unterschiedlich sind die lebensverhältnisse in deutschland? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> armin laschet bei digitalem parteitag zu neuem cdu-vorsitzenden gewählt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> armut in china - staatlich abgeschafft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> artenschutz-konferenz will bedrohte netz-giraffen stärker schützen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> arzneimittelversorung: bundeswirtschaftsminister altmaier soll gesetzesänderung beeinflusst haben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ärzte bei protesten gegen lukaschenko festgenommen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ärztemangel in ländlichen regionen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> asterix-erfinder albert uderzo gestorben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> äthiopien baut größten staudamm afrikas </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> äthiopiens streit um den nil-staudamm </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> äthiopischer premier abiy verkündet einnahme von tigray </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> attentäter von halle zu lebenslänglich mit anschließender sicherheitsverwahrung verurteilt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> audeering: emotionale spracherkennung durch künstliche intelligenz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> auf dem boden der realität: wie die menschen in dover den brexit heute sehen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> auf den spuren von alexander von humboldt in peru </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> auf nummer sicher: freiwillige massentests für österreichs tourismus-beschäftigte </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> auf spurensuche: wer sind die toten auf der balkanroute? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> auf's huhn gekommen: der neueste trend bei tec-milliardären im silicon valley </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> aufarbeitung des missbrauchsfalles in lügde </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> aufräumarbeiten im bayerischen ebersberg nach orkantief &quot;sabine&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> aufstieg und fall von ex-wirecard-chef braun </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> auftakt der bundesweiten corona-impfkampagne in deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> auftakt der filmfestspiele in venedig </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> auftakt der hannovermesse </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> auftakt der vierschanzentournee: skispringer geiger in oberstdorf zweiter </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> auftakt zur tour de france mit verschärften hygiene- und abstandsregeln </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> auktion der bundesnetzagentur: vattenfall kann das kohlekraftwerk moorburg 2021 stilllegen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> aus für germania: wieder eine deutsche fluggesellschaft pleite </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> aus für pkw-maut: eugh hält diese nicht mit eu-recht vereinbar </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ausbau von windkraft stockt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ausbildung: duales system in berufsschule und betrieb steigert chancen von jugendlichen auf einen job </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ausblick auf die wahl des neuen cdu-parteivorsitz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ausgebremst: corona und die jugend </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ausgeflogen aus wuhan wegen coronavirus: 20 deutsche in berlin erwartet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ausländische arbeitskräfte im libanon leiden unter auswirkungen der corona-krise </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ausländischen studenten von us-hochschulen droht die ausweisung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> auslosung der fußball-em 2020: deutsches team trifft in gruppenphase auf portugal und frankreich </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ausnahmegenehmigung für einsatz von remdesivir in den usa </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> außen alt - innen modern: das neue stadtschloss in berlin vor der eröffnung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> außenminister maas fordert &quot;bündnis der hilfsbereiten&quot; zur verteilung geretteter flüchtlinge </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> außenminister maas macht vorstoß zur reform der nato </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ausstattung der schulen trotz &quot;digitalpakt schule&quot; weiterhin ausbaufähig </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ausstellung &quot;wien 1900 - aufbruch in die moderne&quot; eröffnet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ausstellung im museum der bildenden künste in leipzig: 1989 - point of no return </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ausstellung mit bildern der fotografin helga paris in der akademie der künste </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ausstellung über auswirkungen von 100 jahren alpinem tourismus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ausstellung über den könig der tätowierer in hamburg </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ausstellung zum 90. geburtstag von karikaturist hans traxler im caricatura museum </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ausstellung zum künstler ernst barlach im albertinum dresden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> austockung der gesundheitsämter im kampf gegen corona </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> australian open finden trotz corona statt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> australien will millionenbetrag in schutz des &quot;great barrier reef&quot; investieren </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> australien: kletter-verbot für touristen am &quot;uluru&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> auswärtiges amt rät von nordspanien-reisen ab </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ausweisung deutscher korrespondenten: wie die türkei die pressefreiheit weiter beschneidet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> auswirkungen auf deutsche unternehmen bei einem no-deal-brexit </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> auswirkungen der corona-epidemie auf das leben in deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> auswirkungen der corona-infektion des us-präsidenten trump auf den wahlkampf </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> auswirkungen des impfstopps am beispiel eines bremer impfzentrums </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> auswirkungen von trumps umweltpolitik in north dakota </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> auszahlung von soforthilfen gestartet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> auto fährt in nordhessen in karnevalsumzug </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> auto oder öpnv: pendeln in corona-zeiten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> auto rast durch fußgängerzone: tote und verletzte in trier </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> auto-attacke im ruhrgebiet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> autobombe tötet mindestens 78 menschen im somalischen mogadischu </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> autoindustrie in der krise? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> autoindustrie in mexiko: neues bmw-werk vor drohenden strafzöllen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> autoindustrie und lufthansa fordern staatshilfen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> autonomes fahren: zusammenarbeit von daimler und bmw </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> babys von ukrainischen leihmüttern können wegen grenzschließungen nicht abgeholt werden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bahnfahrer teilen ihre skurrilsten erlebnisse </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bahrain und die emirate nehmen diplomatische beziehungen mit israel auf </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> baltische staaten erinnern an freiheitsbewegung &quot;baltischer weg&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> banane-essen als protest: der kulturkampf in polen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bangen vor coronavirus im flüchtlingslager auf lesbos </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bars und kneipen kämpfen mit hygiene-strategien im corona-herbst </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bauernprotest: wie tausende grüne kreuze das höfesterben aufhalten sollen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bauhaus: museumseröffnung in weimar </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> baumärkte haben in der coronakrise hochkonjunktur </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bayer-tochter monsanto wird wegen klagewelle zu risiko für mutterkonzern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bayerischer ministerpräsident söder zum neuen csu-vorsitzenden gewählt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bayern ruft katastrophen-fall aus und verschärft die corona-maßnahmen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bayern trennt sich von fußball-trainer niko kovac </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bayerns landesregierung räumt gravierende verzögerungen bei der übermittlung von corona-testergebnissen ein </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bayerns ministerpräsident söder fordert agrar-ökologie nach bayerischem vorbild </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bayrisches behörden-chaos bei masken und schutzkleidung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bedeutendste interpol-razzia gegen organisiertes doping </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bedeutung des eugh-urteils zur arbeitszeit für deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> beerdigung von fußball-ikone maradona </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> befreiungsschlag durch linksruck? die spd-pläne im realitäts-check </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> begegnungen: am zaun zwischen deutschland und der schweiz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> beginn &quot;tiergartenprozess&quot;: mord mit politischer dimension </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> beginn der corona-testpflicht für reiserückkehrer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> beginn der friedensverhandlungen für afghanistan </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> begrünte bushaltestellen in utrecht als lebensraum für insekten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> behandlung von covid 19-patienten bringen deutsche intensivstationen an ihre grenzen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bei besuch der türkisch-griechischen grenzregion versprechen eu-spitzen milliardenhilfen für griechenland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> beim virtuellen politischen aschermittwoch wirft die bundestagswahl ihren schatten voraus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> beirut nach der katastrophe: reformdruck auf libanesische regierung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> beirut: nach der schweren explosion gibt es nicht nur beim wiederaufbau probleme </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> belarus vor der parlamentswahl </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> belarusische aktivistin maria kolesnikowa verschwunden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> belarusische oppositionsführerin tichanowskaja zu beratungen mit kanzlerin merkel in berlin </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> belarussische behörden weisen ausländische journalisten aus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> beleg der entfremdung: ein youtuber und die politik </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> berater-gremium stellt zukunftskonzept für profi-fußball vor </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> beratergremium der bundesregierung mahnt zur vorsicht bei chinesischen firmenübernahmen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> beratung von bund und ländern nach zwei wochen teil-lockdown </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> beratungen im kanzleramt über die zukunft der deutschen autoindustrie </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> beratungen im kanzleramt über faire lebensmittelpreise </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> beratungen über corona-hilfsprogramm beim video-gipfel der eu </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bercow verweigert abstimmung über brexit-deal </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> berg-karabach-konflikt: reportage aus armenien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> berlin: kleingärten sollen durch wohungsbau verdrängt werden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> berlinale: nora fingscheidts debüt-spielfilm &quot;systemsprenger&quot; vorgestellt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> berlinale: verleihung der hauptpreise und abschied von langjährigem festival-chef dieter kosslick </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> berliner museen und ihre vereinsamten aufpasser </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> berliner philharmoniker spielen europakonzert ohne publikum </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> berliner spd plant gesetz für gebremste mietpreissteigerung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> berliner verwaltungsgericht weist klimaklage ab </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> besorgte apotheker: warum lieferengpässe bei arzneimitteln zunehmen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> besser als nichts?: minimalkonsens beim abschluss der un-klimakonferenz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bestechlichkeitsvorwürfe: bundestag hebt immunität des unionsfraktionsvizes nüßlein auf </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bestürzung bei deutschen politikern nach ausschreitungen in den usa </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> besuch aus hollywood: dicaprio </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> besuche in pflegeheimen: kreative lösungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> besuchsverbot in pflegeheimen aufgelöst </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> betroffene branchen reagieren enttäuscht über die neuen corona-beschlüsse </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bewährungstrafe für früheren kz-wachmann </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bewerbung für den eu-kommissionsvorsitz: die tv-debatte der kandidatinnen und kandidaten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bewohner schwedens offenbar skeptisch gegenüber corona-impfungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bgh-urteil im abgasskandal: vw muss autokäufer entschädigen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bgh: gerichte müssen härtefälle bei eigenbedarfskündigungen genau prüfen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> biathlon der frauen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> biathlon-em </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> biathlon-wm </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> biden gewinnt arizona während trump weiter von wahlbetrug spricht </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> biden oder trump: auf den swing state pennsylvania könnte es ankommen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> biden stellt kabinettsmitglieder vor </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bikini-museum in baden-württemberg eröffnet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bilanz der fußball-wm der frauen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bilanz des ersten einkaufsmontags mit auflagen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bilanz des ersten spieltags der fußball-bundesliga </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bilanz des neustarts nach aufhebung vieler corona-beschränkungen in deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bilanz des zweiten deutschen luftfahrtgipfels </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bilanz einer silvesternacht: ausgebranntes affenhaus und angriffe auf einsätzkräfte </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bilanz nach drei monaten homeschooling </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bilanz nach einem jahr &quot;ankerzentren&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bilanz nach einem jahr brexit </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bilanz nach sechs monaten corona-beschränkungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bilanz zwischen licht und schatten: der schwedische sonderweg in der corona-krise </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bilder vom mars-rover </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> binnenregionen in deutschland profitieren nicht vom touristen-boom </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bio boomt als folge von coronakrise </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> biodiversität: gegen die monokultur </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> biolebensmittel im discounter: eine umstrittene strategie </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> biotech-unternehmen curevac darf corona-impfstoff an menschen testen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> biotechnologen im hessischen zwingenberg entwickeln verfahren zum recyceln von gold aus verbranntem müll </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bislang keine hilfen für gemeinnützige unternehmen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bittere bilanz: zehn jahre nach dem tahrir-platz-aufstand in ägypten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bka-jahrestagung: was tun gegen hetze im netz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> blindes vertrauen? - 20 jahre wikipedia </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> blockiertes land: nichts geht in großbritannien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> blutige kämpfe um kaukasus-region bergkarabach </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> boeing-absturz in äthiopien: sicherheitsdiskussion über den flugzeugtyp boeing 737 max 8 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bolivien: präsident morales tritt zurück </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bolsonaro im corona-aufwind </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> boris johnson droht mit einem &quot;no deal&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> boris johnson wird neuer premierminister von großbritannien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> börsengang von uber: warum taxifahrer den fahrdienstleister fürchten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> botswana: mysteriöses elefantensterben im okavango-delta </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> brandbrief gegen höcke: offener machtkampf in der afd </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> brände im amazonas-gebiet sorgen für politische debatten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> brände im amazonas-regenwald: brasilianische regierung schickt militär zum löschen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> brandenburg: afd und grüne im umfragehoch </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> brasilien bremst verhandlungen auf un-klimakonferenz in madrid </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> brasilien in der krise </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> brasilien: der regenwald brennt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bregenzer festspiele feiern mit verdis &quot;rigoletto&quot; premiere </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> breite debatte über eine endlagersuche für atommüll soll für mehr transparenz sorgen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> breitensport in not: amateurvereine verzweifeln an der corona-krise </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> breitscheid-anschlag - haben behörden informationen über den attentäter zurückgehalten? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bremer koalitionsvertrag: bürgermeister sieling tritt ab </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bremer student erfindet übersetzungsapp für seltene sprachen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> brexit - wie geht es weiter? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> brexit als mittel gegen den fachkräftemangel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> brexit ohne ende: may gewinnt vertrauensabstimmung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> brexit stärkt schottlands wunsch nach unabhängigkeit </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> brexit-abstimmung: britisches parlament lehnt erneut eu-austrittsabkommen ab </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> brexit-aufschub: britische premierministerin may spielt auf zeit </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> brexit-deal könnte noch am britischen unterhaus scheitern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> brexit-deal: london hält abkommen für unwahrscheinlich </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> brexit-hardliner farage ist bereit für neuwahlen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> brexit-profiteur finanzplatz frankfurt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> brexit-streit: may fordert von abgeordneten mehr zeit und gute nerven </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> brexit-verhandlungen: das problem mit den fischereirechten in der nordsee </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> brexit-verhandlungen: problemfall fischerei </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> brexit-verschiebung: eu wartet entwicklung in großbritannien ab </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> brexit: boris johnson besucht merkel in berlin </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> brexit: französischer zoll probt abfertigung am eurotunnel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> brexit: hat sich boris johnson verzockt? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> brexit: kräftemessen im britischen parlament </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> brexit: may und juncker verhandeln weiter </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> brexit: wie geht es weiter? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> brexitvertrag: großbritannien vor der entscheidung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> brisante beziehung: eu und china streiten über &quot;neue seidenstraße&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> briten zwischen trauer und freude über den austritt aus der eu </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> britische premierministerin may hält an brexit-kurs fest </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> britische premierministerin may trifft oppositionsführer corbyn auf suche nach brexit-kompromiss </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> britische premierministerin may wirbt vor eu-sondergipfel um unterstützung für brexit-verschiebung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> britischer corona-held: spendensammler captain tom wird 100 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> britischer premier johnson hält kurze ansprache zum historischen moment des brexits </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> britischer premierminister johnson positiv auf covid-19 getestet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> britischer profi-fußballer rashford im kampf gegen kinderarmut </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> britischer sänger gerry marsden im alter von 78 jahren gestorben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> britisches gericht entscheidet gegen auslieferung von wikileaks-gründer assange in die usa </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> britisches parlament bringt mays brexit-plan erneut zu fall </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> britisches parlament lehnt brexit-abkommen ab </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> britisches parlament lehnt brexit-abkommen deutlich ab </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> britisches parlament stimmt gegen ungeregelten brexit </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> britisches unterhaus geht mit tumulten nach brexit-abstimmung in 5-wöchige zwangspause </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> britisches unterhaus kippt johnsons zeitplan </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> britisches unterhaus stimmt erneut gegen brexit-abkommen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> britisches unterhaus: keine mehrheiten für die acht brexit-alternativen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> brüssel setzt drohnen zur kontrolle der ausgangsbeschränkungen ein </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> buckingham palace: zentrum der britischen monarchie öffnet seine pforten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bund genehmigt nur wenig fördergelder zur sanierung von schwimmbädern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bund plant einmaligen zuschuss von bis zu 5000 euro für solo-selbstständige aus der kulturbranche </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bund und länder beschließen aufstockung des personals und gelder für gesundheitsämter </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bund und länder beschließen erste lockerungen in der corona krise </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bund und länder beschließen verlängerung des lockdowns </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bund und länder einigen sich auf eckpunkte für die grundsteuerreform </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bund und länder einigen sich auf strenge kontaktbeschränkungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bund und länder einigen sich auf strengere corona-regeln </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bund und länder einigen sich auf zielgerichtete maßnahmen bei lokalen corona-ausbrüchen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bund und länder einigen sich im streit über den digitalpakt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bund und länder erwägen harten lockdown bis weihnachten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bund und länder ringen um den kohleausstieg </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bund und länder streiten über verteilung der kosten für künftige corona-hilfen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bund und länder vereinbaren öffnungsstrategie </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bund und länder verlängern teil-lockdown </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bund zahlt überhöhten preis für verbilligte masken </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bund-länder-konferenz diskutiert verlängerung und auch verschärfung der corona-maßnahmen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bund-länder-treffen diskutiert corona-lockerungsstrategien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesanwaltschaft beschuldigt moskau des mordes an georgier </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesarbeitsgericht beschränkt sonderrechte kirchlicher arbeitgeber </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesarbeitsminister heil will recht auf home office </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesarbeitsminister heil will selbstständige zur altersvorsorge verpflichten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesarbeitsminister hubertus heil plant neues &quot;ausländerbeschäftigungsförderungsgesetz&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesaußenminister maas vermittelt im griechisch-türkischen gasstreit </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesaußenminister maas zu beratungen im iran </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesgesundheitsminister spahn befürwortet regionale corona-maßnahmen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesgesundheitsminister spahn ist russischem impfstoff nicht mehr gänzlich abgeneigt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesgesundheitsminister spahn positiv auf corona-virus getestet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesgesundheitsminister spahn will urlaubsrückkehrer aus risikogebieten zu corona-tests verpflichten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesgesundheitsminister spahn zur impfstoffzulassung in ganz europa </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundeshaushalt 2021 beschlossen: fast 180 millionen neue schulden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesinnenminister seehofer verliert afd-verfassungsklage </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesinnenminister seehofer verteidigt abschiebung von amri-helfer ben ammar </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesjustizministerin barley will mietpreisbremse erneut verschärfen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundeskabinett beschließt gutscheinlösung für ausfallende kunst- und kulturevents </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundeskabinett beschließt nachtragshaushalt mit milliardenhilfen wegen corona-epidemie </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundeskabinett verabschiedet  klimaschutzpaket </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundeskanzlerin merkel wirbt in regierungserklärung für solidarität und verteidigt einschnitte </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesländer beschließen verschärfungen der corona-maßnahmen für november </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesländer diskutieren über schulöffnungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesliga: frankfurt verliert gegen mainz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesligastart: die schwierige saison </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesminister scholz und altmaier vor finanzausschuss zu wirecard </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesparteien analysieren ergebnis der hamburger bürgerschaftswahl </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesparteitag der fdp: lindners wiederwahl zum chef </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesparteitag der grünen in bielefeld: parteichef habeck betont regierungswillen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundespolitische maßnahmen gegen den wohnungsmangel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundespolitische reaktionen zum wahlausgang </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundespolizei richtet kontrollstationen an den grenzen zu tirol und tschechien ein </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundespräsident steinmeier besucht israel zu holocaust-gedenken </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundespräsident steinmeier hält fernsehansprache zur corona-krise </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundespräsident steinmeier spricht mit angehörigen von corona-toten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundespräsident steinmeier wünscht &quot;lichtfenster&quot; in gedenken an corona-tote </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesrat lässt e-tretroller zu </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesrat stimmt für kohleausstieg bis spätestens 2038 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesrechnungshof wirft bund versäumnisse bei umgang mit bahn vor </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesregierung beschafft antikörper-medikament für corona-risikopatienten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesregierung diskutiert strategien gegen das coronavirus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesregierung holt erstmals kinder von is-anhängern zurück nach deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesregierung kündigt hilfe für benachteiligte &quot;brennpunktschulen&quot; an </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesregierung plant neues gesetz nach datendiebstahl </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesregierung plant quarantäne für einreisende und neue hilfen für den mittelstand </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesregierung rechnet mit möglicher ausbreitung des coronavirus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesregierung rechnet mit schwerster rezession der nachkriegszeit </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesregierung spricht reisewarnung für ganz spanien mit ausnahme kanaren aus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesregierung startet mit &quot;luftbrücke&quot; größte rückholaktion deutscher geschichte </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesregierung stellt flugverkehr aus großbritannien und südafrika wegen coronavirus-mutation ein </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesregierung streitet bei klimapläne um finanzen und windkraft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesregierung und lufthansa einigen sich auf rettungspaket </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesregierung verlängert reisewarnungen für nicht-eu-länder bis ende august </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesregierung verschärft kampf gegen geldwäsche </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesregierung weitet liste der risikogebiete aus: unsicherheit vor den herbstferien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesregierung will gegen systematische zerstörung von produkten vorgehen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesregierung will reisebeschränkungen verschärfen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesregierung will weitere 1553 flüchtlinge aus griechischen lagern aufnehmen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesschiedsgericht der afd bestätigt parteiausschluss von kalbitz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundessicherheitsrat genehmigt umstrittenen export von rüstungsrelevanten gütern nach saudi-arabien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundestag beschließt grundrente </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundestag beschließt neue diesel-grenzwerte </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundestag beschließt neue fassung des intensivpflegegesetzes </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundestag beschließt reform des umstrittenen &quot;erneuerbare-energien-gesetz&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundestag beschließt überführung von stasi-akten ins bundesarchiv </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundestag billigt haushalt 2021 mit 180 milliarden neuverschuldung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundestag debattiert corona-maßnahmen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundestag debattiert emotional über migrationspaket und beschließt sogenanntes &quot;geordnete-rückkehr-gesetz&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundestag debattiert kontrovers über neues fachkräfteeinwanderungsgesetz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundestag debattiert über abschaffung des blutspendeverbots für homosexuelle </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundestag debattiert über aufnahme von flüchtlingen aus moria </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundestag debattiert über umgang mit extremismus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundestag gedenkt der opfer des holocaust </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundestag stimmt über verschärfte fahndungsmöglichmöglichkeiten im kampf gegen cybergrooming ab </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesumweltministerin schulze konkretisiert co2-steuer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesverfassungsgericht hebt verbot von organisierter sterbehilfe auf </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesverfassungsgericht prüft hartz-iv-sanktionen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesverfassungsgericht sieht afd als rechtsextremen verdachtsfall </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesverfassungsschutz stuft afd als rechtsextremen verdachtsfall ein </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesweit höchste corona-inzidenz in sachsen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesweit wollen fast 140 kommunen weitere flüchtlinge aufnehmen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesweite diskussion über strengere regeln für weihnachten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bundesweiter personalmangel bei finanzbehörden: ver.di fordert mehr gehalt für mitarbeiter des öffentlichen dienstes </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bündnis für die bahn: wie nach der lufthansa nun auch die bahn gerettet werden soll </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bunter protest: landesweite demos gegen johnsons brexit-kurs </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bürger auf rügen zwischen sorge und zuversicht angesichts der lockerung der schutzmaßnahmen vor corona </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bürgerfest und rechte demo: chemnitz ein jahr nach der tödlichen messerattacke </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bürgerkrieg in äthiopien: präsident abiy ahmed schickt truppen gegen separatisten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bürgerschaftswahl in hamburg: die gewinner und verlierer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bürgerschaftswahl in hamburg: triumph für rot-grün </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> buschbrände in australien wüten weiter </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> buschbrände in australien: wut auf die regierung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> buschfeuer in australien breiten sich weiter aus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> busunglück auf madeira </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bverfg befasst sich mit befugnissen des bundesnachrichtendienstes </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bverfg-urteil: auskunft von &quot;bestandsdaten&quot;  für ermittler nur bei konkretem verdacht erlaubt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bverwg-urteil: fußballvereine können an kosten für polizeieinsätze bei hochrisikospielen beteiligt werden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> cameron teilt in seinem buch gegen johnson aus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> cannabis: schwieriger umgang in der medizin </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> cdu auf der suche nach einem partei-vorsitzenden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> cdu bleibt bei kommunalwahlen in nordrhein-westphalen stärkste kraft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> cdu in nrw: wie sich die basis die christdemokratische zukunft vorstellt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> cdu parteitag: akk will sachthemen in den fokus rücken </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> cdu steuert auf kampfkandidatur um parteivorsitz zu </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> cdu-chefin kramp-karrenbauer fordert regeln für die &quot;meinungsmache&quot; im netz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> cdu-klausur nach rücktrittsankündigung von nahles: kramp-karrenbauer und koalition unter druck </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> cdu-krise: diskussion um kramp-karrenbauer-nachfolge </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> cdu-parteichefin annegret kramp-karrenbauer zieht sich von parteispitze zurück </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> cdu-parteivorstand beschließt wahl des neuen vorsitzenden auf digital-parteitag </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> cdu-vorstandsklausur: auftakt zum schicksalsjahr </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> champions league </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> champions league: dortmund verliert gegen tottenham mit 3:0 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> champions-league </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> champions-league-debakel für schalke </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> championsleague: rb leipzig gegen basaksehir istanbul 4:3 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> chancen und risiken von investitionen in afrika </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> chansonniere juliette gréco gestorben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> chanukka als zeichen gegen antisemitismus in deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> chaos in venezuela: guaidó-rivale ernennt sich zum parlamentspräsidenten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> chefarzt gregor hilger über die dramatische lage in seiner klinik </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> chemnitz wird kulturhauptstadt 2025 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> china auf dem sprung zur großmacht im all </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> china ein jahr nach ausbruch der pandemie </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> china feiert 70. jahrestag der gründung der volksrepublik </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> china fördert surfen als olympische disziplin </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> china startet erste eigene mars-mission </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> china: 30 jahre nach tiananmen-massaker </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> china: wachstum statt umweltschutz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> china: wie deutsche in wuhan die lage wahrnehmen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> chinesische regierung hebt isolation für metropole wuhan auf </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> chinesische regierung setzt neues sicherheitsgesetz für hongkong in kraft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> chinesische regierung überwacht touristen mit spähsoftware </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> chinesischer volkskongress verabschiedet neues sicherheitsgesetz für hongkong </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> clan-krieg in berlin im kampf um drogenhandel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> clankriminalität in deutschland: wie weit reicht der einfluss der familien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> club-wm: tigres uanl gegen bayern münchen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> co2-grenzwerte für lkw </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> co2-preis: was kommt auf die verbraucher zu? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> co2-speicherung: comeback der ccs-technologie? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> codex manesse: sammlung mittelalterlicher lyrik wird mit polizeischutz nach mainz transportiert </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> comeback der schiene: reaktivierung stillgelegter zugstrecken? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> comicserie asterix: neues album zum sechzigsten jubiläum </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> container-havarie: flut spült fernseher an den strand von borkum </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona führt zu mehr wilderei durch hunger in südafrika </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona impf- und teststrategie der bundesregierung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona in europa: drastische maßnahmen in südwestlichen nachbarländern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona in frankreich: nächtliche ausgangssperre in paris und weiteren städten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona in indien: millionen mädchen leiden unter schulschließungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona in russland: kreative ideen in der isolation </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona in thailand fördert geschäfte um aberglaube </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona und die auswirkungen auf das leben der studierenden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona und die bundeswehr: wo die soldaten überall aushelfen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona versus karneval: der stille start in die närrische zeit </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona wütet in europa </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-angst: warum alle reiserückkehrer kostenlos getestet werden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-app luca soll kontaktformulare ersetzen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-ärger mit feiernden in der hamburger schanze </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-ausbruch bei erntehelfern in niederbayern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-ausbruch in göttingen: eine stadt und ihr sozialer brennpunkt am pranger </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-ausbruch in italien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-benefizkonzert &quot;one world: together at home&quot; sammelt spenden für helfer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-einschränkungen für junge menschen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-flashmob: deutschland musiziert &quot;ode an die freude&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-herbst: systematisches lüften soll im klassenzimmer vor coronaviren schützen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-hotspot hamm in westfalen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-hotspot landkreis tirschenreuth in bayern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-hotspots: die lage in sachsen spitzt sich weiter zu </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-infektion: britischer premier johnson auf intensivstation verlegt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-konjunkturpaket wird weiter verhandelt - dazu einschätzungen der wirtschaftsweisen veronika grimm </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-krise in ägypten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-krise sorgt für notstand in der häuslichen altenpflege </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-krise und rechtsextreme hetze im internet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-krise: blick nach italien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-krise: bund und ländern schränken öffentliches leben weiter ein </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-krise: diskussion um grenzöffnungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-krise: schwierige zeiten für die tafeln </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-krise: seeleute dürfen nicht von bord </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-listen in bars: datenkrake oder sinnvoll? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-lockdown in new york - kult-friseur kämpft ums überleben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-lockdown wird bis ende januar ausgeweitet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-lockdown zwischen verschärfung und lockerung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-massentests an kitas und schulen in hildburghausen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-maßnahmen spalten madrid </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-maßnahmen: parlamentarier kritisieren vollmachten der regierung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-maßnahmen: unruhen in frankreichs vorstädten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-neuinfektionen sinken nicht mehr: was bedeutet das für mögliche lockerungen? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-pandemie gefährdet betriebe und selbstständige </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-pandemie in brasilien: ultras in sao paulo vereint im protest und in solidarität </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-pandemie in den usa: in new york sind vor allem ärmste von wirtschaftlichen auswirkungen betroffen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-pandemie verschärft den machtkampf zwischen den usa und china </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-pandemie: debatte um grundrechte und lindner im sommerinterview </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-pandemie: die schwierige lage in afrika </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-pandemie: eu-staaten führen grenzkontrollen ein </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-pandemie: immer noch keine schnelltests </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-pandemie: kanzlerin und länderchefs für vorsichtige lockerungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-pandemie: klimaschutz in zeiten wirtschaftlicher krisen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-pandemie: koalitionsausschuss berät über hilfen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-pandemie: londons bürgermeister ruft katastrophenfall aus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-pandemie: situation an den kliniken spitzt sich zu </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-pandemie: wird homeoffice zur pflicht? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-pandemie: zwischenbilanz der reisebranche </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-regeln in den schulen: wie geht es weiter im winter? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-schutz bei spargel-erntehelfern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-schutz ohne isolation in pflegeheim nahe hamburg </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-schutzmaßnahmen für regionen mit mehr als 50 infizierten pro 100.000 einwohner </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-schutzmaßnahmen in tschechien offenbar erfolgreich </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-selbsttests in deutschen supermärkten ausverkauft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-shutdown auf hochseeinsel helgoland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-shutdown: wie sich das öffentliche leben verändert </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-situation in serbien: neuer lockdown und proteste </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-strategie: bundeskanzlerin merkel vereinbart neue corona-auflagen mit großstädten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-studie uni bochum: geringe ansteckungsgefahr durch kinder </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-tests: 46 in bayern getestete urlaubsrückkehrer wissen nichts von ihrer infektion </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-testwerte und ihre bedeutung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-ticker: reisen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-virus: experten stufen risiko für deutschland höher ein </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-virus: krisentreffen mehrerer eu-gesundheitsminister in rom </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona-virus: sorge um die patienten im pflegeheim in wolfsburg </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona: die missverständnisse zwischen wissenschaftlern und politikern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona: pläne für neustart in der fußball-bundesliga </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona: sind die lockerungen ein comeback der sorglosigkeit? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> corona: tschechien vor zweitem teil-lockdown </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> coronagefahr in entwicklungsländern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> coronavirus bedroht internationale lieferketten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> coronavirus in china: mensch-zu-mensch-infektion bestätigt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> coronavirus in österreich: zwischen skisaison und sammelklage </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> coronavirus in spanien: wie lebt es sich im ausnahmezustand? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> coronavirus in spanien: wurde alten menschen in altersheimen die behandlung verwehrt? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> coronavirus lähmt flugverkehr in frankfurt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> coronavirus und ausgangsbeschränkungen: was die einzelnen bundesländer planen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> coronavirus verbreitet sich in china und erreicht europa </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> coronavirus-impfstoff-test trifft in südafrika auf ablehnung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> coronavirus: ausgeflogene deutsche befinden sich in quarantäne </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> coronavirus: dramatische lage in wuhans krankenhäusern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> coronavirus: reportage aus dem infektionsgebiet in nrw </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> coronavirus: was bisher bekannt ist </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> coronavirus: who ruft internationalen gesundheitsnotstand aus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> coronavirus: wie bereiten sich die unternehmen vor? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> crewwechsel auf der neumayer-station in der antarktis </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> csu parteitag findet erstmalig virtuell statt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> csu-parteitag: söder wird wiedergewählt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> csu-vorstoß löst debatte über sondersteuer für billigflüge aus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> da es einen fehler bei der übertragung der meinung gab. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> da es im beitrag &quot;auswirkungen auf deutsche unternehmen bei einem no-deal-brexit&quot; einen insertfehler gab. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> da es im beitrag zu spanien einen insertfehler gab. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> da es im beitrag zum thema &quot;neue beschränkungen nach steigenden infektionszahlen in deutschland&quot; einen insertfehler gab. der beitrag zum thema &quot;formel 1&quot; darf aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> da es in der anmoderation einen versprecher gab. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> da in der fernsehausstrahlung um 22:30 uhr in dem beitrag aus paris die aussage einer protagonistin in einen missverständlichen bezug gesetzt wurde. der beitrag zur &quot;champions-league&quot; darf aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> daimler im diesel-skandal: der betrug nach dem betrug </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> daimler und volkswagen kündigen vorübergehende werksschließungen in europa an </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> daniel-knorr-ausstellung &quot;we make it happen&quot; in tübingen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> dänische schlachtbetriebe ohne corona-infektionen: was machen die dänen besser? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> dänischer koch spioniert in nordkorea </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> dannenröder forst und die grünen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> darf der beitrag zur fußball-bundesliga im internet nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das 750-milliarden-rettungspaket der eu: wiederaufbauplan für europa </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das coronavirus breitet sich aus: patienten in deutschland geht es gut </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das coronavirus verändert die usa </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das dilemma um irland und nordirland im brexit-fall </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das ende der &quot;großen einmütigkeit&quot;: fdp-chef lindner im tagesthemen-gespräch </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das ende der präsidentschaft von donald trump </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das funkloch-drama von nemsdorf-gröhndorf </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das gamestop-phänomen: kleinanleger organisieren sich gegen hedgefonds </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das leseland norwegen auf der frankfurter buchmesse </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das phänomen insta-travel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das problem der schrumpfenden gesellschaft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das revival des shanty durch einen schottischen postboten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das rote haus: dokumentarfilm über soziales projekt in ostgrönland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das schicksal der jesidische kinder </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das schwedische modell in der corona-krise ohne verbote </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das strache-video: was wir wissen und was nicht </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das super bowl-finale: sportereignis mit politischem beigeschmack </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das virus und die wirtschaftlichen auswirkungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das warten hat ein ende: welche hoffnungen in europa mit dem impfstart verbunden sind </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                     “der beitrag zum thema “angriffe auf polizei” darf aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden.” </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                     der beitrag zum thema &quot;liverpool gewinnt die champions-league&quot; darf aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                     die beiträge zum thema &quot;fußball-bundesliga&quot; und  &quot;formel 1&quot; dürfen aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                     die beiträge zur champions league und zum missbrauchsfall in bergisch gladbach dürfen aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                     diese sendung wurde nachträglich bearbeitet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                     diese sendung wurde wegen eines schreibfehlers nachträglich bearbeitet. in der fernsehausstrahlung der tagesthemen um 23:30 uhr stand irrtümlich &quot;vorsitzende&quot; anstelle von &quot;vorsitzender&quot; im beitrag zu südafrika von thomas denzel. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    aufgrund einer fehlerhaften einspielung bei der anmoderation in der fernsehausstrahlung wurde diese sendung nachträglich bearbeitet. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    aufgrund eines fehlers im beitrag zum thema &quot;vereidigung amy coney barrett&quot; wurde die sendung nachträglich bearbeitet. die bilder des beitrags zur fußball-champions-league dürfen aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    aus rechtlichen gründen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    ausschnitte aus dem beitrag &quot;seehofers plan&quot; und der beitrag zum thema “flüchtlingslager” dürfen aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    den beitrag zur fußball-bundesliga können wir aus rechtlichen gründen im internet nicht zeigen. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag &quot;besuch aus hollywood: dicaprio </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag &quot;clan-krieg in berlin&quot; wurde nachträglich redaktionell verändert </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag &quot;digitale hilfe für das leben im alter&quot; darf auf tagesschau.de aus rechtlichen nicht gründen werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag &quot;missbrauchsvorwürfe gegen prinz andrew&quot; darf aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag &quot;notre-dame&quot; wurde wegen eines insert-fehlers nachträglich bearbeitet. der beitrag &quot;geordnete-rückkehr-gesetz&quot; wurde um einen o-ton von linken-chef bernd riexinger ergänzt. die rufnummer der hotline-tafel am ende der sendung wurde nachträglich korrigiert. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag &quot;urlaub im sommer 2020&quot; in deutschland wurde nachträglich bearbeitet. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zu &quot;50 jahre robotron&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zu &quot;hitchcocks blonde schönheiten&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zu den dfb-entscheidungen darf aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zu den historischen beziehungen zwischen großbritannien und der eu darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zu den neuen bußgeldern darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zu den urlaubsrückkehrer darf aus rechtlichen gründen auf tagesschau.de nicht vollständig gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zu fussball-europa-league darf aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zu thema &quot;biathletin laura dahlmeier hört auf&quot; darf aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum &quot;bverfg-urteil&quot; wurde nachträglich bearbeitet und darf aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum &quot;gerüstabbau in notre-dame&quot; darf aus rechtlichen gründen im internet nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum &quot;kampf gegen krebs&quot; darf aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. der beitrag zum &quot;super bowl&quot; darf aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum &quot;letzten film von robert redford&quot; darf aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum &quot;neustart der bundesliga&quot; darf aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. diese sendung wurde nachträglich bearbeitet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum &quot;rechtsrock-festival&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum &quot;wassermangel in spanien&quot; darf aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum abkommen mit den taliban hatte einen fehler und musste daher vom netz genommen werden. die beiträge zum thema &quot;fußball-bundesliga&quot; dürfen auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum fußball-länderspiel deutschland gegen tschechien darf aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema &quot; hopman-cup&quot; darf aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema &quot;ankunft flüchtlingskinder aus griechenland&quot; darf aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema &quot;bilanz des neustarts&quot; darf aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema &quot;bundestrainer löw hört auf&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema &quot;champions league&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema &quot;champions league&quot; darf aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. die bilder in der live-schalte zu markus othmer wurden nachträglich ausgetauscht. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema &quot;championsleague: rb leipzig gegen basaksehir istanbul&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema &quot;dfb-testspiel&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema &quot;dfl-supercup&quot; darf aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema &quot;doping-sperre für russland: gerechte strafe oder zu mildes urteil?&quot; darf aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema &quot;erinnerung an opfer der progromnacht&quot; darf aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema &quot;europa league&quot; darf aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema &quot;formel 1&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema &quot;formel 1&quot; darf aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema &quot;fußball em-qualifikation&quot; darf aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema &quot;fußball-mannschaften paris und istanbul brechen champions-league-spiel nach rassistischem vorfall ab&quot; darf aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema &quot;groko-bilanz: arbeit und soziales&quot; wurde nachträglich bearbeitet. der beitrag darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema &quot;handball-champions-league finale: thw kiel gewinnt gegen barcelona&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema &quot;hockey-em&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema &quot;joachim löw bleibt fußball-bundestrainer&quot; darf aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema &quot;juwelenraub grünes gewölbe: drei festnahmen in berlin&quot; darf aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema &quot;katholische kirche&quot; darf aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema &quot;kriegsende vor 75 jahren&quot; darf aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema &quot;kundenrechte gestärkt - eugh erlaubt rückgabe von matratzen&quot; darf aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden.   diese sendung wurde nachträglich bearbeitet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema &quot;oscarverleihung&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema &quot;rettung durch insolvenz? - der fall 1. fc kaiserslautern&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema &quot;rückkehr der polarstern&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema &quot;sexueller missbrauch in der ddr&quot; darf aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema &quot;super bowl&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. diese sendung wurde wegen eines insertfehlers nachträglich bearbeitet. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema &quot;trauer um olivia de havilland&quot; darf aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema &quot;uefa europa league&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema &quot;umgang mit dem wolf&quot; darf aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema &quot;weltfußballer&quot; darf aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema “gewalt gegen polizisten” darf aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema “tesla-fabrik” darf aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema dirk nowitzki darf aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema europa-league darf aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema fußball-bundesliga darf aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. wegen falscher ortsangabe in der anmoderation zum rücktritt von felix neureuther ist die sendung in einer nachbearbeiteten fassung online sichtbar. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema kita-öffnungen darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum thema krawalle in leipzig darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum tod von hannelore elsner darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zum tod von niki lauda darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zur &quot;champions-league&quot; darf aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zur &quot;fußball-bundesliga&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zur &quot;fußball-bundesliga&quot; darf aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zur &quot;handball-em&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zur &quot;querdenken&quot;-demo darf aus rechtlichen gründen nicht vollständig gezeigt werden. der beitrag zum champions-league-finale der frauen darf aus rechtlichen gründen im internet nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zur &quot;tour de france&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zur &quot;tour de france&quot; darf aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zur berlinale darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zur champions league am ende der sendung darf aus rechtlichen gründen nicht vollständig gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zur champions league darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zur champions league darf aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zur champions-league darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zur eishockey-wm darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zur formel 1 darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zur formel 1 darf aus rechtlichen gründen auf tagesschau.de nicht gezeigt werden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zur fußball-bundesliga bielefeld gegen bremen darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zur fußball-em-qualifikation darf aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beitrag zur polarexpedition darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der beiträge zur fußball-bundesliga und zur formel-1 dürfen aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    der sendemitschnitt wurde nachträglich korrigiert </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die anmoderation zum thema &quot;sprung in die freiheit&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beitrag zum thema coronavirus in der fleischindustrie wurde korrigiert </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beitrag zur fußball-bundesliga darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zu den protesten gegen steigende mieten und zur fußball-bundesliga dürfen aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zu den themen &quot;corona-lage in wintersportgebieten&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zu den themen &quot;corona-lage in wintersportgebieten&quot; sowie &quot;fußball-bundesliga&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zu den themen &quot;fußball bundesliga&quot; und &quot;olyimpia-qualifikation im handball&quot; dürfen aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zu den themen &quot;fußball-bundesliga&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zu den themen &quot;fußball-bundesliga&quot; und &quot;formel 1&quot; dürfen auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zu den themen &quot;fußball-bundesliga&quot; und &quot;tour de france&quot; dürfen aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zu den themen &quot;impfen in sachsen&quot; und &quot;handball-wm&quot; können auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zu den themen dfb-pokal </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zum &quot;fußball supercup&quot; und zur  &quot;formel 1&quot; dürfen auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zum &quot;super-bowl&quot; und zur &quot;fußball bundesliga&quot; darf aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zum drohnenangriff auf die ölraffinerie in saudi arabien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zum sport dürfen aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zum thema &quot;afd-bundesvorstand zu kalbitz&quot; und &quot;fußball-bundesliga&quot; dürfen aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. wegen eines insertfehlers wurde diese sendung nachträglich bearbeitet. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zum thema &quot;frauen-fußball-wm&quot; und &quot;em-qualifikation herren&quot; dürfen aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zum thema &quot;fussball bundesliga&quot; und &quot;formel 1&quot; dürfen aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zum thema &quot;fußball-bundesliga&quot; dürfen aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zum thema &quot;fußball-bundesliga&quot; dürfen aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zum thema &quot;fußball-bundesliga&quot; dürfen aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. diese sendung wurde nachträglich bearbeitet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zum thema &quot;fußball-bundesliga&quot; und &quot;formel 1&quot; dürfen aus rechtlichen gründen auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zum thema &quot;fußball-bundesliga&quot; und &quot;formel 1&quot; dürfen aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zum thema &quot;fußball-bundesliga&quot; und &quot;tennis fed cup&quot; dürfen aus rechtlichen gründen auf tagesschau.de nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zum thema &quot;nations league&quot; und &quot;formel 1&quot; dürfen aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zum thema &quot;wimbledon&quot; sowie zur &quot;formel 1&quot; dürfen aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zum thema fußball-bundesliga dürfen auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zur &quot;basketball-bundesliga&quot; und der &quot;u21-em&quot; dürfen auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zur &quot;formel 1&quot; und  &quot;wimbledon&quot; dürfen auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zur &quot;fußball bundesliga&quot; dürfen aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zur &quot;fußball bundesliga&quot; und &quot;eishockey-wm&quot; dürfen auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. die sendung wurde aus redaktionellen gründen nach der ausstrahlung bearbeitet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zur &quot;fußball-bundesliga&quot; und zum &quot;biathlon der frauen&quot; dürfen aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zur &quot;fußball-bundesliga&quot; und zur &quot;formel 1&quot; dürfen aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zur &quot;fußball-bundesliga&quot; und zur &quot;formel 1&quot; dürfen aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. diese sendung wurde wegen eines inhaltlichen fehlers nachträglich bearbeitet. bei der ausstrahlung um 22:45 uhr war im beitrag zur wahl in georgia irrtümlich davon die rede </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zur &quot;fußball-bundesliga&quot; und zur &quot;handball-wm&quot; dürfen aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zur &quot;fußball-bundesliga&quot; und zur &quot;hockey-em&quot; dürfen aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zur &quot;u21-fußball-em&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zur basketball- und fußball-bundesliga dürfen aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zur berlinale und zum tod von rosamunde pilcher dürfen aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zur berlinale und zur fußball-bundesliga und zur berlinale dürfen aus rechtlichen gründen auf tagesschau.de nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zur formel 1 und der tour de france dürfen auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zur fußball bundesliga dürfen aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zur fußball bundesliga und eishockey-wm dürfen auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zur fußball-bundesliga dürfen aus rechtlichen gründen im internet nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zur fußball-bundesliga dürfen aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. der beitrag zum tod des künstlers christo darf aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zur fußball-bundesliga und handball-em-qualifikation dürfen aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zur fußball-bundesliga und zum atp-turnier dürfen aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zur fußball-bundesliga und zum frauenfußball dürfen auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zur fußball-bundesliga und zum thema &quot;streit in der afd&quot; dürfen aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die beiträge zur fußball-em-qualifikation </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die bilder der beiträge der themen &quot;fußball-bundesliga&quot; und &quot;handball-em&quot; dürfen auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die bilder des beitrags &quot;fc bayern im finale der champions league&quot; dürfen aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die bilder des beitrags &quot;mit alkoholverkaufsverbot gegen partymeilen&quot; von peter kleffmann dürfen aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die bilder des beitrags &quot;vor dem treffen der ministerpräsidenten zur öffnungsstrategie&quot; von michael stempfle dürfen aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die bilder des beitrags zur fußball-bundesliga dürfen auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die bilder zu den themen &quot;fortnite wm in new york&quot; sowie &quot;atp turnier&quot; in hamburg dürfen auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die bilder zum beitrag der fußball-bundesliga dürfen aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die bilder zum beitrag über die fußball-bundesliga und zum basketball supercup dürfen aus rechtlichen gründen nicht gezeigt werden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die bilder zur &quot;handball-bundesliga&quot; und &quot;formel 1&quot; dürfen aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die bilder zur &quot;tour de france&quot; dürfen aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die bilder zur fußball-bundesliga dürfen aus rechtlichen gründen online nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die bilder zur fußball-bundesliga und zum radsport dürfen aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die bilder zur fußball-championsleague dürfen aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die sendung musste wegen eines fehlers in der anmoderation zum eu-sondergipfel nachträglich korrigiert werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die sendung musste wegen eines technischen fehlers nachträglich korrigirt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die sendung wurde aufgrund eines insertfehlers im letzten beitrag nachträglich verändert. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die sendung wurde aufgrund eines logo- und insert-fehlers im sportblock nachträglich bearbeitet.die bilder zum thema &quot;fußball bundesliga&quot; und &quot;super-bowl&quot; dürfen auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die sendung wurde nachträglich bearbeitet aufgrund von fehlerhaften namenseinblendungen. außerdem darf der beitrag über das europa league spiel wolfsburg gegen donezk aus rechtlichen gründen nicht gezeigt werden und wurde daher überblendet. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die sendung wurde nachträglich bearbeitet. aus inhaltlichen gründen wurde die anmoderation zum bundeswehr-beitrag entfernt. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die sendung wurde nachträglich bearbeitet. im &quot;iranischer angriff im irak&quot;  wurde der o-ton eines politikwissenschaftlers entfernt. im beitrag zum thema &quot;weltweit erster 3d-scanner für insekten&quot; wurde ein insertfehler korrigiert. außerdem darf der beitrag zu &quot;harry und meghan&quot; auf tagesschau.de aus rechtlichen gründen nicht vollständig gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die sendung wurde nachträglich bearbeitet. im beitrag zum thema &quot; lukaschenko lässt vor wahl wichtigsten herausforderer verhaften&quot; wurden die datumsangaben vervollständigt. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die sendung wurde nachträglich korrigiert </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die sendung wurde nachträglich korrigiert. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die sendung wurde wegen eines bildfehlers während des kommentars zum kohleausstieg nachträglich bearbeitet. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die sendung wurde wegen eines inhaltlichen fehlers in der anmoderation zum beitrag von sven lohmann zum urteil gegen die auslieferung von julian assange nachträglich bearbeitet. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    die sendung wurde wegen technischer pannen nachträglich bearbeitet. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    diese sendung wurde aus persönlichkeitsrechten nachträglich bearbeitet. der beitrag zur handball-wm darf aus rechtlichen gründen auf tagesschau.de nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    diese sendung wurde bearbeitet. der beitrag &quot;wie fluggesellschaften ihre kunden hängen lassen&quot; darf aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    diese sendung wurde nach der ausstrahlung im fernsehen bearbeitet. es gab im beitrag zum absturz der boeing im iran ein tonfehler und im beitrag zu den &quot;grünen&quot; ein fehler bei der einblendung einer jahreszahl. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    diese sendung wurde nachbearbeitet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    diese sendung wurde nachträglich bearbeitet. im beitrag &quot;sobibor-fotos&quot; wurden fehlende quellenangaben ergänzt. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    diese sendung wurde nachträglich bearbeitet. in der fernsehausstrahlung vom 4.1.19 um 21:45 uhr war im beitrag zum &quot;datenklau&quot; ein link sichtbar. dieser wurde nachträglich unkenntlich gemacht. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    diese sendung wurde wegen eines inhaltlichen fehlers nachträglich bearbeitet. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    diese sendung wurde wegen eines insert-fehlers nachträglich bearbeitet. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    diese sendung wurde wegen eines insertfehlers redaktionell bearbeitet. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    diese sendung wurde wegen eines schreibfehlers nachträglich bearbeitet. in der fernsehausstrahlung stand in einer bauchbinde zu den protesten in den usa irrtümlich &quot;episkopat-kirche&quot; statt &quot;episkopal-kirche&quot; und &quot;trumpf&quot; statt &quot;trump&quot;. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    diese sendung wurde wegen eines schreibfehlers nachträglich bearbeitet. in der fernsehausstrahlung um 22:35 uhr stand in der unterzeile zum literatur-nobelpreis irrtümlich nobelkomittee statt nobelkomitee. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    diese sendung wurde wegen eines technischen fehlers nachträglich redaktionell bearbeitet. in der fernsehausstrahlung um 22:15 uhr fehlte in der moderation vor dem beitrag von stephan stuchlik eine grafik. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    diese sendung wurde wegen eines übersetzungsfehlers im beitrag &quot;einigung über corona-finanzhilfen&quot; nach der ausstrahlung bearbeitet. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    diese sendung wurde wegen wahrung von persönlichkeitsrechten nachträglich bearbeitet. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    im beitrag zu ukrainischen leihmüttern wurde eine person nachträglich unkenntlich gemacht </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    in dem beitrag über die neuordnung der maklerkosten beim immobilienkauf ist uns eine ungenauigkeit unterlaufen. bei der fiktiven beispielrechnung zu einem hauskauf in hessen müsste der letzte posten der nebenkosten &quot;notargebühr + grundbucheintrag&quot; heißen statt nur &quot;notargebühr&quot;. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    irrtümlich wurde boris pistorius mit einem falschen namens-insert versehen. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    wegen eines bildfehlers im beitrag &quot;wiedereröffnung notre dame&quot; bei der fernsehausstrahlung um 22:30 uhr wurde diese sendung nachträglich bearbeitet. der bericht erreichte uns nach der sendung in besserere qualität und wurde ausgetauscht. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    wegen eines insert-fehlers im beitrag zum &quot;corona-flashmob&quot; musste die sendung nachträglich bearbeitet werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
                                
                                    hinweis:
                                    wegen inhaltlicher fehler in den beiträgen zu den themen &quot;geplante eu-asylreform&quot; und &quot;trauer um gwisdek&quot; wurde die sendung nachträglich bearbeitet. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter

                                
                                    hinweis:
                                          diese sendung wurde wegen eines schreibfehlers nachträglich bearbeitet. in der fernsehausstrahlung um 23:05 uhr stand in der illustration zu dem prozess gegen leyla güven irrtümlich &quot;gestern&quot; statt &quot;archiv&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter

                                
                                    hinweis:
                                    der beitrag zum thema &quot;formel 1&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter

                                
                                    hinweis:
                                    der beitrag zum thema &quot;formel-1&quot; darf aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter

                                
                                    hinweis:
                                    der beitrag zum thema &quot;fußball-bundesliga&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter

                                
                                    hinweis:
                                    der beitrag zum thema &quot;fußball-bundesliga&quot; darf aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter

                                
                                    hinweis:
                                    der beitrag zum thema &quot;studie zeigt psychische folgen der corona-krise bei kindern&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter

                                
                                    hinweis:
                                    der beitrag zur &quot;fußball-bundesliga&quot; darf aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter

                                
                                    hinweis:
                                    die beiträge zu den themen &quot;fußball bundesliga&quot; und &quot;hockey-em 2019&quot; dürfen aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter

                                
                                    hinweis:
                                    diese sendung wurde nachträglich bearbeitet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter


                                
                                    hinweis:
                                    der beiträge zur &quot;fußball-bundesliga&quot; und &quot;bahnrad-em&quot; dürfen aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. die sendung wurde nachträglich bearbeitet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter


                                
                                    hinweis:
                                    sendung wegen insertfehler im beitrag &quot;schutzausrüstung&quot; nachträglich geändert </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter




spatenstich in frankfurt: dfb baut neue talentschmiede </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter

02 kirchentag gegen rechts </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter

die beiträge zur fußball-bundesliga dürfen aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter
(der beitrag zum thema &quot;club-wm&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden)
                                
                                    hinweis:
                                    (der beitrag zum thema &quot;club-wm&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden) </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter 
                                
                                    hinweis:
                                    der beitrag zu &quot;fridays for future&quot; wurde aus redaktionellen gründen nach der ausstrahlung bearbeitet. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter 
                                
                                    hinweis:
                                    der beitrag zum thema &quot;europa league&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter 
                                
                                    hinweis:
                                    der beitrag zum thema &quot;fußball bundesliga&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter 
                                
                                    hinweis:
                                    der beitrag zum thema &quot;fußball-bundesliga&quot; darf aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter 
                                
                                    hinweis:
                                    der beitrag zum thema &quot;goldes globes-verleihung&quot; darf aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter 
                                
                                    hinweis:
                                    der beitrag zum thema &quot;schweinsteiger beendet karriere&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter 
                                
                                    hinweis:
                                    der beitrag zum thema &quot;schwimm-wm&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter 
                                
                                    hinweis:
                                    der beitrag zum thema fußball-bundesliga darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter 
                                
                                    hinweis:
                                    der beitrag zur &quot;formel 1&quot; darf aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter 
                                
                                    hinweis:
                                    der beitrag zur &quot;u21-fußball-em&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter 
                                
                                    hinweis:
                                    der beitrag zur bundesliga darf aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter 
                                
                                    hinweis:
                                    die beiträge zu den themen &quot;dfb-pokal&quot; und &quot;us-open&quot; dürfen auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter 
                                
                                    hinweis:
                                    die beiträge zu den themen &quot;frauen-fußball-wm&quot; und &quot;atp stuttgart&quot; dürfen auf tagesschau.de aus rechtlichen gründen nicht vollständig gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter 
                                
                                    hinweis:
                                    die sendung wurde nachträglich bearbeitet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter 

                                
                                    hinweis:
                                    der beitrag zum thema bildung wurde aus rechtlichen gründen hinterher bearbeitet. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter 

                                
                                    hinweis:
                                    diese sendung wurde wegen eines insertfehlers nach der ausstrahlung redaktionell bearbeitet. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter 
(der beitrag zum thema &quot;fußball-bundesliga&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden.)

                                
                                    hinweis:
                                    der beitrag zum thema &quot;fußball-bundesliga&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter  
                                
                                    hinweis:
                                    der beitrag zum thema &quot;70 jahre grundgesetz&quot; darf aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter  
                                
                                    hinweis:
                                    der beitrag zum thema &quot;uefa champions league&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter  
                                
                                    hinweis:
                                    die beiträge zur berlinale und zur fußball-bundesliga dürfen auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter   
                                
                                    hinweis:
                                    die beiträge zur fußball-bundesliga dürfen aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter    
                                
                                    hinweis:
                                    der beitrag zum thema &quot;fußball-bundesliga&quot; darf aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter (der beitrag zum thema &quot;fußball-bundesliga&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden.)

                                
                                    hinweis:
                                    (der beitrag zum thema &quot;fußball-bundesliga&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden.) </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter (der beitrag zum thema &quot;fußball-bundesliga&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden.)

                                
                                    hinweis:
                                    der beitrag zum thema &quot;fußball-bundesliga&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter (die beiträge zu den themen &quot;meinungsverschiedenheiten im britischen königshaus&quot; und &quot;fußball-bundesliga&quot; dürfen auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden.)
                                
                                    hinweis:
                                    (die beiträge zu den themen &quot;meinungsverschiedenheiten im britischen königshaus&quot; und &quot;fußball-bundesliga&quot; dürfen auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden.) </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wetter [die beiträge zur fußball-bundesliga und zum super bowl dürfen aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden] </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> das wunder von bremen: werder rettet sich in die relegation </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> dass die republikaner für eine mehrheit beide sitze gewinnen müssten. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> dass es sich um ein archivbild der cdu-abgeordneten handelte. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> dass rouhani nach biarritz gereist war. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> dass syrer an einer massenschlägerei mit tschetschenen in reinsberg beteiligt waren. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> datenauswertung:wie sehr werden die ausgangsbeschränkungen eingehalten? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> datendiebstahl: forderungen nach politischen konsequenzen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> datenklau bei ärzten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> datenklau: hacker  veröffentlicht private informationen von politikern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> datenreport: corona verschärft soziale ungleichheit in deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> datenschutz in corona-zeiten: überall daten hinterlassen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> dauercamper zurück in mecklenburg-vorpommern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ddr-alltag auf schmalfilm: &quot;open memory box&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ddr-spione: amnestie stand 1990 kurz bevor </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deal oder no deal: tausende jobs könnten finanzplatz london verlassen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> debatte nach unruhen bei corona-protesten in leipzig </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> debatte über aufstockung des kurzarbeitergelds </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> debatte über ausbildungseinsatz der bundeswehr im irak </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> debatte über das eu-türkei-abkommen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> debatte über erneute verschärfung der corona-maßnahmen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> debatte über härteren lockdown aufgrund vieler corona-neuinfektionen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> debatte über herabsetzung der strafmündigkeit </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> debatte über höhere strafen bei kindesmissbrauch </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> debatte über islamistischen terror in deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> debatte über lockerung der corona-kontaktsperre in deutschen schulen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> debatte über lockerung der schutzmaßnahmen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> debatte über neue corona-maßnahmen geht weiter </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> debatte über öffnung der schulen wird fortgesetzt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> debatte über organspenden: das lange warten auf ein spenderorgan </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> debatte über reisebeschränkungen und beherbergungsverbot </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> debatte über schulstreik: klimaaktivistin gretathunberg demonstriert mit hamburger schülern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> debatte über trisomie-bluttest bei schwangeren </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> debatte über umgang mit deutschen risiko-gebieten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> debatte über umgang mit in syrien gefangenen deutschen is-kämpfern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> debatte über untersuchungsausschuss zur pkw-maut </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> debatte über vollendung von erdgasleitung zwischen russland und deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> debatte um bundesweit einheitliche corona-regelungen für schulen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> debatte um corona-maßnahmen: wird die quarantänezeit verkürzt? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> debatte um flüchtlingskinder: habeck-vorschlag und die reaktionen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> debatte um frauenquote in vorständen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> debatte um is-rückkehrer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> debatte um lockerungen der corona-auflagen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> debatte um raubkunst aus afrika </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> debatte um schuldfrage nach stürmung von us-kongress durch trump-anhänger </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> debatte um sicherheit nach tödlicher attacke in frankfurt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> debatte um umgang mit straffälligen asylbewerbern: bundesinnenminister seehofer will asylrecht verschärfen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> debatte zum corona-impfpass </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> debatte zwischen bund und länder um lockerung der corona-auflagen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> debatte: kevin kühnert erntet kritik nach darstellung seiner vorstellung zum sozialismus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deeskalation von nato und bundeswehr: truppen werden in nachbarländer verlegt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> demokratie im sudan: bundespräsident steinmeier verspricht unterstützung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> demokratie-projekte vor dem aus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> demonstration für mehr umwelt-und tierschutz in der landwirtschaft in berlin </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> demonstration in hannover nach npd-hetze gegen journalisten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> demonstrationen gegen nationalismus in europa </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> demonstrationen gegen staatliche regeln zur bekämpfung des coronavirus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> demonstrationen in philippsburg: pro und contra atomkraft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> depression durch die corona-lage </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> der alltag in zeiten von corona-virus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> der attentäter von hanau: frühe warnsignale </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> der beinahe-krieg: us-präsident stoppt nach eigenen angaben angriff auf iran </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> der berg ruft - vielleicht die falschen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> der deutschlandtrend über die bewertung der großen koalition </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> der deutschlandtrend zu fahrverboten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> der einfluss irans auf den irak </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> der erschwerte kampf gegen depressionen in zeiten der corona-pandemie </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> der fall deniz yücel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> der fall max emden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> der fall söring </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> der fall walter lübke: tatverdächtiger gesteht mord an politiker </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> der flugzeugparkplatz in teruel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> der kampf gegen aids in kenia </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> der kampf geht weiter: zweites klimacamp in pödelwitz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> der kleine ost-sandmann feiert 60-jahre-jubiläum </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> der komemntar </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> der kommenmtar </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> der leda-code: forscher enträtseln meisterwerk aus leonardo da vincis werkstatt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> der politische patient: nawalny in der berliner charité </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> der schöne schein: 10 jahre instagram </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> der sport
                                
                                    hinweis:
                                    die beiträge zur fußball-bundesliga können wir aus rechtlichen gründen im internet nicht zeigen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> der sport am samstag </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> der tabubruch von thüringen zeigt zerrissenheit der politischen parteien auf </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> der untergang der &quot;wilhelm gustloff&quot; vor 75 jahren </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> der versöhner: friedensnobelpreis für äthiopischen ministerpräsidenten abiy ahmed </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> der wald stirbt: was können wir tun? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> der weg zurück ins leben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> der werdegang ursula von der leyens </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> der zweite lockdown: israel steht still zum neuen jahr </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> designer colani im alter von 91 jahren gestorben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> designierte eu-präsidentin von der leyen stellt kandidaten für neue eu-kommission vor </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutliche mehrheit im bundestag für zustimmungslösung in der organspende-debatte </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutsch-französische drohnen-entwicklung: streitpunkte bei der bewaffnung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutsch-luxemburgische grenze ist wieder geöffnet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutsche bahn unterzeichnet rahmenplan zur verbesserung  des fahrplan-netzes </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutsche bahn will über die feiertage angebot deutlich ausweiten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutsche bank und commerzbank führen gespräche über mögliche fusion </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutsche bischofskonferenz: massenaustritte aus der katholischen kirche </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutsche einheit: wie sich ost und west an der grenze gegenüber standen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutsche forscher in antarktis beobachten klimawandel am beispiel der kaiserpinguine </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutsche handballer feiern em-auftaktsieg </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutsche impfstrategie in der kritik </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutsche industrie vor brexit </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutsche lehrkräfte berichten zunehmend über klima der einschüchterung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutsche reaktionen auf die us-wahl </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutsche sportvereine im lockdown geraten in existenznot </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutscher bundestag feiert jubiläum: erste sitzung des parlaments vor 70 jahren </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutscher ethikrat spricht sich gegen corona-immunitätsausweise zum jetzigen zeitpunkt aus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutscher film &quot;und morgen die ganze welt&quot; beleuchtet die &quot;antifa&quot; auf den filmfestspielen in venedig </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutscher pflegetag: beratungen auch über pflegende angehörige </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutscher tanzpreis für friedemann vogel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutscher us-häftling söring kehrt nach über 30 jahren nach deutschland zurück </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutsches forschungsschiff &quot;polarstern&quot; nach einjähriger arktis-expedition zurückgekehrt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutsches handball-team verliert bei weltmeisterschaft in ägypten gegen ungarn mit 28:29 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutschland beendet fußball-em-qualifikation als gruppensieger </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutschland bereitet corona-impfstationen vor </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutschland fällt in sachen breitbandausbau zurück </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutschland im handball-fieber </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutschland im lockdown: leere innenstädte </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutschland im schneechaos </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutschland steht im finale der u21-fußball-europameisterschaft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutschland steht still </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutschland übernimmt die eu-ratspräsidentschaft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutschland und eu kritisieren us-sanktionen gegen nord stream 2 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutschland und frankreich starten initiative zur aufnahme minderjähriger flüchtlinge </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutschland und frankreich wollen mit neuem freundschaftsvertrag näher zusammenrücken </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutschland und indien vereinbaren engere zusammenarbeit bei zukunftsthemen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutschland will am flüchtlingsabkommen mit der türkei festhalten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutschlands föderalismus kritisch betrachtet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutschlands gülle-dilemma: bauern lehnen strengere düngevorschriften ab </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutschlandtrend extra zum kandidatenrennen um den cdu-vorsitz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutschlandtrend extra: wachsende sorge vor ansteckung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutschlandtrend zur flüchtlingssituation und zu maßnahmen gegen das coronavirus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutschlandtrend: blitzumfrage zu den geschehnissen in erfurt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutschlandtrend: corona-pandemie </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutschlandtrend: ost-west </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutschlandweit protestieren zehntausende gegen steigende mieten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deutschlandweite proteste gegen polizeigewalt und rassismus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> dfb-auswahl startet mit 3:2 in den niederlanden in em-qualifikation </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> dfb-elf verschenkt sieg gegen serbien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> dfb-frauen starten wm-mission </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> dfb-funktionäre wegen vorwurf der arglistigen täuschung vor gericht </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> dfb-pokal: leverkusen empfängt frankfurt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> dfb-präsident grindel tritt zurück </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> dfb-testspiel: deutschland gegen argentinien 2:2 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> dfb-zentrale wegen verdachts auf steuerhinterziehung durchsucht </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> dfl beschließt unterbrechung der profi-fußballsaison bis 2. april </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> dfl hofft auf spiele vor zuschauern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> dfl-supercup: fc bayern gewinnt gegen bvb </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die &quot;fette &quot;elke&quot;: ein mobiles lokal bringt den osten zum tanzen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die &quot;grüne mauer&quot; ist afrikas projekt gegen wüstenbildung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die &quot;instagram-brücke&quot; im österreichischen zillertal </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die &quot;sendung mit der maus&quot; feiert 50. geburtstag </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die affäre cummings und was sie für boris johnson bedeutet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die angst der gastronomen vor dem corona-shutdown </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die cdu und die frage zum umgang mit der linkspartei </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die corona-app im praxistest </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die corona-maßnahmen könnten eine pleitewelle im einzelhandel auslösen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die die redaktion nicht zeigen wollte. die bilder wurden deshalb ausgetauscht. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die digitale revolution auf dem land hat begonnen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die entdeckung der ostdeutschen heimat </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die ethische zwangslage der ärzte in krankenhäusern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die flüchtlinge in ciudad juarez </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die geschichte der kommandeurin anastasia biefang </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die geschichte des adventskalenders </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die helden des alltags in der corona-krise </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die jecken sind los: hunderttausende feiern rosenmontag </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die katholische kirche und ihr umgang mit dem thema missbrauch </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die koalition und ihr corona-konjunkturpaket </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die komplizierte suche nach einem impftermin </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die krise der spd </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die lage an deutschen häfen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die lage an deutschen kliniken in zeiten der corona-krise </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die lage der kurden nach beginn der türkischen offensive in syrien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die lage im iran und aktuelle entwicklungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die linke im wahljahr 2019 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die linke wählt neue fraktionsspitze </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die linke: das dilemma der partei im osten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die macht der straße - fridays for future-streiks gehen um die welt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die mark-brandenburg ehrt ihren größten dichter im &quot;fontanejahr&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die misere der musikbranche in zeiten von corona </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die nachrichten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die neue lust am demonstrieren: &quot;fridays for future&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die neue rolle der gesundheitsämter </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die neuen influencer in den sozialen medien:petfluencer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die niedrigzinspolitik und ihre folgen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die not der gastronomen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die not der kulturschaffenden während der corona-krise </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die not des einzelhandels und der gastronomie in der corona-krise </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die rheinland-pfälzische ampelkoalition als modell für bundestagswahlen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die rolle der hisbollah im libanon </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die rolle der schiitischen milizen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die russische sicht auf den mauerfall </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die schalltote kammer mitten in tokio </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die schwierige lage der frauen im iran </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die silvesternacht </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die situation vor den landtagswahlen in baden-württemberg und rheinland-pfalz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die stimmung in der spd vor ihrem parteitag </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die stimmungslage vor dem lockdown </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die über wasser laufen - ein foto und seine geschichte </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die umweltdiskussion aus sicht deutscher landwirte </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die weibliche berlinale: mehr frauen im festivalprogramm </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die weiteren nachrichten im überblick </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die welt der &quot;proud boys&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die wetter </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die wirtschaftliche lage in nordkorea </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die zukunft der eu ohne großbritannien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> die zukunft der eu-bürger in großbritannien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> diebe stehlen 95 schmuckstücke im grünen gewölbe in dresden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> diesel-abgasskandal: die ausgebremste hardware-nachrüstung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> diesel-fahrverbote in stuttgart: das chaos am 1. werktag nach inkrafttreten bleibt aus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> diesel-fahrverbote: wie gefährlich sind die abgase wirklich? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> dieselskandal: musterfeststellungsklage gegen vw </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> differenzen beim g20-gipfel in osaka </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> digitale geberkonferenz für afghanistan </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> digitale hilfe für das leben im alter: virtuell betreutes wohnen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> digitale itb: das geschlossene steigenberger hotel in zingst und die not der regionalen zulieferer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> digitale itb: wie sieht es aus auf mallorca? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> digitale kommunikation als helfer in der corona-krise </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> digitale nähe kommunal: videokonferenz zwischen bürgern in bühl </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> digitale revolution: wohin treiben internet- und techfirmen unsere gesellschaft? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> digitale wüste: die nicht-gerüsteten schulen und lehrer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> digitalisierung: gemischte bilanz der großen koalition </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> diplomatie am schlachtfeld: trump und macron am d-day in der normandie </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> diskriminierte brasilianische indigene leiden besonders unter corona </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> diskussion in der cdu um parteivorsitz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> diskussion über aufnahme von flüchtlingen nach brand im griechischen lager moria </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> diskussion über berliner mietendeckel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> diskussion über cdu-parteitag im dezember </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> diskussion über corona-maßnahmen vor bund-länder-beratungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> diskussion über corona-schnelltests </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> diskussion über deutschen alleingang bei flüchtlingen aus griechischen lagern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> diskussion über impfpflicht für pflegepersonal </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> diskussion über lockerungen in den usa </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> diskussion über lockerungen und impfreihenfolge vor bund-länder-konferenz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> diskussion über mehr homeoffice </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> diskussion über private seenotrettung im mittelmeer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> diskussion über privilegien für geimpfte </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> diskussion über verlängerung des lockdowns </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> diskussion über verschärfung der corona-maßnahmen vor bund-länder-beratungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> diskussion über weihnachtsgottesdienste </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> diskussion um &quot;stammbaumforschung&quot; nach krawallen in stuttgart </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> diskussion um ausgangssperren </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> diskussion um berlins verfehlte wohnungspolitik der vergangenheit </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> diskussion um corona-schnelltests </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> diskussion um digitalwährung: wie gefährlich ist libra? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> diskussion um gendergerechte sprache </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> diskussion um geplante abschiebung mutmaßlicher is-anhänger aus türkei nach deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> diskussion um gesundheitsminister spahns vorschlag zum immunitätsausweis </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> diskussion um höhere mehrwertsteuer für fleisch </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> diskussion um messstationen für stickstoffdioxidwerte </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> diskussion um neuregelung der organspende </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> diskussion um organspende </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> diskussionen über cdu-vorsitz und kanzlerkandidatur </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> diskussionen um die vereinbarte notbremse bei hohen inzidenzwerten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> diskussionen um impfstrategie nach astrazeneca-impfstoff </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> dokumentarfilm &quot;nur sag es niemandem&quot; erhöht druck auf katholische kirche in polen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> dokumentarfilm über den vor zehn jahren gestorbenen regisseur christoph schlingensief </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> domspatzen: ende einer aufarbeitung von missbrauch und gewalt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> donald trump kritisiert auszählprozess </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> doping für den perfekten körper </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> doping im ski nordisch: festnahmen bei wm </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> doping-sperre für russland: gerechte strafe oder zu mildes urteil? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> drachenbauer von kairo </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> dramatische lage für flüchtlinge in bosnien: hunderte menschen trotzen eis und schnee ohne obdach </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> dramatische lage in altenheimen: das verzweifelte warten auf corona-schnelltests </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> dramatische situation in flüchtlingslager moria auf lesbos </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> dramatisches artensterben: weckruf für mehr naturschutz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> drei monate nach dem anschlag von hanau </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> dreikönigstreffen der fdp </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> dreiteilige dokumentation &quot;ostfrauen&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> dreyer gegen baldauf im duell der landtagswahlen in rheinland-pfalz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> drittes amtsenthebungsverfahren der us-geschichte </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> drogenanbau hinter stahlbeton - deutschlands erste legale cannabis-plantage </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> drogenanbau in afghanistan nimmt zu </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> drohende eskalation im konflikt mit dem iran? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> drohender stellenabbau bei französischer luftfahrtindustrie wegen corona </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> drohender türkischer einmarsch: syrische kurden fühlen sich verraten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> drohendes defizitverfahren - wie sehr schadet das schuldenmachen  italien? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> droht trump ein amtsenthebungsverfahren? - weißes haus veröffentlicht mitschrift von umstrittenen selenski-telefonat </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> dublin-abkommen zwingt flüchtlinge zur rückkehr in ankunftsländer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> durchbruch beim handelsabkommen zwischen eu und china </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> e-sport: fortnite-wm in new york </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ehemalige eu-justizkommissarin reding zu viktor orban </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ehemaliger us-präsident trump hält erste offizielle rede seit amtsende auf republikaner-konferenz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ehrenamtler spendet mit seinem baritonhorn jeden abend trost </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eilantrag: kalbitz darf vorerst in der afd bleiben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ein demonstrant bei protesten in hongkong von polizist angeschossen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ein dorf wird geräumt: klimawandel an der küste von wales </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ein halbes jahr nach rechtsextremistischem anschlag von hanau: das trauma der opfer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ein jahr §219a: nur wenige ärzte lassen sich auf liste der bundesärztekammer eintragen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ein jahr greta - ein jahr &quot;fridays for future&quot;: was die politik seitdem bewirkt hat </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ein jahr große koalition </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ein jahr nach dem nsu-urteil </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ein jahr nach dem schulmassaker in parkland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ein jahr nach treffen von kim und moon: fußwege an der grenze zu nordkorea </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ein jahr teilhabechancengesetz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ein land im bürgerkrieg: die komplizierten machtverhältnisse in libyen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ein tag auf der covid-station </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ein tag im abschiebeterminal des frankfurter flughafens </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eindringlicher appell der ärzte aus wuhan </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eindrücke aus der hauptstadt washington d.c zum wahlsieg von joe biden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eingeschneit im auto: reportage von der a2 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> einheimische mobilisieren gegen rechtsrock-festival in ostritz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> einige politiker erachten urlaubsreisen zu ostern als unrealistisch </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> einigung bei tarifverhandlungen im öffentlichen dienst der länder </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> einigung in thüringen: ramelow wird gewählt mit neuwahlen im april 2021 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> einreisebeschränkungen aus großbritannien: staus und mögliche versorgungsengpässe </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> einreisekontrollen - situation an den grenzen zu tschechien und tirol </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> einrichten von impfzentren: vorbereitungen laufen auf hochtouren </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> einschätzung des bgh: vw-käufer können im dieselskandal auf entschädigung hoffen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> einschätzungen der börse zur rettung von tui </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> einsturzgefahr notre-dame: wie weit ist der wiederaufbau? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eintauchen ins grün: ausbildung zum waldbademeister </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> einzelhandel in corona-zeiten: diskussion über die fläche von ladengeschäften </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> einzelhandel rechnet mit langer durststrecke wegen corona-pandemie </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> einzelhandel verzeichnet umsatzplus trotz corona </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eishockey-wm: deutschland unterliegt im viertelfinale </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ekd-ratsvorsitzender bedford-strohm will eine übergangslösung für  abschiebungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> elektromobilität: womit will volkswagen punkten? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> elektroschrott: billige neuanschaffung statt teure reparatur </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eltern von us-student warmbier fordern schließung von nordkoreas hostel in berlin </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> emotionaler neustart - was die italiener mit der neuen autobahnbrücke von genua verbinden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> empfehlungen der eu-kommission für die tourismusbranche </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> empörung nach wahl von npd-politiker. regionalwahlen in russland: regierungspartei fährt stimmverluste ein </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ende der fetten jahre: staatseinnahmen gehen zurück </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ende der quarantäne: 500 passagiere verlassen &quot;diamond princess&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ende der sicherheitskonferenz: diskussion über rücknahme von is-kämpfern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ende einer ära: vosskuhles blick auf deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ende einer traumreise: havariertes kreuzfahrtschiff legt im hafen an </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> endlich mallorca - die ersten touristen landen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> energiewirtschaft: wer den höchsten co2-ausstoß in deutschland verursacht </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> enigma: die entschlüsslerinnen von bletchley -park </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> enthüllungsjournalist golunow wieder frei </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> entscheidender schritt zu friedensverhandlungen in afghanistan </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> entscheidung im herbst: mitglieder sollen neuen spd-vorsitz bestimmen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> entscheidung in karlsruhe: hartz-iv-sanktionen teilweise verfassungswidrig </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> entscheidung: gorleben wird kein zwischenlager </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> entsetzen und trauer nach mutmaßlich rassistischem anschlag in hanau </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> entwicklungen des coronavirus in deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> entwicklungszusammenarbeit statt entwicklungshilfe in afrika </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> epstein-vertraute maxwell festgenommen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erdbeben erschüttert erneut kalifornien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erdogan droht eu mit grenzöffnung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erdogan lässt kritik zur nordsyrien-offensive an sich abprallen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erfolg der protestbewegung: hongkong setzt auslieferungsgesetz aus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erfolgreicher kampf gegen lichtverschmutzung: stadt fulda wird &quot;sternenstadt&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ergebnis der kohlekommission: austritt in deutschland bis 2038 empfohlen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ergebnis des spd-mitgliederentscheids: stichwahl zwischen scholz und geywitz sowie walter-borjans und esken </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ergebnis in pennsylvania könnte ausschlaggebend für us-wahl sein </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ergebnis in pennsylvania könnte vorzeitige entscheidung der us-wahl bringen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ergebnisse der bund-länder-beratungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ergebnisse der bund-länder-konferenz: corona-maßnahmen werden verlängert und kontaktbeschränkungen verschärft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ergebnisse der uefa champions league </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ergebnisse von studie zu sexuellem missbrauch im bistum limburg vorgestellt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erhöhte ansteckungsgefahr in brasilianischen favelas </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erhöhte waldbrandgefahr wegen fehlender niederschläge </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erinnerung an erste mondlandung vor 50 jahren </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erinnerungskultur und neue technologien: interaktive hologramme von holocaust-überlebenden im holocaust-museum von chicago </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ermittler beschuldigen vier mutmaßliche verantwortliche des mordes nach mh-17-abschuss </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ermittlungen gegen brustkrebs-bluttest der uniklinik heidelberg eingeleitet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ermittlungen im fall lübcke: generalbundesanwalt übernimmt wegen verdacht auf rechtsextremistisches motiv </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ermittlungserfolg im darknet: auf der dunklen seite des internets </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ernährung der zukunft? - eine ausstellung zeigt gerichte mit kunstfleisch </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erneut großdemonstrationen in belarus gegen präsident lukaschenko </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erneut stirbt ein schwarzer durch polizeigewalt in den usa </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erneute ausschreitungen in leipzig-connewitz nach räumung besetzter häuser </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erneute proteste gegen lukaschenko in belarusischer hauptstadt minsk </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erneuter zitteranfall: bundeskanzlerin merkel weisst zweifel an ihrer gesundheit zurück </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ernste lage im nahen osten: usa bereiten sich auf vergeltungsanschläge vor </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ernteauftakt und klimawandel: wie landwirte ihre flächen auf neue bedingungen einstellen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eröffnung der richard-wagner-festspiele in bayreuth </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eröffnung der weimarer nationalversammlung vor 100 jahren mit festakt gewürdigt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erpressung und prostitution </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erschütternde lebensbedingungen von werkarbeitern in der fleischindustrie </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erste bemannte spacex-rakete in florida erfolgreich zur iss gestartet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erste bilanz nach einem monat - wie gefährlich sind e-scooter? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erste bilanz vier wochen nach dem start der impfkampagne </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erste cholera-fälle nach wirbelsturm &quot;idai&quot; in mosambik </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erste coronavirus-fälle in deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erste deutsche stadt: konstanz ruft &quot;klimanotstand&quot; aus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erste forschungen im privatarchiv von papst pius xii </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erste gruppe unbegleiteter minderjähriger aus überfüllten griechischen flüchtlingslagern in hannover gelandet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erste konzerte mit besuchern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erste messe in notre-dame seit großbrand </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erste studie über sexuellen missbrauch in der ddr </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erster bemannter spacex-start zur iss vorerst verschoben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erster corona-fall in baden-württemberg </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erster coronavirus-fall in den usa </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erster digitaler parteitag der grünen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erster jugend-klimagipfel in new york </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erster koalitionsausschuss nach der sommerpause </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erster prozess gegen is-rückkehrerin am oberlandesgericht münchen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erster tag der verschärften corona-maßnahmen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erstes geisterspiel in der geschichte der fußball-bundesliga in mönchengladbach </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erstes us-werk von general motors schließt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erstmals feiertag: wie berlin den frauentag begeht </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erstmals öffentliche anhörungen im impeachment-verfahren gegen trump </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erstmals wird anklage gegen amtierenden premierminister in israel erhoben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erwartungen an den neuen us-präsidenten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> erzürnte aktionäre: bayer verteidigt den monsanto-ankauf </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> es fehlen bademeister in deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> es muss aber &quot;stickoxid&quot; heißen. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> es war immer zäh: was historisch schief lief zwischen london und brüssel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> esa beginnt projekt zur asteroiden-abwehr </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ethikrat diskutiert impfstrategien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ethikrat spricht sich gegen sonderregeln für corona-geimpfte aus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ethnische durchmischung an us-schulen sinkt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu beschließt milliarden-hilfspaket </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu billigt rahmenvertrag für impfstoff-lieferung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu ringt um kompromiss für corona-milliardenpaket </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu schließt handelsabkommen mit mercosur-staaten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu sichert sich 300 millionen corona-impfdosen der pharmafirmen biontech und pfizer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu stellt sich auf zukunft ohne großbritannien ein </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu und großbritannien setzen brexit-verhandlungen fort </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu und großbritannien stehen kurz vor einigung über handelsvertrag </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu und großbritannien verhandeln über freihandelsabkommen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu-arzneimittelbehörde wird wahrscheinlich noch vor weihnachten impfstoff zulassen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu-außenminister können sich nicht auf sanktionen gegen belarus einigen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu-behörde sperrt gesamten luftraum für boeing 737 max 8 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu-finanzminister ringen um einigung zum thema eurobonds </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu-flüchtlingshilfe in der türkei: erfolg oder desaster? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu-gipfel beschließt verschärfte klimaziele bis 2030 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu-gipfel gewährt verlängerung der brexit-frist bis ende oktober </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu-gipfel in brüssel: der poker um die spitzenposten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu-gipfel in rumänien: staats- und regierungschefs beraten über kurs der union </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu-gipfel streitet über klimaziele </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu-grenzkontrollen verursachen kilometerweite staus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu-innenkommissarin setzt sich in corona-zeiten für unfreiwillig getrennte fernbeziehungen ein </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu-innenminister zur seenotrettung im mittelmeer: suche nach einer lösung auf montag verschoben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu-kommission stellt wasserstoffstrategie vor </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu-kommission: von der leyen stellt plan für klimaneutrales europa vor </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu-kommissionschef juncker kämpft um sein politisches vermächtnis </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu-kommissionspräsidentin von der leyen kritisiert nationale egotrips in der eu </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu-krisengipfel soll brexit-chaos endlich beenden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu-marine-einsatz &quot;sophia&quot; vor lybischer küste wird weitgehend beendet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu-migrationskommissarin </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu-mitglieder beraten über umsetzung des impfpasses </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu-parlament ruft klimanotstand aus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu-parlament stimmt für resolution zur änderung der ausgehandelten gipfel-beschlüssen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu-parlament verabschiedet neues urheberrechtsgesetz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu-parlament verleiht sacharow-preis an uiguren ilham tohti </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu-ratspräsidentschaft: seehofer nimmt sich der eu-flüchtlingspolitik an </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu-richtlinie zu leitungswasser im restaurant </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu-sondergipfel in brüssel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu-sondergipfel nach ergebnislosen verhandlungen vertagt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu-sondergipfel zur corona-pandemie ringt um einheitliches vorgehen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu-sondergipfel: ringen um die nachfolge des kommissionspräsidenten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu-staaten debattieren auf sondergipfel über solidarität oder sparsamkeit </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu-staaten gründen schutzschirm gegen iran-sanktionen der usa </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu-staaten wollen reisebeschränkungen bis spätestens ende juni aufheben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu-staats- und regierungschefs einigen sich im haushaltsstreit mit polen und ungarn </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu-strategie plädiert auf mehr nachhaltigkeit durch &quot;kreislaufwirtschaft&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu-ukraine-gipfel: ukrainischer präsident selenskyj empfängt europäische spitzenpolitiker </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eu-urheberrechtsreform: diskussion um upload-filter im internet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> europäer planen asteroiden-abwehr </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> europäische union erkennt ergebnis der präsidentenwahl in belarus nicht an </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> europawahlkampf unter schwierigen vorzeichen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> evangelikale christen in den usa halten covid-19 für die strafe gottes </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> evangelischer kirchentag in dortmund: merkel und maas beziehen stellung gegen rechtsextremismus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> evp setzt mitgliedschaft von orban-partei bis auf weiteres aus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ex-anwalt cohen wendet sich bei anhörung gegen trump </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ex-außenminister kinkel mit 82 jahren gestorben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ex-beatle und kein bisschen leise: das alterswerk von paul mccartney </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ex-fußball-nationalspieler schweinsteiger beendet aktive karriere </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ex-könig juan carlos verlässt spanien angesichts schwerwiegender korruptionsvorwürfe </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ex-verfassungspräsident maaßen vor untersuchungausschuss zu anschlag auf dem breitscheitplatz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ex-wirecard-chef braun verweigert aussage vor bundestag-untersuchungsausschuss </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> existenznot: hilfsmaßnahmen für die wirtschaft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> exit-debatte: die sicht der corona-risikogruppen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> exit-strategie: österreich will corona-maßnahmen nach ostern lockern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> exit-strategie: wirtschaft drängt nach lockerung der maßnahmen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> expedition ins eis: die reise des forschungsschiffs polarstern zum nordpol </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> expeditionsleiter markus rex zu erkenntnissen der expedition </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> experten fordern deutschlandweite strategie gegen waldbrände </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> experten fordern zerschlagung der stiftung &quot;preußischer kulturbesitz&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> experten warnen vor neuem waldsterben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> explosion in beirut: französischer präsident macron möchte das richtige einsetzen der hilfsgelder kontrollieren </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> extremer wintereinbruch erreicht deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> extremwetter in deutschland: ortsbesuch im harz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> exzellenzinitiative: bundesregierung prämiert elf universitäten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> facebook bemüht sich um transparenz vor europawahl </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> facebooks wandel: warum der konzern sich neu strukturiert </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fahrrad </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fahrradboom in stuttgart: verkehrswende dank corona? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fall nawalny: forderungen nach sanktionen werden lauter </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fälle häuslicher gewalt an frauen in corona-zeiten gestiegen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> familienminister einigen sich auf konzept für kita-öffnungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> familienministerin giffey spricht sich für neuregelung des unterhaltsrechts aus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fast die hälfte aller arbeitenden flüchtlinge als hilfskräfte angestellt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fasten für klima und anderes </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fastenzeit &quot;ramadan&quot; in zeiten von corona </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> faszination antarktis: tourismusansturm am ende der welt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> faszinierender einblick in umfangreichen nachlass: der vergessene foto- und lichtkünstler horst h. baumann </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fc bayern münchen gewinnt die champions league </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fc bayern münchen im finale der champions league </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fc bayern münchen ist neuer deutscher fußballmeister </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fc bayern scheidet aus champions league aus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fdp-europawahlkampf: generalsekretärin wird spitzenkandidatin </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fdp-parteitag stellt weichen für regierungsbeteiligung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> festakt zum 75-jährigen bestehen der vereinten nationen in new york </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> festnahme im mordfall lübcke: spur in die rechte szene </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> festsitzen auf lesbos: von afghanistan nach moria </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> feuerwehr bekommt buschbrände in australien nicht unter kontrolle </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fifa-rat beschließt vergrößerte klub-weltmeisterschaft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> film &quot;judy&quot; über schauspielerin garland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> film über fußballspieler toni kroos feiert weltpremiere </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> film zum geburtstag: wim wenders wird 75 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> filmemacher in der ddr </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> filmfest ohne fans: wie die berlinale trotzdem ein kultur-highlight wird </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> filmkritik: &quot;a rainy day in new york&quot; von woody allen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> filmpremiere &quot;official secrets: geschichte einer whistleblowerin&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> filmpremiere &quot;terminator: dark fate&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> filmpremiere: als hitler das rosa kaninchen stahl </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> filmproduzent artur brauner gestorben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> filmtipp &quot;paranza&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> filmtipp &quot;parasite&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> filmtipp &quot;systemsprenger&quot;: wenn kinder aus der norm fallen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> filmvorstellung &quot;berlin bouncer&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> finanzminister beraten über grundsteuer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> finanzskandale wirecard und warburg bank: was wusste finanzminister scholz? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> finanztransaktionssteuer: finanzminister scholz legt entwurf für steuer auf aktiengeschäfte vor </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fit für die krise: arbeitsminister heil plant gesetz für mehr schutz von arbeitnehmern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> flandern: opfer des brexits </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> flaute beim ausbau der windenergie in baden-württemberg </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fleischindustrie berät sich zu grundsätzlichen änderungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> flucht vor dem coronavirus: deutsche bürger sollen samstag aus china ausgeflogen werden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> flüchtlinge auf dem mittelmeer: debatte um neue staatliche seenotrettungsmission </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> flüchtlinge in griechenland: zustand in lagern katastrophal </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> flüchtlinge vor calais wollen weiterhin nach großbritannien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> flüchtlinge: zurück in  istanbul </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> flüchtlingsdrama nach öffnung der türkisch-griechischen grenze </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> flüchtlingslager im europäischen mittelmeerraum überfüllt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> flüchtlingssituation in griechenland: bundesinnenminister seehofer bietet athen deutsche hilfe an </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fluchtvermeidung: botschafter gegen die illegale migration </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> flügelkampf: wie die afd mit parteinternen kritikern umgeht </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> flughafen ber soll am 31. oktober eröffnen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> flughafen im winterschlaf </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> flugsicherung entwickelt neue technik im kampf gegen drohnen im flugverkehr </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> flugzeugabschuss: die verantwortung des iran </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> flugzeugabsturz im iran: war es eine versehentlich abgefeuerte rakete? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> flut statt feuer: regenfälle sorgen in teilen australiens für überschwemmungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> folgen der angriffe auf die saudischen ölanlagen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> folgen der corona-politik für demenzkranke </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> folgen des kohleausstiegs </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> folgen des sandabbaus in kambodscha </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> folgen von insektensterben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> forderung nach mehr sicherheit: wie der staat seine bürger besser schützen kann </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> formel 1 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> formel 1 und tour de france dürfen auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> formel 1: großer preis von australien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> formel 1: großer preis von bahrain </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> formel 1: großer preis von mexiko </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> forscher entwickeln neue methode zur brustkrebsfrüherkennung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> forscher fordern gemeinsame europäische strategie im kampf gegen das coronavirus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fotograf robert frank verstorben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fotograf sebastio salgado erhält friedenspreis des deutschen buchhandels </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fpö fordert entlassung von orf-moderator wolf </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> frachtsegler &quot;peking&quot; erreicht hamburger hafen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fragen und antworten zu deutschen is-kämpfern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fragwürdige trauer: chemnitzer fc unterstützt gedenkfeier für neonazi </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> frankreich und spanien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> frankreich zwischen rückkehr zur normalität und drohender wirtschaftskrise </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> frankreich:tiefes entsetzen über islamistisches attentat auf lehrer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> frankreichs regierung tritt zurück </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> franzosen demonstrieren gegen antisemitismus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> französischer ex-präsident giscard d'estaing im alter von 94 jahren gestorben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> französischer nationalfeiertag: präsident macron kündigt militärisches weltraumkommando an </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fratelli tutti: neue enzyklika des papstes </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> frauen-weltrekord: brasilianische surferin maya gabeira steht mehr als 22 meter hohe welle </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> frauenfußball erst seit 50 jahren in der brd erlaubt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> frauenfußball-dokumentation &quot;das wunder von taipeh&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> frauenrollen: hitchcocks blonde schönheiten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> frauenstreik in der katholischen kirche </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> frauentag in deutschland: wo stehen wir in sachen gleichberechtigung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> freibäder in nrw dürfen teilweise wieder öffnen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> freiburg erlässt betretungsverbot für öffentliche plätze </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> freie fahrt ans mittelmeer: ceneri-basistunnel eröffnet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> freiheit - und was sich sonst noch geändert hat aus ostdeutscher sicht </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> freiheit für wuhan-rückkehrer: ende der quarantäne in germersheim </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> freiwillige helfer in kliniken: friseurin unterstützt pflegepersonal </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fridays for future debattiert über die formen des freitags-protestes </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fridays for future organisieren proteste im internet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> friedensnobelpreis für äthiopischen ministerpräsidenten abiy ahmed </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> friedensplan für libyen: wie realistisch ist das ergebnis des gipfels? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> friseure dürfen wieder haare schneiden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> friseurtourismus: zum haarschnitt einmal kurz ins ausland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> frust - kampf - novemberhilfen: videotagebuch eines kochs und einer künstlerin </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> frust der laien: scheitert der synodale weg an rom und manchen bischöfen? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> frustrierte gemeinden in nordrhein-westfalen: freiwillige flüchtlingsaufnahme nicht erwünscht </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> führungskrise und machtkampf: cdu union sucht nachfolge von kramp-karrenbauer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fukushima: acht jahre nach der katastrophe </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fünf jahre &quot;wir schaffen das&quot;: rückblick und ausblick auf die situation der flüchtlinge des jahres 2015 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fünf jahre danach: wie es einer flüchtlingsfamilie in berlin ergangen ist </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fünf jahre später: münchner helfer in der flüchtlingskrise </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fünfeinhalb jahre nach abschuss des fluges mh17 beginnt der strafprozess in amsterdam </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fußball bundesliga: fc bayern münchen ist deutscher meister </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fußball-bundesliga: bremen-frankfurt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fußball-bundestrainer joachim löw kündigt rücktritt zum sommer an </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fußball-champions league: leipzig gewinnt gegen manchester united mit 3 zu 0 und dortmund gegen st. petersburg mit 2 zu 1 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fußball-champions-league-frauen-finale </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fußball-champions-league: bayern münchen bei lokomotive moskau </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fußball-em-qualifikation: nordirland verliert gegen deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fußball-länderspiel deutschland-türkei </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fußball-länderspiel: deutschland gegen tschechien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fußball-mannschaften paris und istanbul brechen champions-league-spiel nach rassistischem vorfall ab </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fußball-wm der frauen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fußballerinnen des sc freiburg im finale des dfb-pokals </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fußballmagazin &quot;kicker&quot; wird 100 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> g20-finanzministertreffen: digitalkonzerne sollen steuern zahlen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> g20-gipfel endet mit gemeinsamer abschlusserklärung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> g7-gipfel startet in französischem biarritz: einzelinteressen statt gemeinsamkeiten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> galileo: wie präzise europas satellitennavigationssystem ist </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gasstreit zwischen griechenland und türkei </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gastronomen in sorge um die zukunft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gedenken 80 jahre 2. weltkrieg </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gedenken an 400 jahre sklaverei: warum jetzt us-bürger nach afrika zurückkehren </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gedenken an das oktoberfest-attentat im jahr 1980 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gedenken an die kz-befreiung vor 75 jahren </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gedenken an die opfer der loveparade-katastrophe vor zehn jahren </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gedenken an leipziger montags-demonstrationen vor 30 jahren </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gedenken an opfer der euthanasie-verbrechen der nationalsozialisten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gedenken an opfer von hanau </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gedenken an tote der corona-pandemie </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gedenken in frankreich und großbritannien an das weltkriegsende </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gedenkfeier in polen zum 75. jahrestag des warschauer aufstands </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gefährdete demokratie: bedrohungen  gegen kommunalpolitiker </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gefährliche weltkriegsmunition: kampfstoffe vergiften ostsee </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gegen das vergessen: merkel besucht gedenkstätte auschwitz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> geheilte corona-patienten erzählen bundespräsident steinmeier von ihrem leben nach der infektion </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> geheimtipp aus den 80er: laurie anderson in der elbphilharmonie </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gekipptes beherbergungsverbot in brandenburg und bayern sowie verschärfte corona-maßnahmen in nordrhein-westfalen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gelöbnis zum 65. jahrestag der bundeswehr in berlin </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gemeinsame gottesdienste mit strengen hygieneregeln wieder möglich tag der pressefreiheit: journalisten durch corona weltweit eingeschränkt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gemischtes stimmungsbild zum brexit in london </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> generaldebatte im bundestag: merkel fordert strengere corona-auflagen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> generaldebatte im bundestag: merkel hält an großer koalition fest </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> generaldebatte im bundestag: parteien im bundestag üben kritik an merkels corona-politik </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> generation corona: wie ein virus die pläne vieler jugendlicher vereitelt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> genesene berichten über ihre corona-erkrankung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> genua: gedenken an opfer des brückeneinsturzes vor einem jahr </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> georgien: regierungsgegner demonstrieren für neuwahl </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> geplante digitale krankenakte hat laut chaos computer club massive sicherheitslücken </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> geplante wahlrechtsreform von union und spd stößt parteiübergreifend auf scharfe kritik </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> geringe impfbereitschaft bei pflegepersonal </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> germanwings am boden: wann springt der staat bei der lufthansa ein? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> geschäfte rüsten sich für die wiedereröffnung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gescheiterte fusion der deutschen bank und der commerzbank </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gescheiterte mautpläne: betreiber fordern 560 millionen vom bund </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> geschichte per virtual reality im deutschen historischen museum erleben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> geschlossen: wie museen mit abgesagten ausstellungen umgehen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> geschlossene gotteshäuser: religionsgemeinschaften in deutschland fordern lockerungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> geschlossene grenzen im schengenraum am beispiel görlitz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> geschlossene grenzen und hohe inzidenzen in tschechien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> geschlossene kitas und schulen: familien kommen an ihr limit </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gesetzgeber erlaubt modellprojekt für grippe-impfungen in apotheken </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gesichert rechtsextremistisch: verfassungsschutz stuft &quot;identitäre bewegung&quot; hoch </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gespaltenes land: wahlkampf in polen im endspurt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gesperrt: der ansturm der urlauber an die ostsee </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gespräche über mögliche lockerung der ausgangsbeschränkungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gestreamte konzerte in zeiten von corona </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gesundheits-apps: bundestag stimmt plänen für digitale gesundheitsangebote zu </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gesundheitsminister spahn erlässt anordnung für fluggesellschaften </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gesundheitsminister spahn will die pflege von schwerstpflegebedürftigen reformieren </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gesundheitsminister spahn will verbot von konversionstherapien für homosexuelle </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gesundheitssystem gerät vielfach an belastungsgrenze </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> geteilte meinungen in der bevölkerung und politik zum beschluss über härteren lockdown </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> geteilte reaktionen auf geplante maßnahmen der bundesregierung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gewalt gegen frauen in russland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gewalt in syrischem idlib </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gewerkschaften und arbeitgeber einigen sich auf kompromiss für tarifstreit im öffentlichen dienst </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gewerkschaften warnen bei mai-kundgebungen vor tariffluch </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gewinner und verlierer der landtagswahl in thüringen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gewinner und verlierer des deutschen atomausstiegs zehn jahre nach fukushima </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gewinner und verlierer: die folgen der corona-krise für die wirtschaft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gewinner von cannes: film &quot;parasite&quot; gewinnt die goldene palme </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gibt es bald sanktionen für impfdrängler? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> giftanschlag auf russischen oppositionellen nawalny: weitere hinweise auf kreml-verantwortlichkeit </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gipfeltreffen von eu und arabischer liga im ägyptischen sharm el-sheikh </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gipfeltreffen zwischen trump und kim in vietnam </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> glaubenssache: gottesdienste in zeiten von corona </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gleichberechtigung: pandemie verstärkt offenbar alte rollenverteilung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gleichwertige lebensverhältnisse für alle - bundesregierung stellt paket für strukturschwache gebiete vor </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gletscherschutz durch kunstschnee </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> goldrausch im weltall: raumfahrtsymposium in speyer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gorch-fock-skandal: eine geschichte von fehlplanungen und misswirtschaft bei bundeswehr und werft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gottesdienste werden wegen corona über das internet gestreamt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gratulationen aus der europäischen union </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> grenzenloses lauschen: was darf der bundesnachrichtendienst? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> grenzlager in el paso: kinder als politisches instrument </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> grenzöffnung zu österreich </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> grenzschließungen in europa: alleingänge statt europäischer lösung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> grenzstreit schweiz-italien: alpenhütte wechselt nationalität </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> grenzwert-diskussion: wie reagieren die autofahrer auf den streit? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> griechenlands hoffnung auf tourismus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> griechische insel leros kämpft mit steigenden flüchtlingszahlen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> groko-bilanz zu migration </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> groko-bilanz: aufbauprogramm zum wohnraum </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> groko-bilanz: fachkräftemangel in deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> groko-kompromiss: wer von der neuen grundrente profitiert </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> groko-streit über entschuldung der kommunen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> großbrand in mecklenburg-vorpommern: umliegende ortschaften evakuiert </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> großbritannien beginnt mit corona-massenimpfungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> großbritannien bereitet sich auf europawahl vor </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> großbritannien hat gewählt: konservative liegen nach befragungen klar vorn: annette dittert mit informationen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> großbritannien impft ohne bedenken weiter </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> großbritannien in der krise ohne premierminister </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> großbritannien lässt als erstes europäisches land corona-impfstoffe zu </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> großbritannien steuert neuwahlen am 12. dezember an </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> großbritannien verhängt quarantänepflicht für zurückkehrende spanien-urlauber </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> großbritannien vor der wahl: erstes tv-duell von johnson und corbyn </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> großbritannien: die kandidaten der conservative party stehen fest </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> großbritannien: mit minigolf gegen leere kirchen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> großbritannien: parlament wird in zwangsurlaub geschickt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> großbritannien: wie premierministerin may ihren brexit-deal noch retten will </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> großbritanniens rassismus-debatte trifft denkmäler kolonialer vergangenheit </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> großbritanniens wirtschaft durch coronavirus und brexit in doppelter gefahr </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> großdemonstration gegen corona-politik der bundesregierung wegen missachtung der hygienevorschriften aufgelöst </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> große anteilnahme nach busunglück auf madeira </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> große erwartungen an merkels zweite eu-ratspräsidentschaft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> große karl lagerfeld retrospektive in halle </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> große koalition einigt sich auf 130-milliarden-euro-konjunkturpaket zur rettung der wirtschaft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> große koalition einigt sich beim migrationspaket </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> große koalition streitet über gesetzesentwurf für paketdienste </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> große koalition unter zugzwang: klimakabinett auf der suche nach schnellen lösungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> große koalition zieht halbzeitbilanz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> großes theater: deutsches schauspielhaus trotzt corona mit neuem stück von rainald goetz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> größter waldbrand in der geschichte kaliforniens </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> großveranstaltungen trotz corona? medizin-experiment bei konzert von tim bendzko </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gründe für unruhen in chile </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> grundrente beschlossen: was kritiker zu durchführbarkeit und finanzierbarkeit sagen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> grundrente mit bedarfsprüfung: der kompromiss der groko </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> grundsatzpolitischer schlagabtausch: bundeskanzlerin merkel trifft ungarns ministerpräsidenten orban </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> grundschulen und kitas in sachsen öffnen wieder </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> grundwasser und ameisen - wieder ärger für tesla in grünheide </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> grüne debattieren 40 jahre nach gründung über grundsatzprogramm </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> grüne kündigen vorstoß in debatte über wahlrecht ab 16 an </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> grüne pisten: mountainbiker befördern alpinen sommertourismus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> grüne vor den landtagswahlen in baden-württemberg vor der cdu </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> grüne welle bei eu-wahl - warum die wähler den klassischen volksparteien nicht mehr vertrauen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> grünen-parteitag: höhenflug ohne ende? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> grünen-spitzenkandidatin maike schaefer im gespräch </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> grünes investment: klimawandel als geschäft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gundam - der riesenroboter von yokohama </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hackerangriff auf dax-konzerne: recherche zu industriespionage </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> halle gedenkt der opfer des terroranschlags auf die synagoge vor einem jahr </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hambacher fest: rechte vereinnahmen symbolträchtige orte der deutschen geschichte </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hamburger polizei bilanziert das alkohol-verkaufsverbot auf der reeperbahn </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hamburgs justizsenator steffen will &quot;containern&quot; straffrei machen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hamburgtrend vor der bürgerschaftswahl </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> handball-champions-league-finale: thw kiel gewinnt gegen barcelona </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> handball-em </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> handball-wm in kairo im schatten von corona </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> handball-wm und formel 1 dürfen aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> handball-wm: deutschland - uruguay </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> handball-wm: deutschland besiegt im letzten gruppenspiel serbien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> handball-wm: deutschland und frankreich trennen sich unentschieden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> handball-wm: die deutsche mannschaft verliert gegen spanien mit 28:32 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> handelsstreit: china und usa besiegeln abkommen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hannah arendt und ihr 20. jahrhundert </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hans traxler: caricatura museum in frankfurt am main stellt werke des künstlers aus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> happy end im ddr-kunstkrimi: &quot;alte meister&quot; zurück in gotha </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> harmonie: weltweite aufführung der johannes-passion </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> harmoniebestreben: kramp-karrenbauer bei der csu </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hass im netz im hamburger wahlkampf </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hass im netz: beleidigungen gegen politikerinnen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hass-klima in den usa: die konsequenzen nach el paso und und dayton </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> häufung von hand-fehlbildungen in nordrhein-westfalen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hauptangeklagter im fall walter lübcke gesteht mord </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hauptstadtflughafen ber mit kleiner feier eröffnet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> haus- und klinikärzte stehen wegen corona unter druck </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> haus-besuch: im wohnzimmer von altkanzler helmut schmidt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hausärzte  an der belastungsgrenze: die angst der praxen vor dem herbst </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> haushaltsdebatte: investitionen kontra &quot;schwarze null&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> haushaltsstrategie: siegburg investiert trotz schulden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> haute-couture-woche in paris: nachwuchsstar imane ayissi aus kamerun </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> heidenau - fünf jahre danach </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> heimsieg für karl geiger beim weltcup-skispringen in willingen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> heinsberg-studie macht hoffnung auf lockerung der corona-auflagen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> heiraten in zeiten von corona </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> heiße suppe für obdachlose </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> held des alltags - dhl fahrer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> held des alltags: die ambulante suppenküche </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> held des alltags: freiwillige erntehelfer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> held*in des alltags: arztmobil hamburg </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> held*in des alltags: der lieferservice </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> held*in des alltags: die physiotherapeutin </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> held*in des tages: syrische familie näht masken für krankenhäuser </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> helden des alltags in der corona-krise </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> helden des alltags: alleinerziehende </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> helden des alltags: lkw-fahrer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> helden des alltags: rettungssanitäter </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> heldin des alltags </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> heldin des alltags - lehrerin </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> heldin des alltags am sorgentelefon </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> heldin des alltags: die abiturientin </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> heldin des alltags: die hamburger pastorin imke sander </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> heldin des alltags: die hebamme </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> heldin des alltags: medizinstudentin aus brandenburg </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> herausforderer biden liegt bei us-präsidentschaftswahl uneinholbar vorne </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> herausforderung bei corona-impfungen in mali </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> herbert diess gibt chef-posten der kernmarke volkswagen ab </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> herbststrategie gegen corona-ausbreitung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> herbstvollversammlung der deutschen bischofskonferenz in fulda </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hessen und niedersachsen melden häufung von corona-neuinfektionen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hessischer polizeipräsident tritt zurück </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hetze gegen homosexuelle: &quot;lgbt&quot;-freie zonen in polen befremden deutsche partnerstädte </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> heuschreckenplage: gefahr für millionen menschen in ostafrika </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> heute ist welt-nichts-tag </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hitze in deutschen städten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hitze-wochenende sorgt für überfüllte strände an nord- und ostsee </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hitzerekord in deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hitzerekord von 1947 gebrochen: wie geht es den alten menschen mit dem heißen wetter? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hochrechnungen zur europawahl </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> höchste terror-warnstufe ausgerufen: französischer präsident macron verurteilt messerangriff in nizza mit drei toten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hochwasser venedig: warum die schutzmaßnahmen nicht helfen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hochzeiten - das neue spreader-event? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hoffen auf einen corona-impfstoff </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hoffnung auf baldigen einsatz des corona-impfstoffs </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hoffnung auf kriegsende: gefangenenaustausch in der ukraine </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hoffnungen und ängste der jüdischen gemeinde in hamburg </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hohe erwartungen an den neu gewählten vorsitzenden der deutschen bischofskonferenz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hohe haftstrafe für mutmaßlichen deutschland-is-chef: was heißt das für die islamistische szene </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hohe haftstrafe gegen doping-arzt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hohe infektionszahlen in tschechien: grenzschließung ab sonntag </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hollywood-legende kirk douglas verstorben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> holocaust- überlebender sucht deutschen retter </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> holocaust-ausstellung: martin schoeller fotografierte 75 überlebende der naziherrschaft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> holocaust-gedenken: friedländer nennt deutschland bollwerk gegen antisemitismus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> holocaust-gedenken: instagram-projekt soll jugendliche aufklären </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> holocaust-gedenktag für sinti und roma in auschwitz erinnert an völkermord </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> holocaust-gedenktag: &quot;schindlers liste&quot; wieder im kino </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> holocaust-überlebende berichten von auschwitz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> holz muss nicht hölzern sein: künstlerin meißelt mode </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> homeoffice nach corona </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hongkong: digitale plattformen stoppen chinesische desinformation über soziale medien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> housten trauert um georg floyd </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> huawei darf vorerst weiter geschäfte mit us-firmen machen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> huawei droht ausschluss von 5g-ausbau </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> humanitäre katastrophe im syrischen idlib: reportage aus einer stadt unter beschuss </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hundertausende demonstrieren in london für neues brexit-referendum </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hunderte ärzte verlassen den libanon wegen perspektivlosigkeit </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hunderttausende demonstrieren in barcelona </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hunderttausende erntehelfer fehlen in der deutschen landwirtschaft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hunderttausende protestieren in hongkong gegen geplantes auslieferungsgesetz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hunderttausende syrer harren in flüchtlingslager nahe idlib aus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hurrikan &quot;dorian&quot; richtet schwere zerstörungen auf den bahamas an </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> iaa publikumstag: demo gegen suvs </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ikone des deutschen films: hannelore elsner ist tot </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> illegale autorennen in deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> illegaler handel mit tieren </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> illustrator edel rodriguez: auf der suche nach dem wahren gesicht des us-präsidenten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> im interview: ehemaliger chefankläger der nürnberger prozesse benjamin ferencz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> im kino: &quot;capernaum&quot; - stadt der hoffnung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> im kino: &quot;enfant terrible&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> im niederrheinischen heinsberg stagniert die zahl der neuinfektionen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> im porträt: ursula von der leyen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> im zweifel für die kunstfreiheit: warum der eugh das sampling als eigene kulturtechnik sieht </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> image einer stadt: konzert gegen rechts in chemnitz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> immer mehr infizierte: wie die chinesische regierung versucht zu beruhigen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> immer mehr kinder  in deutschland leben in pflegefamilien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> immer mehr kinder besuchen privatschulen: hier beginnt die spaltung der gesellschaft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> immer weniger helfer für minderjährige flüchtlinge </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> immobilienkongress &quot;quo vadis&quot; stellt konzepte für bezahlbares wohnen vor </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> impeachment-verfahren: präsident trumps verteidiger halten ihre plädoyers </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> impeachment: wie war das nochmal bei clinton? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> impfen und testen - lösung für den lockdown </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> impfgiptel in berlin: viel streit um wenig stoff </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> impfpflicht - was bedeutet das? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> impfstoffmangel in afrika </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> impfstoffpolitik der bundesregierung erntet kritik </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> impfungen mit astrazeneca-wirkstoff vorsichtshalber ausgesetzt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> in 14 us-bundesstaaten bestimmen demokraten am &quot;super tuesday&quot; ihren kandidaten für die präsidentschaftswahl </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> in 80 tagen um die welt: segler boris herrmann bei der regatta vendée globe </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> in den niederlanden soll ein freibad im winter corona-ängste vertreiben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> in der grenzregion saarland/lothringen erstarken längst überwunden geglaubte ressentiments </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> in der landwirtschaft fehlen saisonarbeiter </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> in italien zeichnet sich regierungsbildung ab </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> in schottland wächst die angst vor einem no-deal-brexit </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> in syrien ist der is wieder aktiv </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> indien im ausnahmezustand nach ausgangssperre für 1 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> indonesien reagiert auf ansteigenden tourismus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> industriestandort osaka: japans heimliche hauptstadt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> inf-abrüstungsvertrag liegt auf eis </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> infektionszahlen in sachsen bis zu dreimal höher als im bundesdurchschnitt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> influencer: ist schleichwerbung zulässig? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> initiative &quot;maria 2.0&quot; kirchenreformen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> initiative gegen müll: hersteller sollen kosten für entsorgung mittragen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> initiativen gegen rechts in chemnitz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> inklusive wohngemeinschaft in bremen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> innenausschuss des bundestages befasst sich mit rassismus in deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> innenminister seehofer sieht kein strukturelles problem mit rechtsextremisten bei sicherheitsbehörden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> innenminister-konferenz: abschiebung von gefährdern im einzelfall wieder möglich </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> innenministerkonferenz: beratungen über nutzung von sprachassistentenaufzeichnungen als beweismittel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> inner-irische grenze trennt künftig die eu und großbritannien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> innerdeutsche grenze: mecklenburg-vorpommern schottet sich in der corona-krise ab </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> inselstadt sassnitz im pipeline-streit zwischen usa und russland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> insolventes touristikunternehmen thomas cook: bund will kunden helfen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> integrationsgipfel verabschiedet nationalen aktionsplan </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> intensivstationen im grenzgebiet: hilfe für europas nachbarn </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> interessenten für cdu-vorsitz bringen sich in stellung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> internationale kampagne zum verbot von streumunition legt lagebericht vor </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> internationale syrien-geberkonferenz stellt etwa sieben milliarden euro zur verfügung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> internationale umweltmesse &quot;grüne woche&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> internationaler aktionstag: frauen als opfer häuslicher gewalt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> internationaler asteroidentag: einblicke in die unendlichen weiten des weltalls </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> internationaler energiewende-dialog: eine zwischenbilanz für deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> internationaler frauentag: gewalt gegen frauen in mexiko </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> internationaler gerichtshof verurteilt ex-rebellenkommandanten ongwen: der schwere weg zur versöhnung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> internationaler sportsgerichtshof cas reduziert olympia-sperre russlands auf zwei jahre </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> internationaler tag der jugend: zunahme von depressionen im kinder- und jugendalter </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> internationaler tag der patientensicherheit: who fordert mehr anstrengungen gegen behandlungsfehler </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> internationaler tag gegen rassismus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> internationaler tag zur abschaffung des sklavenhandels: die nachfahren von sklaven in brasilien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> interview löst diskussion um cdu-austritt von maaßen aus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> interview mit cdu-chefin kramp-karrenbauer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> interview mit cdu-vorsitzender annegret kramp-karrenbauer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> interview mit gesundheitsminister spahn </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> interview mit spd-generalsekretär lars klingbeil </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> investition ins klima: die finanzierung des &quot;green deal&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> iran greift us-truppen im irak an </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> iran kündigt nach tötung von soleimani rückzug aus atomabkommen an </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> iran trauert um soleimani </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> iran vor 40 jahren: was von der revolution übrig blieb </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> iran vor den wahlen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> iran-konflikt: washington erhöht den druck </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> irans führender atomwissenschaftler mohsen fachrisadeh bei anschlag getötet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> is-chef al-bagdadi nimmt sich bei angriff das leben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> is-kämpfer: deutscher bestreitet mord- und foltervorwürfe </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> islamismus-debatte eskaliert in kleinem französischem ort trappes </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> islamkonferenz: der kampf gegen den islamismus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> island öffnet schulen ohne restriktionen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> israel führt impfpass ein </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> israelische forscher stellen einen prototyp eines herzens aus dem 3d-drucker her </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> israelisches parlament löst sich vier wochen nach vereidigung auf und beschließt neuwahlen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> israels präsident reuven rivlin im bundestag zum gedenken an die ns-opfer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> istanbul: erdogans akp erzwingt wiederholung der bürgermeisterwahl </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> istanbul: oppositionskandidat imamoglu klarer sieger bei bürgermeisterwahl </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> italien führt das bürgergeld ein </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> italien öffnet grenzen für tourismus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> italien riegelt wegen coronavirus gebiete im norden ab </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> italien und spanien fordern europäische solidarität in form von bonds </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> italienische insel capri hofft sommersaison noch zu retten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> jahrespressekonferenz des russischen präsidenten putin per videoübertragung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> japan erlaubt züchtung menschlicher organe in tieren </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> japan verkauft leerstehende häuser in der provinz zu symbolpreisen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> japan: erstmals anwerbung von arbeitsmigranten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> japans kaiser akihito dankt ab </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> jeder vierte israeli bereits gegen corona geimpft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> jetzt kommt der brexit: erste abschiede in großbritannien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> joachim löw bleibt fußball-bundestrainer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> joachim löw sortiert drei weltmeister aus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> joe biden als neuer präsident der usa vereidigt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> joe biden offiziell präsidentschaftskandidat der us-demokraten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> john bercow verlässt britisches parlament </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> johnsons brexit-pläne verunsichern britische autoindustrie </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> journalismus in zeiten der pandemie: zwischen informationspflicht und panikmache </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> jubiläum und bilanz: 75 000. stolperstein erinnert an das schicksal von memminger juden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> jubiläum: 100 jahre jugendherberge </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> juden in deutschland: antisemitismusbeauftragter warnt vor kippa-tragen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> jüdisches leben in deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> jüdisches museum eröffnet mit neuer dauerausstellung nach zwei jahren umbaupause </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> jugend und internet: die rolle der influencer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> junge malerei in deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> jüngste ereignisse überlagern anti-iran-protest im irak </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> juristischer streit um großdemo gegen corona-politik entfacht </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> justizministerin barley schlägt neuregelung der maklergebühr vor </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> juwelenraub &quot;grünes gewölbe&quot;: drei verdächtige in untersuchungshaft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kabinett beschließt investitionen von mehr als einer milliarde euro in das mobilfunknetz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kabinett beschließt neun-punkte-plan gegen rechtsextremismus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kabinett beschließt zulassung von elektro-scootern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kabinettsausschuss zur bekämpfung von rechtsextremismus und rassismus berät sich mit vertretern von migrantenorganisationen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kabinettsumbildung und stimmung in der großen koalition </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kahlschlag bei karstadt: sorge vor verödung der innenstädte </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kahlschlag in den kaparten: rumäniens urwald in gefahr </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kahlschlag: wie brasilien den regenwald und seine ureinwohner opfert </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kalbitz kämpft weiter um seine afd-parteimitgliedschaft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kalbitz verliert rechtsstreit um parteiausschluss </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kalifornien führt zahlreiche anti-corona-maßnahmen wieder ein </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kälteeinbruch in nordamerika lässt niagara-fälle vereisen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kältewelle: arktische temperaturen machen weiten teilen der usa zu schaffen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kambodschas textilindustrie: kampf ums überleben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kampf gegen corona-virus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kampf gegen corona: biontech weckt hoffnungen auf hochwirksamen impfstoff </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kampf gegen corona: schwierige suche nach einheitlichem vorgehen in den bundesländern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kampf gegen die clan-kriminalität </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kampf gegen krebs: die chancen der immuntherapie </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kampf gegen malaria: weltweit erste impfkampagne
                                
                                    hinweis:
                                    der beitrag zum thema &quot;dürre&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kampf gegen waffenschmuggel: eu-außenminister vereinbaren neuen marinemission im mittelmeer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kampf um brexit: letzter showdown vor der parlamentspause </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kämpfe um letzte is-bastion baghus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kampfmittelräumung mit ki - die lesenden maschinen von kiel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kandidaten-duell in großbritannien entscheidet nachfolge von premierministerin may </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kandidatensuche: die ersten us-vorwahlen in new hampshire </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kann's cannes noch? das filmfestival zwischen frauenquote und rechtspopulismus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kanzlerin merkel berät mit länderchefs über einheitliche vorgehensweise während der pandemie </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kanzlerin merkel berät sich mit vertretern des öffentlichen gesundheitsdienstes über entlastungen der gesundheitsämter </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kanzlerin merkel besorgt über hohe infektionswerte </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kanzlerin merkel ruft zu solidarität in corona-krise auf </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kanzlerin merkel und präsident macron schlagen eu-wiederaufbaufonds im umfang von 500-milliarden euro vor </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kanzlerin merkel: impf-angebot für alle deutschen bis zum 21. september </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kapitänin der &quot;sea-watch 3&quot; äußert sich im ard-interview </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kapitänin der &quot;sea-watch 3&quot; rackete bleibt unter hausarrest </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> karnevalshochburgen feiern rosenmontag </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kartellamt gegen facebook: konzern soll entflochten werden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> katar: einblicke in das gastgeberland der fußball-wm 2022 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> katastrophe über mecklenburg-vorpommern: zwei eurofighter abgestürzt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kaufrausch - wie manipulierend ist der &quot;black friday&quot;? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> keeper trautmanns erstaunliche lebensgeschichte im kino </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kein bock auf europawahl? was die wähler sagen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kein ende der buschbrände in australien in sicht </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kein ende des notstandsgesetzes: wie orban seine macht zementiert </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kein erfolgsmodell: warum sich die corona-app in singapur nicht durchsetzt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kein happy end in hollywood: corona legt die filmbranche lahm </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> keine abschiedsparty nach der highschool in virginia: bleibende erinnerung für den jahrgang 2020 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> keine corona-beschränkungen in belarus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> keine einigung beim eu-sondergipfel über corona-finanz-paket in sicht </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> keine hoffnung auf baldige lockerungen nach jahreswechsel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> keine maskenpflicht mehr im unterricht in nrw </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> keine touristen in prag - airbnb-wohnungen werden wieder bezahlbar </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kfw-kredite: unternehmen mit insolvenzängsten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kindergrundsicherung: spd will kinder besser vor armut schützen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kindertagesstätten agieren mit notbetreuung weiter am limit </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kindheit auf youtube: das geschäft mit den familienkanälen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kino trotz corona: 60 jahre autokino </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kinofilm &quot;green book&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kinofilm &quot;tolkien&quot; über herr der ringe-autor j.r.r. tolkien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kinotipp: &quot;corpus christi&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kinotipp: königshaus-drama &quot;the favourite&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kirchen in der corona-adventszeit </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kita- und schulöffnungen: grundschule in altenmünster als positivbeispiel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kita-erzieherin unter mordverdacht: personalnot und strukturversagen? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kitas öffnen schrittweise </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> klage beim bundesverfassungsgericht gegen &quot;triage&quot;-empfehlungen bei medizinischen behandlungsengpässen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> klausurtagung: union und spd bekräftigen handlungsfähigkeit </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> klima-forschungschiff &quot;polarstern&quot; unternimmt größte arktis-expedition aller zeiten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> klimaaktivistin greta thunberg fordert stärkeren einsatz für eu-klimaziele </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> klimabilanz 2020: sterbende wälder und neue wege der aufforstung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> klimakiller &quot;cloud&quot; -  wie viel co2 verursachen wir beim surfen im netz? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> klimaliste baden-württemberg: konkurrenz für die grünen? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> klimapaket: stahlhersteller in deutschland stehen beim thema klimaschutz besonders unter druck </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> klimapolitik im fokus der generaldebatte im bundestag </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> klimapolitik: kritik an den plänen der bundesregierung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> klimaschutz-kommission einigt sich nicht auf verkehrswende </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> klimaschutz: rat der wirtschaftsweisen empfehlen co2-steuer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> klimawandel sorgt in norwegen für überdurchschnittlich viel schnee </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> klimawandel: chiles verschwundener see </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> klimawandel: islands gletscher schmelzen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kliniken in belgien durch hohe corona-neuansteckungszahl unter druck </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kliniken in portugal arbeiten an belastungsgrenze </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> klinikskandal: eltern mutmaßlicher missbrauchsopfer wurden nicht informiert </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> klinische befunde der berliner charité bestätigen hinweise auf vergiftung beim kreml-kritiker nawalny </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> knabenchöre: wieso mädchen nicht mitsingen dürfen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> knastleben in den usa: der marathonlauf von st. quentin </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> koalition auf crashkurs? in der groko tun sich immer mehr konflikte auf </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> koalition aus konservativer öpv und grüne stellt nach langen verhandlungen regierungsprogramm vor </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> koalition einigt sich auf durchführung einer rassismus-studie in der gesellschaft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> koalition einigt sich auf lieferkettengesetz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> koalition streitet über erhöhung der luftverkehrsabgabe </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> koalitionsausschuss berät im kanzleramt über weitere corona-beschlüsse </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> koalitionsausschuss berät über milliardenschweres konjunkturprogramm </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> koalitionsausschuss berät über wahlrechtsreform und kurzarbeitergeld </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> koalitionsausschuss beschließt weiteres corona-hilfspaket </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> koalitionsausschuss diskutiert über modell zur grundrente </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> koalitionsausschuss von cdu/csu und spd </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> koalitionsstreit in sachsen-anhalt: wie die cdu zur afd steht </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kohleausstieg: bundeskabinett beschließt milliardenhilfe für strukturwandel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kohleausstieg: wie der strukturwandel in sachsen gelingen soll </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kolumbianer bernal gewinnt die tour de france </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kolumbianer egan bernal gewinnt tour de france </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kommen jetzt die anklagen gegen ex-präsident trump? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kommentar </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kommissionspräsidentin von der leyens erste rede zur lage der union </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kommunalwahlen in der türkei </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kommunen droht finanznot durch wegfall der gewerbesteuer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kommunen fordern mehr hilfe von bund und ländern für klimaschutz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kommunen hoffen auf hilfen aus dem konjunkturpaket </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> komplizierte verbindungen zwischen der ukraine und joe bidens sohn hunter </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kompromiss beim klimapaket: bund und länder einigen sich auf höheren co2-preis </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kompromiss im streit um nord stream 2 gefunden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> konferenz des bundesjustizministeriums zum thema rechtsextremismus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> konflikt in nordsyrien: akk fordert sicherheitszone </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> konflikt um straße von hormus: die strategien der europäer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> konflikt zwischen iran und usa: fragen an außenminister maas </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> konjunkturdelle: warum die fetten jahre schon bald vorbei sein könnten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> konjunkturflaute: maschinenbauer als seismografen für drohende rezession </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> konjunkturpaket mit mehrwertsteuersenkung von bundestag und bundesrat auf den weg gebracht </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> konkurrenz für weißrusslands präsident lukaschenko </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> können löw und bierhoff die fußball-nationalelf aus dem tief führen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> konsequenzen nach maskenaffäre: cdu-fraktionsspitze fordert ehrenerklärung von abgeordneten bis freitag </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> konservative triumphieren bei wahl in griechenland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kontakt in die nazi-szene: umgang mit cdu-politiker möritz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kontaktsperren bleiben auch über ostern bestehen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kontrolle der corona-maßnahmen: eine reportage aus bremen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kontrollen an deutschen flughäfen aufgrund neuer einreisebeschränkungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kontrollen zur einhaltung der deutschen kontaktbeschränkungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kontroverse um beteiligung der bundesrepublik an marineeinsatz in straße von hormus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> konzernumbau: deutsche bank baut massiv stellen ab </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kopenhagen will bis 2025 die weltweit erste klimaneutrale hauptstadt werden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kopf-an-kopf-rennen bei der us-wahl: präsident trump erklärt sich vorzeitig zum sieger der us-wahl </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> korruptionsskandal an us-elite-unis </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> korruptionsurteil in frankreich: ex-präsident sarkozy zu haftstrafe verurteilt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kostenloser öpnv in augsburgs innenstadt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kraftfahrtbundesamt genehmigt erstmals hardware-nachrüstung für euro-5-diesel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kramp-karrenbauer fordert stärkeres militärisches engagement deutschlands in der welt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kramp-karrenbauer schätzt die sicherheitspolitische rolle deutschlands ein </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kramp-karrenbauer stärkt position auf cdu-parteitag in leipzig </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kramp-karrenbauer zeigt sich offen für verlängerung des bundeswehr-einsatzes in afghanistan </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kramp-karrenbauer zur wahl der eu-kommisssionspräsidentin und koalitionsfragen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kramp-karrenbauers vorschlag zur schutzzone in syrien sorgt für irritationen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kranke wirtschaft durch corona - wann kommt die große pleitewelle? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kranken- und altenpfleger demonstrieren in köln für mehr geld und personal </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> krankenhäuser in new york überlastet: ärzte und pfleger verzweifeln </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> krankenhäuser schließen: grund zur sorge? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> krankenhäuser versuchen in corona-krise oberhand zu behalten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> krawalle in stuttgart </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kreml-kritiker festgenommen - nawalny könnte zur bedrohung für putin werden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kreml-kritiker nawalny möglicherweise vergiftet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kreml-kritiker nawalny wird bei rückkehr nach russland festgenommen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kreuzfahrtschiff in quarantäne: geplatzte traumreise wegen coronavirus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> krieg im jemen: reportage über flüchtlingslager bei sanaa </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kriegsende vor 75 jahren: steinmeier bezeichnet 8. mai als tag der befreiung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kriminalitätstatistik 2018 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> krise an türkisch-griechischer grenze: koalition diskutiert über flüchtlingsaufnahme </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> krise bei kulturschaffenden zwingt zu kreativen maßnahmen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> krise der koalition: italien drohen neuwahlen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> krise in nahost: wieso der iran teilweise aus dem atomabkommen aussteigt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> krise in österreich: alle fpö-minister verlassen regierung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> krise in thüringen: was wird aus ramelows lieberknecht-coup </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> krisengipfel der britischen royals: queen unterstützt rückzugspläne von harry und meghan </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> krisenstab berät über schutzmaßnahmen gegen das coronavirus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> krisentreffen der ministerpräsidenten: kultusminister uneinig über frage der schulschließungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kritik an kramp-karrenbauer: verteidigungsausschuss zu kampfjet-plänen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kritik an privilegien für profi-fußballer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kritik an us-präsident: trumps ausfahrt und umgang mit dem coronavirus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kritik an verschärften corona-regelungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kuba hofft mit biden auf wiederaufnahme diplomatischer beziehungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kultur in zeiten von corona: die oberammergauer bangen um die passionsspiele </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kulturschaffende sorgen sich trotz corona-hilfen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kultusminister beraten über fehlende digitale ausstattung und hygienekonzepte an schulen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kultusministerkonferenz berät über die art des schulunterrichts während des lockdowns </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kundenrechte gestärkt - eugh erlaubt rückgabe von matratzen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kundus-bombardierung 2009: europäisches gericht für menschenrechte weist klage wegen aufklärung des kundus-angriffs 2009 ab </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kunst im u-boot-bunker: lichtinstallation &quot;bassins de lumières&quot; in bordeaux präsentiert motive von klimt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kunst und musik im dialog: ausstellung &quot;hyper&quot; in hamburg </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kunst unter schwierigen bedingungen in zeiten von corona </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kunstmetropole ade? - private kunstsammlungen verlassen berlin </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kunstsensation: vermeers &quot;brieflesendes mädchen am offenen fenster&quot; von übermalung befreit </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kurden beschießen türkische städte als reaktion auf erdogans offensive </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kurden flüchten aus angst vor der türkischen armee </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kurfürstendamm: aufstieg und verfall des berliner prachtboulevards </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kurzarbeit: sorgen der betroffenen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> kurzfilm-dokus: liebe in zeiten von corona </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> labour-partei bricht brexit-gespräche in london ab </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> läden in österreich dürfen wieder öffnen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ladenhüter astra zeneca: was tun mit dem liegengebliebenen impfstoff </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> lady in yellow: amanda gorman und ihr gedicht zur amtseinführung bidens </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> lage auf lesbos bleibt angespannt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> lage auf lesbos spitzt sich zu </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> lage der flüchtlinge an der griechisch-türkischen grenze </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> lage im corona-hotspot moselle an der französisch-deutschen grenze </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> lage im sudan: junge aktivisten demonstrieren gegen diktator al-bashir </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> lage im türkisch-syrischen grenzgebiet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> lage in den usa nach der polizeigewalt gegen einen schwarzen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> lage in syrien: türkei droht mit öffnung der grenzen für flüchtlinge </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> lambrecht will verstärkt gegen hass im netz vorgehen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> land- und forstwirtschaft befürchten dürre </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> länder diskutieren nach anstieg der corona-neuinfektionen über einheitliche maßnahmen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> länder mit regelung zur sterbehilfe: beispiel schweiz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> landleben in zeiten von corona </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> landtag thüringen wählt fdp-kandidaten kemmerich zum neuen regierungschef </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> landtagswahl in brandenburg: spd trotz verlusten vor afd </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> landtagswahl in sachsen:  cdu stärkste kraft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> landwirte brechen zu protesten nach berlin auf </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> langjährige haftstrafen für unterstützer des anschlags auf &quot;charlie hebdo&quot; 2015 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> langzeitbeobachtung von 5000 lernenden im lockdown </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> langzeitfolgen für covid-19-patienten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> langzeitstudie über soldaten im afghanistan-einsatz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> lawinengefahr in österreich und bayern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> lawinengefahr: unterwegs mit einem experten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> lebenslange haftstrafe für hauptangeklagten im lübcke-mordprozess </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> lebensmittelverschwendung nach weihnachten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> lehre aus corona-lockdown: inder fordern mehr umweltschutz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> lehrer fordern mehr schutz vor corona an den schulen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> lehrer in nordrhein-westfalen dürfen sich selbst auf corona testen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> leichtathletik-wm in doha: leere zuschauerränge und unglaubliche hitze </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> leipziger buchmesse mit festakt eröffnet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> lernen in corona-zeiten - schulstart in mecklenburg-vorpommern nach den sommerferien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> letzte amtshandlung im eu-parlament: britische eu-abgeordnete packen ihre koffer in brüssel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> letzte diskussionsrunde der cdu-kanzlerkandidaten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> letzter einkaufstag vor weihnachten in berlin </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> letzter flug vom berliner flughafen tegel gestartet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> letzter royaler auftritt von harry und meghan in london </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> letzter spieltag in der 2. fußball-bundesliga </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> letztes tv-duell zwischen trump und biden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> libyen-konferenz einigt sich auf plan zur konfliktlösung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> libyen-konferenz in berlin: stopp aller waffenlieferungen und aufruf zu einem dauerhaften waffenstillstand </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> libyens ministerpräsident al-sarraj wirbt in europa um unterstützung im kampf gegen aufständische </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> licht und schatten der e-mobilität </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> linken-parteitag streitet über eu-kurs </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> linkspartei wählt neues führungsduo </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> literatur-nobelpreisträgerin toni morrison gestorben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> lockdown im berchtesgadener land scheint erste erfolge zu zeigen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> lockdown-debatte: warum die maßnahmen gegen das coronavirus aufrechterhalten werden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> lockdown-ende: wuhans vorsichtige rückkehr in den alltag </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> lockdown-soundtrack: lieder aus dem nonnen-kloster </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> lockern statt lockdown: wie der stillstand bald beendet werden könnte </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> lockerung der corona-beschränkungen in österreich </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> london wendet sich im hongkong-konflikt von peking ab </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> london: mehr als ein drittel der kinder leben in armut </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> londonderry: strafverfolgung gegen &quot;bloody sunday&quot;-schützen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> löschung von handy-daten der ex-verteidigungsministerin von der leyen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> lösungsfinder: wie ein gesundheitskiosk in hamburg versorgungslücken auffängt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> lufthansa börse an </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> lufthansa kündigt &quot;absturz&quot; ohne staatliche hilfe an </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> lufthansa-rettung: hauptversammlung der aktionäre </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> lufthansa: 22.000 jobs auf der kippe </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> luftverkehr: der traum vom klimaschonenden fliegen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> lügde: staatsanwaltschaft erhebt anklage gegen zwei männer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> lukaschenko lässt vor wahl wichtigsten herausforderer verhaften </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> lyrikerin louise glück erhält literaturnobelpreis </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> macht des geldes im us-wahlkampf: michael bloomberg gegen donald trump </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> machtkampf in libyen: vereinte nationen fordern humanitäre korridore für zivilisten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> machtkampf in russland zwischen nawalny und putin </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> machtkampf in venezuela </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> machtkampf in venezuela dauert an </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> machtkampf in venezuela droht zu eskalieren </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> machtkampf in venezuela: guaidó will entscheidung erzwingen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> machtkampf in venezuela: militär warnt vor bürgerkrieg </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> machtkampf zwischen erdogan und imamoglu </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> machtpolitik: russland und die usa mischen sich in venezuela ein </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> macron lädt staatschefs der sahelzone zu anti-terror-gipfel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> macrons vision und europäische realität </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> madeira: 29 menschen sterben bei busunglück </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> malawi will versorgung mit drohnen sichern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> maler sean scully gibt karlsruhe einen korb </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mallorquinische tourismusbranche wegen steigender corona-zahlen vor reisewarnung besorgt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> malta: die schleppenden ermittlungen im mordfall galizia </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mangel an pflege-fachkräften: gesundheitsminister spahn auf werbetour im kosovo </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mangelnde aufsicht in pflege-wgs </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> markus preiß mit einer einschätzung aus brüssel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> marokko: bildung soll beim bleiben helfen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> marsch auf washington: tausende menschen demonstrieren gegen rassismus und polizei-gewalt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> masernausbruch: drastische maßnahmen in hildesheim </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> maske auf im einzelhandel? die sicht der kunden und ladenbesitzer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> maskenaffäre in der union: wenn die krise zum geschäft wird </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> maskenangebot und maskenzwang: viele wollen sie </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> maskenpflicht auf den ballearen nach missachtung der corona-maßnahmen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> maskenpflicht in deutschland: so lief der erste tag </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> maskenverweigerer in der schule: die nöte der lehrer und schulleiter </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> massenandrang auf dem mount everest fordert tote </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> massenandrang in österreichs skigebieten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> massenprotest gegen neue umweltauflagen: wie ein dialog zwischen landwirten und naturschützern gelingen kann </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> massiver datendiebstahl: 20-jähriger gesteht tat </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> massiver hackerangriff auf us-behörden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> massiver protest gegen rentenpläne: generalstreik legt frankreich lahm </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mauerstreit: trump und der kompromissvorschlag </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> may bringt verschiebung des brexits ins spiel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> may will weitere brexit-verschiebung beantragen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mazedonien und griechenland streiten um einen namen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mecklenburg-vorpommern erlaubt öffnung von gastronomie </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mecklenburg-vorpommerns innenminister tritt wegen waffen-affäre zurück </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mediziner kämpfen gegen ebola-epidemie in der demokratischen republik kongo </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> medizinische lage im nordsyrischen camp al-hol </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> meer-lust: wie die deutschen kreuzfahrtschiffe wieder in see stechen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> megaprojekt: türkei plant kanalbau zwischen schwarzem und marmara-meer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mehr als 1.000 kindesmissbrauchsfälle auf campingplatz in nrw aufgedeckt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mehr als 1000 infizierte: massive kritik an fleisch-produzent tönnies </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mehr als ein denkzettel: akp verliert istanbul und ankara </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mehr aufwand als ertrag?: vom sinn und unsinn der mehrwertsteuersenkung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mehr ausgeben fürs fleisch?- was die verbraucher denken </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mehr flüchtlinge erreichen die eu über die türkei </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mehr videokonferenz und weniger dienstreisen durch corona </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mehrere europäische länder wollen flüchtlinge der &quot;sea watch 3&quot; aufnehmen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mehrere industrieunternehmen bauen eigenes 5g-netz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mehrheit der russen stimmt für verfassungsreform </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> meinung von markus preiß </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> meinungen zur offiziellen corona-warn-app des bundes </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> menschen meiden arztbesuche wegen corona </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> menschen mit behinderung: werkstätten drängen auf weiterbetrieb </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> menschenunwürdige zustände im flüchtlingslager vucjak in nordbosnien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> menschenunwürdige zustände in flüchtlingslagern: un fordert bei globalem flüchtlingsforum mehr einsatz für menschenrechte </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> menschenversuche an ddr-freizeitsportlern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> menschmaschinen: chancen und risiken von künstlicher intelligenz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> merkel bei erdogan: annäherung in schwierigen zeiten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> merkel hält eindringlichen appell an die bevölkerung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> merkel im bundestag: regierungserklärung zur corona-krise </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> merkel ruft zu wachsamkeit und zusammenhalt gegen corona auf </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> merkel verteidigt corona-maßnahmen im bundestag </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> merkel zur corona-krise: appell auf nicht notwendige sozialkontakte zu verzichten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> merkels ansprache zur bevölkerung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> merz und röttgen stellen sich jungen union </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> messenger-dienst &quot;telegram&quot; als radikalisierungsmedium </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mexiko benötigt dringend eine corona-strategie </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mexiko kämpft mit corona-pandemie </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mexikos hoffnung auf den wahlsieg von joe biden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> michel houellebecqs dystopie &quot;serotonin&quot; und die wirklichkeit in der französischen provinz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mieten-explosion: wohnungsnot verschärft auch fachkräftemangel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mietwahnsinn in der hauptstadt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> migranten erleben dramatische zustände an der us-mexikanischen grenze </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mikroplastik: wie die kleinen teilchen unser leben bestimmen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> miliz-offensive: truppen rücken auf libysche hauptstadt tripolis vor </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> milliarden für den klimaschutz: union und spd stecken positionen für gesamtpaket ab </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> milliarden steuerloch: eine schadensmeldung und die konsequenzen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> millionenfund im lager der dresdner kunstsammlung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> milllionen kinder weltweit in not durch schwerhörigkeit </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mindestens 49 tote nach anschlag auf moscheen in neuseeland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mindestens sieben tote und 21 vermisste nach schiffsunglück in budapest </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> minister unter druck: kritik der industrie an peter altmaier </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ministerpräsident von schleswig-holstein über corona-maßnahmen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ministerpräsidenten einigen sich auf gemeinsame linie im kampf gegen corona </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> missbrauchsfall in bergisch gladbach vermutlich größer als in lügde </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> missbrauchsskandal in bergisch-gladbach: über 30.000 hinweise zu tatverdächtigen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> missbrauchsvorwürfe gegen prinz andrew </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mit einer handy-app gegen die pandemie: sinnvoll oder nicht? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mit sicherheit ins kühle nass: aktionswoche gegen sinkende schwimmfähigkeit von kindern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mittelstand: gemischte erwartungen für 2019 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mittendrin: anwohner des erfurter wohnviertels herrenberg wehren sich gegen rechtsextremismus in der nachbarschaft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mittendrin: streit um galopprennbahn in bremen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mitterteich verhängt als erster ort in deutschland ausgangssperre </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mobilitäts-flatrate in augsburg </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mode-ikone karl lagerfeld ist tot </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> moderne verkehrsprojekte in deutschland: seilbahnen und fliegende busse </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> modernisierung der wasserstraßen per &quot;masterplan binnenschifffahrt&quot; geplant </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mönchengladbach gegen 1. fc köln endet 2:1 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> monsanto: der klotz am bein von bayer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mord an georgier in berlin: ein hauch von kaltem krieg? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> morddrohungen gegen politiker: was macht das mit den betroffenen? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mordfall lübcke: ermittler gehen hinweisen auf mittäter nach </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mordfall lübcke: verdächtiger hatte offenbar enge kontakte in die neonazi-szene </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> moskau weist drei eu-diplomaten aus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> motorradlärm - der schwarzwald will seine ruhe zurück </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mueller-bericht: keine beweise für absprachen zwischen trump und russland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> münchen feiert ersatz-wiesn trotz hoher corona-infektionszahlen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> münchener museen projizieren kunstwerke mit beamern nach draußen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> münchener staatsanwaltschaft erhebt anklage gegen ehemaligen audi-chef stadler </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> münchner sicherheitskonferenz beginnt mit forderung nach militärisch stärkerem europa </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> münchner sicherheitskonferenz: steinmeier beklagt egoismus der großmächte </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> munitionsaffäre bei elitetruppe der bundeswehr: kramp-karrenbauer räumt fehler ein </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> munitionsräumung nach waldbränden in lübtheen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> musik sichtbar machen: beethovens fünfte für gehörlose </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> musik-lieferdienst: eine danksagung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> musikalische demonstration: ein festival gegen maduro - und eines für ihn </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> musikwelt trauert um us-opernstar jessye norman </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> muslimische dating-app &quot;hawaya&quot; auf erfolgskurs </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mutation des coronavirus einer der gründe für strengeren lockdown </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mutiertes coronavirus dominiert das infektionsgeschehen in flensburg </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mutmaßlich rechtsextremes netzwerk bei nrw-polizei aufgeflogen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mutmaßlicher rechtsextremist erschießt zwei menschen in halle (saale) </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mutmaßlicher täter der cyberattacke soll aus youtuber-szene stammen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mutmaßlicher terroranschlag in wien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> myanmars militär geht verschärft gegen proteste vor </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nabucco aus dem hausarrest: wie ein festgesetzter russischer regisseur die premiere in hamburg inszeniert </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach angriff auf ölanlagen in saudi arabien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach angriffen auf us-botschaft in bagdad schickt die us-armee verstärkung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach anschlag auf zwei moscheen in neuseeland: gedenken an die opfer des terroranschlags </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach anschlag in halle </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach anschlag in halle: ermittler gehen von einem anti-semitischen motiv aus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach anschlag in halle: solidaritätskonzert für die opfer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach anschlag in halle: synagoge bekommt eine neue tür </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach anschlag in neuseeland wird mutmaßlicher täter angeklagt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach brand in notre-dame: computerspiel könnte bei rekonstruktion für wiederaufbau helfen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach brandkatastrophe in moria weiterhin tausende menschen ohne unterkunft und verpflegung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach corona-ausbrüchen in schlachtbetrieben: bundesregierung beschließt weitgehendes verbot von werkverträgen in der fleischindustrie </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach dem brand in moria </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach dem coronaausbruch im hanns-lilje-heim in wolfsburg </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach dem is-terror: noch immer großes leid in syrien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach dem spd-mitgliederentscheid: zukunft der groko ist offen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach dem sturz von diktator al-baschir im sudan: chancen und risiken für das land </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach dem tod eines schwarzen us-amerikaners durch weißen polizisten: proteste und ausschreitungen in minneapolis </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach dem wirbelsturm &quot;idai&quot;: die schwierige versorgung der menschen in mosambik </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach den landtagswahlen in brandenburg und sachsen: wie weiter mit der großen koalition? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach den terroranschlägen in sri lanka </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach der einigung von thüringen: was sagt die bundes-cdu? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach der invasion in nordsyrien: usa verhängen sanktionen gegen die türkei </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach der rechtsradikalen inszenierung vor dem reichstag: debatte um die sicherheit des bundestages </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach der us-wahl: neue wirtschaftspolitik? was bidens sieg für deutsche unternehmen bedeutet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach der wahl in großbritannien: johnson macht tempo </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach der wahl ist vor der abschiebung: griechenland und die flüchtlinge </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach drohnenangriff auf saudische ölanlagen: debatte in den usa über reaktionen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach einem jahr corona-pandemie droht nun die dritte welle </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach eskalationen im sudan: milizen gehen gewaltsam gegen oppositionelle und zivilisten vor </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach eurofighter-absturz: die umstrittenen übungsflüge </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach explosion in beirut: lage für syrische flüchtlinge im libanon spitzt sich zu </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach explosion in beirut: libanesische regierung tritt zurück </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach hurrikan: aufräumarbeiten auf den bahamas </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach ibiza-affäre - österreichs parlament stürzt kanzler kurz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach impfgipfel: nationaler impfplan soll impfkampagne besser koordinieren </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach kritik von wirtschaftsverbänden will bundeswirtschaftsminister altmaier öffnungsstrategie erarbeiten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach massiver explosion in beirut: wie hart trifft die katastrophe die menschen im libanon? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach militäraktion: lage der kurden in der türkei </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach mohring-rücktritt: wohin steuert die cdu in thüringen? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach mord an georgier in berlin: bundesregierung weist zwei russische diplomaten aus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach mordfall lübcke: diskussion um schutz von politikern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach präsidentschaftswahlen: furcht vor neuer instabilität im kongo </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach prügelattacke gegen schiedsrichter: debatte über mehr sicherheit im amateurfußball </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach rekordminus: deutsche wirtschaft hofft auf langsame erholung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach selbstverbrennung: proteste gegen stadionverbot für frauen im iran </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach skandal um wursthersteller wilke kommen zweifel an kontrollmechanismen auf </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach sturm auf kapitol in washington: sorge vor weiteren unruhen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach terroranschlag in wien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach terroranschlag von halle: bundestag debattiert über antisemitismus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach thomas-cook-pleite: suche nach lösung für konzerntöchter in deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach thüringen-krise: das verhältnis zwischen cdu und die linke </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach verhandlungsmarathon: koalition einigt sich auf paket zum klimaschutz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach verheerendem dammbruch: die wut der angehörigen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach wahl der neuen spd-führung: diskussion über zukunft der großen koalition </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach wahldebakel in thüringen: cdu diskutiert über kurs und führung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach zahlreichen corona-fällen konsequenzen für arbeitsbedingungen in schlachthöfen gefordert </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nach zyklon &quot;idai&quot;: helfer sprechen von „massiver katastrophe“ in mosambik </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nachhaltig reisen: greta-effekt auch bei der urlaubsplanung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nachlass und versöhnung: das neue fassbinder-center </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nachruf auf den schwarzen bürgerrechtler john lewis </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nachtragshaushalt und rekordverschuldung: fragen an bundesfinanzminister olaf scholz zur finanzierung der neuverschuldung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nachverfolgung von infektionsketten überlastet die gesundheitsämter </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nah an der obergrenze - wie der corona-hotspot rosenheim gegen das virus kämpft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nahles gibt amt als fraktionsvorsitzende der spd auf </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nahles unter druck: vorsitzende muss sich in der spd-fraktion erklären </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nasa-rover &quot;perseverance&quot; landet auf dem mars </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nasa: erstmals rein weiblicher außeneinsatz an der iss </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nation league: deutschland und spanien trennen sich unentschieden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nationale gedenkfeier in frankreich nach islamistisch motiviertem mord an lehrer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nationale industriestrategie: bundeswirtschaftsminister altmaier will schlüsselindustrie schützen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nationaler volkskongress in china beginnt mit fokus auf militärische und wirtschaftliche entwicklung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nationalfeiertag in frankreich: präsident macron dankt helfern in der corona-krise </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nations league: deutschland und die schweiz trennen sich unentschieden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nato berät über truppenabzug aus afghanistan </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nato-jubiläumsgipfel in london </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nato-jubiläumsgipfel in london endet mit gemeinsamer abschlusserklärung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nazinotstand in dresden? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neu im kino: dokumentation &quot;diego maradona&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neubau im weltkulturerbe - james-simon-galerie in berlin eröffnet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neubauten in deutschland für viele mieter nicht bezahlbar </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neue  verteidigungsministerin: die stimmung bei der bundeswehr </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neue &quot;orthodoxe kirche der ukraine&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neue ausstellung in &quot;gedenkstätte deutsche teilung marienborn&quot; eröffnet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neue beschränkungen nach steigenden infektionszahlen in deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neue bnd-zentrale in berlin eröffnet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neue corona-rekordzahlen überfordern gesundheitsämter </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neue details im fall des cyberbunkers in rheinland-pfalz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neue erkenntnisse im missbrauchsfall von lügde </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neue friedensinitiative für den persischen golf? - der iranische präsident rohani spricht vor den vereinten nationen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neue hoffnung: klinische studie zu einem corona-impfstoff </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neue impfverordnung: debatte um priorisierung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neue landwirtschaft: kommission soll zukunft diskutieren </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neue pisa-studie: deutsche schüler schneiden wieder schlechter ab </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neue regeln für schiffsdiesel sollen ausstoß von schwefeldioxid verringern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neue richtlinien beim  urheberrecht: die folgen für verlage und user </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neue screening-methode im kampf gegen gebärmutterhaltskrebs </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neue studie belegt die wirksamkeit des russischen corona-impfstoffs sputnik v </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neue studie zur gletscherschmelze in grönland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neue waffenruhe im konflikt um bergkarabach nach us-amerikanischer vermittlung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neue wortbildungen in der corona-krise </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neue zerreißprobe für die eu: können die regierungschefs polen und ungarn noch umstimmen? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neuer &quot;speaker&quot; im londoner unterhaus: der skurrile mister hoyle </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neuer corona-hotspot: coesfeld und die fleischindustrie </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neuer internet-trend: &quot;trash-challenge&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neuer pflanzenhorrorfilm &quot;little joe&quot; kommt in die kinos </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neuer plan: warum der soli schon bald für (fast) alle abgeschafft werden soll </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neuer premierminister: johnson will briten selbstvertrauen zurückgeben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neuer schulalltag nach schulschließungen: herausforderungen für lehrende und familien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neuer star-wars-film im kino </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neuer us-kongress tagt erstmals </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neues banksy-kunstwerk </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neues bauhaus-museum in dessau vor der eröffnung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neues europäisches urheberrecht: proteste von internet-aktivisten und streit in der koalition </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neues täterprofil: der amokläufer als held rechter netzwerke </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neues tierwohl-label: diskussion um klöckners vorschlag </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neuinszenierung der &quot;walküre&quot; in der deutschen oper berlin feiert trotz corona premiere </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neujahrsfest in china: millionen auf dem weg zur familie </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neuneinhalb jahre haft für angeklagten nach messerattacke von chemnitz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neuseeland ist fast corona-frei </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neustart der bundesliga sorgt weiterhin für diskussionen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neustart: was bringt die generalistische pflegeausbildung? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neuverfilmung von &quot;berlin alexanderplatz&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> neuwahlen in großbritannien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> new horizons: rekord-rendevouz im weltall </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> new york entwickelt sich zum corona-zentrum amerikas </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> new york ist besonders schwer vom coronavirus betroffen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> newcomer im us-kongress wollen politik umkrempeln </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> niederlande: europas ältestes naturhistorisches museum in leiden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> niki lauda: legende der formel 1 gestorben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nobelpreis für chemie an drei batterie-forscher verliehen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nobelpreis für physik für die erforschung von schwarzen löchern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nobert blüm ist tot </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nominierungsparteitag der us-demokraten: michelle obama übt scharfe kritik an trump </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> norbert röttgen gibt kandidatur für cdu-parteivorsitz bekannt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nordsyrien: feuerpause wird nicht eingehalten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> not der flüchtlinge: warum eltern in idlib ihre kinder ins waisenhaus geben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> not macht erfinderisch: kreative ideen von selbständigen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nöte der wanderarbeiter in china: herr qu fährt in sein überflutetes dorf </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> notfall-rettung: versorgung auf dem land durch den tele-notarzt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> notre dame: schwieriger abbau des alten gerüst </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nrw-innenminister reul für bekanntmachung der nationalität von tatverdächtigen bei pressemitteilungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> nsu 2.0: ehepaar aus bayern wegen drohmails an politiker und prominente vorläufig festgenommen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> obdachlosigkeit in der corona-krise </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> öffnung des städel museums in frankfurt am main </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> öffnungsdiskussionen: warum deutschland keine einheitlichen regeln schafft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> öffnungsstrategien der bundesregierung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> oktoberfest 2020 fällt aus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> olaf scholz äußert sich zu seiner kandidatur für den spd-vorsitz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> olympia-frust: wie tokio die ausgefallene eröffnungsfeier verarbeitet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> olympische spiele wegen corona in gefahr? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> olympische spiele werden auf 2021 verschoben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> online-casinos: bundesländer einig über entwurf für neuen staatsvertrag zum glücksspielwesen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> op-art-ausstellung im kunstmuseum in stuttgart </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> opioid-epidemie in den usa </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> opposition in venezuela demonstriert für humanitäre hilfe </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> opposition in venezuela resigniert schon vor der parlamentswahl </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> oppositionskandidatin tichanowskaja aus belarus nach litauen geflüchtet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> oppositionspolitiker alexej nawalny darf nach deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> oppositionspolitikerin maria kolesnikowa in belarus in haft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> orcas attackieren segelboote vor spanien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> organisationschaos erwartet: deutschland muss rasch auf &quot;turbo&quot; schalten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> orkan hinterlässt weniger schäden als befürchtet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ortsteil gadheim von veitshöchheim in bayern wird neuer geographischer mittelpunkt der eu </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ost-strategien von cdu und spd </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ostern in corona-zeiten: familienfest ohne familie </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> österreich auf kurssuche nach skandal um fpö-vorsitzenden und vizekanzler strache </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> österreich beendet harten lockdown </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> österreich schränkt durch corona-maßnahmen tourismus stark ein </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> österreich vor der nationalratswahl: ex-kanzler kurz schließt koalition mit fpö nicht aus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ötzi in der uckermark: frauenskelett bringt viele neue erkenntnisse </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> overtourism: kreuzfahrtschiffe vor civitavecchia </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> pakistan und indien melden abschuss von kampfjets über kaschmir </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> palliativmediziner fordern andere behandlung von unheilbar erkrankten covid-19-patienten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> palmen oder parkbank: start in die sommerferien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> pannenflughafen ber: keiner fliegt mehr-ausgerechnet jetzt kommt die freigabe </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> papst als umweltschützer: amazonas-synode im vatikan </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> papst franziskus besucht erstmals die arabische halbinsel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> papst franziskus setzt irakbesuch fort und ruft zu widerstand gegen gewalt und extremismus auf </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> papst gedenkt in rom der anschlagsopfer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> papst prüft vertuschungsvorwürfe gegen erzbischof woelki </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> papstbesuch in marokko </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> pariser kathedrale notre-dame brennt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> parlament prüft rechtliche konsequenzen auf störungen im bundestag </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> parlament verschiebt brexit-abstimmung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> parlamentswahl in dänemark - wie die sozialdemokraten punkten wollen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> parlamentswahl in der ukraine: partei von präsident selenskyj gewinnt laut prognose </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> parlamentswahl in großbritannien: erdrutschsieg für boris johnson </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> parlamentswahl in indien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> parlamentswahl in israel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> parlamentswahl in israel bringt keine klaren machtverhältnisse </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> parlamentswahlen in indien gehen in die zweite phase </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> parlamentswahlen in polen: regierungspartei pis baut vorsprung aus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> parteien holen zu verbalattacken auf politische gegner aus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> parteien stellen sich die kanzlerkandidatenfrage </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> parteitag der konservativen: johnson unterbreitet brüssel einen neuen vorschlag </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> parteitag in braunschweig: afd-parteispitze schwört mitglieder auf regierungsfähigkeit ein </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> partielle mondfinsternis </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> pekings corona-propaganda </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> pendlerreportage aus südbaden: die situation an den grenzen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> per rad durch die republik: reise durch ein anderes land </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> perfekter &quot;hole-in-one&quot;-schlag von golfprofi rahm </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> peru erobert die gourmetwelt: haute cuisine im armenviertel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> peter handke und olga tokarczuk erhalten literatur-nobelpreise </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> peter lindbergh ist mit 74 jahren gestorben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> peter sellars inszeniert mozarts &quot;idomeneo&quot; bei den salzburger festspielen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> pfingsturlaub mit abstand: ausgebuchte campingplätze </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> pflegeheime verhängen besuchsverbote </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> pflegekräfte einer intensivstation dokumentieren corona-alltag in einem video-tagebuch </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> pfleger dringend gesucht: wo die kliniken ihr personal rekrutieren </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> pflichttests für rückkehrer aus risikogebieten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> pharmakonzern novartis verlost gentherapie für 100 todkranke babys </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> philosoph und politikwissenschaftler prof. nida-rümelin mit einer einschätzung des beherbergungsverbots </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> pieter bruegel der ältere: brüssel feiert den berühmten sohn </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> pilotprojekt balearen: urlaub auf mallorca für deutsche ab montag möglich </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> pilotprojekt in nordwest-mecklenburg: impfungen nun auch in arztpraxen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> pitt und tarantino plaudern über ihren neuen film </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> pitt und tarantino plaudern über ihren neuen film&quot; darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> plan für kohleausstieg stößt auf breite kritik von opposition und umweltverbänden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> pläne der spd: neues kindergeld-konzept soll sozialer sein </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> pläne von minister scheuer: milliarden für die verkehrswende </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> pleite thomas cook: reisekonzern verhandelt über weiteres geld </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> polen vor der stichwahl: der schmutzige wahlkampf </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> polen will abtreibungsgesetz verschärfen - frauen suchen hilfe im ausland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> polens präsident duda gewinnt knapp wiederwahl </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> politik </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> politik auf der straße: wahlkampf trifft auf klimaschutz-demonstrationen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> politik berät über strategien zur bewältigung der corona-erkrankten zur grippezeit im herbst und winter </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> politik debattiert über zukunft von nord stream 2 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> politik in zeiten der corona- pandemie </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> politiker debattieren über vereinheitlichung von corona-regelungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> politiker der grünen und fdp kritisieren aktuelle teillockdown-strategie </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> politische kehrtwende: usa akzeptieren jüdische siedlungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> politische kooperation in zeiten von corona </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> politische korrektheit: niederlande streiten um &quot;schwarzen piet&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> politische kultur: die macht der straße </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> politische reaktionen aus berlin auf die gewalttat </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> politischer kampf im us-rentnerparadies florida </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> politischer showdown in westminster </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> politischer spagat: das programm der neuen spd-führung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> politischer stillstand nach den heutigen wahlen in israel wahrscheinlich </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> politischer umgang mit verschwörungstheorien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> polizei geht gegen hochzeitskonvois vor </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> polizei in münster ermittelt gegen kindermissbrauchs-netzwerk: elf tatverdächtige verhaftet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> polizei räumt breidscheidplatz und nimmt verdächtige fest </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> polizei räumt demonstrationen gegen corona-politik in berlin nach regelverstößen und ausschreitungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> polizeigewalt in den usa: positivbeispiel camden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> polizeiskandal in nrw: ein ehemaliger polizeischüler berichtet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> popkömodie &quot;yesterday&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> porträt von joe biden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> position der grundrente noch immer ungeklärt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> positive bilanz nach ende des probebetriebs am flughafen ber </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> post-brexit-verhandlungen: treffen von johnson und von der leyen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> pränatale diagnostik  - warum die bluttests auf trisomie 21 umstritten sind </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> präsident des robert koch-instituts wieler rät zu einhaltung und verschärfung der maßnahmen gegen die verbreitung des coronavirus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> präsidenten- und parlamentswahlen in uganda: präsident musevenis herausforderer bobi wine ist hoffnungsträger der jugend </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> präsidenten-wahl: exodus in guatemala </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> präsidentenwahl in afghanistan: geringe wahlbeteiligung nach drohungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> präsidentschaftsanwärter der demokraten in den usa </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> präsidentschaftskandidat biden will sich an nation wenden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> präsidentschaftswahl im politisch angespannten belarus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> präsidentschaftswahl in bolivien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> präsidentschaftswahl in der ukraine: polit-neuling selenskij gewinnt ersten wahlgang </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> präsidentschaftswahl in polen: kopf-an-kopf-rennen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> präsidentschaftswahl usa: erste tv-debatte der demokratischen kandidaten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> präsidentschaftswahlen in bolivien und die deutschen lithium-verträge </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> präsidentschaftswahlen in polen: amtsinhaber duda verpasst absolute mehrheit </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> preisanstiege bei obst und gemüse </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> preisträger der berlinale 2021 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> preisverleihung bei den 70. internationalen filmfestspielen in berlin </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> premier johnsons verordnete parlamentspause stößt auf massiven widerstand </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> premier-league: fc liverpool ist meister </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> premierministerin may kündigt rücktritt an </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> premierministerin may muss plan b für den brexit vorlegen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> prinz harry und meghan geben teil ihrer royalen verpflichtungen ab </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> privatfeiern wahrscheinlich für die steigenden corona-infektionen verantwortlich </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> problematische betreuung von menschen mit behinderung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> probleme bei privatisierung des gesundheitssystems </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> profitieren von der eu: wahlsdorf zeigt wie's geht </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> prognosen zur bremenwahl </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> promi-botschaften für schulabsolventen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> protest der &quot;mütter von srebrenica&quot; gegen nobelpreis für peter handke </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> protest gegen abitur: petition kritisiert mathematik-prüfungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> protest und parteitag: was die polizeigewalt in wisconsin für den us-wahlkampf bedeutet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> protest und revolutions-erinnerung: demonstrationen in prag </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> protestbewegung im libanon fordert politisch runderneuerten staat </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> proteste für faire wahlen: wieder hunderte festnahmen in moskau </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> proteste gegen &quot;südlink&quot;-stromtrassen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> proteste gegen corona-maßnahmen: esoteriker und rechtsextreme demonstrieren nebeneinander </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> proteste gegen die inbetriebnahme des steinkohlekraftwerk datteln 4 im ruhrgebiet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> proteste gegen geplante stromtrasse &quot;suedlink&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> proteste gegen militär-junta in myanmar werden von buddhistische mönchen unterstützt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> proteste gegen pekings sicherheitsgesetz in hongkong </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> proteste gegen präsident keita: mali auf dem weg ins chaos </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> proteste gegen präsident lukaschenko in belarus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> proteste gegen präsident lukaschenko in belarus: mehr teilnehmer als jemals zuvor </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> proteste gegen rassismus und polizeigewalt - plünderungen in new york </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> proteste hongkong: stadtregierung nimmt mehrere aktivisten vorübergehend fest </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> proteste im iran nach flugzeugabschuss </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> proteste in algerien gegen neue amtszeit von präsident bouteflikas </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> proteste in belarus: lukaschenkos strategie und putins interessen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> proteste in belarus: regierung lässt 2000 inhaftierte demonstranten frei </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> proteste in belarus: tausende festnahmen und erster toter nach protesten gegen präsident lukaschenko </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> proteste in hongkong: hunderte demokratie-aktivisten verschanzen sich in universität </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> proteste in hongkong: lage spitzt sich zu </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> proteste in mehreren us-städten nach tod von schwarzem bei polizeieisatz in minneapolis </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> proteste in minsk gegen staatschef lukaschenko von frauen angeführt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> prozess gegen israels premierminister netanjahu </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> prozessauftakt gegen den mutmaßlichen attentäter von halle: wie umgehen mit rechtsextremer gewalt? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> prozessauftakt gegen hollywood-filmproduzent weinstein </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> prozessauftakt gegen rechtsextreme gruppe &quot;revolution chemnitz&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> prozessauftakt im missbrauchsfall von lügde </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> prozessauftakt in koblenz: anklage wegen verbrechen in syrien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> prozessbeginn gegen mutmaßliche komplizen des attentats auf satirezeitschrift &quot;charlie hebdo&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> prozessbeginn im fall des ermordeten kasseler regierungspräsidenten lübcke </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> prozessbeginn in neuseeland nach rassistischen anschlägen mit 51 toten in christchurch </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> prozessbeginn nach mord an enthüllungsjournalist und seiner verlobten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> psychologe erläutert die bedenken der impfgegner </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> putin und erdogan einigen sich auf verlängerung der waffenruhe </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> putins macht: die stärke russlands im nahen osten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> putsch in myanmar: regierungschefin aung san suu kyi festgenommen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> queen elizabeth ii. hält rede an die nation </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> queen verliest regierungserklärung von johnson </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> quoten-diskussion: wieviel frau braucht die cdu? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ramadan in deutschland: warum viele muslimische schüler fasten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ramelow wird im dritten wahldurchgang wieder thüringer ministerpräsident </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rapper bushido verliert rechtsstreit </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rassismus bei der polizei: der politische streit um die racial-profiling-studie </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rassismus in belgien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rassismus-vorwürfe gegen bremer feuerwehr </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rassistische morddrohung gegen frankfurter anwältin </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ratlosigkeit und mögliche auswege </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rätsel um brand im russischen u-boot: kreml stuft erkenntnisse zur havarie als staatsgeheimnis ein </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> raumfahrt-pionier sigmund jähn gestorben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> räumung in berlin: wie gefährlich sind die autonomen? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rbb-recherche: querdenker-bewegung offenbar mit verbindungen zur szene der sogenannten reichsbürger </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> re:publica: wie die internetgiganten reguliert werden sollen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> reaktionen auf beschlüsse der koalition zum klimaschutz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> reaktionen auf beschlüsse des corona-gipfels und vertagte entscheidungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> reaktionen auf das freihandelsabkommen zwischen eu und mercosur </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> reaktionen auf den machtwechsel in der usa </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> reaktionen auf die ministerpräsidenten-wahl aus berlin </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> reaktionen auf die us-wahl aus deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> reaktionen auf lockerungen in der corona-krise </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> reaktionen auf neue corona-beschlüsse </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> reaktionen aus deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> reaktionen aus deutschland zu dieser geschichtsträchtigen wahl </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> reaktionen aus tripolis </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> reaktionen der bürger dazu </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> reaktionen in brüssel auf brexit-abstimmung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> reaktionen von briten in deutschland auf brexit-abstimmung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rebellenkämpfer in nordsyrien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> recherche von ndr und wdr: wikileaks-gründer assange in ecuadorianischer botschaft in großem ausmaß ausspioniert </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> recherche zu datenleak &quot;fincen-files&quot;: gravierende probleme im kampf gegen geldwäsche </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> recherche zu rechtsextremismus: &quot;revolution chemnitz&quot; und die folgen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> recherche-netzwerk bekräftigt verdacht auf hetzjagden in chemnitz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> recherche: frontex-menschenrechtsverletzungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> recherche:wirecard-skandal und die hinweise auf verstrickungen in mafia-geschäfte </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> recherchen der nazijäger in ludwigsburg </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> recherchen über verdeckte parteispenden belasten österreichischen vizekanzler strache </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> recherchen zu afd-spendenaffäre führen zu deutscher milliardärsfamilie </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> recherchen zu spannervideos im internet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rechter terror in deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rechtsextremismus: afd-vorstand verlangt &quot;flügel&quot;-auflösung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rechtsextremistische tendenzen: afd brandenburg als verdachtsfall eingestuft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rede zur eu-ratspräsidentschaft: merkel betont bedeutung der grundrechte für europa </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rededuell um das ukrainische präsidentenamt vor entscheidender stichwahl am sonntag </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> referendum zu verfassungsänderung in ägypten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> reform des infektionschutzgesetzes verabschiedet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> reformen in der katholischen kirche: wie kommen die vorschläge an? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> regierung beschließt gesetze zur bekämpfung von rechtsextremismus und hasskriminalität im internet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> regierung und lufthansa signalisieren baldiges ende der verhandlungen eines rettungspakets </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> regierungserklärung gesundheitsminister spahn: im sommer gibt es genügend corona-impfstoff </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> regierungskrise in italien - lega-chef salvini fordert neuwahlen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> regierungskrise in italien: ministerpräsident conte erklärt rücktritt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> regierungskrise in österreich löst vorgezogene neuwahlen aus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> regierungskrise in österreich: bundeskanzler kurz will vorgezogene neuwahlen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> regierungskrise in österreich: was wird aus kanzler kurz? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> regierungskrise in sachsen-anhalt spitzt sich zu </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> regierungskrise in thüringen: offizieller rückzug von kemmerich </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> regierungskrise in thüringen: ramelow schlägt lieberknecht als übergangsregierungschefin vor </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> regierungskrise italien: senat bremst entscheidung über neuwahlen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> regierungskritische proteste in hongkong </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> regierungskritische proteste in russland fordern freilassung von kreml-kritiker nawalny </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> regierungspartei anc gewinnt wahlen in südafrika </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> regional-reportage: corona-hilfe auf dem land </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> regionale unterschiede bei den corona-lockerungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> regionalkonferenzen: wie die kandidatenwahl die spd verändern könnte </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> regionalreportage: fans erwarten erstes spiel von fc erzgebirge aue nach corona-pause </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> regionalreportage: sylt ohne touristen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> reihenfolge für corona-impfungen in deutschland festgelegt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> reise ins ungewisse: urlaub im risikogebiet türkei </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> reisebeschränkungen für touristen aus deutschen corona-hotspots </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> reisebüros sind durch internationale reisewarnung existentiell bedroht </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> reisekino für kinder in nordsyrien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> reiselust: urlaub trotz coronavirus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> reisen zu corona-zeiten: letzter teil der reportage </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> reisende flüchten vor strengen corona-maßnahmen nach schweden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> reisewarnung für außereuropäische länder problematisch für türkischen tourismus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> reisewarnung für teile frankreichs </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rekord-bilanz im weihnachtsgeschäft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rekordwahlbeteiligung bei kommunalwahlen in hongkong </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> religion </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rembrandt-ausstellung in amsterdam zeigt künstler als selbstdarsteller </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rennen um den corona-impfstoff </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> reportage aus libyen: die schwierige lage der bevölkerung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> reportage beleuchtet steigende jugendarbeitslosigkeit in deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> reportage über das leben im corona-hotspot neukölln </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> reportage über die lage der bevölkerung im türkisch besetzten tall abjad </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> reportage zur amtseinführung des neuen brasilianischen präsidenten bolsonaro </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> reportage: buschbrände in australien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> reportage: die kinder von skid row </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> reportage: flüchtlinge im bosnischen bihac </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> reportage: grenzregion in polen erinnert an den zweiten weltkrieg </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> reportage: reise per rad in deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> reportage: streit um donald trumps grenzsicherung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> repressives ungarn - das letzte freie radio ist ausgeschaltet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> reproduktionsrate: corona-ansteckungen gehen zurück </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> republikaner küren trump offiziell zum kandidaten für us-wahlkampf </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> republikanische partei am scheideweg am ende von trumps amtszeit </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> restaurierte möbel: mit schleifpapier und hobel gegen die wegwerfgesellschaft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> restaurierung von rembrandts &quot;nachtwache&quot;: operation am offenen herzen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rettung durch insolvenz? - der fall 1. fc kaiserslautern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rettungsschiffe ohne hafen: die flüchtlinge vor malta </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> richter fällt urteil im ersten cum-ex-prozess </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> richtungsentscheidung i: spd und die sozialstaatsreform </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> richtungsentscheidung ii: cdu und die migrationspolitik </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> richtungswechsel bei corona-app </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rietschen in der oberlausitz: wie ein truppenübungsplatz zum hoffnungsträger wird </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ringen um eine brexit-lösung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ringen um lockerungen und geld: merkel und die ostdeutschen ministerpräsidenten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> risikogebiet luxemburg soll sonderregelung für corona-testpflicht erhalten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> riskanter klettertrip: oscar-gewinner &quot;free solo&quot; kommt in die kinos </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rki blickt mit sorge auf erneut steigende corona-infektionszahlen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rki prognostiziert harten corona-winter in deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rki stuft kosovo und kroatien offiziell als corona-risikogebiete ein </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> robert habeck bekräftigt machtanspruch der grünen auf virtuellem parteitag </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> robert koch-institut stellt antikörper-studie zu corona-hotspot kupferzell vor </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> robert redfords letzter film </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rock n' roll-pionier little richard gestorben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rockband karat gibt hotelzimmer-konzert </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rocken auf dem bauernhof - aufnahmestudio in wales </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rodel-weltcup in innsbruck </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rodungsstopp bei tesla: die folgen für das großprojekt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> roland emmerichs neues actiondrama &quot;midway – für die freiheit&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rosa luxemburg - wer war sie wirklich? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rosenmontag und alles ist anders </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> roxette-sängerin marie fredriksson erliegt krebsleiden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rückgang bei corona-neuinfektionen: fragen zur belastbarkeit der jüngsten zahlen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rückkehr nach syrien: die geschichte von moaz abu ali </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rückkehr und asylantrag des clanchefs ibrahim m. sorgen für empörung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rückkehr von kriegsvertriebenen nach polen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rückkehr zum regelbetrieb: zwei schulschließungen in mecklenburg-vorpommern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rücktrittsgerüchte und eu-wahl im vereinigten königreich </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> russische polizei geht hart gegen demonstranten für freilassung von kremlkritiker nawalny vor </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> russischer ministerpräsident medwedjew erklärt überraschend rücktritt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> russischer oppositionspolitiker alexej nawalny zu dreieinhalb jahren haft verurteilt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> russischer präsident putin warnt in rede zur lage der nation vor neuem wettrüsten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> russisches kopfgeld auf us-soldaten: was wusste präsident trump? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> russland beginnt mit auslieferung seines umstrittenen corona-impfstoffs &quot;sputnik v&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> russland schließt seine grenzen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> russland und china blockieren syrien-hilfe im un-sicherheitsrat </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> russland und die geplante verfassungsänderung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> russland und ukraine vollziehen gefangenenaustausch </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> russland-ukraine-gipfel: auf der suche nach frieden für die ostukraine </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> russlandaffäre: robert mueller will nicht vor kongress aussagen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> russlands politischer einfluss auf belarus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sacharbeit statt selbstbeschäftigung: cdu vertagt personalentscheidungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sacharow-menschenrechtspreis des eu-parlaments geht an belarusische oppositionspolitiker*innen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sachsen und brandenburg vor der wahl </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sachsen-anhalt stimmt gegen erhöhung des rundfunkbeitrags </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> saison-aus für ski-ass rebensburg </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> salvini vor comeback? - regionalwahlen in italien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> samsung: dauerprotest eines ehemaligen mitarbeiters </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sänger und songschreiber bill withers mit 81 jahren gestorben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sasa stanisic erhält deutschen buchpreis 2019 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> saudi-arabien: inhaftierte frauenrechtlerin loujain al hathloul im hungerstreik </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> säulenhalle &quot;stoa169&quot; nach antikem vorbild in bayern eingeweiht </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schalker ehrenrat tagt zu äußerung des aufsichtsratsvorsitzenden tönnies </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schall auf beton: faszinierende klangkunst im berghain </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> scharfe kritik an trump von barack obama bei parteitag der us-demokraten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schäuble schlägt schule statt sommerurlaub vor </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schaulaufen für cdu-vorsitz bei der jungen union </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schauspieler chadwick boseman verstorben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schauspieler max wright verstorben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schauspieler peter fonda gestorben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schauspieler sean connery mit 90 jahren gestorben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schauspielerin doris day gestorben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schaustellerbetriebe in der krise </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> scheidender us-präsident trump nimmt reibungslosen machtwechsel nicht hin </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> scheuer will klagerecht bei großen verkehrsprojekten beschneiden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schicksals-wahlen: die wichtigsten informationen zum urnengang in europa und bremen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schlachtbetrieb-skandal: ist nicht nur tönnies schuld? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schlagabtausch beim politischen aschermittwoch </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schlagerstar karel gott gestorben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schließung eines friedensabkommens in der region dafur </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schlittenhunde über wasser: ein symbolträchtiges foto und seine geschichte </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schmerzhafter abschied: eu-gipfel berät über brexit-krise </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schnee und eis in österreich </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schneechaos in bayern und österreich </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schneechaos in spanien: madrid versinkt im schnee </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schneechaos: reportage aus dem abgeschnittenen skiort hochkar/österreich </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schneemassen sorgen für chaos in bayern und österreich </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schnelle corona-tests vor weihnachten im parkhaus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schneller anstieg von neuinfektionen in russland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> scholz verteidigt haushaltspläne: schwerpunkt sozialer zusammenhalt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> scholz' reformplan für finanzaufsicht erntet kritik von opposition und koalitionspartner </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schon wieder neuwahl: politikverdrossenheit in spanien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schriftsteller daniel kehlmann zur einschränkung der grundrechte durch corona </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schulabschlussprüfungen werden trotz corona-krise durchgeführt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schulbeginn mit hindernissen in corona-zeiten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schulden des bundes steigen auf etwa 218 milliarden euro durch das verabschiedete konjunkturpaket </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schulen reagieren mit notbetreuung und online-unterricht auf schließungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schüler demonstrieren für mehr klimaschutz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schülervertreter von rheinland-pfalz verfassen offenen brief an landesregierung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schulpreis 2019 - was macht eine schule gut? fragen an youtuber &quot;lehrer schmidt&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schulschließungen in großem stil: die folgen für kinder und eltern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schulstart nach den sommerferien mit strengen hygiene-vorgaben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schusswaffenangriff: mehrere tote im texanischen el paso </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schutz vor rechtem terror: was sich nach dem anschlag von hanau ändern soll </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schutzausrüstungen für krankenhauspersonal sind weltweit mangelware </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schwach oder stark: europas rolle in der weltpolitik </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schwarz und schwul: politischer karneval in rio erzürnt die religiösen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schwarze vor der us-wahl </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schwedens wechsel zu strenger asylpolitik nach 2015 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schwedische ermittler halten mordfall palme für geklärt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schwedische schulen unterrichten e-sport </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schwedisches gericht entscheidet über rechte der sami-ureinwohner </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schweiz und frankreich soll ab samstag schrittweise erfolgen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schwere gefechte in der region bergkarabach: armenien und aserbaidschan verhängen kriegsrecht </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schweres busunglück auf madeira: offenbar viele deutsche unter den mindestens 28 toten - hotline des auswärtigen amtes für angehörige: 030 - 5000 3000 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schwierige entscheidung: bundesverfassungsgericht verhandelt klagen gegen sterbehilfe-verbot </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schwierige regierungsbildung: rot-rot-grün verliert mehrheit in thüringen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schwieriges erinnern an rassistische anschläge von mölln: rednerin trotzt morddrohungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schwieriges verfahren: loveparade-prozess könnte eingestellt werden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schwieriges zusammenleben: hagen und die zuwanderer aus südosteuropa </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> schwierigkeiten bei den verhandlungen zu den corona-beschlüssen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sechs wochen vor der us-wahl: wie ist das stimmungsbild in pennsylvania? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> seehofer vor dem maut-u-ausschuss </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> seehofer-besuch in der türkei: wie stabil ist das flüchtlingsabkommen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> seehofers plan zur bekämpfung von rechtsextremismus auch in sicherheitsbehörden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> segelschulschiff &quot;gorch fock&quot; hat wieder eine handbreit wasser unter dem kiel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sehnsucht nach normalität trotz möglicher zweiter corona-welle </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> seit einem jahr ist brasiliens ministerpräsident bolsonaro im amt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> selbstdarstellung per app: das phänomen &quot;tik tok&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> selbstversorger in corona-zeiten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> seltenes tv-duell vor neuwahl des bürgermeisters in istanbul </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> senats-stichwahlen im us-bundesstaat georgia </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sensation in der astrophysik: foto zeigt millionen lichtjahre entferntes schwarzes loch </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sexualisierte gewalt gegen kinder: prozessauftakt in bergisch-gladbach </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sexuelle gewalt gegen kinder und wie man sie erkennen kann </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sexueller missbrauch: beauftragter der regierung fordert gesetze zum schutz von kindern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> showdown in london? das britische parlament tagt wieder </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> shutdown vorerst beendet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sicherheit und nebenwirkungen der corona-impfungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sicherheitskonferenz in münchen endet mit treffen zur libyen-krise </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sicherheitskonferenz in münchen: merkel sorgt mit rede für stehende ovationen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sicherheitskräfte besorgt über gewalteskalationen in leipzig </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sicherheitskräfte im irak gehen weiter hart gegen protestierende vor </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sicherheitszone syrien: reportage aus kobane </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sieg für netanyahu-lager bei wahl in israel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sieger der us-präsidentschaftswahl weiterhin offen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> singapur will ausreisen für urlaubswillige unattraktiv zu machen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sir simon rattle wird neuer chefdirigent beim bayerischen rundfunk </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sir tom moore zum ritter geschlagen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> situation der menschen im flüchtlingslager moria verbessert sich nur langsam </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> situation vor dem fußball-länderspiel deutschland gegen serbien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ski alpin: dreßen gewinnt abfahrt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> skirennfahrer neureuther beendet seine karriere </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> skirennläufer neureuther beendet karriere </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> skispringen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> smart speaker: hören die sprachassistenten auch ohne sprachkommando mit? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> snowfarming: ist schneebunkern sinnvoll? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> so bereitet sich deutschland auf den coronavirus vor </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> so blicken us-bürger auf das drohende amtsenthebungsverfahren von präsident trump </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> so hat europa gewählt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> söder will steuern auf bahntickets senken </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> soleimani als strippenzieher im nahen osten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> solidarität in corona-zeiten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> soll an einen kapitän des nazi-regimes erinnert werden?: langsdorff-entscheidung vor 80 jahren </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sommer-interviews: söder/meuthen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sommer-pressekonferenz: merkel zeigt keine amtsmüdigkeit </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sommer: in teilen deutschlands wird das wasser knapp </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sommerinterviews von ard und zdf mit lindner und habeck </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sommersemester bietet vielen studierenden erste erfahrungen im online-studium </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sonderbericht zum klimawandel: der teufelskreis aus intensiver landwirtschaft und klimawandel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sonntag ohne auto: ein versuch im landkreis pfaffenhofen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sorge in einigen regionen frankreichs über explosion der covid-zahlen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sorge um ausbreitung des coronavirus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sorge um corona-neuinfektionen nach illegaler silvester-party in frankreich </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sorge um die umwelt: teslas gigafabrik in grünheide spaltet die gemüter </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sorge um historische parkanlage von sanssoucci wegen trockenheit </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sorge um lieferung von johnson &amp; johnson-vakzin wegen us-exportstopp für corona-impfstoffe </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sorge vor eskalation im kaschmir-konflikt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sorge vor häuslicher gewalt als folge der ausgangssperre </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sorge vor neuen unruhen in nordirland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sorge vor weiterem corona-lockdown im kreis gütersloh </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sorge wegen covid19-mutationen in deutschland wächst </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sozial benachteiligte jugendliche in zeiten von homeschooling </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sozialbetrug durch scheinstudenten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sozialisten bei europawahl in den niederlanden vorn </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sozialisten liegen bei parlamentswahl in spanien vorne </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sozialminister heil präsentiert konzept zur grundrente </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sozialverbände und gewerkschaften fordern mehr unterstützung für bedürftige </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> spahn plant pflichttests für reiserückkehrer aus risikogebieten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> spahn will impfpflicht einführen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> spanien hat mehr todesfälle durch coronavirus als china </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> spanien lockert strenge ausgangssperre </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> spanien verhängt coronavirus-notstand </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> spaniens gesundheitssystem offenbar in der krise </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> spannungen in kaschmir </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> spatenstich der neuen dfb-akademie </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> spd beginnt beratungen für wahlprogramm </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> spd im umfragetief vor bürgerschaftswahl in bremen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> spd im umfragetief: wer übernimmt den spd-parteivorsitz? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> spd parteitag: beschlüsse zu sozialstaatskonzept und personalentscheidungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> spd prescht vor: kanzlerkandidat scholz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> spd stellt konzept für wiedereinfühurng der vermögensteuer vor </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> spd verabschiedet leitantrag für parteitag </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> spd verliert in saarbrücken das oberbürgermeisteramt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> spd vor der stichwahl zum parteivorsitz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> spd will leistungen für arbeitslose reformieren </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> spd-politiker eva högl zur neuen wehrbeauftragten des bundestages gewählt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> spd-politiker manfred stolpe ist tot </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> spd-politiker und bundestagsvizepräsident thomas oppermann im alter von 66 jahren gestorben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> spd-regionalkonferenz: wettstreit um spd-vorsitz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> spd-spitze zeigt sich offen für koalition mit linkspartei </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> spd-vorsitz: warum nun auch vizekanzler scholz kandidiert </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> spd: der lange weg zu einer neuen führung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> spd: schuldenbremse weg - vermögenssteuer her </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sperrung von trumps twitterfeed stößt debatte über regulierung sozialer netzwerke in deutschland an </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> spießrutenlauf vor abtreibungskliniken - wie frauen im us-bundesstaat alabama von abtreibungsgegnern verfolgt werden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> spionagevorwurf: huawei klagt gegen us-regierung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sport und kultur stellen öffnungskonzept vor </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sportgericht: bittere niederlage für olympiasiegerin semenya </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> spurensuche in halle: die schwierige aufarbeitung des attentats </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> spurensuche in polen: wo der zweite weltkrieg wirklich begann </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> spurensuche: wo kommt das virus wirklich her? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> spurwechsel für asylbewerber: unternehmer kämpfen für bleiberecht </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> srebrenica: unterwegs in einer verwundeten stadt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sri lanka erwartet nach anschlag geringeres wirtschaftswachstum </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> staatsanwaltschaft erlässt haftbefehl wegen mordes nach angriff im frankfurter bahnhof </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> staatsanwaltschaft ermittelt gegen 43-jährigen deutschen im fall der 2007 verschwundenen madeleine mccann </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> staatsbesuch in china: merkel mahnt hongkong-lösung an </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> staatskrise in venezuela spitzt sich dramatisch zu </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> staatsoper in stuttgart: lieb und teuer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> staatsschutz ermittelt nach mutmaßlichem terror auf berliner stadtautobahn </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> stadt ohne trinkwasser: misswirtschaft in bulgarien löst wasserkrise aus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ständige impfkommission empfiehlt astrazeneca-vakzin für menschen unter 65 jahren </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> stark und sozial: wie die spd in den bundestagswahlkampf gehen will </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> stasi-unterlagen sollen ins bundesarchiv </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> staus an corona-teststationen und überlastete gesundheitsämter </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> steigende corona-zahlen: kommt nun der zweite lockdown und wie sieht er aus? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> steigende fallzahlen bei weniger impfungen begünstigen dritte corona-welle </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> steigende opferzahlen durch türkischen angriffskrieg in syrischen kurdengebieten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> steinmeier in yad vashem: &quot;die bösen geister in neuem gewand&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> steinmeier würdigt historische bedeutung der nürnberger prozesse vor 75 jahren </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> stellenabbau durch strukturwandel in der automobilindustrie </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> steuereinnahmen bis 2023 offenbar rund 124 milliarden euro geringer als erwartet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> stichwahl um senatsposten in us-bundesstaat georgia </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> stillgelegte bergwerke müssen zum schutz des trinkwassers weiter betreut werden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> stimmenverluste bei landtagswahlen dämpfen erwartungen der union für bundestagswahl </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> stimmung vor den us-wahlen: reportage aus new orleans </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> stimmung vor der europawahl </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> stimmungstest an der basis - wie kommt kramp-karrenbauer bei den kommunalpolitikern der union an? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> strategie der grünen für die landtagswahlen in ostdeutschland und die europawahl </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> strategien gegen bevölkerungsexplosion </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> strategien gegen starkregen: wie städte sich vor den wassermassen schützen können </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> streik im öffentlichen dienst: so positionieren sich die arbeitgeber </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> streik im öffentlichen dienst: was wollen die streikenden? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> streit im schwimmbad: was tun gegen aggressive jugendgruppen? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> streit in der afd nach kalbitz-rauswurf </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> streit über geplante huawei-beteiligung am 5g-netzausbau </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> streit über mögliche inntal-bahntrasse für den güterverkehr </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> streit über verzicht auf auto-kaufprämien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> streit um atomabkommen: krisentreffen in brüssel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> streit um corona-bonds: warum italien die hilfszusagen nicht reichen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> streit um den brexit: kampagne gegen das parlament </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> streit um den cdu-parteitag </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> streit um deutsche rüstungsexporte für saudi arabien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> streit um erbe der hohenzollern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> streit um erhalt oder abriss der ehemaligen tierversuchsanstalt in berlin </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> streit um hybrid-unterricht in schulen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> streit um impfpflicht </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> streit um impfstoff-beschaffung erreicht koalition </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> streit um menschenrechte bei gipfeltreffen zwischen europäischer union und arabischer liga </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> streit um personalpolitik im literaturarchiv marbach </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> streit um tempolimit 130 auf deutschen autobahnen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> streit um vermutete gasvorkommen: konflikt zwischen griechenland und der türkei </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> streit um windkraft: koalition streitet um abstand zwischen wohnhäusern und windrädern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> streit zwischen eu und astrazeneca über impfstoff-lieferungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> streit zwischen stadtplanern und architekten um tahrir-platz in ägypten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> strom und wärme durch windenergie: der besondere weg eines kleine dorfes in der uckermark </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> stromausfall: venezuela fast landesweit lahmgelegt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> strompreise steigen weiter an </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> studie der friedrich-ebert-stiftung: zunehmende ablehnung gegenüber asylbewerbern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> studie stellt zusammenhänge zwischen wirtschaftsförderung und geringerer produktivität in ostdeutschland fest </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> studie über die wirkung der corona-einschränkungen auf kinder und jugendliche </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> studie zeigt psychische folgen der corona-krise bei kindern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> studie zu kinder und corona: zwischenergebnisse ermutigen baden-württemberg zur baldigen öffnung der kitas </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> studie zu klimazielen: umweltbundesamt fordert schärfere maßnahmen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> studie zu sexueller belästigung am arbeitsplatz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> studie zum brexit: sex-flaute und angstzustände </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sturmtief &quot;sabine&quot; wütet über teilen europas </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> stuttgart 21: wie die polizeigewalt am &quot;schwarzen donnerstag&quot; die protestbewegung verändert hat </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> suche nach antworten: was auf einen boeing-abschuss im iran hindeutet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> suche nach lösungen: wie europa die usa-iran-krise entschärfen will </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> suche nach veränderung: wo die grünen nach 40 jahren stehen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> suche nach verschollenen containern in der nordsee </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sudan: unterzeichnung der verfassungserklärung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> südkoreanischer sozialsatire &quot;parasite&quot; als erster nicht-englischsprachiger film mit oscar für besten film ausgezeichnet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sündenböcke der pandemie: die verfolgung der bahai im iran </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> supercup-finale in budapest vor 15.000 fans </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> supreme court erklärt parlamentspause für unrechtmäßig </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> syrer in der passklemme </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> syrien wählt neues parlament </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tag der deutschen einheit: merkel hält rede in kiel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tag des notrufs: wie sich der alltag der feuerwehren verändert </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tag des parlaments: abstimmungen über anträge zum brexit </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tagesschau-chefsprecher jan hofer nimmt abschied </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tagesthemen &quot;mittendrin&quot; in bollstedt in thüringen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tagesthemen #mittendrin: geboren nach der wende - altwarp </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tagesthemen mittendrin: ärger um den zulauf zum brenner-basistunnel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tagesthemen mittendrin: berliner szene vor und nach der wende </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tagesthemen mittendrin: brandenburger kämpfen gegen putenmast </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tagesthemen mittendrin: das höchste dorf am rhein </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tagesthemen mittendrin: das leiden der sexarbeiterinnen in der corona-krise </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tagesthemen mittendrin: ddr-bürgerrechtler und dissidenten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tagesthemen mittendrin: der wasser-streit von lüneburg </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tagesthemen mittendrin: drohungen gegen muslimische ladenbesitzer im steintorviertel von hannover </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tagesthemen mittendrin: dürre im thüringischen artern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tagesthemen mittendrin: letzter bus nach amerika </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tagesthemen mittendrin: lychen - streit um nacktbadeverbot in der uckermark </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tagesthemen mittendrin: nürburgring in zeiten von corona </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tagesthemen mittendrin: schrebergarten-idylle statt fernreise </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tagesthemen mittendrin: spurensuche connewitz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tagesthemen mittendrin: streit um baumschwebebahn im harz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tagesthemen mittendrin: streit um wohnungsbauprojekt &quot;6-seen-wedau&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tagesthemen mittendrin: wie war dein sommer? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tagestourismus sorgt für hohe besucherzahlen am bayrischen walchensee </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> taiwan vor der wahl: der lange arm von pekings propaganda </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> taliban kündigen vor präsidentschaftswahl in afghanistan anschläge an </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> talkshow-legende larry king gestorben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tanker-angriff: sorge vor eskalation zwischen usa und iran wächst </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tarif-einigung im öffentlichen dienst </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tausende anhänger der &quot;sardinen&quot;-bewegung protestieren in rom gegen rechtspopulismus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tausende demonstranten in hong kong bilden menschenkette </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tausende demonstrieren in belarus trotz einschüchterung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tausende demonstrieren in nrw für schnelleren kohleausstieg </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tausende flüchtlinge müssen an der türkischen grenze zu griechenland ausharren </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tausende menschen demonstrieren in berlin gegen antisemitismus und rechte gewalt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tausende menschen im landkreis gütersloh wegen corona-ausbruch in quarantäne </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tausende motorradfahrer protestieren in münchen gegen mögliche fahrverbote </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tausende protestieren in russland gegen die festnahme des gouverneurs furgal </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tausende russen gedenken ermordetem oppositionellem nemzow in moskau </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tausende solidarisieren sich bundesweit auf demonstrationen mit seenotrettern im mittelmeer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> teamlösung für die cdu?: fragen an schleswig-holsteins ministerpräsidenten günther </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> technikmesse ces in las vegas </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> teile pekings wegen corona-neuinfektionen abgeriegelt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> teilnehmer der corona-demonstration in berlin stehen in der kritik </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> telemedizin: mehr videosprechstunden wegen der corona-pandemie </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> terror in sri lanka: anschläge treffen kirchen und hotels </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> terror in sri lanka: is reklamiert anschläge für sich </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> terrormiliz is in syrien für besiegt erklärt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tesla kommt in die provinz: wie der autokonzern grünheide in brandenburg verändert </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tesla plant neue fabrik in brandenburg </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> test bei stromkonzern: cyberabwehr als neuer werkschutz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> test für putin: wie sich die vergiftung nawalnys auf die regionalwahl in russland auswirkt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> testpanne in bayern: söder kämpft um sein gewinner-image </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> testpflicht für reiserückkehrer aus risikogebieten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tests für rückkehrer: neue corona-pläne für urlauber </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> thailand krönt neuen könig </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> theater- und filmemacher kirill serebrennikow nimmt erstmals seit ende des hausarrests an premiere teil </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> themenwoche bildung: was eine quereinsteigerin in der grundschule erlebt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> thomas-cook-insolvenz: zurich-versicherung übernimmt 50 prozent der kosten für deutsche urlauber </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> thüringen empfiehlt eingeschränkten bewegungsradius </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> thüringen und die folgen für cdu und fdp </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> thüringen vor der wahl </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> thüringen: konsequenzen des rückzugs von ministerpräsident kemmerich </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> thüringens cdu debattiert über kooperation mit der afd </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> thüringer cdu will mit rot-rot-grün über projekte sprechen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> thüringer verfassungsgerichtshof kippt paritätsgesetz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> thyssenkrupp-fusion und aufspaltung scheitern an eu-kommission </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tief im sumpf: neue haftbefehle im wirecard-skandal </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tiefe einschnitte bei audi: 9500 stellen sollen bis 2025 wegfallen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tierklinik für koalas in australien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tobender machtkampf in venezuela </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tod eines insassen: der mysteriöse fall des us-finanzberaters jeffrey epstein </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tod von us-verfassungsrichterin ruth bader ginsburg </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tödliche attacke: prozessbeginn nach messerangriff von chemnitz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tödliche messerattacke in dresden: bundesanwaltschaft geht von radikal-islamistischem hintergrund aus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> totale mondfinsternis sorgt für seltenes naturschauspiel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tote bei messerstecherei in der nähe von london </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tote bei zugunglück in dänemark: personenzug wird auf brücke über großen belt schwer beschädigt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tourismus trotz corona: ägypten öffnet hotspots </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tourismus-infarkt und gegenmaßnahmen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tourismusindustrie leidet stark unter den einschränkungen durch die corona-krise </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> traditionelle vatertagsausflüge unterliegen corona-maßnahmen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> transitgipfel: 10-punkte-plan beschlossen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> trauer in coronazeiten: reportage einer bestatterin </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> trauer um cartoonisten uli stein </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> trauer um dramatiker rolf hochhuth </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> trauer um ehemaligen spd-vorsitzenden hans-jochen vogel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> trauer: tagesschau-legende wilhelm wieben ist tot </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> trauerfeier für robert mugabe: wo steht simbabwe seit dem machtwechsel? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> trauerfeier von george floyd in heimatstadt houston </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> traurige weihnachtsmänner: chinas export im corona-tief </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> trauriger rekord in brasilien: mehr als 1.000 corona-tote innerhalb von 24 stunden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> treffen der koalitionsspitzen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> treffen von merkel und pompeo - wie gut ist das verhältnis zwischen deutschland und den usa? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> treffen von putin und lukaschenko in sotschi: russland verspricht milliardenkredit für belarus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> trend verschlafen: wann wird deutschland e-scooter-land? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> trockene böden und missernten - landwirtschaft sucht neue strategien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> trockenheit: deutsche winzer sorgen sich um ihre ernte </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> trockenheit: die sorgen der landwirte </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> trockenheit: wetterexperten warnen vor neuem dürresommer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> trotz aller proteste - wieder stierkämpfe in palma de mallorca </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> trotz einigung über corona-finanzhilfen wird der ton innerhalb der eu rauher </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> trotz fachkräftemangels: ausländische pflegekräfte scheitern an der deutschen bürokratie </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> trotz massiver drohungen gehen in minsk tausende gegen lukaschenko auf die straße </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> trump besucht konferenz des konservativen aktivistenverbands cpac </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> trump besucht von waldbränden bedrohten us-staat kalifornien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> trump facht die rassismus-debatte in den usa weiter an </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> trump nach corona-infektion im krankenhaus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> trump nimmt nach covid-19-erkrankung wahlkampf wieder auf </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> trump nominiert konservative juristin barrett für obersten gerichtshof </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> trump oder biden: wie der wahlkampf amerikanische familien spaltet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> trump stellt wahltermin in frage: widerspruch auch aus den reihen der republikaner </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> trump unter druck: kehrtwende in corona-krise </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> trump verteidigt den us-angriff - wie reagiert der iran? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> trump verteidigt mauer-idee vor nation und droht wieder mit notstand </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> trump vs. pelosi: stand der impeachment-debatte </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> trump wird nationalen notstand erklären </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> trump-anhänger dringen in kapitol ein </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> trump-rede vor der un-vollversammlung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> trump-show: wie der us-präsident seine wiederwahl sichern will </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> trump's &quot;drain the swamp&quot;: ein zustandsbericht </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> trumps reaktion auf raketenangriff </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> trumps umgang mit der corona-krise </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tt-gespräch mit annalena baerbrock </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tt-gespräch mit ursula von der leyen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tuberkulose in deutschland: wie groß ist die gefahr? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tunesien und die toten flüchtlinge </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> türkei verweigert journalisten verlängerung der akkreditierung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> türkische militäroffensive gegen die kurden in nordsyrien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> türkische offensive: lage in syrien und der türkei </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> türkische offensive: zehntausende kurden in nord-syrien auf der flucht </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> türkischer präsident erdogan öffnet grenzen zur eu </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> türkischer präsident erdogan will hagia sophia in moschee umwandeln </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> türkisches militär setzt offensive gegen kurden in nordsyrien fort </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> türkisches parlament genehmigt truppeneinsatz in libyen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tv-duell der spd-kandidatenteams </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tv-interview mit prinz harry und meghan sorgt für meinungsverschiedenheiten im britischen königshaus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> twitter schließt den account von donald trump </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> u21-em: deutschland steht im halbfinale </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> über den vorschlag der kommission </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> über die seelischen auswirkungen der pandemie forscht die psychologin cornelia betsch </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> über drei millionen menschen gehen mehreren jobs gleichzeitig nach </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> überforderte notaufnahmen: wie kann entlastung funktionieren? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> überfüllte bars in berlin </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> übergangsfrist für den brexit endet: noch keine einigung in sicht </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> überlastete krankenhausärzte: reportage von einer intensivstation </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> überlastung der krankenhäuser in belgien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> überraschende freisprüche im gezi-prozess in istanbul </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> überraschendes abkommen zwischen israel und den emiraten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> überraschung in biarritz: warum der iranische außenminister das g7-treffen besuchte </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> überwiegend frauen aufgrund von corona-beschränkungen zu hause </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> überwintern im warmen: hunderttausende &quot;snowbirds&quot; zieht es nach arizona </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> uefa champions league </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ukraine gedenkt der opfer der massenproteste auf dem maidan vor fünf jahren </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ukraine vor der wahl: reportage aus der ostukraine </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ukraine-stichwahl: komiker selenskyj als neuer präsident gefeiert </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ukraine-wahlen: viele hoffen auf wladimir selenskij </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> uli hoeneß legt präsidentenamt des fc bayern münchen nieder </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> uli hoeneß zieht sich von spitze des fc bayern münchen zurück </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> umbruch im einzelhandel: gewinner und verlierer der krise </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> umbruch in algerien: bouteflika tritt ab </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> umfrage: reisen in corona-zeiten -  ja oder nein? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> umgang mit bootsflüchtlingen: bundesregierung will 25 prozent aufnehmen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> umgang mit krebs: kommissarische spd-vorsitzende schwesig legt parteiamt wegen krebserkrankung nieder </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> umgang mit strukturwandel: was brandenburg vor der landtagswahl bewegt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> umkämpfte wahlkreise: großbritannien vor der wahl </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> umsatzausfälle wegen coronavirus: unternehmen hoffen auf soforthilfen und kredite </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> umstrittene ausstellung über die ästhetik der nationalsozialisten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> umstrittene präsidentschaftswahl in polen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> umstrittene umbettung der sterblichen überreste von spaniens ex-diktator franco </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> umstrittener banker: abschied von ezb-chef mario draghi </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> umstrittener impfstoff: dosen von astrazeneca bleiben ungenutzt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> umstrittener kandidat: wie boris johnson polarisiert </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> umstrittenes kunstobjekt: michael-jackson-ausstellung in bonn </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> umstrittenes maskenverbot: erneut proteste in hongkong </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> umstrittenes ökologiekonzept beim bau der tesla-fabrik im brandenburgischen grünheide </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> umweltschutz an silvester: diskussion über böllerverbot </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> umweltschützer befürchten zerstörung der biotope auf fehmarn durch kitesurfer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> umweltschützer starten volksbegehren zur rettung der artenvielfalt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> umweltschützer wütend über ausgang der weltklimakonferenz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> umweltverbände kritisieren geplante eu-reform der agrarpolitik </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> un stufen singapurs &quot;hawker&quot;-zentren als weltkulturerbe ein </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> un-flüchtlingshilfswerk geleitet 98 flüchtlinge aus lybischen lagern nach italien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> un-klimagipfel in new york: regierungen stellen pläne gegen klimawandel vor </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> un-konferenz berät über zukunft von bergtourismus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> un-konferenz in nairobi zur entwicklung der weltbevölkerung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> un-resolution: gegen sexualisierte gewalt als kriegswaffe </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> un-sonderorganisation ilo setzt seit 100 jahren weltweit gerechte arbeitsbedingungen durch </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> unbekannte einblicke: bilder aus vernichtungslager sobibor </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> uneinigkeit über das löschen von afd-daten beim sächsischen verfassungsschutz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> unfall mit tesla: entsorgung von batterien bei elektroautos bereitet große probleme </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ungarische zwickmühle: warum sich die evp mit victor orban so schwer tut </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ungewöhnlicher protest: wikipedia für 24 stunden offline </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> union begegnet spd-plänen mit skepsis </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> union berlin steigt erstmals in die 1. fußballbundesliga auf </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> union holt auf: neueste zahlen im ard-deutschlandtrend </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> union ringt um gemeinsame klimapolitik </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> unions- und oppositionspolitiker fordern von finanzminister scholz aufklärung über kenntnisse zum wirecard-skandal </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> unionsabgeordnete geben nach maskenaffäre ehrenerklärung ab </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> unmut über lockdown in den kreisen gütersloh und warendorf </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> unstimmigkeiten in der großen koalition über grundrente </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> unter staatlicher kontrolle: mülltrennen in schanghai </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> unternehmer äussern sich zum fachkräftemangel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> unterricht mit maske: schulstart in nrw unter strengen hygiene-auflagen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> unterrichtsausfall an schulen: immer mehr lehrer fehlen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> unterschätzter schutz - debatte um mundschutzpflicht </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> unterschätztes risiko: aktive vulkane als ziel von touristen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> unterschiedliche auffassungen zwischen ost- und west-verbänden der union </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> unterschiedlichste politische spektren kommen auf sogenannten hygiene-demos zusammen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> unterstützung für die opposition in belarus: bewegung erhält sacharow-preis </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> untersuchungsausschuss soll wirecard-skandal aufklären </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> untersuchungsausschuss zur pkw-maut setzt verkehrsminister scheuer unter druck </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> urlaub im sommer 2020 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ursula von der leyen zur eu-kommissionspräsidentin gewählt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> urteil für mehr tierschutz: küken-töten wird bald verboten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> urteil im missbrauchsfall lügde - welche konsequenzen ziehen die behörden? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> urteil im weinstein-prozess: der schuldspruch der jury </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> urteil in istanbul: haftstrafe für deniz yücel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> urteil zu neuem fenster für die marktkirche in hannover </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> urteil zum bnd-gesetz: auslandsüberwachung verfassungswidrig </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> us-außenminister pompeo sagt berlin-besuch ab und reist stattdessen kurzfristig nach bagdad </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> us-außenminister pompeo zu antrittsbesuch in berlin </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> us-demokrat sanders macht den weg für die kandidatur joe bidens frei </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> us-demokraten treiben amtsenthebungsverfahren gegen trump voran </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> us-fotograf moore mit world press photo award 2019 ausgezeichnet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> us-justizministerium veröffentlicht bericht des sonderermittlers mueller </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> us-marineschiff zerstört iranische drohne </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> us-präsident biden bekennt sich bei münchner sicherheitskonferenz zu transatlantischen beziehungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> us-präsident biden unterzeichnet an erstem arbeitstag 17 dekrete </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> us-präsident entlässt nationalen sicherheitsberater bolton </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> us-präsident trump benutzt kenosha-besuch für seinen wahlkampf </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> us-präsident trump erkennt israels souveränität über golanhöhen an </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> us-präsident trump nutzt supreme court und tiktok für seinen wahlkampf </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> us-präsident trump ruft nationalen notstand aus </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> us-präsident trump stellt seinen nahost-plan vor und die reaktion der palästinenser darauf </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> us-präsident trump trifft premierministerin may und stellt handelsabkommen in aussicht </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> us-präsident trump übertritt nordkoreas grenze und trifft machthaber kim </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> us-präsident trump will soziale netzwerke wegen seines streits mit twitter stärker reglementieren </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> us-präsident trump zu gast bei der queen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> us-präsident trumps riskanter kurs im nahen osten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> us-präsidentschaftskandidat biden wählt kalifornische senatorin harris zu seiner vize-präsidentin </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> us-präsidentschaftskandidat biden wählt kamala harris als seine mögliche vizepräsidentin </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> us-regierung kündigt download-sperre für chinesische video app tiktok an </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> us-repräsentantenhaus beschließt zweites amtsenthebungsverfahren gegen präsident trump </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> us-sanktionen gegen kuba treffen das land hart </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> us-senat: befragung der supreme court-kandidatin amy coney barrett </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> us-sicherheit in gefahr: streit um zölle </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> us-sonderermittler mueller übergibt bericht an justizministerium </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> us-strafzölle - aus für riesling-winzer? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> us-vizepräsident pence sagt venezolanischem übergangspräsidenten guaidó bei beratungen der &quot;lima-gruppe&quot; unterstützung zu </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> us-vorwahlen in iowa: peinliche panne bei den demokraten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> us-wahlkampf: tv-duell der vizepräsidentschaftskandidierenden pence und harris </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> us-wahlkampf: warum das tv-duell zwischen trump und biden aus dem ruder lief </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> usa drohen mit sanktionen gegen nord stream 2 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> usa kaufen medizinische ausrüstung in russland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> usa kündigen inf-vertrag: beide seiten werfen sich vertragsbruch vor </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> usa lassen von strafzöllen gegen mexiko ab - im gegenzug soll das land die grenzkontrollen verstärken </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> usa planen offenbar truppen-präsenz in deutschland zu reduzieren </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> usa steigert die impfkapazität auf zwei millionen impfungen pro tag </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> usa und china einigen sich auf ein teilabkommen im handelskonflikt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> usa und eu einigen sich auf handelsabkommen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> usa unterzeichnen friedensabkommen mit den taliban </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> usa verzeichnet rekordhoch bei corona- neuinfektionen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> usa vor längstem shutdown aller zeiten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> usa wählen präsidenten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> usa wollen fast 12.000 soldaten aus deutschland abziehen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> usa: billionenschweres corona-hilfspaket </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> usa: diskussion um mueller-report </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> usa: nach dem anschlag in el paso und dayton </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> usa: shutdown und kein ende in sicht </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> userin umgeht china-zensur mit make-up-tutorial bei tik tok </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> vakuumverpackte baumstämme als mittel gegen dramatische waldschäden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> van gogh im städel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> venezuela zwischen stillstand und neuanfang </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> venezuela: der machtkampf um die hilfsgüter </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> venezuela: wer ist juan guaidó </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> veränderungen in der arbeitswelt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> veranstaltungsbranche durch corona in der krise </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> verbot der &quot;grauen wölfe&quot; gefordert: wie türkische nationalisten minderheiten in deutschland bedrohen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> verbot von &quot;combat 18&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> verbot von wildtieren in der chinesischen volksmedizin </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> verdacht auf abrechnungsbetrug in millionenhöhe bei krebsmedikamenten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> verfassungsgericht in brandenburg kippt paritätsgesetz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> verfassungsschutz beobachtet afd in sachsen-anhalt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> verfassungsschutz beobachtet bewegung &quot;querdenken 711&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> verfassungsschutz darf afd nicht als prüffall bezeichnen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> verfassungsschutzbericht: zahl der extremisten und extremistischen straftaten gestiegen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> verhaftung mehrerer regierungskritiker in hongkong </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> verhaltene reaktion der nato auf kramp-karrenbauers vorschlag zu schutzzone in syrien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> verhältnismäßig lockere corona-maßnahmen in san marino sorgen für touristen aus italien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> verhandlungen über neue corona-maßnahmen zwischen bund und ländern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> verhärtete fronten beim eu-haushaltsgipfel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> verhüllungskünstler christo gestorben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> verkehrsgerichtstag in goslar beschäftigt sich mit dem autonomen fahren </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> verkehrsminister scheuer muss sich vor untersuchungsausschuss verantworten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> verkehrsminister scheuer plant höhere bußgelder für verstöße gegen die straßenverkehrsordnung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> verkehrswende in den städten: zuschüsse für lastenfahrräder </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> verlängerung der corona-beschränkungen geplant </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> verlängerung der corona-maßnahmen - einschätzungen zur bedeutung der ergebnisse </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> verleihung der goldenen löwen beim 77. internationalen filmfestival von venedig </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> verleihung der nobelpreise 2019 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> verloren an den &quot;islamischen staat&quot; - wie ein vater um seine tochter  kämpft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> verlust der mitte: welche zukunft haben bürgerliche parteien? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> verordnung für e-roller tritt ab heute in kraft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> verschärfte fahrverbotsregeln unter beschuss: verkehrsminister scheuer erntet für neuen vorstoß zustimmung und kritik </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> verschärfte pandemie-maßnahmen in london und teilen südwest-englands eingeführt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> verschärfung der corona-kontrollen in berlin </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> verschwörungstheorien oder doch nur verschwörungs-&quot;erzählungen&quot;? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> verschwörungstheorien um corona verbreiten sich rasant </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> versteigerung der 5-g-mobilfunkfrequenzen hat begonnen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> verstöße gegen nottötungen in der schweinemast </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> verteidigungsministerin kramp-karrenbauer will bundeswehr-elitetruppe ksk zum teil auflösen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> verteilung von flüchtlingskindern aus griechenland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> vertreter der protestbewegung und militär einigen sich auf einrichtung eines übergangsrats im sudan </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> verunsicherung bei schwerstkranken - wie etwa krebspatienten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> verunsicherung nach reisebeschränkungen auf mallorca </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> verwirrung um identität von hsv-profi bakery jatta </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> verzögerter impfstart in deutschland wegen lieferengpässen bei biontech und pfizer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> vfl wolfsburg steht im finale der fußball-champions-league </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> viel ehrenamtliches engagement und solidarität in corona-krise </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> viele bürgerkriegsflüchtlinge aus äthiopien stranden im sudan </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> viele fragen vor schulschließungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> viele jugendliche von der pandemie gebeutelt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> viele staats- und regierungschefinnen und chefs gratulieren biden zum sieg </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> viele tote bei hochwasser in sibirien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> vielversprechende ansätze bei der entwicklung von corona-medikamenten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> vier berliner bezirke wegen hoher corona-infektionszahlen zu risikogebiete erklärt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> vier eu-länder beschließen übergangslösung für bootsflüchtlinge </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> vierschanzentournee: überflieger kobayashi hängt eisenbichler ab </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> viertelfinal-auslosung im dfb-pokal </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> virologen zu herdenimmunität </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> virtuelle gamescom: wie sich corona auf die messe und auch auf ideen für computerspiele auswirkt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> volkskongress in china: kein wirtschaftswachstum und steigende arbeitslosigkeit durch corona </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> voll </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> volle fußgängerzonen vor dem lockdown </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> vollendung von nord stream 2 durch fall nawalny gefährdet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> voller </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> vom vorzeige- zum sorgenland: massiv steigende corona-neuinfektionen in israel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> von der leyen muss sich im untersuchungsausschuss zur berateraffäre heiklen fragen stellen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> von der leyen wirbt um zustimmung im eu-parlament </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> von wegen schachmatt: das revival der brettspiele </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> vor &quot;compact with africa&quot;: verschlechterte menschenrechtslage in ägypten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> vor aufhebung der reisewarnung: reisen in zeiten von corona </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> vor cdu-migrationstreffen: einige unionspolitiker fordern schärferen umgang mit flüchtlingen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> vor dem bund-länder-treffen: plan und perspektive gesucht </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> vor dem champions-league-finale: deutsche fußballtrainer auf erfolgskurs </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> vor dem eu-gipfel: streit zwischen italien und niederlande </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> vor dem prozess gegen julian assange: ein vater kämpft um seinen sohn </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> vor dem sturm: orkantief &quot;sabine&quot; nähert sich deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> vor dem treffen der ministerpräsident*innen zur öffnungsstrategie </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> vor den wahlen in thüringen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> vor der cdu-klausur: wie die politische jugend auf die parteien schaut </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> vor der dritten welle: bilanz über ein jahr corona </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> vor der kundgebung: trump und das tulsa-massaker </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> vor der landtagswahl: &quot;was brandenburg bewegt&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> vor der nächsten schneefront in den alpen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> vor der parlamentswahl in israel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> vor der wahl in der ukraine: reportage aus odessa </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> vor der wahl in thüringen: situation der menschen in gera </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> vor finale des eurovision song contest in tel aviv </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> vor russland-afrika-gipfel: russlands interessen in zentralafrika </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> vor wahl im eu-parlament: von der leyen kündigt rücktritt als verteidigungsministerin an </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> vorbereitungen in ramsgate auf brexit </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> vorbild trump: afd sät zweifel an briefwahl </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> vorerst kein lockdown in nordrhein-westfalen nach corona-ausbruch </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> vorfälle auf tankern: spannungen zwischen usa und iran steigen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> vorläufige ergebnisse der bund-länder-beratungen zu neuen lockerungsstrategien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> vorschlag der eu-kommission zur migrationspolitik: schneller abschieben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> vorstellung der corona-app </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> vorwurf der vetternwirtschaft: diskussion um titel für chemnitz als kulturhauptstadt 2025 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> vorzeitiger abbruch des gipfeltreffens zwischen us-präsident trump und nordkoreas machthaber kim </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> vw beginnt die elektro-ära in zwickau </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> vw entschädigungen an folteropfer 35 jahre nach ende der brasilianischen militärdiktatur </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> vw startet produktion in wolfsburg mit minimierter belegschaft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wachsender antisemitismus in deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wachsendes selbstbewusstsein: der chinesische führungsanspruch </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wacken: festival versucht müll-problem zu lösen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> waffenrecht in brasilien: präsident bolsonaro will lockerung durchsetzen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wahl der neuen eu-kommission </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wahl in israel: kopf-an-kopf-rennen zwischen netanyahu und gantz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wahl in österreich: övp ist stärkste kraft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wahl in portugal: aufschwung nach der schuldenkrise </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wahlen in argentinien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wahlkampf in großbritannien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wahlkampf in österreich </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wahlkampf in spanien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wahlkampf um cdu-vorsitz: laschet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wahlkampf usa: wen wählen latinos und latinas? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wahlkampfauftakt der afd in brandenburg </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wahlkampfauftakt nach trumps ankündigung für zweite amtszeit zu kandidieren </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wahlkampfendspurt: netanyahu spaltet die wähler in israel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wahlkampfendspurt: wo stehen trump und biden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wahlkampfhilfe für afd-vorsitzenden meuthen: strohleute auf der liste der geldgeber </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wahlkampfveranstaltung von trump in tulsa </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wahlkampfzirkus: politik-duell um präsidentenamt in der ukraine </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wahlkreistausch auf zeit: was abgeordnete in anderen wahlkreisen lernen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wahlkreistausch auf zeit: was ost und west voneinander lernen können </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> währungsunion vor 30 jahren: als die d-mark in die ddr kam </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> waldbrände amazonas: macron fordert sanktionen gegen brasilien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> waldbrände in kalifornien zwingen mehr als 100.000 menschen zur flucht </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> waldbrände in sibirien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> walfängergräber auf spitzbergen durch erderwärmung aufgetaut </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> walter-borjans und esken sollen spd-parteiführung übernehmen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wandel in der autoindustrie: deutsche hersteller setzen auf elektro </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wanderurlaub in zeiten von corona </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> warnstreiks an deutschen flughäfen führen zu vielen ausfällen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> warnstreiks der sicherheitsleute lähmen flugverkehr </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> warnstreiks im öffentlichen dienst: auch mitarbeiter von krankenhäusern in kaiserslautern legen arbeit nieder </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> warum dauert die eu-zulassung der impfstoffe so lange? die interessen moskaus bei der vergabe des impfstoffs sputnik v </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> warum die flüchtlingsfamilien auf lesbos kaum noch hoffnung haben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> warum die neue schnelltest-strategie nur langsam anläuft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> warum die wirtschaftshilfen der bundesregierung noch nicht ankommen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> warum diesmal so viele junge menschen in den usa wählen gehen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> warum es bei der prävention von waldbränden hapert </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> warum sich die impfung durch hausärzte weiter verzögert </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> warum sich hass im netz so schwer verbieten lässt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> warum viele start-up-unternehmen vergleichsweise gut durch die corona-zeit kommen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> was bedeuten die coronavirus-mutationen für deutschland? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> was bedeutet das aus der pkw-maut für die csu? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> was bringt die mietpreisbremse? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> was der abzug von 9500 us-soldaten für deutsche us-stützpunkte bedeutet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> was der corona-tag sonst noch brachte </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> was der sondergipfel über den zustand der eu offenbart </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> was deutschland bewegt: umgang mit dem wolf </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> was die urteile gegen kremlkritiker nawalny für die russische opposition bedeuten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> was drehverbote für die film- und fernsehindustrie bedeuten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> was dürfen politische witze im karneval? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> was europa bewegt: &quot;arm &amp; reich in europa&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> was europa bewegt: jugend &amp; zukunftschancen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> was europa bewegt: sicherheit und grenzschutz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> was ist uns die pflege wert? interview mit gesundheitsminister jens spahn </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> was macht fußball-deutschland in corona-zeiten? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> was sachsen bewegt: fremdenfeindlichkeit und zivilgesellschaft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> was sachsen bewegt: wie rückkehrer die lausitz verändern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> was sagen trumps fans? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> was sich menschen für das jahr 2021 wünschen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> was sind die größten klimakiller? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> was thüringen bewegt: streit um windkraft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> was tun gegen rechten terror? pinar atalay im gespräch mit bundesinnenminister seehofer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> waschanlage reinigt grachten mit luftblasen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> washington d.c feiert wahlsieg von joe biden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> washington gleicht in vorbereitung auf inauguration von joe biden einer festung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> washington wappnet sich gegen qanon: trump und der 4. märz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wasserknappheit bedroht große region im osten deutschlands </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wasserknappheit in deutschland bereitet bauern sorge </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wassermangel in spanien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wasserstoff als hoffnungsträger der flugindustrie </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wasserwacht am starnberger see wegen corona im ausnahmezustand </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wdr </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wdr-sportchefin töpperwien verabschiedet sich nach 35 jahren </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wechselwirkung zwischen hetzrede und realer gewalt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wegen eines insert-fehlers wurde die sendung nachträglich bearbeitet.
                                
                                    hinweis:
                                    wegen eines insert-fehlers wurde die sendung nachträglich bearbeitet. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wegen hasskommentaren und falschmeldungen laufen facebook die anzeigenkunden weg </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wegen rechtsstaats-gebot: ungarn und polen legen veto gegen eu-haushalt ein </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wehrbeauftragte högl will rückkehr zur wehrpflicht neu diskutieren </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weihnachten in corona-zeiten: die gefühlslage in deutschland nach dem fest </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weihnachtsbaum ohne reue - geht das? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weihnachtsshopping-hotspot berlin: umsatz vs. gesundheit </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil der vorspann der tagesthemen in der fernsehausstrahlung um 22:15 uhr aus technischen gründen nicht abgespielt werden konnte. außerdem wurde in dem beitrag von justus kliss die einblendung „archiv“ irrtümlich über den o-ton des präsidenten des bundesrechnungshofes gelegt. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil es in der fernsehausstrahlung im beitrag &quot;humanitäre katastrophe im syrischen idlib&quot; ein insertfehler gab. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil es in der fernsehausstrahlung um 22:15 uhr einen versprecher in der anmoderation zum beitrag aus straßburg gab. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil es in der fernsehausstrahlung um 22:45 uhr in den beiträgen zur &quot;brexit-verschiebung&quot; und zum &quot;friedenspreis des deutschen buchhandels&quot;  insertfehler gab. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil im beitrag von gerrit rudolph zu den rassistischen morden in hanau ein fehlerhaftes namens-insert enthalten war. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil im beitrag zu den cdu-delegierten ein cdu-mitglied nicht korrekt bezeichnet wurde. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil in den tagesthemen um 23:15 uhr im beitrag zum ergebnis der kohlekommission irrtümlich namens-inserts vertauscht wurden. der beitrag zum thema bundesliga darf aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil in der ersten moderation der fernsehausstrahlung um 22:15 uhr die studiobeleuchtung fehlerhaft war. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil in der fernsehausstrahlung eine kameraeinstellung kurzzeitig fehlerhaft war. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil in der fernsehausstrahlung im &quot;kommentar&quot; ein insert-fehler vorlag. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil in der fernsehausstrahlung im beitrag um 22:08 uhr ein falsches insert eingeblendet wurde. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil in der fernsehausstrahlung um 21:15 uhr im beitrag zu corona-kurztests bildrechte beeinträchtigt wurden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil in der fernsehausstrahlung um 21:35 uhr im beitrag aus italien der italienische innenminister salvini irrtümlich in der bauchbinde als luigi di maio bezeichnet wurde. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil in der fernsehausstrahlung um 21:45 uhr im beitrag aus berlin: &quot;spd-vorsitz: warum nun auch vizekanzler scholz kandidiert&quot; - versehentlich ein name nicht eingeblendet wurde. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil in der fernsehausstrahlung um 21:45 uhr im beitrag von jennifer lange in der unterzeile irrtümlich die falsche schule angegeben wurde. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil in der fernsehausstrahlung um 21:45 uhr in der anmoderation der meinung von christiane meier irrtümlich die falsche landesrundfunkanstalt eingeblendet wurde. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil in der fernsehausstrahlung um 22:00 uhr im beitrag zum klimacamp die augsburger oberbürgermeisterin eva weber irrtümlich in der unterzeile als oberbürgermeisterin bayerns bezeichnet wurde. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil in der fernsehausstrahlung um 22:02 uhr im beitrag aus leipzig &quot;theaterpädagogin&quot; in der unterzeile/bauchbinde falsch buchstabiert wurde. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil in der fernsehausstrahlung um 22:15 uhr eine unterzeile falsch gesetzt wurde. die beiträge zum thema &quot;fußball-champions-league&quot; und &quot;ansturm auf den uluru&quot; dürfen aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil in der fernsehausstrahlung um 22:15 uhr im beitrag &quot;was thüringen bewegt&quot; der waldbesitzer erhard müller fälschlicherweise als erhard gruber bezeichnet wurde. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil in der fernsehausstrahlung um 22:15 uhr im beitrag aus venezuela in der unterzeile/bauchbinde ein fehler vorlag. in der anmoderation zum thema diesel-fahrverbote spricht die moderatorin fälschlicherweise von &quot;stickstoff&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil in der fernsehausstrahlung um 22:15 uhr im beitrag maut irrtümlich die falsche autorin genannt wurde. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil in der fernsehausstrahlung um 22:15 uhr im beitrag über &quot;free solo&quot; irrtümlich in der unterzeile/bauchbinde ein fehler vorlag. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil in der fernsehausstrahlung um 22:15 uhr im beitrag zum umgang mit stasi-akten eine bauchbinde irrtümlich zweimal zu sehen war. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil in der fernsehausstrahlung um 22:15 uhr im beitrag zur fußball-nationalmannschaft ein fehler in der unterzeile/bauchbinde vorlag. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil in der fernsehausstrahlung um 22:15 uhr im beitrag zur wahlrechtsreform der spd vorsitzende norbert walter-borjans in der bauchbinde falsch bezeichnet wurde. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil in der fernsehausstrahlung um 22:15 uhr in der moderation zum beitrag &quot;der iranische präsident rouhani spricht vor den vereinten nationen&quot; irrtümlich gesagt wurde </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil in der fernsehausstrahlung um 22:17 uhr im beitrag &quot;gelöbnis zum 65. jahrestag der bundeswehr in berlin&quot; ein insertfehler vorlag. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil in der fernsehausstrahlung um 22:30 uhr im beitrag &quot;donald trump kritisiert auszählprozess&quot; ein insertfehler vorlag . </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil in der fernsehausstrahlung um 22:30 uhr im beitrag aus kambodscha die dame &quot;oum pom&quot; irrtümlich als &quot;yaok kem&quot;  bezeichnet wurde. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil in der fernsehausstrahlung um 22:37 uhr im beitrag der hinweis fehlte </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil in der fernsehausstrahlung um 22:45 uhr im beitrag zur &quot;zukunft der groko&quot; ein quellenhinweis auf zdf-material fehlte. die beiträge zur fußball-bundesliga </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil in der fernsehausstrahlung um 22:45 uhr uhr im beitrag zum tod von sigmund jähn dessen geburtsort irrtümlich als morgenröthe-rautenkranz &quot;im erzgebirge&quot; bezeichnet wurde. die beiträge zur fußball-bundesliga und formel1 können aus rechtlichen gründen im internet nicht vollständig gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil in der fernsehausstrahlung um 22:55 uhr im beitrag zu lesbos ein falsches insert eingeblendet wurde. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil in der fernsehausstrahlung um 22.15 uhr im beitrag &quot;hackerangriff auf dax-konzerne: recherche zu industriespionage&quot; eine fehlerhafte unterzeile eingeblendet wurde. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil in der fernsehausstrahlung um 22.15 uhr im beitrag &quot;streit um vermutete gasvorkommen: konflikt zwischen griechenland und der türkei&quot; in der unterzeile ein tippfehler unterlaufen ist </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil in der fernsehausstrahlung um 23:09 uhr im beitrag aus spanien irrtümlich falsche bilder verwendet wurden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil in der fernsehausstrahlung um 23:15 uhr im beitrag &quot;pläne für neustart der fußball-bundesliga&quot; karl lauterbach in der unterzeile irrtümlich der falschen fraktion zugeordnet wurde. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil in der fernsehausstrahlung um 23:15 uhr im beitrag &quot;ukraine-wahlen&quot; wladimir selenski irrtümlich falsch benannt wurde. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil in der fernsehausstrahlung um 23:15 uhr im beitrag aus new york der amerikanische präsident irrtümlich falsch geschrieben wurde. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil in der fernsehausstrahlung um 23:15 uhr im beitrag zu dem is-kämpfer aus sachsen-anhalt eine quellenangabe fehlte. die beiträge zur fußball-bundesliga dürfen aus rechtlichen gründen nicht auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil in der fernsehausstrahlung um 23:35 uhr im beitrag zur pop-art-ausstellung irrtümlich die unterzeile/bauchbinde zur kuratorin julia nebenführ zu früh eingeblendet wurde. diese sendung wurde wegen eines inhaltlichen fehlers nachträglich bearbeitet. in der fernsehausstrahlung um 23:35 uhr wurde in der animierten grafik zu den ergebnissen der basketball-bundesliga ein ergebnis nicht eingeblendet. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil in der moderation zum beitrag über die afd das hintergrundbild aus technsichen gründen nicht ausgespielt werden konnte. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil in der passage über das infektionsgeschehen in einer putenschlachterei ein inhaltlicher fehler war. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil in der um 23.15 uhr ausgestrahlten sendung in der meldung zu danzig irrtümlich bilder vom attentäter verwendet wurden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weil irrtümlich behauptet wurde </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wein-training in japan </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weiße flocken ohne ende: filzmoos abgeschnitten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weiter meldungen aus dem sport </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weitere medlungen im überblick </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weitere melduingen im überblick </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weitere meldungen des tages </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weitere meldungen im überblick
                                
                                    hinweis:
                                    der beitrag zum fußball bundesliga darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weitere meldungen im überblick
                                
                                    hinweis:
                                    diese sendung wurde nachträglich bearbeitet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weitere meldungen im überblick. die &quot;neue normalität&quot; in chinas unternehmen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weitere meldungen im überlick </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weitere nachrichten im überblick ii </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weiterer umweltskandal in sibirien aufgedeckt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weiterhin hartes vorgehen gegen proteste in belarus: opposition trauert um toten mitstreiter </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weiterhin hohes lawinenrisiko im alpenraum </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> welche effekte schwedens co2-steuer hat </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> welche folgen der insolvenzantrag der &quot;gorch fock&quot;-werft für die marineausbildung hat </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> welche stimme zählt? kann trump die auszählung  vom obersten gericht stoppen lassen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> welche themen bewegen die menschen in hamburg vor der bürgerschaftswahl? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> welche themen entscheiden die wahl? die stimmung in sachsen und brandenburg </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> welle der hilfsbereitschaft: millionenspenden für den wiederaufbau von notre-dame </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> welternährungsprogramm der vereinten nationen erhält friedensnobelpreis </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weltflüchtlingstag: 80 millionen menschen sind weltweit auf der flucht </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weltjugendtag - hoffnungen der indigenen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weltklimakonferenz beginnt in madrid </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weltweit eine million registrierte corona-tote </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weltweit erster 3d-scanner für insekten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weltweit größtes wasserkraftwerk in brasilien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weltweit protestieren schulen gegen den klimawandel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weltweiter wettlauf: russen wollen beim impfen die ersten sein </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weltweiter wettlauf: russland lässt ersten corona-impfstoff zu </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weltwirtschaftsforum in davos zwischen trump und greta </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weltwirtschaftsforum startet in davos </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wenig begeisterung für ostdeutschland-pläne </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wenige kriegen sie </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weniger bedarf an dolmetschern bei virtuellem eu-gipfel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wenn lehrer &quot;querdenker&quot; sind: wie sich berliner schüler gegen beeinflussung wehren </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wer macht was? - die zentralen institutionen der eu </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wer wird der nächste cdu-parteichef? antworten aus der partei-basis </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> werbetour auf schwierigem terrain: von der leyen startet im eu-parlament kampagne zur wahl als kommissionspräsidentin </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> werbung für wahlen in georgia: trump leugnet erneut wahlniederlage </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> werbung in eigener sache: von der leyen in brüssel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wetterbilanz 2019 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wetterextreme: kommunen und privatleute müssen sich künftig anders gegen starkregen wappnen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> whistleblower im arbeitsleben: juan moreno und der fall relotius </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wichtiger zeuge in impeachment-anhörungen belastet us-präsident trump schwer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> widerspruchslösung oder freiwilligkeit: die debatte in der politik </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> widerstand gegen geplanten kohleausstieg für 2038 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie bekommt südkorea den corona-virus in den griff? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie belastbar ist die gesundheitsversorgung der usa? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie besinnlich wird die weihnachtszeit? diskussion um die verlängerung des teil-lockdowns </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie chinas führung der wirtschaftskrise begegnen will </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie co2-ampeln vor corona schützen können </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie das thema umweltschutz die grüne woche dominiert </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie der buchhandel der corona-krise trotzen will </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie der corona-virus die gesellschaft verändert </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie der tönnies-skandal die auswüchse der fleischindustrie offenbart </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie der zweite lockdown an unseren kräften zehrt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie deutsche waffen den jemen-konflikt befeuern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie deutschland mit der rekordhitze umgeht </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie deutschland sich mit grenzkontrollen gegen die corona-mutationen isoliert </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie die afd die wende von '89 für sich instrumentalisiert </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie die autoindustrie ihre arbeitsplätze retten will </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie die berliner das clubsterben verhindern wollen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie die bewohner im berchtesgadener land mit den ausgangsbeschränkungen umgehen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie die corona-einschränkungen familien belasten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie die corona-pandemie unseren alltag &quot;digitalisiert&quot; hat </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie die eu den brexit-deal doch noch retten will </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie die französischen winzer ihre edlen bordeaux-weine durch pestizideinsatz in verruf bringen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie die italienische kleinstadt codogno auf das pandemiejahr zurückblickt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie die neue generalsekretärin der fdp punkten will </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie die sicherheit von impfzentren und impfstoff-transporten gewährleistet werden soll </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie die thomas-cook-insolvenz sich auf die fluggesellschaft condor auswirkt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie digitales lernen funktioniert </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie eine co2-steuer funktionieren könnte </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie es nach der videokonferenz der kanzlerin weitergeht </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie fluggesellschaften ihre kunden hängen lassen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie freiwillig die corona-warn-app ist </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie geht es weiter in den schulen? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie geht es weiter mit dem bundeswehreinsatz in afghanistan? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie geht es weiter nach brand im größten europäischen flüchtlingslager moria auf der insel lesbos? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie gesellschaft und polizei mit ausschreitungen umgehen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie grönland den klimawandel für sich nutzen will </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie hoch die belastung durch den corona-lockdown in deutschland ist </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie im vergangenen jahr nur schlimmer: hotspot tirschenreuth </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie kinder- und jugendwerke jugendlichen aus sozialen brennpunkten im lockdown helfen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie kliniken das frühwarnsystem für fehlende intensivbetten beeinträchtigen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie leben kinder mit der corona-krise? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie new york city seine hochhäuser energetisch saniert </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie opfer und ersthelfer des attentats am breitscheidplatz mit den folgen leben </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie rechte die prügelattacke von amberg für sich nutzen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie schlachtbetriebe auch ohne werkverträge funktionieren </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie sich briten in deutschland auf den brexit vorbereiten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie sich die schulen auf einen neustart des unterrichts vorbereiten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie sich die union der k-frage stellt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie sich mallorca ohne massentourismus anfühlt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie sicher ist das? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie sicher sind busreisen? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie soll klimaschutz gehen? koalitionsstreit um co2-steuer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie städte das radfahren sicherer machen könnten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie steht es um die sicherheit deutscher kirchen? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie umgehen mit den russland-sanktionen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie umgehen mit der öffnung von kitas und schulen trotz steigender inzidenzzahlen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie verantwortlich handeln unternehmen in der krise? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie viel bringt das plastiktüten-verbot? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie viele schüler wissen vom holocaust? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie vollendet ist die wende?: interview mit schauspieler jan-josef liefers </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie weit dürfen corona-bestimmungen das versammlungsrecht beeinträchtigen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie weit ist deutschland bei der suche nach dem corona-impfstoff? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wie weiter im dezember: neubewertung der corona-lage </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wieder unterwegs: &quot;fridays for future&quot;-bewegung beendet corona-pause </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wiedereröffnung der gemäldegalerie &quot;alte meister&quot; in dresden </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wiedereröffnung des jüdischen museums in frankfurt am main </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wiedereröffnung des museums ludwig in köln mit einer andy warhol-retrospektive </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wiedervereinigtes irland: neue hoffnungen und alte sorgen der nordiren </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wien als musterbeispiel für sozialen wohnungsbau </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wikileaks-gründer assange in london festgenommen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wilde haare: frisieren in corona-zeiten </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wilhelm-hack-museum in ludwigshafen zeigt den sommer in der pop-art </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wimbledon finale der männer: djokovic besiegt federer in fünf-satz-krimi </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wimbledon: roger federer erreicht finale </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> windenergiegipfel: bericht aus rheinland -pfalz </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> winterschule in kleingruppen für kinder aus bildungsfernen familien </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> winzige tröpfchen: wie groß ist die ansteckungsgefahr durch aerosole? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wird das mutierte virus den erfolg der corona-impfung beeinträchtigen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wird kramp-karrenbauer zum problem für die cdu? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wirecard: das versagen der berater </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wirtschaft in den usa steht unter schock </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wirtschaftsexperten berechnen milliardenschwere steuer-einbußen für 2021 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wirtschaftsminister altmaier will verstöße gegen corona-regeln härter bestrafen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wissenschaftler auf dem forschungsschiff &quot;polarstern&quot; erkunden klimawandel in der arktis </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wissenschaftlicher zwischenbericht: geologische karte für mögliche atommüll-endlager </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wm-aus für deutsche handballer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wo bleibt der soziale wohnraum? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> woher stammt der terror von sri lanka: spurensuche in der muslimischen gemeinde </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wohin geht es mit der deutschen wirtschaft </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wohnungsbaupolitik in hamburg soll aus der akuten wohnungsnot helfen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wohnungsnot: politik streitet über enteignung von immobilienkonzernen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wolfgang schäuble kritisiert den reichstags-eklat </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wolfsburgerinnen feiern sechsten dfb-pokalsieg </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> womöglich bis zu 150 tote bei flüchtlingsdrama vor libyen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wuhan: ein jahr nach abriegelung der stadt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> würdigung des sofas in der corona-krise </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> wut und aufatmen - gütersloh macht wieder auf </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ylva johansson </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ypg im türkisch-syrischen grenzgebiet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zahl der asylanträge sinkt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zahl der corona-infektionen in großbritannien steigt bedrohlich weiter </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zahl der corona-infizierten in china ist so niedrig wie seit 14 tagen nicht mehr </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zahl der neuinfektionen weiter hoch: münchen verschärft maskenpflicht </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zehn jahre bürgerkrieg in syrien und die langzeitfolgen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zehn jahre danach: wie das aktionsbündnis winnenden das trauma überwand </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zehntausende äthiopier auf der flucht vor kämpfen im norden des landes </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zehntausende demonstranten fordern rücktritt von tschechiens ministerpräsident babis </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zehntausende protestieren im osten russlands gegen russische zentralregierung </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zehntausende protestieren in deutschland gegen reform des eu-urheberrechts </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zeitz und pirmasens: zwei der schwächsten kreise </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zentralafrikanische republik: theaterstück will mit humor zur verarbeitung von kriegsverbrechen beitragen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zentralkomitee deutscher katholiken beraten bei vollversammlung über reformprozess </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zerreißprobe der koalition in sachsen-anhalt im streit um den medienstaatsvertrag </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zerrieben zwischen den fronten: türkische kurden im grenzgebiet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zieleinlauf bei der solo-weltumseglung vendée globe </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zinsentscheidung der ezb und die wirkung auf deutsche kleinsparer </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zoff im dannenröder forst: wie umweltaktivisten gegen den ausbau der a49 protestieren </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zu besuch bei trump-fans in der demokratenhochburg staten island </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zu spät und zu wenig geld: studenten am limit </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zuckerbergs cyberwährung: bedenken gegen libra </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zulassung von astrazeneca-vakzin: eu-kommission veröffentlicht vertrag über lieferungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zum geburtstag der kanzlerin: bilanz nach 14 jahren als regierungschefin </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zum weltfrauentag: der blick in deutsche führungsetagen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zunehmend trump-kritik aus den reihen der republikaner </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zur &quot;fußball-bundesliga&quot; und zur &quot;basketball-wm&quot; dürfen auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zur eu-asylpolitik </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zur formel 1 und zum ironman dürfen aus rechtlichen gründen nicht vollständig auf tagesschau.de gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zurück in die vergangenheit: grenzen zu trotz schengenraum </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zustand der deutsch-amerikanischen beziehungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zuwanderung bleibt herausforderung für gesamteuropa </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zwanzig jahre soldatinnen bei der bundeswehr </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zwei abgeordnete tauschen ihre wahlkreise: hamburg-karlsruhe </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zwei filme - eine geschichte: film-ereignis „feinde&quot; </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zwei jahre &quot;synodaler weg&quot;: katholische kirche beginnt mit aufarbeitung von sexuellem missbrauch in der kirche </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zwei menschen und der angreifer sterben bei messerattacke in london </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zwei weitere festnahmen im mordfall lübcke </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zwei wochen nach rassistischem anschlag: trauerfeier in hanau </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zwei wochen vor start der bundesliga: diskussion über unterschiedliche regelungen für fans im stadion </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zweifel an datensicherheit: europäischer gerichtshof kippt datenschutz-abkommen zwischen eu und usa </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zweifel und kritik am härteren lockdown </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zweifelhafte blutspenden: dollar gegen gesundheit </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zweifelsfreie vergiftung nawalnys: bundesregierung fordert kreml zu stellungnahme auf </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zweite chance: wie die grünen im osten doch noch durchstarten wollen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zweites impeachment-verfahren gegen donald trump beginnt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zweitligist-verein dynamo dresden wegen positiver corona-tests gesperrt </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zwischen corona und brexit: wie geht es weiter in großbritannien? </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zwischen den fronten: deutsche kinder von is-angehörigen in syrischen lagern </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zwischen versammlungsfreiheit und gesundheitsschutz: berlin verbietet geplante corona-demo </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zwischenbilanz in nrw nach der ersten woche der lockerungen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zwischenbilanz nach sechs monaten corona in deutschland </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> zwischenbilanz zu verschärften maßnahmen vor corona-gipfel </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
</tbody>
</table>

Interessant, hier ist schon einiges an neuen Features zu holen, nämlich:

- Gab es einen Kommentar?
- Gab es eine Meinung?
- Gab es das Wetter?
- Gab es einen Sportblock?
- Gab es den #mittendrin-Bericht? (sh. Post xxx)
- Wie viele Nachrichtenblöcke gab es in der Sendung?
- Gab es Lottozahlen?

Und diese werden nun extrahiert!
Bei den "Weiteren Meldungen" prüfe ich zunächst, wie viele Nachrichtenblöcke es maximal pro Sendung gab.


```r
tagesthemen_text %>%
  mutate(
    weitere_meldungen = if_else(str_detect(thema, "weitere meldungen im") == TRUE, thema, ""),
    weitere_meldungen = str_remove(weitere_meldungen, "weitere meldungen im überblick")
  ) %>%
  count(weitere_meldungen)
```

```
## # A tibble: 8 x 2
##   weitere_meldungen                                                            n
##   <chr>                                                                    <int>
## 1 ""                                                                        5886
## 2 "\n                                \n                                  …     1
## 3 "\n                                \n                                  …     1
## 4 " i"                                                                        13
## 5 " ii"                                                                       16
## 6 ". die \"neue normalität\" in chinas unternehmen"                            1
## 7 "weitere meldungen im überlick"                                              1
## 8  <NA>                                                                    10341
```
Die Zeile 3 und 4 sind interessant für mich. Offensichtlich gab es maximal eine "II". Für den Fall dass ich die Analyse irgendwann wiederhole und auch mehr als 2 Nachrichtenblöcke vorhanden sind, setze ich die Zählung entsprechend flexibel auf (mit einer Summe).
Bei den anderen Features interessiert mich nur ob es das gab oder nicht; entsprechend nutze ich in der mutate-Anweisung die max-Funktion.


```r
tagesthemen_text <- tagesthemen_text %>%
  filter(!is.na(thema)) %>%
  mutate(
    anzahl_nachrichtenbloecke = if_else(str_detect(thema, "weitere meldungen im überblick") == TRUE, 1, 0),
    hatte_wetter = if_else(str_detect(thema, "das wetter") == TRUE, 1, 0),
    hatte_kommentar = if_else(str_detect(thema, fixed("der kommentar")) == TRUE, 1, 0),
    hatte_meinung = if_else(str_detect(thema, "die meinung") == TRUE, 1, 0),
    hatte_lotto = if_else(str_detect(thema, "lottozahlen") == TRUE, 1, 0),
    hatte_mittendrin = if_else(str_detect(thema, "mittendrin") == TRUE, 1, 0)
  ) %>%
  group_by(datum_zeit) %>%
  mutate(
    anzahl_nachrichtenbloecke = sum(anzahl_nachrichtenbloecke, na.rm = TRUE),
    hatte_wetter = max(hatte_wetter, na.rm = TRUE),
    hatte_kommentar = max(hatte_kommentar, na.rm = TRUE),
    hatte_meinung = max(hatte_meinung, na.rm = TRUE),
    anzahl_themen = max(nummer_thema, na.rm = TRUE),
    hatte_lotto = max(hatte_lotto, na.rm = TRUE),
    hatte_middendrin = max(hatte_mittendrin, na.rm = TRUE)
  )


tagesthemen_text <- tagesthemen_text %>%
  filter(
    str_detect(thema, "weitere meldungen im") == FALSE,
    str_detect(thema, "das wetter") == FALSE,
    str_detect(thema, "kommentar") == FALSE,
    str_detect(thema, "die meinung") == FALSE,
    str_detect(thema, "lottozahlen") == FALSE,
    str_detect(thema, "mittendrin") == FALSE
  )
```



```r
tagesthemen_text <- tagesthemen_text %>%
  mutate_at(vars(contains("hatte")), as.factor)
```




```r
tagesthemen <- tagesthemen %>%
  left_join(
    tagesthemen_text %>%
      select(datum_zeit, anzahl_nachrichtenbloecke, hatte_wetter, hatte_kommentar, hatte_meinung, anzahl_themen) %>%
      distinct()
  )
```



```r
tagesthemen %>%
  filter(extra == 0) %>%
  ggplot(aes(x = hatte_meinung, y = hatte_kommentar)) +
  geom_point() +
  geom_jitter(width = 0.1, height = 0.1)
```

<img src="/posts/2021-05-18-tagesthemen-textanalyse/post_files/figure-html/unnamed-chunk-9-1.png" width="2400" />

```r
tagesthemen %>%
  filter(extra == 0) %>%
  select(date, hatte_kommentar, hatte_meinung) %>%
  pivot_longer(cols = c("hatte_kommentar", "hatte_meinung")) %>%
  filter(value == "1") %>%
  ggplot(aes(x = date, y = value, fill = name)) +
  geom_tile()
```

<img src="/posts/2021-05-18-tagesthemen-textanalyse/post_files/figure-html/unnamed-chunk-9-2.png" width="2400" />
Aha, Mitte 2020 hat man sich bei den Tagesthemen wohl gedacht, "Den Kommentar" umzubenennen in "Die Meinung". Ich schätze das hat im Rahmen der Fake-News-Bewegung mit den Bemühungen der Medien zu tun, Meinungen als solche noch kenntlicher zu machen. 


```r
tagesthemen <- tagesthemen %>%
  mutate(
    hatte_wetter = relevel(hatte_wetter, ref = "0"),
    hatte_kommentar = relevel(hatte_kommentar, ref = "0"),
    hatte_meinung = relevel(hatte_meinung, ref = "0")
  )


model <- lm(dauer ~ hatte_meinung + hatte_wetter + hatte_kommentar, tagesthemen %>% filter(extra == 0))

summary(model)
```

```
## 
## Call:
## lm(formula = dauer ~ hatte_meinung + hatte_wetter + hatte_kommentar, 
##     data = tagesthemen %>% filter(extra == 0))
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -21.577   0.153   0.826   1.357  32.186 
## 
## Coefficients:
##                  Estimate Std. Error t value Pr(>|t|)    
## (Intercept)       10.8435     1.1939   9.082  < 2e-16 ***
## hatte_meinung1    14.4739     0.4684  30.903  < 2e-16 ***
## hatte_wetter1      8.5302     1.1980   7.120 2.42e-12 ***
## hatte_kommentar1  10.0035     0.3444  29.042  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 4.437 on 792 degrees of freedom
## Multiple R-squared:  0.6336,	Adjusted R-squared:  0.6322 
## F-statistic: 456.5 on 3 and 792 DF,  p-value: < 2.2e-16
```


