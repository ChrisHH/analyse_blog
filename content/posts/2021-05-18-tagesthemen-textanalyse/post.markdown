---
title: "Tagesthemen - Featureextraktion aus den Themen den Sendung"
date: '2021-05-15'
keywords:
  - R
  - Explorative Datenanalyse
  - NLP
  - Feature Engineering
tags:
  - R
  - Tagesthemen
---



## Einleitung

Ich habe in vorigen Posts schon die eine oder andere deskriptive Analyse der Tagesthemen durchgeführt.
Bisher noch völlig ausser acht gelassen ist die Themenvielfalt der Sendungen. Und darum geht es im heutigen Post!

Zunächst werden die bis zu diesem Punkt verfeinerten Daten eingelesen.


```r
tagesthemen <- read_csv("https://funalytics.netlify.app/files/data_clean.csv")
```

## Ableitung von Features aus den Themen der Sendung

Aus den Themen der Sendung lassen sich bestimmt noch Features ableiten. Um dies zu realisieren kommt das R-Paket "tidytext" von Julia Silge und David Robinson zum Einsatz. Ich kann das zugehörige Buch "Text Mining with R (O'Reilly Verlag) sehr empfehlen. Es existiert aber auch als open source Version unter diesem Link: https://www.tidytextmining.com

Damit sie analysierbar sind, müssen im ersten Schritt die Themen der Sendung "vereinzelt" werden. Nach kurzer Betrachtung des Strings stellte ich fest, dass die Theman durch ein Komma getrennt sind. Es bietet sich also ein String-Split mit Hilfe der separate Funktion an. Diese verlangt jedoch die Vorgabe einer Anzahl von Variablen, in die gesplittet werden soll. Ich habe die Anzahl einmal auf 20 gesetzt und prüfe im Nachgang ob das ausreichend war. Außerdem ist es bei Textanalysen immer sehr sinnvoll, Leerzeichen zu "strippen" und den gesamten String in Groß- oder Kleinschreibung umzuwandeln, wobei sich im NLP Umfeld die Kleinschreibung durchgesetzt hat. 

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
In der letzten Variable, also 'thema.20' bzw. der Variable 'nummer_thema' mit der Ausprägung 20, müssten alle Themen ein 'NA' besitzen.
Das heißt die Anzahl der 'NA's muss der Anzahl aller Fälle entsprechen.


```r
tagesthemen_text %>%
  filter(nummer_thema == 20) %>%
  count(thema) %>%
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
   <td style="text-align:right;"> 813 </td>
  </tr>
</tbody>
</table>

Das sieht gut aus: 813 NA's, also ja, die Anzahl war ausreichend!

Nun verschaffe ich mir einen Überblick über die Häufigkeit einzelner Themen; dabei filtere ich auf Themen die mind. 10 mal vorkommen und stelle die Häufigkeit dar.


```r
tagesthemen_text %>%
  filter(!is.na(thema)) %>%
  count(thema, sort = TRUE) %>%
  filter(n >= 10) %>%
  arrange(n) %>%
  ggplot(aes(x = reorder(thema, n), y = n, label = n)) +
  geom_bar(stat = "identity") +
  coord_flip() +
  labs(x = "Thema", y = "Häufigkeit") +
  geom_label(size = 3)
```

<img src="/posts/2021-05-18-tagesthemen-textanalyse/post_files/figure-html/unnamed-chunk-4-1.png" width="2400" />

Interessant, denn hier ist schon einiges an neuen Features zu holen, nämlich:

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
  count(weitere_meldungen, sort = TRUE) %>%
  kable()
```

<table>
 <thead>
  <tr>
   <th style="text-align:left;"> weitere_meldungen </th>
   <th style="text-align:right;"> n </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:right;"> 10341 </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:right;"> 5886 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ii </td>
   <td style="text-align:right;"> 16 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> i </td>
   <td style="text-align:right;"> 13 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hinweis:
                                    der beitrag zum fußball bundesliga darf auf tagesschau.de aus rechtlichen gründen nicht gezeigt werden. </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hinweis:
                                    diese sendung wurde nachträglich bearbeitet </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> . die &quot;neue normalität&quot; in chinas unternehmen </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> weitere meldungen im überlick </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
