---
title: Kaffeepause
author: Christian Jähnert
date: '2021-06-02'
slug: []
categories:
  - R
  - Tidyverse
  - Tidytuesday
  - Kaffee
tags: []
draft: yes
---
Er ist wortwörtlich in aller Munde: Kaffee. Ob als Filterkaffee am Morgen zum Wachwerden, als Espresso nach dem Mittagessen oder als Cafe Creme zum Kuchen am Nachmittag. Ob heiß gebrüht oder auf Eiswürfeln serviert. Mal kommt er schwarz daher, mal weiß, mit viel Zucker und Milchschaumkrone. Wie auch immer Kaffee serviert wird: allen Leuten die Kaffee konsumieren geht es darum, dass er auch gut schmeckt.

Und um den Geschmack des Kaffees geht es im heutigen Post.
...



```r
library(tidytuesdayR)
library(tidyverse)
```

```
## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.1 ──
```

```
## ✓ ggplot2 3.3.5     ✓ purrr   0.3.4
## ✓ tibble  3.1.2     ✓ dplyr   1.0.7
## ✓ tidyr   1.1.3     ✓ stringr 1.4.0
## ✓ readr   1.4.0     ✓ forcats 0.5.1
```

```
## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
## x dplyr::filter() masks stats::filter()
## x dplyr::lag()    masks stats::lag()
```

```r
tuesdata <- tidytuesdayR::tt_load('2020-07-07')
```

```
## --- Compiling #TidyTuesday Information for 2020-07-07 ----
```

```
## --- There is 1 file available ---
```

```
## --- Starting Download ---
```

```
## 
## 	Downloading file 1 of 1: `coffee_ratings.csv`
```

```
## --- Download complete ---
```

```r
coffee <- tuesdata$coffee_ratings

rm(tuesdata)
```



```r
coffee %>%  
  count(country_of_origin, sort = TRUE)
```

```
## # A tibble: 37 x 2
##    country_of_origin                n
##    <chr>                        <int>
##  1 Mexico                         236
##  2 Colombia                       183
##  3 Guatemala                      181
##  4 Brazil                         132
##  5 Taiwan                          75
##  6 United States (Hawaii)          73
##  7 Honduras                        53
##  8 Costa Rica                      51
##  9 Ethiopia                        44
## 10 Tanzania, United Republic Of    40
## # … with 27 more rows
```

```r
coffee %>% 
  group_by(country_of_origin) %>% 
  summarise(avg_aroma = mean(flavor)) %>% 
  arrange(-avg_aroma)
```

```
## # A tibble: 37 x 2
##    country_of_origin avg_aroma
##    <chr>                 <dbl>
##  1 Papua New Guinea       8.42
##  2 Ethiopia               8.01
##  3 United States          7.99
##  4 Rwanda                 7.92
##  5 Kenya                  7.78
##  6 Uganda                 7.75
##  7 Japan                  7.75
##  8 Peru                   7.66
##  9 El Salvador            7.65
## 10 Ecuador                7.64
## # … with 27 more rows
```

