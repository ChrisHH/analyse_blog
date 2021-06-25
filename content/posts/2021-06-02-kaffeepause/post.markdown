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

Er ist wortwörtlich in aller Munde: der Kaffee.
...




```r
library(tidytuesdayR)

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
table(coffee$species) 
```

```
## 
## Arabica Robusta 
##    1311      28
```

