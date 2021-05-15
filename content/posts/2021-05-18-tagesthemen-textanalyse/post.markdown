---
title: "Tagesthemen - Analyse der Themen"
subtitle: "Dieser Post ist noch work in progress"
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

Ich habe jetzt schon die eine oder andere deskriptive Analyse der Tagesthemen durchgeführt.
Bisher noch völlig ausser acht gelassen ist die Themenvielfalt der Sendungen. Und darum geht es im heutigen Post!

Zunächst werden die bis zu diesem Punkt verfeinerten Daten eingelesen.


```r
tagesthemen <- read_csv("https://funalytics.netlify.app/files/data_clean.csv")
```

## Ableitung von Features aus den Themen der Sendung

Aus den Themen der Sendung lassen sich bestimmt noch Features ableiten. Um dies zu realisieren kommt das R-Paket "tidytext" von Julia Silge und David Robinson zum Einsatz. Ich kann das zugehörige Buch "Text Mining with R (O'Reilly Verlag) sehr empfehlen. Es existiert aber auch als open source Version unter [diesem Link] (https://www.tidytextmining.com).

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
# Check ob Anzahl von 20 Variablen in separate Anweisung ausreicht

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

Das sieht gut aus!

Nun verschaffe ich mir einen Überblick über die Häufigkeit einzelner Themen; dabei filtere ich auf Themen die mind. 10 mal vorkommen 


```r
tagesthemen_text %>%
  filter(!is.na(thema)) %>%
  count(thema, sort = TRUE) %>%
  filter(n >= 10) %>%
  arrange(n) %>%
  ggplot(aes(x = reorder(thema, n), y = n)) +
  geom_bar(stat = "identity") +
  coord_flip()
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


