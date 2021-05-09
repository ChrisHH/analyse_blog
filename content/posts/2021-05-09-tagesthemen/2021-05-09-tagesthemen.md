---
title: "Tagesthemen"
date: '2021-05-09'
keywords:
- tagesthemen
- R
tags:
- R
- sracping with R
category: blog-post
---

# Analyse der Tagesthemen

## Vorgeschichte

"Sie trägt es wieder, dieses hellblaue Kleid!"

![Caren Miosga, Quelle: Tagesthemen.de](/posts/2021-05-09-tagesthemen/sendungsbild-596897~_v-grossgalerie16x9.jpg)

Diesen Gedanken habe ich hin und wieder, wenn ich Caren Miosga als Moderatorin der Tagesthemen zuschaue, wie sie das Tagesgeschehen in den Tagesthemen einordnet. Die Gedanken gehen dann weiter: "Super, dass der NDR nachhaltig agiert, verantwortungsbewusst mit GEZ-Geldern umgeht und die Kleidung der Moderator*innen mehrfach getragen werden!"

Irgendwann wuchs aus diesen Gedanken die Idee, einmal genauer unter die Lupe zu nehmen, was bei den Tagesthemen so passiert.
Ich kam spontan auf folgende Fragen:

- Welche Personen moderieren eigentlich die Tagesthemen wie häufig?
- Wie lange dauert eine Sendung durchschnittlich?
- Gibt es Unterschiede nach Tagen oder eine Saisonalität?
- Haben die Moderator*innen erkennbar bevorzugte Lieblingsfarben ihrer Kleidung?
- Und ist die Farbe der Kleidung beeinflusst durch die Themen der Sendung?
- Wie oft wird ein Kleidungsstück genutzt; über welchen Zeitraum?

... die Liste ließe sich noch erweitern.

Diese Auswahl an Fragen war für mich Grund genug mit Hilfe verschiedener Instrumente aus dem Bereich Data Science loszulegen (und wie so oft, wird sich zwischendurch noch die eine oder andere Frage ergeben). In loser Folge versuche ich auf die einzelnen Fragen Antworten zu geben und darzustellen, wie ich die Lösung vorangetrieben habe.

![Caren Miosga, Quelle: Tagesthemen.de](/posts/2021-05-09-tagesthemen/sendungsbild-661551~_v-grossgalerie16x9.jpg)

## Datenbasis

Unter dem Link www.tagesschau.de ist das gesamte Nachrichtenangebot, dass die ARD hergibt, gebündelt. Es ist also eine dankbare Datenquelle, um die aufgeworfenen Fragen zu beantworten.

Nach einer kurzen Inspektion der Seite und der URLs bin ich darauf aufmerksam geworden, dass jede Ausgabe der Tagesthemen eine eigene 
URL besitzt, wie zum Beispiel diese Ausgabe
vom 8. Mai 2021: https://www.tagesschau.de/multimedia/sendung/tt-8257.html

Bei Inspektion mehrerer URLs von Tagesthemen-Seiten stellte ich fest, dass die Zahl "8257" in der URL ein Index ist, der hochzählt.
So habe ich über die Kalenderfunktion (ebenfalls auf der Webseite vorhanden) die URL der Sendung vom 01.01.2019 und vom 17.03.2021 ermittelt.
Das ist also der Start- und Endpunkt des Scrapings und dazwischen wird der Zähler einfach erhöht! (Mehr dazu später.)

### Was findet man nun auf dieser Seite?

Die wesentlichen Bereiche sind in dem Screenshot farblich markiert:

![Screenshot, Quelle: Tagesthemen.de](/posts/2021-05-09-tagesthemen/screenshot_tt.png)

Zunächst können wir herausfinden, wann die Sendung ausgestrahlt wurde (organgefarbener Kasten). Das Standbild der Sendung (grüner Kasten) liefert natürlich die Information, welche Person die Sendung moderiert hat und welche Farbe die Kleidung hatte, die diese Person trug. Außerdem ist rechts unten im Standbild die Dauer der Sendung zu erkennen (gelber Kasten; es handelt sich um ein overlay, sodass der Text ermittelbar ist).Schlussendlich sind unter dem Bild die Themen der Sendung einsehbar (violetter Kasten).

