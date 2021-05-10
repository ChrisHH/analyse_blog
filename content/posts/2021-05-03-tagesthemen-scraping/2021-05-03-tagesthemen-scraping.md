---
title: "Tagesthemen - Scraping"
date: '2021-05-03'
keywords:
- tagesthemen
- R
tags:
- R
- sracping with R
category: blog-post
---

# Scraping der Webseite www.tagesschau.de

## Untersuchung der Webseite

Unter dem Link www.tagesschau.de ist das gesamte Nachrichtenangebot, dass die ARD hergibt, gebündelt. Es ist also eine dankbare Datenquelle, um die aufgeworfenen Fragen zu beantworten.

Nach einer kurzen Inspektion der Seite und der URLs bin ich darauf aufmerksam geworden, dass jede Ausgabe der Tagesthemen eine eigene 
URL besitzt, wie zum Beispiel diese Ausgabe
vom 8. Mai 2021: https://www.tagesschau.de/multimedia/sendung/tt-8257.html

Bei Inspektion mehrerer URLs von Tagesthemen-Seiten stellte ich fest, dass die Zahl "8257" in der URL ein Index ist, der hochzählt.
So habe ich über die Kalenderfunktion (ebenfalls auf der Webseite vorhanden) die URL der Sendung vom 01.01.2019 und vom 17.03.2021 ermittelt.
Das ist also der Start- und Endpunkt des Scrapings und dazwischen wird der Zähler in der URL einfach erhöht! (Mehr dazu später.)

### Was findet man nun auf dieser Seite?

Die wesentlichen Bereiche sind in dem Screenshot farblich markiert:

![Screenshot](/posts/2021-05-03-tagesthemen-scraping/screenshot_tt.png "Quelle: www.tagesschau.de")

Zunächst können wir herausfinden, wann die Sendung ausgestrahlt wurde (organgefarbener Kasten). Das Standbild der Sendung (grüner Kasten) liefert natürlich die Information, welche Person die Sendung moderiert hat und welche Farbe die Kleidung hatte, die diese Person trug. Außerdem ist rechts unten im Standbild die Dauer der Sendung zu erkennen (gelber Kasten; es handelt sich um ein overlay, sodass der Text ermittelbar ist).Schlussendlich sind unter dem Bild die Themen der Sendung einsehbar (violetter Kasten).

Das ist doch schomal eine ganze Menge! (Und es kommt schon der erste Gedanke: Oh oh, wie kann man denn die Kleidung der Moderator*innen von den Personen im Hintergrund (auf der Medienwand) unterscheiden? Dieser Gedanke wird erstmal aufgeschoben und zu gegebener Zeit behandelt - es wird aber mit Sicherheit ein Thema, das in Richtung Bild-Segmentierung geht.)

Nun also erstmal zum Scraping!

## Scraping mit R

Das Scraping mit R ist mit den Paketen tidyverse, rvest, lubridate und xml2 überraschend einfach möglich.
Ich habe zunächst eine Funktion geschrieben, die für jede URL, die ihr übergeben wird, folgendes ermittelt:

- Datum und Sendezeit
- Themen der Sendung
- Länge der Sendung
- Link zum Standbild

Zunächst wird in der Funktion der HTML-Inhalt der Seite ausgelesen und in der Variable tt gespeichert.
Dieser Inhalt wird dann nach o.g. Kriterien untersucht.