</tbody>
</table>

Die Zeile 3 und 4 sind interessant für mich. Offensichtlich gab es maximal eine "II". Für den Fall dass ich die Analyse irgendwann wiederhole und auch mehr als 2 Nachrichtenblöcke vorhanden sind, setze ich die Zählung entsprechend flexibel auf (mit einer Summe). Bei den anderen Features interessiert mich nur ob es das gab oder nicht; entsprechend nutze ich in der mutate-Anweisung die max-Funktion. Abschließend verwandle ich diese Variablen noch in Faktoren, da die Tatsache ob es in den Tagesthemen z.B einen Kommentar gab ein kategoriales Merkmal ist. Zuguterletzt lösche ich die Themen Themen für die weitere Analyse heraus.


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
    hatte_mittendrin = max(hatte_mittendrin, na.rm = TRUE)
  ) %>%
  mutate_at(vars(contains("hatte")), as.factor)


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

Um mit den Features, abgeleitet aus den Themen noch weitere Analysen machen zu können, joine ich diese Features zurück an den regulären DataFrame.
Warum? Der DataFrame 'tagestehmen_text' ist vertikalisiert und damit sind weitere deskriptive Analysen eher umständlich.


```r
tagesthemen <- tagesthemen %>%
  left_join(
    tagesthemen_text %>%
      select(datum_zeit, anzahl_nachrichtenbloecke, hatte_wetter, hatte_kommentar, hatte_meinung, hatte_lotto, anzahl_themen, hatte_mittendrin) %>%
      distinct()
  )
```

Super! Jetzt kann es weitergehen und ich kann weitere Analysen vornehmen.

Als erstes interessiert mich, was hat es eigentlich mit "Der Kommentar" und "Die Meinung" auf sich. Beides klingt sehr ähnlich und ich schaue mir das Auftreten mal im Zeitverlauf an.



```r
tagesthemen %>%
  filter(extra == 0) %>%
  select(date, hatte_kommentar, hatte_meinung) %>%
  pivot_longer(cols = c("hatte_kommentar", "hatte_meinung")) %>%
  filter(value == "1") %>%
  ggplot(aes(x = date, y = value, fill = name)) +
  geom_tile() +
  labs(x = "Datum", y = "", fill = "Variable")
```

<img src="/posts/2021-05-18-tagesthemen-textanalyse/post_files/figure-html/unnamed-chunk-8-1.png" width="2400" />
OK, Mitte 2020 hat man sich bei den Tagesthemen wohl gedacht, "Den Kommentar" umzubenennen in "Die Meinung". Ich schätze das hat im Rahmen der Fake-News-Bewegung mit den Bemühungen der Medien zu tun, Meinungen als solche noch kenntlicher zu machen.

Ich fasse beide Variablen mal kurz zusammen und arbeite dann damit weiter. Das mache ich, in dem ich die zwei Variablen einfach addiere (da sie 0-1 codiert sind). Eine Anmerkung: dazu müssen die Faktoren zunächst in eine numerische Variable umgewandelt werden. Das gelingt etwas umständlich, in dem man die Faktoren zunächst in Charakter- und dann in numerische Variablen umwandelt 


```r
tagesthemen <- tagesthemen %>%
  mutate(
    kommentar_meinung = as.numeric(as.character(hatte_meinung)) +
      as.numeric(as.character(hatte_kommentar)),
    kommentar_meinung = as.factor(kommentar_meinung)
  )
```

Und auch hier wieder ein kurzer Check ob es funktioniert hat.


```r
tagesthemen %>%
  filter(
    extra == 0,
    kommentar_meinung == 1
  ) %>%
  ggplot(aes(x = date, y = kommentar_meinung)) +
  geom_tile(color = "midnightblue") +
  labs(x = "Datum", y = "", fill = "Variable")
```

<img src="/posts/2021-05-18-tagesthemen-textanalyse/post_files/figure-html/unnamed-chunk-10-1.png" width="2400" />

In diesem Plot sieht man schon sehr gut, dass gegen Ende des Jahres, die Kommentar- bzw. Meinungsbeiträge in den Tagesthemen eher rar sind.

