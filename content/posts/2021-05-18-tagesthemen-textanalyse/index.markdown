---
title: Tagesthemen - Textanalyse
author: Christian
date: '2021-05-18'
slug: []
categories:
  - Natural Language Processing
  - Textanalytics
tags:
  - NLP
  - Tidytext
  - Tidymodels
draft: yes
---



## Einleitung

Ich habe jetzt schon die eine oder andere deskriptive Analyse der Tagesthemen durchgeführt.
Bisher noch völlig ausser acht gelassen ist die Themenvielfalt der Sendungen. Und darum geht es im heutigen Post
Wie schon beschrieben, 



```r
tagesthemen <- read_csv("https://funalytics.netlify.app/files/data_clean.csv")
```


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


```r
view(tagesthemen_text %>%
  count(thema, sort = TRUE))
```

Der Ausdruck "Das Wetter" und "Weitere Meldungen im Überlbick" taucht im Ranking mehrfach auf. Zunächst widme ich mich dem letztgenannten string. Hierraus lässt sich ein weiteres Feature für die Sendungen ableiten, nämlich ob es nur einen oder sogar zwei Nachrichtenblöcke gab.


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

```r
tagesthemen_text <- tagesthemen_text %>%
  filter(!is.na(thema)) %>%
  mutate(
    anzahl_nachrichtenbloecke = if_else(str_detect(thema, "weitere meldungen im überblick") == TRUE, 1, 0),
    hatte_wetter = if_else(str_detect(thema, "das wetter") == TRUE, 1, 0),
    hatte_kommentar = if_else(str_detect(thema, fixed("der kommentar")) == TRUE, 1, 0),
    hatte_meinung = if_else(str_detect(thema, "die meinung") == TRUE, 1, 0)
  ) %>%
  group_by(datum_zeit) %>%
  mutate(
    anzahl_nachrichtenbloecke = sum(anzahl_nachrichtenbloecke, na.rm = TRUE),
    hatte_wetter = sum(hatte_wetter, na.rm = TRUE),
    hatte_kommentar = sum(hatte_kommentar, na.rm = TRUE),
    hatte_meinung = sum(hatte_meinung, na.rm = TRUE),
    anzahl_themen = max(nummer_thema, na.rm = TRUE)
  )


tagesthemen_text <- tagesthemen_text %>%
  filter(
    str_detect(thema, "weitere meldungen im") == FALSE,
    str_detect(thema, "das wetter") == FALSE,
    str_detect(thema, "kommentar") == FALSE,
    str_detect(thema, "die meinung") == FALSE
  )

tagesthemen_text %>%
  ungroup() %>%
  select(contains("hatte"), anzahl_themen) %>%
  skimr::skim()
```


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
   <td style="text-align:left;"> Piped data </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Number of rows </td>
   <td style="text-align:left;"> 3913 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Number of columns </td>
   <td style="text-align:left;"> 4 </td>
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
   <td style="text-align:left;"> numeric </td>
   <td style="text-align:left;"> 4 </td>
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
   <td style="text-align:left;"> hatte_wetter </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 0.98 </td>
   <td style="text-align:right;"> 0.14 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:left;"> ▁▁▁▁▇ </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hatte_kommentar </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 0.47 </td>
   <td style="text-align:right;"> 0.50 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:left;"> ▇▁▁▁▇ </td>
  </tr>
  <tr>
   <td style="text-align:left;"> hatte_meinung </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 0.19 </td>
   <td style="text-align:right;"> 0.39 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:left;"> ▇▁▁▁▂ </td>
  </tr>
  <tr>
   <td style="text-align:left;"> anzahl_themen </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 7.79 </td>
   <td style="text-align:right;"> 1.74 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 7 </td>
   <td style="text-align:right;"> 8 </td>
   <td style="text-align:right;"> 9 </td>
   <td style="text-align:right;"> 17 </td>
   <td style="text-align:left;"> ▁▅▇▁▁ </td>
  </tr>
</tbody>
</table>

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

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-7-1.png" width="2400" />

```r
tagesthemen %>%
  filter(extra == 0) %>%
  select(date, hatte_kommentar, hatte_meinung) %>%
  pivot_longer(cols = c("hatte_kommentar", "hatte_meinung")) %>%
  filter(value == "1") %>%
  ggplot(aes(x = date, y = value, fill = name)) +
  geom_tile()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-7-2.png" width="2400" />
Aha! Mitte 2020 hat man sich bei den Tagesthemen wohl gedacht, "Den Kommentar" umzubenennen in "Die Meinung". Ich schätze das hat mit den Bemühungen der Medien zu tun, Meinungen als solche noch transparenter darzustellen. 


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