```{r}

library(tidyverse)
library(rvest)
library(lubridate)


extract_tagestehmen <- function(url) {

    tt <- read_html(url)
  
    
    # Extrahiere Tag und Uhrzeit ----------------------------------------------
    
    datum_zeit <- tt %>% 
      rvest::html_nodes('body') %>% 
      xml2::xml_find_all("//span[@class='multimediahead__date']") %>% 
      rvest::html_text() 
    
    
    datum_zeit <- str_remove(datum_zeit, "Sendung vom ") %>% 
      str_remove("Uhr") %>% 
      trimws()
    
    datum_zeit <- as.POSIXct(datum_zeit, format = "%d.%m.%Y, %H:%M")
    
    # Extrahiere Themen der Sendung -------------------------------------------
    
    themen <- tt %>% 
      rvest::html_nodes('body') %>% 
      xml2::xml_find_all("//div[@class='copytext__video__details']")  %>% 
      rvest::html_text()
    
    
    # Extrahiere Länge der Sendung in Sekunden --------------------------------
    
    dauer <- tt %>% 
      html_nodes('body') %>% 
      xml_find_first("//div[@data-ts_component='ts-mediaplayer']") %>% 
      xml_attr("data-config")
    
    dauer <- as.tibble(t(str_split(dauer, ",", simplify = TRUE))) %>%
      filter(str_detect(V1,"_duration")) 
    
    dauer <- as.character(dauer[1,1]) %>% 
      str_extract("(\\d)+")
    
    
    # Extrahiere Standbild der Sendung ----------------------------------------
    
    standbild_url <- tt %>% 
      html_nodes('body') %>% 
      xml_find_first("//div[@data-ts_component='ts-mediaplayer']") %>% 
      xml_attr("data-config")
    
    standbild_url <- as.tibble(t(str_split(standbild_url, ",", simplify = TRUE))) %>%
      filter(str_detect(V1,"\"xl\":\"")) %>% 
      mutate(V1 = str_remove(V1, "\"xl\":\""),
             V1 = str_sub(V1, end = -3))
    
    standbild_url <- paste("https://www.tagesschau.de",standbild_url[1,1], sep = "")
    
    
    return(tibble(datum_zeit, dauer, themen, standbild_url, url))
    
}

```

Diese Funktion habe ich in einem for-loop aufgerufen, und die Ergebnisse in einem Tibble gespeichert.
Da der Index nicht immer exakt um 1 erhöht wird, existieren manche URLs nicht. Diese werden mit dem tryCatch Aufruf behandelt, sodass die Schleife nicht einfach abbricht.


```{r}
tagesthemen <- tibble()

for(i in 6471:8147) {
  
  link <- paste("https://www.tagesschau.de/multimedia/sendung/tt-",i,".html", sep = "")
  
  tryCatch(
    {
      tagesthemen <- tagesthemen %>% 
        bind_rows(extract_tagestehmen(link))
    }, error = function(err) {print(paste("URL:",link,"existiert nicht!"))
      return(0)} )
  
  Sys.sleep(5)
  
}
```

Am Ende steht der Tibble mit dem Namen "tagestehmen" und enthält alle wichtigen Basisinformationen - genau: BASISinformationen.
Was fehlt? Natürlich: die Namen der Moderator*innen, denn bisher ist ja nur die URL des Standbildes bekannt.

Mit den nachfolgenden Zeilen lassen sich diese Bilder einfach herunterladen.

```{r}

linklist <- tagesthemen$standbild_url
linklist <- linklist[linklist != "https://www.tagesschau.deNA"]


for (link in linklist){
  
  url <- link
  destfile <- str_remove(link, "https://www.tagesschau.de/multimedia/bilder/")
  
  tryCatch(
    {
      download.file(link, destfile)
    }, error = function(err) {print(paste("URL:",link,"existiert nicht!"))
      return(0)} )

  Sys.sleep(1)
}

```

### Vom Bild zum Namen ohne viel Aufwand?

Ich habe kurz darüber nachgedacht, wie ich nun aus den Bildern den Namen ableiten kann. Hier habe ich mir das Leben leicht gemacht und diese Bilder einfach per Batch nach Amazon Photos hochgeladen. Dort habe ich ein paar Gesichter markiert und per Tag den Namen der Person vergeben; nach einem Tag hatte Amazon den Job erledigt und alle Namen für alle Fotos getaggt. Die Bilder habe ich dann wieder heruntergeladen und schon konnte ich über den Dateinamen den Namen der moderierenden Person zuordnen. Dieses Vorgehen müsste eigentlich auch gut mit der Foto-App auf den Geräten von Apple funktionieren; das habe ich aber nicht ausprobiert.

## Ausblick

In den nächsten Posts geht es ganz im Sinne von CRISP-DM erstmal um die Datensichtung, Datenbereinigung und eine erste explorative Datenanalyse, bevor das Instrumentarium dann komplexer wird.