Nun interessieren mich noch die Lottozahlen. Diese sind ja gleichzeitig eine gute Möglichkeit, die Datenqualität zwischendurch erneut zu überprüfen, denn sie sollten nur an den Ziehungstagen, also mittwochs oder samstags auftauchen.


```r
tagesthemen %>%
  filter(
    extra == 0,
    hatte_lotto == 1
  ) %>%
  count(day) %>%
  kable()
```

<table>
 <thead>
  <tr>
   <th style="text-align:left;"> day </th>
   <th style="text-align:right;"> n </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> Samstag </td>
   <td style="text-align:right;"> 28 </td>
  </tr>
</tbody>
</table>

Schade, das hat nicht geklappt. Denn demzufolge wären im Analysezeitraum nur 28 mal die Lottozahlen gesendet worden.
Aber das ist nicht korrekt; vielmehr ist es so, dass die Lottozahlen als Thema nur 28 mal separat aufgeführt wurden.
Das Feature könnte man nun einfach über den Mittwoch und Samstag ergänzen, aber ich entscheide mich, dies einfach zu löschen.


```r
tagesthemen <- tagesthemen %>%
  select(-hatte_lotto)
```

Ein oft gehörter Satz in jeder Nachrichtensendung? "Wie immer zum Schluss: das Wetter!".


```r
tagesthemen %>%
  filter(extra == 0) %>%
  count(hatte_wetter) %>%
  kable()
```

<table>
 <thead>
  <tr>
   <th style="text-align:left;"> hatte_wetter </th>
   <th style="text-align:right;"> n </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> 1 </td>
   <td style="text-align:right;"> 782 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 0 </td>
   <td style="text-align:right;"> 14 </td>
  </tr>
</tbody>
</table>

Interessant, laut Auszählung gab es in 14 Sendungen kein Wetter?
Hier lohnt nochmal ein Blick auf die Sendungen.


```r
tagesthemen %>%
  filter(extra == 0, hatte_wetter == 0) %>%
  select(datum_zeit, dauer) %>%
  kable()
```

<table>
 <thead>
  <tr>
   <th style="text-align:left;"> datum_zeit </th>
   <th style="text-align:right;"> dauer </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> 2019-01-19 21:10:00 </td>
   <td style="text-align:right;"> 7.833333 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2019-02-17 22:45:00 </td>
   <td style="text-align:right;"> 20.083333 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2019-04-02 21:33:00 </td>
   <td style="text-align:right;"> 7.950000 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2019-04-24 21:35:00 </td>
   <td style="text-align:right;"> 7.666667 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2019-06-17 21:50:00 </td>
   <td style="text-align:right;"> 8.150000 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2019-06-30 23:45:00 </td>
   <td style="text-align:right;"> 6.650000 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2019-07-31 22:15:00 </td>
   <td style="text-align:right;"> 30.183333 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2020-01-31 23:40:00 </td>
   <td style="text-align:right;"> 25.650000 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2020-02-04 21:35:00 </td>
   <td style="text-align:right;"> 6.683333 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2020-08-17 22:15:00 </td>
   <td style="text-align:right;"> 30.700000 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2020-09-07 22:15:00 </td>
   <td style="text-align:right;"> 35.200000 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2020-12-23 21:35:00 </td>
   <td style="text-align:right;"> 7.016667 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-01-13 21:35:00 </td>
   <td style="text-align:right;"> 7.683333 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-03-03 23:15:00 </td>
   <td style="text-align:right;"> 14.850000 </td>
  </tr>
</tbody>
</table>

Die Sendungen, in denen es keinen Wetterbericht gab, waren überwiegend relativ kurz. 
Es handelte sich um kurze Tagesthemen, die z.B häufig während der Halbzeit eines Fußballspiels ausgestrahlt werden.
Bei den anderen? Da hat jemand vergessen, den Wetterbericht in den Themen einzutragen.
Dennoch belasse ich die Varialbe im Datensatz und ändere sie nicht weiter, dies nicht sehr häufig vorkommt (in Relation zu allen Sendungen).

Schlussendlich erfolgt nun eine erste multivariate Analyse. Ich möchte wissen, ob und in welchem Ausmaß die Sendedauer beeinflusst wird durch:

- Moderator*in
- Kommentar / Meinung
- Anzahl der Themen
- Anzahl Nachrichtenblöcke
- Extrausgabe
- Wochentag

Dafür wird auf die lineare Regression zurückgegriffen. Damit später das Ergebnis der linearen Regression nachvollziehbar interpretiert werden kann, wird zunächst für die Faktor-Variablen das Referenz-Level gesetzt. 


```r
tagesthemen <- tagesthemen %>%
  mutate(
    kommentar_meinung = relevel(kommentar_meinung, ref = "0"),
    hatte_mittendrin = relevel(hatte_mittendrin, ref = "0"),
    name = as.factor(name)
  )
```

Nun erfolgt die lineare Regression


```r
linear_model <- lm(dauer ~ name + kommentar_meinung + anzahl_themen + anzahl_nachrichtenbloecke + extra + day, tagesthemen)

summary(linear_model)
```

```
## 
## Call:
## lm(formula = dauer ~ name + kommentar_meinung + anzahl_themen + 
##     anzahl_nachrichtenbloecke + extra + day, data = tagesthemen)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -19.495  -1.424   0.178   1.853  39.553 
## 
## Coefficients:
##                           Estimate Std. Error t value Pr(>|t|)    
## (Intercept)               10.42598    0.90798  11.483  < 2e-16 ***
## nameIngo Zamperoni        -0.09487    0.33413  -0.284  0.77654    
## namePinar Atalay          -0.47445    0.40015  -1.186  0.23610    
## nameVertretung            -1.20067    1.36849  -0.877  0.38055    
## kommentar_meinung1         7.04232    0.48755  14.444  < 2e-16 ***
## anzahl_themen              1.71687    0.11622  14.773  < 2e-16 ***
## anzahl_nachrichtenbloecke -1.34031    0.43344  -3.092  0.00206 ** 
## extra                     -4.72382    1.16618  -4.051  5.6e-05 ***
## dayDonnerstag              0.75450    0.55178   1.367  0.17189    
## dayFreitag                -1.47978    0.57252  -2.585  0.00992 ** 
## dayMittwoch                0.14604    0.54436   0.268  0.78856    
## dayMontag                  1.15994    0.55096   2.105  0.03558 *  
## daySamstag                -1.08411    0.63368  -1.711  0.08751 .  
## daySonntag                 0.65747    0.62164   1.058  0.29054    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 4.168 on 799 degrees of freedom
## Multiple R-squared:  0.7061,	Adjusted R-squared:  0.7013 
## F-statistic: 147.7 on 13 and 799 DF,  p-value: < 2.2e-16
```
Gut, was sagt uns dieser Output?

Der Intercept sind 10.4 Minuten und bezieht sich darauf, dass Caren Miosga die Sendung an einem Dienstag moderiert (da sie bei der Variable "name" im weiteren nicht auftaucht; d.h. sie ist das Referenz-Level - dsgl. gilt für den Wochentag). Wenn Ingo Zamperoni die Sendung moderiert, dann verkürzt sich die Sendezeit um 0.1 Minute marginal. Um 1.2 Minuten kürzer ist die Sendung, wenn sie von einer Vertretung moderiert wird. Wenn ein Kommentar/Meinungsbeitrag in der Sendung stattfindet, dann erhöht sich die Sendedauer um 7 Minuten (was aber nicht heißt, dass der Meinungsbeitrag so lange dauert). Mit jedem zusätzlichen Thema erhöht sich die Sendedauer um 1.7 Minuten; hingegen reduziert sie sich um 1.3 Minuten mit jedem Nachrichtenblock. Handelt es sich um eine Extra-Ausgabe, dann reduziert sich die Sendedauer um 4.7 Minuten. An einem Montag ist die Sendung um 1.2 min länger; hingegen an einem Freitag um 1.5 min kürzer.

Diese Erklärung ist schon in einem gelben Bereich (Richtung) grün, da das angepasste R^2 bei 0.701 liegt. Das heißt, dass die Varianz in der Sendedauer durch die unabhängigen Variablen zu knapp 70% erklärt wird. Ich denke ein weiterer Fakt der die Sendedauer bestimmt, ist die Anfangszeit der Sendezeit. Je früher, desto länger und je später, desto kürzer.
