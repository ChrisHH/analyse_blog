---
title: "Tagesthemen - Ableitung von Features aus den Themen"
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
  count(day)
```

```
## # A tibble: 1 x 2
##   day         n
##   <chr>   <int>
## 1 Samstag    28
```

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
  select(datum_zeit, dauer)
```

```
## # A tibble: 14 x 2
##    datum_zeit          dauer
##    <dttm>              <dbl>
##  1 2019-01-19 21:10:00  7.83
##  2 2019-02-17 22:45:00 20.1 
##  3 2019-04-02 21:33:00  7.95
##  4 2019-04-24 21:35:00  7.67
##  5 2019-06-17 21:50:00  8.15
##  6 2019-06-30 23:45:00  6.65
##  7 2019-07-31 22:15:00 30.2 
##  8 2020-01-31 23:40:00 25.6 
##  9 2020-02-04 21:35:00  6.68
## 10 2020-08-17 22:15:00 30.7 
## 11 2020-09-07 22:15:00 35.2 
## 12 2020-12-23 21:35:00  7.02
## 13 2021-01-13 21:35:00  7.68
## 14 2021-03-03 23:15:00 14.8
```

Die Sendungen, in denen es keinen Wetterbericht gab, waren überwiegend relativ kurz. 
Es handelte sich um kurze Tagesthemen, die z.B häufig während der Halbzeit eines Fußballspiels ausgestrahlt werden.
Bei den anderen? Da hat jemand vergessen, den Wetterbericht in den Themen einzutragen.
Dennoch belasse ich die Varialbe im Datensatz und ändere sie nicht weiter, dies nicht sehr häufig vorkommt (in Relation zu allen Sendungen).

Schlussendlich erfolgt nun eine erste multivariate Analyse. Ich möchte wissen, ob und in welchem Ausmaß die Sendedauer beeinflusst wird durch:

- Moderator*in
- Kommentar / Meinung
- Anzahl der Themen
- Anzahl Nachrichtenblöcke

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
linear_model <- lm(dauer ~ name + kommentar_meinung + anzahl_themen + anzahl_nachrichtenbloecke, tagesthemen)

summary(linear_model)
```

```
## 
## Call:
## lm(formula = dauer ~ name + kommentar_meinung + anzahl_themen + 
##     anzahl_nachrichtenbloecke, data = tagesthemen)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -19.682  -1.646   0.215   1.882  39.790 
## 
## Coefficients:
##                           Estimate Std. Error t value Pr(>|t|)    
## (Intercept)                 8.6680     0.7228  11.992   <2e-16 ***
## nameIngo Zamperoni         -0.1234     0.3422  -0.361   0.7185    
## namePinar Atalay           -0.6411     0.4090  -1.567   0.1174    
## nameVertretung             -1.8135     1.3746  -1.319   0.1874    
## kommentar_meinung1          7.3719     0.3981  18.518   <2e-16 ***
## anzahl_themen               1.8843     0.1099  17.140   <2e-16 ***
## anzahl_nachrichtenbloecke  -0.9790     0.4285  -2.285   0.0226 *  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 4.271 on 806 degrees of freedom
## Multiple R-squared:  0.6886,	Adjusted R-squared:  0.6863 
## F-statistic: 297.1 on 6 and 806 DF,  p-value: < 2.2e-16
```