Das ist doch schomal eine ganze Menge! (Und es kommt schon der erste Gedanke: Oh oh, wie kann man denn die Kleidung der Moderator*innen von den Personen im Hintergrund (auf der Medienwand) unterscheiden? Dieser Gedanke wird erstmal aufgeschoben und zu gegebener Zeit behandelt - es wird aber mit Sicherheit ein Thema, das in Richtung Bild-Segmentierung geht.)

Nun also erstmal zum Scraping!

## Webscraping

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

Wer sich dafür interessiert, dem verlinke ich die Zuordnung zwischen Bild und Moderator*innen hier.


## Explorative Datenanalyse

Während meiner Analyse ist mir aufgefallen, dass es an manchen Tagen mehrere Tagesthemen gibt.
Es handelt sich um Extra-Ausgaben; diese habe ich identifiziert in dem ich von der Anzahl der Tagesthemen an einem Tag die Zeilennummer subtrahiert habe.
Insofern bleibt im Falle einer Sondersendung die Zahl "1" und damit kann dann die Filterung vorgenommen werden. 


```{r}

moderatoren <- read_csv("./Tagesthemen_Standbilder/moderatoren_sendung.csv")

# Verfeinerung des Datensatzes --------------------------------------------

tagesthemen <- tagesthemen %>% 
  mutate(dauer = as.numeric(dauer),
         date = as.Date(strftime(datum_zeit, format = "%Y-%m-%d"))) %>% 
  group_by(date) %>% 
  arrange(date, .by_group = TRUE) %>% 
  mutate(extra = n() - row_number(),
         day = factor(weekdays(datum_zeit), levels = c("Montag", "Dienstag", "Mittwoch",
                                                       "Donnerstag","Freitag","Samstag","Sonntag")),
         dateiname_standbild = str_remove(standbild_url, "https://www.tagesschau.de/multimedia/bilder/")) %>% 
  ungroup() %>% 
  left_join(moderatoren, by = c("dateiname_standbild" = "file"))


# Beleuchtung der Sendezeit -----------------------------------------------

tagesthemen %>% 
  ungroup() %>% 
  filter(extra == 0) %>%
  ggplot(aes(x=datum_zeit, y=round(dauer/60),1)) +
  theme_plex() +
  geom_line(color = "midnightblue") +
  geom_smooth() +
  labs(x = "Datum", y = "Sendezeit in Minuten", caption = "Quelle: www.tagesschau.de", 
       subtitle = "01.01.2019 - 16.03.2021 (ohne Extrausgaben) ", title = "Sendezeit in Minuten nach Datum")
```

![Zeitreihe Sendezeit, Quelle: Tagesthemen.de](/posts/2021-05-09-tagesthemen/sendezeit_over_days.png)

Juhu, das erste Ergebnis - und schon so interessant!

Zunächst ist starkes "Zucken" in den Daten erkennbar; in jeder Woche gibt es starke Schwankungen.
Daher ist es natürlich sinnvoll, die Werte auch noch einmal nach Wochentagen anzuschauen. Das passiert gleich.

Weiterhin ist erkennbar, dass ab ca. August 2020 eine kleine Treppe in der durchschnittlichen Sendezeit erkennbar ist.
Das ist kein Fehler sondern hat nach meiner Recherche einen einfachen Grund! Mehr dazu in diesem Video:

Und natürlich fällt ein starker Extremwert ins Auge undzwar gleich zu Beginn des Jahres 2021.
Hierbei handelt es sich um die Sendung vom 06. Januar 2021 als in den USA das Kongressgebäude gestürmt wurde.

![Sendezeit nach Wochentag, Quelle: Tagesthemen.de](/posts/2021-05-09-tagesthemen/sendezeit_nach_wochentag.png)

Das die Tagesthemen am Wochenende kürzer sind, das wusste ich. Neu war mir, dass dies offensichtlich schon am Freitag der Fall ist. 
Erkennbar ist, das an den Tagen von Montag bis Donnerstag eine Sendung im Durchschnitt 28-30 Minuten dauer und von Freitag bis Sonntag knapp über 20 Minuten.
Das ist doch gut zu wissen, wenn man mal wieder dringend aufs Klo muss, aber den nächsen Beitrag nicht verpassen möchte ;-)

Mit diesen ersten Einblicken endet dieser Post zunächst.

In den nächsten Posts gehe ich auf die Suche nach weiteren Insights - stay tuned! :-)

