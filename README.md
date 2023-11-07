# wescraping_exemples
#Step 1:collecting links: Rstudio  (not sure that it's the right code)
#0. Avant de commencer
#0.1. Travailler quelque part
setwd("~/Desktop/thesis")

#0.2. Chargement des packages
library(RSelenium)
#install.packages("RSelenium")
library(rvest)
library(tidyverse)
library(stringr)
library(tibble)
library(rJava)
devtools::install_github('colearendt/xlsx')
library(readxl)
#devtools::install_github("strengejacke/strengejacke")
library(sjmisc)
library(xml2)


###1. Scraper les adresses URL d'une seule page web ####

###1.1. Aspirer tous les liens sur une page ####

# Charger la page sur laquelle sont les liens qu'on souhaite récolter
page_URL <- read_html ("https://ria.ru/search/?query=%D0%BA%D1%83%D0%B4%D1%80%D0%BE%D0%B2%D0%BE")
#on peut se promener sur la page en utilisant le clic droit "Inspecter"
#pour trouver où sont les liens url vers les pages expo dans le code html
#après la balise <a    toujours associée à un lien
#après le marqueur href=


# On récolte tous les liens hypertexts:
URL1 <- data.frame(url = as.character(page_URL %>% html_nodes("a") %>%
                                        html_attr("href")))
#on précise ici que ce qui nous intéresse n'est pas du contenu comme la semaine dernière (html_text)
#mais le contenu de l'attribut : d'où html_attr

View(URL1)
#cette base concerne trop de liens: on va devoir affiner


#1.2. Selectionner uniquement les adresses URL qui nous intéressent

#il faudrait juste garder les lignes 17 à 30
URL2 <- data.frame(URL1[c(6:45),])
View(URL2)

#on remarque qu'il manque le début de l'adresse http, que l'on va donc rajouter

#colnames(URL2) <- c("fin") #on renomme pour avoir un nom de colonne plus commode
#on crée un colonne avec le début à rajouter
#URL2$debut <- "https://ria.ru"
#et on colle
#URL2$url <- paste(URL2$debut, URL2$fin, sep = "")
#View(URL2) #cela semble bien, je conserve uniquement la colonne qui m'intéresse et je l'enregistre
#liste_url <- URL2[,-c(1,2)]


#on a une liste d'adresses url à scrapper, que l'on peur enregistrer et qui servira pour faire tourner la boucle
#write.csv (liste_url , "liste_url_orsay22.csv") 


#>> mettre en application sur votre cas


#### 2. Utiliser RSelenium pour simuler un humain qui navigue ####


#ouvrir un browser sur l'ordinateur

#Attention: il faut changer le numéro du port a chaque utilisation.
#Attention2: les wifi institutionnels peuvent poser problème
#update.packages("RSelenium")



rD <- rsDriver(browser="firefox", port=1237L, verbose=F)
remDr <- rD[["client"]] 
remDr$open(silent = TRUE)


remDr$navigate("https://ria.ru/search/?query=%D0%BA%D1%83%D0%B4%D1%80%D0%BE%D0%B2%D0%BE")
#cliquer sur des éléments:
#on utilise à nouveau le selector gadget pour identifier l'endroit où l'on veut cliquer.
#on peut aussi utiliser le xpath

#cliquer sur un bouton: la petit flèche > pour aller sur les autres pages
remDr$findElements("css", '.color-btn-second-hover')[[1]]$clickElement()

#une alternative: le xpath plutôt que le css. Même effet
#remDr$findElements("xpath", '//*[contains(concat( " ", @class, " " ), concat( " ", "pager__item--next", " " ))]//a')[[1]]$clickElement()


#### 3. Construire une boucle pour constituer la liste d'url ####

#on va aspirer l'ensemble des adresses url présentes sur toutes les pages
#ce qui implique de cliquer sur les boutons année précédente

####3.1. Mêler RSelenium et Rvest sur une page ####
#aller sur la page d'intérêt n°1
remDr$navigate("https://ria.ru/search/?query=%D0%BA%D1%83%D0%B4%D1%80%D0%BE%D0%B2%D0%BE")

#lire la page
page_URL <- read_html(remDr$getPageSource()[[1]])

#aspirer tous les liens
URL1 <- data.frame(url = as.character(page_URL %>% html_nodes("a") %>%
                                        html_attr("href")))

View(URL1)

### 3.2 Construire une boucle en général ####

#la structure clasique d'une boucle:
for (i in 1:n) {
  do_something_with(i)}

#on peut aussi ajouter des conditions:
if (this happens) {do_this}

#et faire d'autres choses qui ne correspondent pas à cette condition:
else {}

#ou casser la boucle
else {break}



####3.3. Créer une boucle adaptée

for (i in 1:5) {
  #cliquer sur un bouton "page suivante"
  remDr$findElements("css", '.color-btn-second-hover')[[1]]$clickElement()
  Sys.sleep(5) #pour laisser le temps à la page de charger
  
  #lire la page
  page_URL <- read_html(remDr$getPageSource()[[1]])
  
  #aspirer tous les liens
  temp_data_frame <- data.frame(url = as.character(page_URL %>% html_nodes("a") %>%
                                                     html_attr("href")))
  
  #et les ajouter dans la base  
  URL1 <- rbind(URL1, temp_data_frame)    } 



#et on attend en regardant sur Firefox si ça fonctionne!
View(URL1)




#Step 2: Web scraping of the article content on phyton (Jupiterbook):
pip install beautifulsoup4
pip install requests
from bs4 import BeautifulSoup
import requests
import re
from re import sub
from decimal import Decimal
import io
from datetime import datetime
import pandas as pd

# Step 2. Example 1: 
import requests
from bs4 import BeautifulSoup
import pandas as pd

# Загрузка URL-адресов из CSV-файла
data = pd.read_csv("ria.csv")
urls = list(data['URL'])

results = []

# Итерация по URL-адресам
for url in urls:
    response = requests.get(url)
    soup = BeautifulSoup(response.content, "html.parser")

    # Извлечение нужных данных с веб-страницы
    title = soup.title.text
    body = [paragraph.text for paragraph in soup.find_all(class_="article__text")]
    date = soup.find(class_="article__info-date").text

    # Добавление результатов в список
    results.append({
        "URL": url,
        "Title": title,
        "Body": body,
        "Date": date
    })

# Создание DataFrame из результатов
df = pd.DataFrame(results)

# Сохранение данных в CSV
df.to_csv("1ria_results.csv", index=False)

#Step 2. Exmple 3. 

import requests
from bs4 import BeautifulSoup
import pandas as pd

# Load URL addresses from CSV file
data = pd.read_csv("neva_links.csv")
urls = list(data['URL'])

results = []

# Iterate over URLs
for url in urls:
    response = requests.get(url)
    soup = BeautifulSoup(response.content, "html.parser")

    # Extract the title
    title = soup.title.text

    try:
        # Find the main content container
        main_container = soup.find(class_="article-text")

        # Extract the paragraphs from the main content
        paragraphs = main_container.find_all("p")

        # Extract the text from each paragraph
        body = [paragraph.text.strip() for paragraph in paragraphs]
    except AttributeError:
        # Handle the case when main_container is not found
        body = []

    # Extract the date
    date_element = soup.find(class_="article-info-item")
    date = date_element.text if date_element else None

    # Print the extracted data
    print("Title:", title)
    print("Body:")
    for paragraph in body:
        print("-", paragraph)
    print("Date:", date)

    # Add the results to the list
    results.append({
        "URL": url,
        "Title": title,
        "Body": body,
        "Date": date
    })

# Create a DataFrame from the results
df = pd.DataFrame(results)

# Save the data to CSV
df.to_csv("neva_results.csv", index=False)


### STEP : Statistics on the corpus (Rstudio)

##  Création d'un corpus de textes 


```{r packages1, include=FALSE, message=FALSE, warning=FALSE }
library(tidyverse)
library(rvest)
library("readxl")
library(stringr)
library(openair)
library(data.table)
library(tidyverse)
library(dplyr)
library(lubridate)
library(xlsx)
library(kableExtra)
```

Notre corpus de textes est composé d'articles sur Kudrovo, publiés sur une période allant de 2010 (moment du début de la construction de ce territoire) à 2020, par les médias locaux et nationaux les plus populaires. La liste des médias nationaux comprend des sources d'information parmi les plus lues en Russie, telles que RIA Novosti(120-130 millions de visites du site par mois), Komsomolskaya Pravda(80-90 millions), Lenta.ru (100-110 millions), Izvestiya.RU (30-40 millions), TASS (30-45 millions), Interfax (15-20 millions), Gazeta.ru (55-60 millions), RBC (RosBusinessConsulting)(160-170 millions), Moskovskiy Komsomolets (MK)(85-95 millions), Rossiyskaya Gazeta (85-95 millions), Kommersant (55-60 millions) et Argumenty i Fakty (30-35 millions). 

Dans la liste des médias locaux, nous avons inclus les sources d'information les plus lues de Saint-Pétersbourg et de l'oblast de Léningrad, parmi elles : Neva aujourd'hui (environ 300 000 visites par mois), Online 47 (300 000 à 350 000 visites), Komsomolskaya Pravda SPb (environ 2,5 millions de visites), Delovoy Petersburg (environ 2,5 millions de visites), 47news (environ 2 millions de visites), Nevskie novosti, 78.ru (environ 1,5 million de visites), Télékanal Saint-Pétersbourg, Petérbourgski dnevnik (2 à 3 millions de visites), Fontanka (de 25 à 35 millions de visites), et Boumaga (de 200 à 300 000 visites).

Au total, nous avons 2185 articles.

```{r masterframe, message=FALSE, warning=FALSE, include=FALSE, paged.print=FALSE}
rm(list = ls()) #clear environment
setwd("~/Desktop/thesis") 

## data frames with dates that need to be converted into the correct format using the monthsconversion table
#monthsconversion <- read_excel("months.xlsx")

library(readxl)

corpus<- read_excel("corpus_count.xlsx")
#View(corpus)

corpus$date <- as.Date(corpus$date)

NAS <- corpus[rowSums(is.na(corpus)) > 0,]  #check missing values
corpus <- corpus %>% drop_na()
#view(corpus)

corpus <- corpus %>% filter(str_count(body, pattern = "Кудрово") >= 1)


save(corpus, file = "masterframe.RData")
#install.packages("openxlsx")
library(openxlsx)
#write.xlsx(corpus, "corpus_done.xlsx")

masterframe <- corpus
```



```{r countbysource, include=FALSE, message=FALSE, warning=FALSE }

countbysource <- data.frame(table(masterframe$source))
countbysource <- countbysource %>% 
  rename("Source" = Var1) %>%
  rename("Count" = Freq) %>%
  mutate(Count = as.numeric(Count))
```

### Répartition des articles par source au fil du temps.

Le premier graphique représente le nombre total d'articles sur Kudrovo publiés par les journaux russes de 2010 à 2020. 


```{r ggplot1, echo=FALSE, message=FALSE, warning=FALSE }
#dev.new(width=5, height=4)
ggplot(data = countbysource, mapping = aes(x = reorder(Source, Count), Count)) + 
  geom_bar(stat = "identity", fill = "cadetblue") + 
  coord_flip() +
  scale_y_continuous(breaks = round(seq(0, 700, by = 25),1)) +
  labs(y="Nombre d'articles publiés", 
       x= "Source d'articles",
       #title="Graphique 18: Nombre total d'articles sur Kudrovo publiés par les journaux russes (2010-2020)", 
       caption = "Période couverte : janvier 2010 - décembre 2019 ") +
  theme(axis.text=element_text(size=9),
        axis.title=element_text(size=9),
        plot.title = element_text(size =9.5, hjust =1, vjust = 1),
        axis.text.x = element_text(angle = 45, hjust = 1),
        plot.caption = element_text(size = 9)) 
```
Les résultats montrent que le plus grand nombre d'articles a été publié par le journal local "Fontanka" (623), suivi par le journal national "Kommersant" (346) et le journal local "online47" (259).

Les mêmes résultats sont présentés en details dans un tableau ci-dessus.

```{r table 1, echo=FALSE, message=FALSE, warning=FALSE}
library(kableExtra)

kable(countbysource%>% arrange(desc(Count)),
      caption = "**Nombre total d'articles sur Kudrovo publiés par les journaux russes (2010-2020)**",
      align="lcc", escape = FALSE) %>%
  kable_styling(bootstrap_options = c("striped", "hover"), full_width = FALSE)%>%
  column_spec(2, background = "cadetblue")

```

Ces résultats représentent le nombre total d'articles sur Kudrovo publiés dans les journaux russes. Analysons-les et tirons les conclusions suivantes :

Le nombre d'articles sur Kudrovo publiés par différents sources varie considérablement. "Fontanka" est la principale source avec le plus grand nombre d'articles publiés - 623. Ensuite viennent "Kommersant" avec 346 articles et "online47" avec 259 articles. Ces sources suivent probablement activement les événements à Kudrovo et publient régulièrement des nouvelles sur ce quartier.

"RBK" et "Telekanal Spb" ont le même nombre d'articles sur Kudrovo - 189. Cela peut indiquer que ces sources s'intéressent également de manière équivalente à ce quartier et couvrent activement les nouvelles qui y sont liées.

Certaines sources, telles que "MK", "TASS", "NevaToday", "RG" et "47news", ont publié un nombre relativement faible d'articles sur Kudrovo, ce qui peut indiquer une spécialisation plus étroite ou un intérêt moindre pour ce quartier dans leurs flux d'actualités.

Il existe des sources telles que "Bumaga", "RIA", "Lenta" et "NevNov", qui ont publié un nombre relativement faible d'articles sur Kudrovo, probablement en raison d'un intérêt limité pour ce sujet.

"Interfax", "KP SPB" et "Argumenty i fakty" ont le plus faible nombre d'articles sur ce quartier - 7, 3 et 2 respectivement. Ces sources semblent ne pas accorder une attention particulière à Kudrovo dans leurs publications d'actualités.

Dans l'ensemble, ces résultats permettent de voir quelles sont les sources russes qui couvrent activement les événements et les nouvelles liées à Kudrovo, et lesquelles y accordent moins d'attention, voire pas du tout. Cela est souvent lié à la localisation géographique de chaque source, à leur spécialisation ou à d'autres facteurs influençant l'intérêt pour ce sujet.

### Nombre d'articles publiés par source par année


L'analyse suivante montre l'intérêt par année pour chaque source sous forme de tableau et de graphique.

```{r table2, echo=FALSE, message=FALSE, warning=FALSE}

library(dplyr)
library(kableExtra)

# Group articles by year and news source
countbysourcebyyear <- masterframe %>%
  group_by(source, year = substr(date, 1, 4)) %>%
  summarise(Freq = n())

kable(countbysourcebyyear %>% arrange(desc(Freq)),
      caption = "**Nombre d'articles sur Kudrovo publiés par chaque source d'actualités au fil du temps**",
      align = "lcc", escape = FALSE) %>%
  kable_styling(bootstrap_options = c("striped", "hover"), full_width = FALSE) %>%
  column_spec(3, background = "cadetblue")

```
Ces résultats représentent le nombre d'articles sur Kudrovo publiés par chaque source d'actualités au fil du temps.

Principales conclusions : Les sources d'actualités "Fontanka", "Kommersant" et "online47" ont le plus grand nombre d'articles sur Kudrovo. Cela peut indiquer que ces sources accordent plus d'attention à ce sujet et couvrent activement les événements liés à Kudrovo (1); Au cours des dernières années, en particulier de 2017 à 2019, le nombre d'articles sur Kudrovo dans de nombreuses sources a augmenté. Cela peut être lié à un intérêt accru pour ce sujet, peut-être en raison d'un développement actif ou d'événements se produisant dans ce quartier(2); Certaines sources, telles que "Gazeta", "Interfax", "KP SPB" et "Lenta", ont un nombre limité d'articles sur Kudrovo, ce qui peut indiquer qu'elles publient moins fréquemment des nouvelles sur ce sujet ou le considèrent comme moins prioritaire(3);La tendance générale montre une augmentation de l'intérêt pour Kudrovo au fil du temps, en particulier à partir de 2017. Cela peut refléter une augmentation de la popularité du quartier ou des événements importants s'y déroulant(4).

```{r ggplot2, echo=FALSE, message=FALSE, warning=FALSE}
# Create the ggplot graphic
ggplot(countbysourcebyyear, aes(x = year, y = Freq, fill = source)) +
  geom_bar(stat = "identity", position = "dodge") +
  scale_y_continuous(breaks = seq(0, max(countbysourcebyyear$Freq), by = 25)) +
  labs(x = "Année", y = "Nombre d'articles publiés",
       #title = "Graphique 19: Nombre d'articles sur Kudrovo publiés par chaque source d'actualités au fil du temps",
       fill = "Source d'artciles") +
  theme_minimal()+
  theme(axis.text=element_text(size=9),
        axis.title=element_text(size=9),
        plot.title = element_text(size =9, hjust =0.1, vjust = 1),
        plot.caption = element_text(size = 9)) 
```

##STEP 4: Text analysis 

```{r corpus, include=FALSE, message=FALSE, warning=FALSE}
#install.packages("R.temis")
library(R.temis)
library(tidyverse)

corpus2 <- import_corpus("кудрово.txt", format="alceste", language="rus")
```


```{r dic, include=FALSE  }
# Metadonnées du corpus
## ----TLE----------------------------------------------------------------------------
dtmsmo <-build_dtm(corpus2, remove_stopwords = T )
#dtmsmo
## ----Dico, eval=TRUE, include=FALSE-------------------------------------------------
dictionary(dtmsmo)
dic <-dictionary(dtmsmo)
```

La première étape de l'analyse du corpus consiste à supprimer automatiquement les stopwords tels que les prépositions. De plus, les mots les plus utilisés dans le corpus qui ne sont pas pertinents pour le sujet ont été retirés manuellement :


```{r include=FALSE, message=FALSE, warning=FALSE}
dtmsmo
```

```{r asupp, echo=TRUE, message=FALSE, warning=FALSE}
asupp <- c("xa", "это", "года","году","также","которые","будут","около", "говорит","год","пока","который","например","тыс", "тысяч","однако","несколько","очень","таких", "ранее","кроме","поэтому","могут","которых","которая","двух","ооо", "считает","числе", "нужно","должны","составляет", "менее", "случае","рядом","является","рамках","наиболее","стоит","этих","должен","другие", "других","сообщили","день", "находится","просто","среди","свои", "сообщает", "либо","годы","должна", "рассказал","сообщил","одной", "говорят", "идет","хотя","отмечает","такие","одного","нам","лишь","эта","меньше", "больше",
           "две","которого", "второй", "большой", "одна","достаточно", "которой","напомним","стали","конце","поскольку","отметил","таким", "раньше","выше", "станет","ru","свой","какие","пять","касается","своих","этим","декабря", "каждый", "те", "общей","сентября","первых","ру","четыре","заявил","составит","получить","рассказывает","го","xaв","особенно", "тех", "которое","несмотря","первую","помимо", "первом", "речь", "действительно","некоторые", "сделать", "итоге", "июля", "планирует", "вместе", "возможно", "новая","скорее", "стать", "вообще","имеет", "тому","февраля","марта","правда","семь","новом","прошлом", "будем", "среднем", "причем", "давно","первые", "считают", "своего","часто","всем","ниже","средняя","xaкв","первого","октября","этому","крти","которую", "никто", "трех", "данный", "одним","число","позволит","придется","наш","такого","такое", "января", "находятся","значит", "такая","котором","сами", "итогам", "месяц", "остается","минут","отмечают","общего", "уверен", "появятся","самых", "одно","месяцев","пор","августа","важно","первой", "ук","ведется","стала","ходе", "гораздо", "мая","смогут","затем","самый","сколько","читайте","далее","самые","дня", "пройдет","вполне","делать", "нем","являются","всей", "разных","апреля","добавляет","собой","многих","наших", "about","александр","фото", "данным", "xaтыс","xaм", "первый","части", "назад","стало","сергей","примерно","михаил","господин","многие","необходимо","xaмлн", "образом", "сразу","стал", "порядка", "дело","внимание", "точки", "дмитрий", "начале", "целом", "известно", "полностью", "начала", "появится", "говорить", "ольга", "xaмлрд", "течение","своей","должно", "андрей","александра", "алексей","екатерина", "сравнению", "владимир", "соответственно","игорь","основном","полагает","вдоль","недавно","евгений","получил","свое", "составила","хотят","июня","которым", "пишет","почему","сообщают","зависит", "кстати","николай", "ноября", "примеру","прежде","xaи","вместо","следует", "георгий","идут","рассказала","приняли","её","писал","точно","некоторых","решили","самое","всё","xaже","немного","весь","втором","одну","мере","оао","оказался","самом","xaга","сих","наши","елена","чаще","еще","гу","другое","согласно","bg") 



dtmsmo2 <-dtmsmo[, !colnames(dtmsmo)  %in% asupp]
dtmsmo2

```

Cela nous a laissé 71 446 termes.

Les données représentent une matrice "Document-Term" (documents-mots), composée de 2352 documents et 71446 termes (mots ou termes). La matrice contient 471060 entrées non nulles et 167569932 entrées nulles, ce qui signifie que la plupart des cellules de la matrice sont vides (données éparses). La densité de la matrice est de 100%, ce qui indique une très forte éparsité des données.

La matrice a été pondérée en utilisant la méthode "term frequency (tf)" (fréquence des termes). Le poids de chaque terme dans le document est déterminé par sa fréquence d'apparition dans ce document.

Les données de cette matrice peuvent être utilisées pour l'analyse de données textuelles, par exemple pour identifier les termes fréquemment utilisés dans les documents, regrouper les documents en fonction de leur contenu similaire ou créer des modèles d'apprentissage automatique pour la classification des données textuelles.


## Nuage de mots

Ici, nous pouvons voir un nuage de mots composé de 600 mots avec une fréquence d'occurrence d'au moins 100 en russe et les 100 termes les plus fréquents ci-dessus. 

```{r cloud, echo=FALSE, message=FALSE, warning=FALSE}
cloud<-word_cloud(dtmsmo2, color="cadetblue", n=600, min.freq= 100)
```

## Termes les plus fréquents.


```{r frequency, echo=FALSE, message=FALSE, warning=FALSE}
library(kableExtra)
tab2 <-frequent_terms(dtmsmo2, n=100)

kable(tab2,
      caption = "**100 termes les plus fréquents**",
      align="lcc", escape = FALSE) %>%
  kable_styling(bootstrap_options = c("striped", "hover"), full_width = FALSE)%>%
  column_spec(2, background = "cadetblue")
```

En étudiant les 100 termes les plus fréquemment rencontrés dans le corpus d'articles sur Kudrovo, on peut faire les conclusions suivantes :

Le terme "Kudrovo" est le plus fréquemment mentionné dans le corpus, ce qui indique que les données concernent principalement cette ville spécifique.

Les thèmes importants des articles sont la construction et l'immobilier, car la liste comprend des termes tels que "construction", "projets de construction", "logements", "compagnies", "projets", "maisons", "complexe résidentiel", "promoteurs immobiliers" et autres.

On trouve également des termes liés à l'infrastructure et aux itinéraires de transport dans la ville, tels que "métro", "lignes de bus", "infrastructure de transport".

Le marché immobilier est représenté par des termes tels que "marché immobilier", "coût du logement", "prix des appartements" et autres, ce qui indique une activité dans ce domaine à Kudrovo.

La mention du gouverneur et des structures gouvernementales ("gouverneur", "gouvernement") peut indiquer l'importance de cette ville et l'intérêt des autorités pour son développement.

Certains termes peuvent se référer à des événements ou des caractéristiques spécifiques de la ville, tels que "Murino", "Dybenko", "mètres", "numéros cadastraux".

Certains termes peuvent indiquer des aspects socio-économiques de la ville, tels que "résidents", "emplois", "complexe résidentiel", "infrastructure" et "développement".

La présence de valeurs numériques telles que "millions", "mètres carrés", "kilomètres" liées à des mesures ou des superficies peut indiquer des caractéristiques spécifiques d'objets ou de projets dans la ville.

Ainsi, l'analyse de ces termes les plus fréquemment utilisés permet de conclure que le corpus d'articles sur Kudrovo se concentre principalement sur la construction, l'immobilier, l'infrastructure et le développement socio-économique de cette ville spécifique. Ces résultats peuvent être utiles pour une compréhension plus approfondie des caractéristiques et de la thématique des articles concernant cette ville.

Dans la liste des termes, nous pouvons constater quelques inconvénients sérieux. Par exemple, le nom de la ville de Saint-Pétersbourg est divisé en deux termes (Saint et Petersburg) et compté séparément. De plus, le nom de la région de Léningrad (Leningrad oblast) est présenté par plusieurs termes en même temps (oblast, Leningrad, Leniblast). Un autre problème que nous pouvons voir dans la liste des termes les plus populaires concerne les différentes formes grammaticales du même mot (par exemple, "projet").

Dans ce cas, la lemmatisation du corpus nous aidera.

## Lemmatisation of the corpus

La lemmatisation est le processus de regrouper les formes fléchies d'un mot afin qu'elles puissent être analysées comme un seul élément, identifié par la forme lemmatisée du mot, ou sa forme de dictionnaire. Dans notre cas, nous avons effectué une lemmatisation des 600 termes les plus fréquents sur 71 446.

En appliquant la lemmatisation, les termes "Saint" et "Petersburg" peuvent être regroupés en "Saint-Pétersbourg", et les termes "oblast", "Leningrad" et "Leniblast" peuvent être fusionnés en "Léningrad oblast". Cela aidera à représenter plus précisément les occurrences de ces termes dans l'analyse.

De plus, la lemmatisation convertira des mots tels que "projects" et "project" en leur forme de base "projet", consolidant ainsi le décompte de fréquence pour les variations du même mot.

La présence de cas en russe rend la tâche particulièrement complexe, car la plupart des mots ont différentes formes selon le cas, le genre et le nombre, tout en ayant la même signification.

De plus, nous corrigeons les noms propres dans le corpus. Par exemple, "Aleksandr" et "Beglov" sont remplacés par "Beglov" (le nom du responsable de la ville de Saint-Pétersbourg).

Nous divisons le corpus lemmatisé en nouvelles unités (paragraphes au lieu d'articles) : de cette manière, il sera plus facile d'analyser les textes par année et les ressources dans lesquelles ils sont publiés.

```{r corpus1, message=FALSE, warning=FALSE, echo=FALSE}
corpus_d <-split_documents(corpus2,1)
dtmsmo_d <-build_dtm(corpus_d, remove_stopwords=T)
dtmsmo_d2 <-dtmsmo_d[, !colnames(dtmsmo_d)  %in% asupp]
dtmsmo_d2
```

```{r tab1, message=FALSE, warning=FALSE, echo=FALSE}
library(questionr)
tab1 <-freq(meta(corpus_d)$Газета)
#install.packages("kableExtra")
tab1 <-tab1 %>% arrange(desc(n))
tab1
kable(tab1 %>% arrange(desc(n)),
      caption = "**Contribution de chaque source au corpus d'articles.**",
      align="lcc", escape = FALSE) %>%
  kable_styling(bootstrap_options = c("striped", "hover"), full_width = FALSE)%>%
  column_spec(3, background = "cadetblue")
```

Cette tableau présente la contribution de chaque source dans le corpus d'articles en pourcentage. La colonne "n" montre le nombre d'articles de chaque source, et "val%" représente le pourcentage de ce nombre par rapport au nombre total d'articles dans le corpus.

Interprétation et conclusions : Les sources "Фонтанка" (Fontanka) (22,8 %), "Коммерсант" (Kommersant) (12,2 %) et "47онлайн" (47online) (10,5 %) ont apporté la plus grande contribution au corpus d'articles. Cela indique que ces sources sont les plus importantes et fournissent la plus grande partie de l'information dans l'ensemble du corpus.
En général, les premières sources représentent une part significative des données, tandis que les autres sources contribuent moins, leur part étant inférieure à 7 % chacune.
"НеваСегодня" (Neva Segodnya), "Интерфакс" (Interfax), "Новости47" (Novosti47) et "МосковскийКомсомолец" (Moskovskiy Komsomolets) apportent également une contribution significative au corpus global, représentant entre 4,7 % et 3,5 % chacun.
Il existe plusieurs sources, telles que "Газера.ру" (Gazera.ru), "РИАновости" (RIA Novosti), "Argumentyifakty" (Argumenty i Fakty), "КПСПБ" (KPS PB) et "Невскийновости" (Nevskiy Novosti), qui apportent une petite contribution en pourcentage (moins de 2 %) au corpus d'articles.
Dans l'ensemble, ces informations permettent de comprendre quelles sources fournissent le plus grand volume de données dans le corpus d'articles et quelle est leur part de contribution à l'information globale.

```{r tab111, message=FALSE, warning=FALSE, echo=FALSE}
library(questionr)
tab1 <-freq(meta(corpus_d)$Статус)
#install.packages("kableExtra")
library(kableExtra)
kable(tab1 %>% arrange(desc(n)),
      caption = "**Contribution de chaque type de source au corpus d'articles.**",
      align="lcc", escape = FALSE) %>%
  kable_styling(bootstrap_options = c("striped", "hover"), full_width = FALSE)%>%
  column_spec(3, background = "cadetblue")
```
Ces résultats indiquent la contribution de chaque type de source au corpus d'articles.

Le type de source "местная" (locale) représente 58.8% du corpus total d'articles, ce qui en fait la source dominante en termes de nombre d'articles publiés. Cela signifie que la majorité des articles proviennent de sources locales qui couvrent probablement des sujets et événements spécifiques à la région ou à la ville de Kudrovo.

Le type de source "всероссийская" (panrusse) représente 41.2% du corpus d'articles, ce qui en fait une part significative mais légèrement inférieure à celle des sources locales. Les sources "всероссийская" sont des médias nationaux et des agences de presse qui couvrent des sujets d'intérêt national et des nouvelles ayant un impact à l'échelle nationale.

On peur resumer que les résultats montrent que le corpus d'articles sur Kudrovo est principalement composé d'articles provenant de sources locales, ce qui met en évidence l'importance de la couverture médiatique au niveau local pour ce sujet. Cependant, les sources panrusses jouent également un rôle significatif dans la couverture des événements et des sujets liés à Kudrovo à l'échelle nationale.


```{r tab112, message=FALSE, warning=FALSE, echo=FALSE}
library(questionr)
tab1 <-freq(meta(corpus_d)$Год)
tab1
#install.packages("kableExtra")
library(kableExtra)
kable(tab1 %>% arrange(desc(n)),
      caption = "**Contribution des articles de différentes années au corpus d'articles.**",
      align="lcc", escape = FALSE) %>%
  kable_styling(bootstrap_options = c("striped", "hover"), full_width = FALSE)%>%
  column_spec(3, background = "cadetblue")
```

Ces résultats indiquent la répartition du texte par paragraphes pour chaque année dans le corpus d'articles sur la ville de Kudrovo. Voici l'interprétation et les conclusions:

En 2019, 27,9% du texte dans le corpus est attribué aux articles concernant Kudrovo pour cette année. Cela suggère que l'année 2019 a généré une quantité substantielle de contenu dans les articles sur cette ville.

En 2018, la contribution est de 22,3%, ce qui est également significatif, mais légèrement inférieur à 2019.

Les années antérieures montrent une diminution progressive de la contribution du texte. En 2017, elle représente 13,0%, en 2016, 10,8%, et ainsi de suite.

Les années 2014, 2015 et 2013 montrent une contribution stable autour de 7,5%, 6,4% et 5,4% respectivement.

Les années 2011, 2012 et 2010 ont la plus faible contribution en termes de texte, avec 3,2%, 1,8% et 1,7% respectivement.

Le development actif du village de Kudrovo a commencé en 2010, et son statut de ville a été attribué en 2018. Cela peut être l'une des raisons pour lesquelles il y a eu plus d'articles et de publications sur cette ville en 2019 et 2018.


```{r dic2, include=FALSE}
dic <-dictionary(dtmsmo)
dic2 <- dic[!rownames(dic) %in% asupp,]
#write.csv2(dic2, "кудровонов.csv")
dic3 <- read.csv("lemm.csv",row.names = 1)
dic4 <- read.csv("lemm.csv",row.names = 1)
str(dic4)
# Заменить "санктпетербург" на "спб" в столбце "Term"
dic4$Term <- gsub("санктпетербург", "spb", dic4$Term)
dic4$Term <- gsub("спб", "spb", dic4$Term)
dic4$Term <- gsub("ленобласть", "lenoblast", dic4$Term)
dic4$Term <- gsub("проект", "projet", dic4$Term)
dic4$Term <- gsub("строительство", "construction", dic4$Term)
dic4$Term <- gsub("районы", "quartiers", dic4$Term)
dic4$Term <- gsub("кудрово", "koudrovo", dic4$Term)
dic4$Term <- gsub("город", "ville", dic4$Term)
dic4$Term <- gsub("новые", "nouveau", dic4$Term)
dic4$Term <- gsub("компания", "entreprise", dic4$Term)
dic4$Term <- gsub("транспорт", "transport", dic4$Term)
dic4$Term <- gsub("рубли", "roubles", dic4$Term)
dic4$Term <- gsub("станция", "station", dic4$Term)
dic4$Term <- gsub("территория", "territoire", dic4$Term)
dic4$Term <- gsub("квартиры", "appartements", dic4$Term)
dic4$Term <- gsub("объекты", "objets", dic4$Term)
dic4$Term <- gsub("рынок", "marché", dic4$Term)
dic4$Term <- gsub("жилье", "logement", dic4$Term)
dic4$Term <- gsub("дом", "maison", dic4$Term)
dic4$Term <- gsub("работы", "travaux", dic4$Term)
dic4$Term <- gsub("застройщик", "promoteur", dic4$Term)
dic4$Term <- gsub("места", "places", dic4$Term)
dic4$Term <- gsub("развитие", "développement", dic4$Term)
dic4$Term <- gsub("квм", "m²", dic4$Term)
dic4$Term <- gsub("улицы", "rues", dic4$Term)
dic4$Term <- gsub("центр", "centre", dic4$Term)
dic4$Term <- gsub("метро", "métro", dic4$Term)
dic4$Term <- gsub("жилые", "résidentiel", dic4$Term)
dic4$Term <- gsub("участок", "parcelle", dic4$Term)
dic4$Term <- gsub("жители", "habitants", dic4$Term)
dic4$Term <- gsub("время", "temps", dic4$Term)
dic4$Term <- gsub("инфраструктура", "infrastructure", dic4$Term)
dic4$Term <- gsub("комплекс", "complexe", dic4$Term)
dic4$Term <- gsub("дороги", "routes", dic4$Term)
dic4$Term <- gsub("цены", "prix", dic4$Term)
dic4$Term <- gsub("недвижимость", "immobilier", dic4$Term)
dic4$Term <- gsub("россия", "Russie", dic4$Term)
dic4$Term <- gsub("решение", "décision", dic4$Term)
dic4$Term <- gsub("строить", "construire", dic4$Term)
dic4$Term <- gsub("покупатели", "acheteurs", dic4$Term)
dic4$Term <- gsub("предложение", "proposition", dic4$Term)
dic4$Term <- gsub("директор", "directeur", dic4$Term)
dic4$Term <- gsub("проспект", "avenue", dic4$Term)
dic4$Term <- gsub("продажи", "ventes", dic4$Term)
dic4$Term <- gsub("губернатор", "gouverneur", dic4$Term)
dic4$Term <- gsub("регион", "région", dic4$Term)
dic4$Term <- gsub("стоимость", "coût", dic4$Term)
dic4$Term <- gsub("всеволожск", "Vsevolozhsk", dic4$Term)
dic4$Term <- gsub("слова", "mots", dic4$Term)
dic4$Term <- gsub("создание", "création", dic4$Term)
dic4$Term <- gsub("городской", "urbain", dic4$Term)
dic4$Term <- gsub("площадь", "carré", dic4$Term)
dic4$Term <- gsub("квартал", "quartier", dic4$Term)
dic4$Term <- gsub("вопрос", "question", dic4$Term)
dic4$Term <- gsub("годы", "années", dic4$Term)
dic4$Term <- gsub("объем", "volume", dic4$Term)
dic4$Term <- gsub("мурино", "Murino", dic4$Term)
dic4$Term <- gsub("группавк", "VKcommunauté", dic4$Term)
dic4$Term <- gsub("власти", "autorités", dic4$Term)
dic4$Term <- gsub("строительный", "constructionnel", dic4$Term)
dic4$Term <- gsub("проблемы", "problèmes", dic4$Term)
dic4$Term <- gsub("социальный", "social", dic4$Term)
dic4$Term <- gsub("зоны", "zones", dic4$Term)
dic4$Term <- gsub("спрос", "demande", dic4$Term)
dic4$Term <- gsub("линии", "lignes", dic4$Term)
dic4$Term <- gsub("метры", "mètres", dic4$Term)
dic4$Term <- gsub("построено", "construit", dic4$Term)
dic4$Term <- gsub("эксперты", "experts", dic4$Term)
dic4$Term <- gsub("открытие", "ouverture", dic4$Term)
dic4$Term <- gsub("детские", "enfants", dic4$Term)
dic4$Term <- gsub("миллион", "million", dic4$Term)
dic4$Term <- gsub("очереди", "files d'attente", dic4$Term)
dic4$Term <- gsub("правительство", "gouvernement", dic4$Term)
dic4$Term <- gsub("девелопер", "promoteur", dic4$Term)
dic4$Term <- gsub("комфорт", "confort", dic4$Term)
dic4$Term <- gsub("люди", "gens", dic4$Term)
dic4$Term <- gsub("готовность", "préparation", dic4$Term)
dic4$Term <- gsub("автомобили", "voitures", dic4$Term)
dic4$Term <- gsub("качество", "qualité", dic4$Term)
dic4$Term <- gsub("начать", "démarrer", dic4$Term)
dic4$Term <- gsub("активный", "actif", dic4$Term)
dic4$Term <- gsub("жк", "complexe résidentiel", dic4$Term)
dic4$Term <- gsub("комитет", "comité", dic4$Term)
dic4$Term <- gsub("застройка", "construction", dic4$Term)
dic4$Term <- gsub("северный", "nord", dic4$Term)
dic4$Term <- gsub("возможности", "opportunités", dic4$Term)
dic4$Term <- gsub("сети", "réseaux", dic4$Term)
dic4$Term <- gsub("детскийсад", "jardin d'enfants", dic4$Term)
dic4$Term <- gsub("план", "plan", dic4$Term)
dic4$Term <- gsub("администрация", "administration", dic4$Term)
dic4$Term <- gsub("класс", "classe", dic4$Term)
dic4$Term <- gsub("сдача", "livraison", dic4$Term)
dic4$Term <- gsub("планирование", "planification", dic4$Term)
dic4$Term <- gsub("школы", "écoles", dic4$Term)
dic4$Term <- gsub("программа", "programme", dic4$Term)
dic4$Term <- gsub("москва", "Moscou", dic4$Term)
dic4$Term <- gsub("финансирование", "financement", dic4$Term)
dic4$Term <- gsub("бюджет", "budget", dic4$Term)
dic4$Term <- gsub("километры", "kilomètres", dic4$Term)
dic4$Term <- gsub("появиться", "apparaître", dic4$Term)
dic4$Term <- gsub("сроки", "délais", dic4$Term)
dic4$Term <- gsub("ситуация", "situation", dic4$Term)
dic4$Term <- gsub("оценка", "évaluation", dic4$Term)
dic4$Term <- gsub("аренда", "location", dic4$Term)
dic4$Term <- gsub("машины", "voitures", dic4$Term)
dic4$Term <- gsub("планировка", "planification", dic4$Term)
dic4$Term <- gsub("бизнес", "entreprise", dic4$Term)
dic4$Term <- gsub("маршрут", "itinéraire", dic4$Term)
dic4$Term <- gsub("реализовать", "réaliser", dic4$Term)
dic4$Term <- gsub("происходить", "se produire", dic4$Term)
dic4$Term <- gsub("возведение", "construction", dic4$Term)
dic4$Term <- gsub("средства", "fonds", dic4$Term)
dic4$Term <- gsub("последние", "derniers", dic4$Term)
dic4$Term <- gsub("рост", "croissance", dic4$Term)
dic4$Term <- gsub("служба", "service", dic4$Term)
dic4$Term <- gsub("ипотека", "hypothèque", dic4$Term)
dic4$Term <- gsub("увеличение", "augmentation", dic4$Term)
dic4$Term <- gsub("миллиарды", "milliards", dic4$Term)
dic4$Term <- gsub("поселок", "village", dic4$Term)
dic4$Term <- gsub("расположение", "emplacement", dic4$Term)
dic4$Term <- gsub("инвестиция", "investissement", dic4$Term)
dic4$Term <- gsub("дорожный", "routier", dic4$Term)
dic4$Term <- gsub("деньги", "argent", dic4$Term)
dic4$Term <- gsub("кад", "personnel", dic4$Term)
dic4$Term <- gsub("парк", "parc", dic4$Term)
dic4$Term <- gsub("изменение", "modification", dic4$Term)
dic4$Term <- gsub("получить", "obtenir", dic4$Term)
dic4$Term <- gsub("направление", "direction", dic4$Term)
dic4$Term <- gsub("население", "population", dic4$Term)
dic4$Term <- gsub("количество", "quantité", dic4$Term)
dic4$Term <- gsub("основной", "principal", dic4$Term)
dic4$Term <- gsub("обеспечение", "provision", dic4$Term)
dic4$Term <- gsub("общественный", "public", dic4$Term)
dic4$Term <- gsub("реализация", "réalisation", dic4$Term)
dic4$Term <- gsub("торговые", "commerciales", dic4$Term)
dic4$Term <- gsub("уровень", "niveau", dic4$Term)
dic4$Term <- gsub("стороны", "côtés", dic4$Term)
dic4$Term <- gsub("инвесторы", "investisseurs", dic4$Term)
dic4$Term <- gsub("интерес", "intérêt", dic4$Term)
dic4$Term <- gsub("вид", "vue", dic4$Term)
dic4$Term <- gsub("начало", "début", dic4$Term)
dic4$Term <- gsub("руководитель", "chef", dic4$Term)
dic4$Term <- gsub("шоссе", "autoroute", dic4$Term)
dic4$Term <- gsub("федеральный", "fédéral", dic4$Term)
dic4$Term <- gsub("предполагать", "supposer", dic4$Term)
dic4$Term <- gsub("часть", "partie", dic4$Term)
dic4$Term <- gsub("состав", "composition", dic4$Term)
dic4$Term <- gsub("ближайшие", "proches", dic4$Term)
dic4$Term <- gsub("запад", "ouest", dic4$Term)
dic4$Term <- gsub("фонтанка", "Fontanka", dic4$Term)
dic4$Term <- gsub("дрозденко", "Drozdenco", dic4$Term)
dic4$Term <- gsub("прессслужба", "service de presse", dic4$Term)
dic4$Term <- gsub("счет", "compte", dic4$Term)
dic4$Term <- gsub("генеральныйдиректор", "directeur général", dic4$Term)
dic4$Term <- gsub("метрополитен", "métro", dic4$Term)
dic4$Term <- gsub("дыбенко", "Dybenko", dic4$Term)
dic4$Term <- gsub("банк", "banque", dic4$Term)
dic4$Term <- gsub("местные", "locaux", dic4$Term)
dic4$Term <- gsub("доступность", "accessibilité", dic4$Term)
dic4$Term <- gsub("площадка", "site", dic4$Term)
dic4$Term <- gsub("отметить", "remarquer", dic4$Term)
dic4$Term <- gsub("земля", "terre", dic4$Term)
dic4$Term <- gsub("новостройки", "nouveaux bâtiments", dic4$Term)
dic4$Term <- gsub("скоростной", "rapide", dic4$Term)
dic4$Term <- gsub("представители", "représentants", dic4$Term)
dic4$Term <- gsub("высокий", "élevé", dic4$Term)
dic4$Term <- gsub("наш", "notre", dic4$Term)
dic4$Term <- gsub("будущее", "avenir", dic4$Term)
dic4$Term <- gsub("главный", "chef", dic4$Term)
dic4$Term <- gsub("деревня", "village", dic4$Term)
dic4$Term <- gsub("крупный", "important", dic4$Term)
dic4$Term <- gsub("эксплуатация", "exploitation", dic4$Term)
dic4$Term <- gsub("мнение", "opinion", dic4$Term)
dic4$Term <- gsub("приобрести", "acheter", dic4$Term)
dic4$Term <- gsub("движение", "mouvement", dic4$Term)
dic4$Term <- gsub("девяткино", "Deviatkino", dic4$Term)
dic4$Term <- gsub("помещение", "espace", dic4$Term)
dic4$Term <- gsub("гк", "société de planification urbaine", dic4$Term)
dic4$Term <- gsub("необходим", "nécessaire", dic4$Term)
dic4$Term <- gsub("этажи", "étages", dic4$Term)
dic4$Term <- gsub("глава", "chef", dic4$Term)
dic4$Term <- gsub("конкурс", "concours", dic4$Term)
dic4$Term <- gsub("новости", "actualités", dic4$Term)
dic4$Term <- gsub("поселение", "peuplement", dic4$Term)
dic4$Term <- gsub("вариант", "option", dic4$Term)
dic4$Term <- gsub("водители", "chauffeurs", dic4$Term)
dic4$Term <- gsub("друг", "ami", dic4$Term)
dic4$Term <- gsub("перспективы", "perspectives", dic4$Term)
dic4$Term <- gsub("участие", "participation", dic4$Term)
dic4$Term <- gsub("конец", "fin", dic4$Term)
dic4$Term <- gsub("невский", "Nevsky", dic4$Term)
dic4$Term <- gsub("граница", "frontière", dic4$Term)
dic4$Term <- gsub("региональный", "régional", dic4$Term)
dic4$Term <- gsub("результат", "résultat", dic4$Term)
dic4$Term <- gsub("системы", "systèmes", dic4$Term)
dic4$Term <- gsub("смольный", "Smolny", dic4$Term)
dic4$Term <- gsub("участники", "participants", dic4$Term)
dic4$Term <- gsub("управление", "gestion", dic4$Term)
dic4$Term <- gsub("документ", "document", dic4$Term)
dic4$Term <- gsub("схема", "schéma", dic4$Term)
dic4$Term <- gsub("предлагать", "proposer", dic4$Term)

dic4$Term <- gsub("момент", "moment", dic4$Term)
dic4$Term <- gsub("информация", "information", dic4$Term)
dic4$Term <- gsub("данные", "données", dic4$Term)
dic4$Term <- gsub("сегмент", "segment", dic4$Term)
dic4$Term <- gsub("коммерческий", "commercial", dic4$Term)
dic4$Term <- gsub("ввод", "introduction", dic4$Term)
dic4$Term <- gsub("комплексный", "complexe", dic4$Term)
dic4$Term <- gsub("существует", "existe", dic4$Term)
dic4$Term <- gsub("дтп", "accident de la route", dic4$Term)
dic4$Term <- gsub("совет", "conseil", dic4$Term)
dic4$Term <- gsub("магистраль", "artère principale", dic4$Term)
dic4$Term <- gsub("проезд", "passage", dic4$Term)
dic4$Term <- gsub("конкурентоспособность", "compétitivité", dic4$Term)
dic4$Term <- gsub("пассажиры", "passagers", dic4$Term)
dic4$Term <- gsub("ставка", "taux", dic4$Term)
dic4$Term <- gsub("чиновник", "fonctionnaire", dic4$Term)
dic4$Term <- gsub("столица", "capitale", dic4$Term)
dic4$Term <- gsub("занима", "occupe", dic4$Term)
dic4$Term <- gsub("принято", "accepté", dic4$Term)
dic4$Term <- gsub("трамвай", "tramway", dic4$Term)
dic4$Term <- gsub("соответствие", "conformité", dic4$Term)
dic4$Term <- gsub("собствен", "propriété", dic4$Term)
dic4$Term <- gsub("дела", "affaires", dic4$Term)
dic4$Term <- gsub("пригородный", "banlieue", dic4$Term)
dic4$Term <- gsub("работать", "travailler", dic4$Term)
dic4$Term <- gsub("проектирование", "conception", dic4$Term)
dic4$Term <- gsub("развивать", "développer", dic4$Term)
dic4$Term <- gsub("парковка", "stationnement", dic4$Term)
dic4$Term <- gsub("экономическ", "économique", dic4$Term)
dic4$Term <- gsub("земельные", "terrestres", dic4$Term)
dic4$Term <- gsub("производство", "production", dic4$Term)
dic4$Term <- gsub("образование", "éducation", dic4$Term)
dic4$Term <- gsub("мосты", "ponts", dic4$Term)
dic4$Term <- gsub("трет", "troisième", dic4$Term)
dic4$Term <- gsub("студия", "studio", dic4$Term)
dic4$Term <- gsub("снижение", "réduction", dic4$Term)
dic4$Term <- gsub("миллионы", "millions", dic4$Term)
dic4$Term <- gsub("прав", "droits", dic4$Term)
dic4$Term <- gsub("сложно", "complexe", dic4$Term)
dic4$Term <- gsub("действ", "action", dic4$Term)
dic4$Term <- gsub("условия", "conditions", dic4$Term)
dic4$Term <- gsub("следующий", "suivant", dic4$Term)
dic4$Term <- gsub("сам", "soi-même", dic4$Term)
dic4$Term <- gsub("цел", "but", dic4$Term)
dic4$Term <- gsub("организация", "organisation", dic4$Term)
dic4$Term <- gsub("разрешение", "autorisation", dic4$Term)
dic4$Term <- gsub("суд", "tribunal", dic4$Term)
dic4$Term <- gsub("кольцевая", "boulevard périphérique", dic4$Term)
dic4$Term <- gsub("покупка", "achat", dic4$Term)
dic4$Term <- gsub("проценты", "pourcentage", dic4$Term)
dic4$Term <- gsub("клуб", "club", dic4$Term)
dic4$Term <- gsub("локация", "emplacement", dic4$Term)
dic4$Term <- gsub("государствен", "état", dic4$Term)
dic4$Term <- gsub("янино", "Ianiño", dic4$Term)
dic4$Term <- gsub("большинство", "majorité", dic4$Term)
dic4$Term <- gsub("максимальный", "maximum", dic4$Term)
dic4$Term <- gsub("концепция", "concept", dic4$Term)
dic4$Term <- gsub("позволять", "permettre", dic4$Term)
dic4$Term <- gsub("услуги", "services", dic4$Term)
dic4$Term <- gsub("освоение", "exploitation", dic4$Term)
dic4$Term <- gsub("ряд", "rangée", dic4$Term)
dic4$Term <- gsub("очевидцы", "témoins", dic4$Term)
dic4$Term <- gsub("развязка", "échangeur", dic4$Term)
dic4$Term <- gsub("прошлое", "passé", dic4$Term)
dic4$Term <- gsub("здание", "bâtiment", dic4$Term)
dic4$Term <- gsub("сделки", "transactions", dic4$Term)
dic4$Term <- gsub("период", "période", dic4$Term)
dic4$Term <- gsub("частный", "privé", dic4$Term)
dic4$Term <- gsub("причин", "raisons", dic4$Term)
dic4$Term <- gsub("каждый", "chaque", dic4$Term)
dic4$Term <- gsub("выбор", "choix", dic4$Term)
dic4$Term <- gsub("связано", "lié", dic4$Term)
dic4$Term <- gsub("договор", "contrat", dic4$Term)
dic4$Term <- gsub("дополнительн", "supplémentaire", dic4$Term)
dic4$Term <- gsub("полиция", "police", dic4$Term)
dic4$Term <- gsub("председатель", "président", dic4$Term)
dic4$Term <- gsub("сотрудник", "employé", dic4$Term)
dic4$Term <- gsub("втор", "deuxième", dic4$Term)
dic4$Term <- gsub("цдс", "Centrodorstroy", dic4$Term)
dic4$Term <- gsub("спортивн", "sportif", dic4$Term)
dic4$Term <- gsub("рабочий", "travailleur", dic4$Term)
dic4$Term <- gsub("вход", "entrée", dic4$Term)
dic4$Term <- gsub("приморский", "maritime", dic4$Term)
dic4$Term <- gsub("современ", "moderne", dic4$Term)
dic4$Term <- gsub("жизнь", "vie", dic4$Term)
dic4$Term <- gsub("показатели", "indicateurs", dic4$Term)
dic4$Term <- gsub("пункт", "point", dic4$Term)
dic4$Term <- gsub("сумма", "somme", dic4$Term)
dic4$Term <- gsub("практический", "pratique", dic4$Term)
dic4$Term <- gsub("прода", "vendre", dic4$Term)
dic4$Term <- gsub("выход", "sortie", dic4$Term)
dic4$Term <- gsub("массов", "de masse", dic4$Term)
dic4$Term <- gsub("автобус", "autobus", dic4$Term)
dic4$Term <- gsub("железнодорожн", "ferroviaire", dic4$Term)
dic4$Term <- gsub("оста", "reste", dic4$Term)
dic4$Term <- gsub("setlcity", "Setlcity", dic4$Term)
dic4$Term <- gsub("настоящее", "actuel", dic4$Term)
dic4$Term <- gsub("больше", "plus", dic4$Term)
dic4$Term <- gsub("этап", "étape", dic4$Term)
dic4$Term <- gsub("как", "comment", dic4$Term)
dic4$Term <- gsub("муниципальн", "municipal", dic4$Term)
dic4$Term <- gsub("называ", "appeler", dic4$Term)
dic4$Term <- gsub("аналитики", "analystes", dic4$Term)
dic4$Term <- gsub("южн", "sud", dic4$Term)
dic4$Term <- gsub("помощь", "aide", dic4$Term)
dic4$Term <- gsub("микрорайон", "quartier", dic4$Term)
dic4$Term <- gsub("серьезн", "sérieusement", dic4$Term)
dic4$Term <- gsub("купить", "acheter", dic4$Term)
dic4$Term <- gsub("люб", "aimer", dic4$Term)
dic4$Term <- gsub("юг", "sud", dic4$Term)
dic4$Term <- gsub("реконструкция", "réhabilitation", dic4$Term)
dic4$Term <- gsub("вицегубернатор", "vice-gouverneur", dic4$Term)
dic4$Term <- gsub("жилищный", "résidentiel", dic4$Term)
dic4$Term <- gsub("быстро", "rapidement", dic4$Term)
dic4$Term <- gsub("оказа", "fournir", dic4$Term)
dic4$Term <- gsub("налоги", "impôts", dic4$Term)
dic4$Term <- gsub("добавить", "ajouter", dic4$Term)
dic4$Term <- gsub("приходить", "venir", dic4$Term)
dic4$Term <- gsub("хорош", "bien", dic4$Term)
dic4$Term <- gsub("гатчина", "Gatchina", dic4$Term)
dic4$Term <- gsub("говор", "parler", dic4$Term)
dic4$Term <- gsub("клиент", "client", dic4$Term)
dic4$Term <- gsub("соглашение", "accord", dic4$Term)
dic4$Term <- gsub("един", "unique", dic4$Term)
dic4$Term <- gsub("материалы", "matériaux", dic4$Term)
dic4$Term <- gsub("пострада", "souffrir", dic4$Term)
dic4$Term <- gsub("доли", "parts", dic4$Term)
dic4$Term <- gsub("заместители", "remplaçants", dic4$Term)
dic4$Term <- gsub("проживание", "résidence", dic4$Term)
dic4$Term <- gsub("благоустройство", "aménagement", dic4$Term)
dic4$Term <- gsub("технология", "technologie", dic4$Term)
dic4$Term <- gsub("провод", "câble", dic4$Term)
dic4$Term <- gsub("так", "ainsi", dic4$Term)
dic4$Term <- gsub("требование", "exigence", dic4$Term)
dic4$Term <- gsub("граждане", "citoyens", dic4$Term)
dic4$Term <- gsub("сво", "son", dic4$Term)
dic4$Term <- gsub("двор", "cour", dic4$Term)
dic4$Term <- gsub("пространство", "espace", dic4$Term)
dic4$Term <- gsub("инженерн", "ingénieur", dic4$Term)
dic4$Term <- gsub("мужчина", "homme", dic4$Term)
dic4$Term <- gsub("градостроительн", "urbanisme", dic4$Term)
dic4$Term <- gsub("продолжа", "continuer", dic4$Term)
dic4$Term <- gsub("отделы", "départements", dic4$Term)
dic4$Term <- gsub("охта", "Ochta", dic4$Term)
dic4$Term <- gsub("трест", "trust", dic4$Term)
dic4$Term <- gsub("среда", "mercredi", dic4$Term)
dic4$Term <- gsub("зелен", "vert", dic4$Term)
dic4$Term <- gsub("тосно", "Tosno", dic4$Term)
dic4$Term <- gsub("гектары", "hectares", dic4$Term)
dic4$Term <- gsub("group", "groupe", dic4$Term)
dic4$Term <- gsub("дирекция", "direction", dic4$Term)
dic4$Term <- gsub("пояснить", "expliquer", dic4$Term)
dic4$Term <- gsub("баллы", "points", dic4$Term)
dic4$Term <- gsub("дети", "enfants", dic4$Term)
dic4$Term <- gsub("общ", "commun", dic4$Term)
dic4$Term <- gsub("запланировать", "planifier", dic4$Term)
dic4$Term <- gsub("подземка", "métro", dic4$Term)
dic4$Term <- gsub("найти", "trouver", dic4$Term)
dic4$Term <- gsub("частности", "particulièrement", dic4$Term)
dic4$Term <- gsub("безопасность", "sécurité", dic4$Term)
dic4$Term <- gsub("заверш", "terminer", dic4$Term)
dic4$Term <- gsub("документация", "documentation", dic4$Term)
dic4$Term <- gsub("коммерсант", "commerçant", dic4$Term)
dic4$Term <- gsub("ремонт", "réparation", dic4$Term)
dic4$Term <- gsub("наблюдать", "observer", dic4$Term)
dic4$Term <- gsub("узел", "nœud", dic4$Term)
dic4$Term <- gsub("процесс", "processus", dic4$Term)
dic4$Term <- gsub("рассматрива", "examiner", dic4$Term)
dic4$Term <- gsub("важн", "important", dic4$Term)
dic4$Term <- gsub("закон", "loi", dic4$Term)
dic4$Term <- gsub("ответ", "réponse", dic4$Term)
dic4$Term <- gsub("первичный", "primaire", dic4$Term)
dic4$Term <- gsub("средн", "moyen", dic4$Term)
dic4$Term <- gsub("статус", "statut", dic4$Term)
dic4$Term <- gsub("вконтакте", "VKontakte", dic4$Term)
dic4$Term <- gsub("корпус", "corps", dic4$Term)
dic4$Term <- gsub("небольш", "petit", dic4$Term)
dic4$Term <- gsub("получа", "recevoir", dic4$Term)
dic4$Term <- gsub("маркетинг", "marketing", dic4$Term)
dic4$Term <- gsub("отдельн", "individuel", dic4$Term)
dic4$Term <- gsub("строители", "constructeurs", dic4$Term)
dic4$Term <- gsub("значительно", "significativement", dic4$Term)
dic4$Term <- gsub("связи", "communications", dic4$Term)
dic4$Term <- gsub("совместно", "ensemble", dic4$Term)
dic4$Term <- gsub("состояние", "état", dic4$Term)
dic4$Term <- gsub("восточн", "est", dic4$Term)
dic4$Term <- gsub("набережная", "quai", dic4$Term)
dic4$Term <- gsub("часы", "heures", dic4$Term)
dic4$Term <- gsub("мир", "monde", dic4$Term)
dic4$Term <- gsub("сем", "famille", dic4$Term)
dic4$Term <- gsub("депутат", "député", dic4$Term)
dic4$Term <- gsub("проектн", "projet", dic4$Term)
dic4$Term <- gsub("отсутствие", "absence", dic4$Term)
dic4$Term <- gsub("самый", "le plus", dic4$Term)
dic4$Term <- gsub("сдела", "faire", dic4$Term)
dic4$Term <- gsub("текущ", "actuel", dic4$Term)
dic4$Term <- gsub("высот", "hauteur", dic4$Term)
dic4$Term <- gsub("сообщен", "rapporté", dic4$Term)
dic4$Term <- gsub("ожида", "attendre", dic4$Term)


dic4$Term <- gsub("поликлиник", "clinique", dic4$Term)
dic4$Term <- gsub("доход", "revenu", dic4$Term)
dic4$Term <- gsub("красносельск", "Krasnoselsk", dic4$Term)
dic4$Term <- gsub("разработк", "développement", dic4$Term)
dic4$Term <- gsub("эконом", "économie", dic4$Term)
dic4$Term <- gsub("кризис", "crise", dic4$Term)
dic4$Term <- gsub("пробки", "embouteillages", dic4$Term)
dic4$Term <- gsub("city", "city", dic4$Term)
dic4$Term <- gsub("имеют", "ont", dic4$Term)
dic4$Term <- gsub("оккервиль", "Okkerville", dic4$Term)
dic4$Term <- gsub("популярн", "populaire", dic4$Term)
dic4$Term <- gsub("центральный", "central", dic4$Term)
dic4$Term <- gsub("отделк", "finition", dic4$Term)
dic4$Term <- gsub("сообща", "informer", dic4$Term)
dic4$Term <- gsub("официальн", "officiellement", dic4$Term)
dic4$Term <- gsub("начальник", "chef", dic4$Term)
dic4$Term <- gsub("комплексы", "complexes", dic4$Term)
dic4$Term <- gsub("лучш", "meilleur", dic4$Term)
dic4$Term <- gsub("трассы", "routes", dic4$Term)
dic4$Term <- gsub("беглов", "Beglov", dic4$Term)
dic4$Term <- gsub("обрат", "retour", dic4$Term)
dic4$Term <- gsub("отношен", "relation", dic4$Term)
dic4$Term <- gsub("размещение", "placement", dic4$Term)
dic4$Term <- gsub("выдел", "allouer", dic4$Term)
dic4$Term <- gsub("полис", "police", dic4$Term)
dic4$Term <- gsub("заневский", "Zanevsky", dic4$Term)
dic4$Term <- gsub("кредит", "crédit", dic4$Term)
dic4$Term <- gsub("сайт", "site web", dic4$Term)
dic4$Term <- gsub("специалисты", "spécialistes", dic4$Term)
dic4$Term <- gsub("плат", "paiement", dic4$Term)
dic4$Term <- gsub("существенный", "important", dic4$Term)
dic4$Term <- gsub("выборгск", "Vyborgsk", dic4$Term)
dic4$Term <- gsub("растет", "croît", dic4$Term)
dic4$Term <- gsub("предварительн", "préliminaire", dic4$Term)
dic4$Term <- gsub("составля", "constituer", dic4$Term)
dic4$Term <- gsub("счита", "considérer", dic4$Term)
dic4$Term <- gsub("газпром", "Gazprom", dic4$Term)
dic4$Term <- gsub("дальн", "distant", dic4$Term)
dic4$Term <- gsub("задача", "tâche", dic4$Term)
dic4$Term <- gsub("масштабн", "à grande échelle", dic4$Term)
dic4$Term <- gsub("подход", "approche", dic4$Term)
dic4$Term <- gsub("карт", "carte", dic4$Term)
dic4$Term <- gsub("пулково", "Pulkovo", dic4$Term)
dic4$Term <- gsub("управля", "gérer", dic4$Term)
dic4$Term <- gsub("пример", "exemple", dic4$Term)
dic4$Term <- gsub("затраты", "coûts", dic4$Term)
dic4$Term <- gsub("наличие", "disponibilité", dic4$Term)
dic4$Term <- gsub("никак", "en aucune façon", dic4$Term)
dic4$Term <- gsub("магазин", "magasin", dic4$Term)
dic4$Term <- gsub("мега", "méga", dic4$Term)
dic4$Term <- gsub("стар", "vieux", dic4$Term)
dic4$Term <- gsub("подобн", "similaire", dic4$Term)
dic4$Term <- gsub("правобережный", "rive droite", dic4$Term)
dic4$Term <- gsub("техническ", "technique", dic4$Term)
dic4$Term <- gsub("год", "année", dic4$Term)
dic4$Term <- gsub("предприят", "entrepreneurial", dic4$Term)
dic4$Term <- gsub("яндекс", "Yandex", dic4$Term)
dic4$Term <- gsub("большие", "grands", dic4$Term)
dic4$Term <- gsub("мал", "petit", dic4$Term)
dic4$Term <- gsub("обычный", "ordinaire", dic4$Term)
dic4$Term <- gsub("промышлен", "industriel", dic4$Term)
dic4$Term <- gsub("сниз", "réduire", dic4$Term)
dic4$Term <- gsub("треб", "exiger", dic4$Term)
dic4$Term <- gsub("крупн", "gros", dic4$Term)
dic4$Term <- gsub("вечер", "soir", dic4$Term)
dic4$Term <- gsub("подрядчик", "entrepreneur", dic4$Term)
dic4$Term <- gsub("введ", "introduire", dic4$Term)
dic4$Term <- gsub("принима", "accepter", dic4$Term)
dic4$Term <- gsub("разн", "différent", dic4$Term)
dic4$Term <- gsub("расход", "dépense", dic4$Term)
dic4$Term <- gsub("учет", "comptabilité", dic4$Term)
dic4$Term <- gsub("зрение", "vision", dic4$Term)
dic4$Term <- gsub("институт", "institut", dic4$Term)
dic4$Term <- gsub("переход", "transition", dic4$Term)
dic4$Term <- gsub("повышен", "augmenté", dic4$Term)
dic4$Term <- gsub("правила", "règles", dic4$Term)
dic4$Term <- gsub("задержа", "retarder", dic4$Term)
dic4$Term <- gsub("паркинг", "stationnement", dic4$Term)
dic4$Term <- gsub("пригород", "banlieue", dic4$Term)
dic4$Term <- gsub("чп", "urgence", dic4$Term)
dic4$Term <- gsub("xaэт", "xaэт", dic4$Term)
dic4$Term <- gsub("большое", "grand", dic4$Term)
dic4$Term <- gsub("опыт", "expérience", dic4$Term)
dic4$Term <- gsub("полн", "plein", dic4$Term)
dic4$Term <- gsub("сегодняшний", "d'aujourd'hui", dic4$Term)
dic4$Term <- gsub("тенденция", "tendance", dic4$Term)
dic4$Term <- gsub("архитектурн", "architectural", dic4$Term)
dic4$Term <- gsub("вод", "eau", dic4$Term)
dic4$Term <- gsub("скор", "vite", dic4$Term)
dic4$Term <- gsub("тысячи", "milliers", dic4$Term)
dic4$Term <- gsub("школа", "école", dic4$Term)
dic4$Term <- gsub("источник", "source", dic4$Term)
dic4$Term <- gsub("перв", "premier", dic4$Term)
dic4$Term <- gsub("переезд", "déménagement", dic4$Term)
dic4$Term <- gsub("польз", "utilité", dic4$Term)
dic4$Term <- gsub("продолжен", "continué", dic4$Term)
dic4$Term <- gsub("выбра", "choisir", dic4$Term)
dic4$Term <- gsub("пересечение", "intersection", dic4$Term)
dic4$Term <- gsub("сферы", "domaines", dic4$Term)
dic4$Term <- gsub("включение", "inclusion", dic4$Term)

dic4$Term <- gsub("генеральныйдиректор", "directeurgénéral", dic4$Term)
dic4$Term <- gsub("вицегубернатор", "vicegouverneur", dic4$Term)
dic4$Term <- gsub("университет", "université", dic4$Term)
dic4$Term <- gsub("проектирование ", "Aménagement", dic4$Term)
dic4$Term <- gsub("команд", "diriger", dic4$Term)
dic4$Term <- gsub("команда", "équipe", dic4$Term)
dic4$Term <- gsub("правобережный", "rivedroite", dic4$Term)


dic4$Term <- gsub("генеральныйdirecteur", "directeurgénéral", dic4$Term)
dic4$Term <- gsub("вицеgouverneur", "vicegouverneur", dic4$Term)
dic4$Term <- gsub("projetирование", "Aménagement", dic4$Term)
dic4$Term <- gsub("droitsобережный", "rivedroite", dic4$Term)


dic4$Term <- gsub("травм", "blessures", dic4$Term)
dic4$Term <- gsub("госпитализирова", "hospitalisé", dic4$Term)
dic4$Term <- gsub("авария", "accident", dic4$Term)
dic4$Term <- gsub("происшествие", "incident", dic4$Term)
dic4$Term <- gsub("летн", "été", dic4$Term)
dic4$Term <- gsub("уголовн", "criminel", dic4$Term)
dic4$Term <- gsub("очеvueцы", "témoins", dic4$Term)
dic4$Term <- gsub("килоmètres", "kilomètres", dic4$Term)

view(dic4)
```

```{r}
#frequent_terms(dtmsmo2l, n=500)
```



```{r lemmatiser, message=FALSE, warning=FALSE, include=FALSE}
library(tm)
#setdiff(rownames(dic3), rownames(dic2))
frenchdtmsmo2l <- combine_terms(dtmsmo2,dic4) #без лишних слов 
frenchdtmsmo_d2l <- combine_terms(dtmsmo_d2,dic4)

```



```{r lemmatiser, message=FALSE, warning=FALSE, include=FALSE}
library(tm)
#setdiff(rownames(dic3), rownames(dic2))
dtmsmo2l <- combine_terms(dtmsmo2,dic3) #без лишних слов 
dtmsmo_d2l <- combine_terms(dtmsmo_d2,dic3)

```

### Nuage de mots 2

```{r cloud2, echo=FALSE, message=FALSE, warning=FALSE}
cloud <-word_cloud(frenchdtmsmo2l, color="#8968CD", n=500, remove_stopwords = TRUE, min.freq= 200)
#заменить санктпетебург на спб 
```

Le nuage de mots mis à jour des 200 termes les plus fréquents est le suivant :
```{r cloud2, echo=FALSE, message=FALSE, warning=FALSE}
cloud<-word_cloud(dtmsmo2l, color="cadetblue", n=500, remove_stopwords = TRUE, min.freq= 200)
#заменить санктпетебург на спб 
```
Voici le nuage de mots des 700 termes les plus fréquents avec une fréquence d'au moins 100.

Les termes les plus populaires de notre corpus sont : Saint-Pétersbourg (le nom de la mégapole) avec une fréquence globale de 8907 (1,47%), Lenoblast (le nom de la région où se trouve la ville étudiée) avec une fréquence globale de 7943 (1,3%), Projet avec une fréquence globale de 5137 (0,84%), Construction avec une fréquence globale de 4424, Quartiers avec une fréquence globale de 4336, et enfin, le nom de la ville étudiée, Kudrovo, avec une fréquence de 4122 (0,68%).

Dans la liste des 100 termes les plus fréquents, on trouve également les termes suivants : ville, nouveau, entreprise, transport, roubles (monnaie russe), station, territoire, appartements, objets, marché, logement, maison, travaux de construction, développeurs, développement, mètres carrés, métro.

### Les termes les plus fréquents sont :
```{r frequency2, echo=FALSE, message=FALSE, warning=FALSE}
tab3 <- frequent_terms(dtmsmo2l, n=100)
kable(tab3,
      caption = "**100 termes les plus fréquents après lemmatisation**",
      align="lcc", escape = FALSE) %>%
  kable_styling(bootstrap_options = c("striped", "hover"), full_width = FALSE)%>%
  column_spec(2, background = "cadetblue")
```
### Les termes les plus fréquents sont :
```{r frequency2, echo=FALSE, message=FALSE, warning=FALSE}
tab3 <- frequent_terms(frenchdtmsmo2l, n=50)
tab3
kable(tab3,
      caption = "**20 termes les plus fréquents après lemmatisation**",
      align="lcc", escape = FALSE) %>%
  kable_styling(bootstrap_options = c("striped", "hover"), full_width = FALSE)%>%
  column_spec(2, background = "#8968CD")

#save_kable(tab3, file = "my_table4.png")
```
Après analyse des 100 termes les plus fréquents après lemmatisation dans le corpus d'articles sur Kudrovo, on peut tirer les conclusions suivantes :

Les termes "Санктпетербург" (Saint-Pétersbourg) et "Ленобласть" (oblast de Léningrad) sont les plus fréquents dans le corpus, ce qui indique que les articles portent principalement sur cette région.

Les thèmes les plus importants dans les articles sont liés à la construction et à l'immobilier, comme en témoignent les termes "проект" (projet), "строительство" (construction), "квартиры" (appartements), "дом" (maison), "жилые" (résidentiel) et d'autres.

Le secteur des transports est également un sujet fréquent, comme en témoignent les termes "транспорт" (transport), "станция" (station), "улицы" (rues), "линии" (lignes) et "метро" (métro).

Les termes "компания" (entreprise), "работы" (travaux), "застройщик" (promoteur immobilier) et "власти" (autorités) indiquent une implication importante des entreprises et des autorités dans le développement de la région.

Les termes "цены" (prix), "стоимость" (coût) et "рынок" (marché) suggèrent un intérêt pour les tendances des prix et le marché de l'immobilier dans la région.

Les termes "инфраструктура" (infrastructure), "развитие" (développement) et "планирование" (planification) soulignent l'importance de l'infrastructure et du développement de la région.

La présence de termes liés aux services publics, tels que "детские сады" (jardins d'enfants), "школы" (écoles) et "здравоохранение" (santé) indique l'attention portée aux services sociaux dans la région.

Les termes "эксперты" (experts), "губернатор" (gouverneur) et "комитет" (comité) suggèrent une implication active des experts et des autorités locales dans le développement de la région.

En conclusion, l'analyse des termes les plus fréquents dans les articles sur Kudrovo met en évidence les principaux thèmes abordés, tels que la construction, l'immobilier, les transports et le développement régional. Ces résultats peuvent être utiles pour une meilleure compréhension des caractéristiques et des sujets traités dans les articles sur cette région.

## Specific terms

Après la lemmatisation du dictionnaire, nous obtenons les termes spécifiques suivants utilisés par chaque média :

```{r specific terms, echo=FALSE, message=FALSE, warning=FALSE}

tab4 <- specific_terms(dtmsmo2l, meta(corpus2)$Газета, 
  p=0.1,                     
  n = 5,
  sparsity = 0.98,
  min_occ = 100)


tab4

#kable(tab4,
     # caption = "**Specific_terms for each news outlet**")

#to fix this table 

#исправить 

#https://www.rdocumentation.org/packages/R.temis/versions/0.1.3/topics/specific_terms
#https://rtemis.hypotheses.org/r-temis-dans-rstudio
```

Les données présentent les résultats de l'analyse d'un corpus d'articles provenant de différentes sources en utilisant des termes spécifiques associés à chaque journal. Chaque journal possède ses propres termes clés qui représentent les caractéristiques et les accents exprimés dans ses publications. 

Voici une brève description des résultats de l'analyse pour chaque journal : 1) 47online : Le journal se concentre sur l'administration, les activistes et les événements liés à différentes localités, y compris Vsevolozhsk et Gatchina; 2) Argumentyifakty : Le journal met l'accent sur divers aspects de la vie dans la ville de Tosno, les clubs et les ligues sportives; 3) AgenceTASS : Le journal couvre les activités du gouverneur et les événements dans la région de Leningrad, y compris Saint-Pétersbourg et l'oblast de Leningrad; 4) Paper :Le journal traite principalement des lignes de transport, des stations de métro, des règles et des règlements, ainsi que des questions relatives aux entreprises et aux affaires; 5) Gazera.ru : Le journal se concentre sur Gazprom et le district d'Okhta, ainsi que sur les aspects commerciaux du centre-ville; 6) Interfax :Le journal se concentre également sur Gazprom, ainsi que sur les routes et les événements commerciaux dans la région de Leningrad; 7)Kanal78 : Le journal se concentre sur les incidents, les résidents et les réunions qui ont lieu à Saint-Pétersbourg et dans les environs; 8) Kommersant : Le journal Kommersant met l'accent sur les aspects financiers tels que les intérêts, les loyers, les marques et les projets; 9) KPSPB :Le journal KPSPB met l'accent sur Murino, les criminels récemment découverts, les étudiants et la police; 10)Lenta : Le journal Lenta met l'accent sur la détection, les scores, les appartements, les millions et les mètres; 11)MoskovskyKomsomolets :Le journal est dominé par des informations sur les banques, l'émission d'hypothèques, les appartements et les étudiants; 12) NevaToday : Le journal relate les incidents et les accidents (accidents de la route) survenus à Kudrovo, ainsi que les événements de la soirée; 13)NevskyNovosti : Le journal se concentre sur l'eau, le changement de statut, Kudrovo, le métro et les résidents; 14) Novosti47 :Le journal couvre l'administration, Vsevolozhsk, Zanevsky, les nouvelles et Okkervil; 15) RBC : Le journal RBC met l'accent sur le groupe ainsi que sur Gazprom, les intérêts, les hectares et les projets; 16) RBCSPb : Le journal RBCSPb met l'accent sur "not", "Petersburg", "eta", ainsi que sur l'agglomération urbaine et les promoteurs; 17) RIAnovosti : Le journal RIAnovosti traite de l'oblast de Leningrad, des nouvelles, des pompiers, des régions et de Drozdenko; 18) RossiyskayaGazeta : Le journal RossiyskayaGazeta se concentre sur les pourcentages, les étudiants, les trains, les millions et les projets; 19)SPbDnevnik : Le journal SPbDnevnik se concentre sur Dybenko, Kudrovo, les lignes de métro et le district de Pravoberezhny; 20) TelekanalSPb :Le journal TelekanalSPb se concentre sur Beglov, VKontakte, Kudrovo, le métro et les nouvelles.

Ces analyses donnent un aperçu des sujets et des événements les plus importants et les plus discutés dans chaque journal.

Les conclusions de ces résultats peuvent être formulées comme suit : Chaque journal a sa propre thématique unique et ses intérêts, qui se reflètent dans ses termes clés principaux. Certains journaux se concentrent sur des emplacements spécifiques et des événements liés à ces lieux (par exemple, "Vsevolozhsk" et "Gatchina" dans "47online").

D'autres journaux se concentrent sur le sport, les affaires, le métro et d'autres aspects de la vie urbaine. Il existe des journaux qui mettent l'accent sur certaines organisations ou entreprises, comme "Gazprom" dans "Gazera.ру" et "Interfax".

Les résultats indiquent également les intérêts géographiques et les accents de chaque journal, tels que la focalisation sur Saint-Pétersbourg et la région de l'oblast de Léningrad. Cette analyse aide à comprendre quelles sont les thématiques et les événements les plus importants et discutés dans chaque journal.

Exemples de termes clés liés à Kudrovo : "Kanal78"  souligne l'importance des quartiers voisins, y compris Kudrovo, et reflète les événements qui se déroulent dans ces endroits. "NevaToday" met l'accent sur les incidents et les accidents, y compris les accidents de la route, survenus à Kudrovo. Le domaine d'intérêt du journal "NevskyNovosti" comprend le changement de statut des quartiers, y compris Kudrovo. "TelekanalSPb" couvre les événements liés à Kudrovo dans le contexte d'autres lieux, tels que les événements marquants liés à Beglov.



```{r specific terms2, echo=FALSE, message=FALSE, warning=FALSE}
tab41 <- specific_terms(frenchdtmsmo2l, meta(corpus2)$Год, 
  p=0.1,                     
  n = 5,
  sparsity = 0.98,
  min_occ = 100)


tab41




```

```{r specific terms2, echo=FALSE, message=FALSE, warning=FALSE}
tab41french <- specific_terms(frenchdtmsmo2l, meta(corpus2)$Год, 
  p=0.1,                     
  n = 5,
  sparsity = 0.98,
  min_occ = 100)


tab41french
```

L'analyse des termes spécifiques utilisés chaque année montre qu'en 2010, les journaux étudiés ont utilisé les termes Gasprom (géant russe du gaz), Okhta (un autre territoire en développement actif), Okkervil (une partie de Saint-Pétersbourg reliée à Kudrovo), mètres carrés et "площадь жилья" (surface habitable). 

En 2011, les termes spécifiques tels que Gasprom, la réalisation de projets, les kilomètres et le volume ont été mis en avant. En 2012, on a pu observer l'émergence des termes tels que Chef Général, Chef, prix et AH (?). En 2013, les sujets abordés ont inclus Vice-Chef, développement de projets, villes, marché et prix. En 2014, l'accent a été mis sur l'hébergement, les appartements, les mètres carrés, le confort et le développement. En 2015, on a constaté l'importance des termes tels que Société de développement urbain, bâtiments résidentiels, prêt et promoteur. En 2016, les sujets prédominants ont été les appartements et les écoles. En 2017, les discussions ont porté sur les groupes, les équipes et les clubs. En 2018, les sujets phares étaient les lignes de métro, Dybenko, les lignes de métro, les voyages et le métro.сEn 2019, les termes pertinents incluaient Beglov (gouverneur de Saint-Pétersbourg), Vkontakte (réseau social populaire), la Ville et Drozdenko (gouverneur de Kudorovo).

Cette analyse met en évidence les termes spécifiques pour chaque année, reflétant les sujets d'intérêt et les tendances marquantes de cette période. Ces termes illustrent les préoccupations et les développements importants liés à Kudrovo et à la région de Saint-Pétersbourg tout au long de cette décennie.

Les conclusions tirées de l'analyse des données sur les termes clés utilisés dans les publications concernant Kudrovo de 2010 à 2019 sont les suivantes :

Dynamique des intérêts : Au cours de cette décennie, les intérêts et les accents dans les publications concernant Kudrovo ont changé. Chaque année, on peut identifier des termes clés spécifiques qui reflètent les sujets les plus importants et les plus discutés dans différents journaux.

Importance de "Gazprom" : "Gazprom" a été l'un des termes clés les plus fréquemment mentionnés au fil des ans, ce qui indique l'importance de cette organisation et son rôle dans les événements et les intérêts liés à Kudrovo.

Diversité des sujets : Les publications sur Kudrovo ont couvert des sujets variés, tels que la construction, le logement, le sport, les affaires, l'éducation, les transports et d'autres aspects de la vie urbaine.

Accent sur le logement : À plusieurs reprises, des termes clés liés au logement et à l'urbanisme ont été mentionnés, ce qui peut refléter des questions pertinentes en matière de politique de logement.

Mise en avant des aspects régionaux : Certains termes clés indiquaient des intérêts géographiques, tels que "Ohta", "Lignes", "Itinéraire" et "Métro", qui sont liés à des lieux spécifiques à Kudrovo.

Évolution du statut : Les termes clés liés à l'évolution du statut des quartiers, comme "Vice-gouverneur" et "Dybenko", ont également attiré l'attention à différentes périodes.

Influence des personnalités clés : Certains termes clés sont liés à des personnalités spécifiques, telles que "Beglov" et "Drozdenco", ce qui indique leur influence et leur activité en relation avec Kudrovo.

Dans l'ensemble, l'analyse des termes clés permet de comprendre la dynamique des intérêts et des accents dans les publications concernant Kudrovo, ainsi que d'identifier les sujets les plus importants et les plus discutés à différentes époques. Ces informations peuvent être utiles pour étudier le développement de la ville et ses aspects les plus importants.


```{r specific terms3, echo=FALSE, message=FALSE, warning=FALSE}
tab42 <- specific_terms(dtmsmo2l, meta(corpus2)$Статус, 
  p=0.1,                     
  n = 5,
  sparsity = 0.98,
  min_occ = 100)


tab42
```

```{r}
tab42 <- specific_terms(frenchdtmsmo2l, meta(corpus2)$Газета, 
  p=0.1,                     
  n = 5,
  sparsity = 0.98,
  min_occ = 100)
tab42

```


```{r specific terms3, echo=FALSE, message=FALSE, warning=FALSE}
tab42 <- specific_terms(frenchdtmsmo2l, meta(corpus2)$Статус, 
  p=0.1,                     
  n = 5,
  sparsity = 0.98,
  min_occ = 100)


tab42
```
Ici, nous pouvons voir les termes spécifiques pour les journaux locaux et nationaux sur l'ensemble de la période. Les termes les plus spécifiques pour les journaux nationaux sont "atterrissage", "banque", "gazprom", "tête générale" et "administration". Pour les journaux locaux, les termes spécifiques sont "accident de voiture", "voitures", "soirée", "vkontakte" et "conducteurs". Cela s'explique par le fait que les journaux locaux accordent souvent une attention particulière aux accidents.

Voici les conclusions tirées de ces résultats concernant le statut des journaux :

Les journaux à l'échelle nationale se concentrent sur divers aspects liés à la location, aux banques et aux autorités. Cela peut indiquer une large gamme de sujets concernant l'économie, les finances et la politique à l'échelle nationale. "Gazprom" et "Directeur général" sont des termes clés, ce qui pourrait indiquer une activité et un intérêt pour ces sujets à l'échelle nationale.

Les journaux locaux se concentrent sur des événements et des sujets locaux tels que les accidents, les automobiles, les événements du soir, les réseaux sociaux et les conducteurs. Cela suggère que ces journaux sont axés sur la discussion des questions locales et des intérêts des résidents de zones spécifiques. L'absence du terme "Gazprom" pourrait indiquer que ce journal ne se concentre pas sur les sujets liés à cette grande entreprise nationale, mais plutôt sur des sujets plus locaux et régionaux.

En conclusion, l'analyse des termes clés permet de tirer des conclusions sur les domaines d'intérêt et la portée des journaux, reflétant leur orientation vers des sujets nationaux ou des questions locales.

```{r tree, echo=FALSE, message=FALSE, warning=FALSE}
#tree <- terms_graph(dtmsmo2l)
```

### Les termes co-occurrents avec le terme "кудрово" (Kudrovo) :


```{r cooccurrences1, echo=FALSE, message=FALSE, warning=FALSE}
tab5 <-cooc_terms(frenchdtmsmo_d2l,"koudrovo",
             p = 0.1,
  n = 25,
  sparsity = 0.98,
  min_occ = 100
)
tab5
kable(tab5,
      caption = "**Cooccurrences for the term Kudrovo**",
      align="lcc", escape = FALSE) %>%
  kable_styling(bootstrap_options = c("striped", "hover"), full_width = FALSE)%>%
  column_spec(3, background = "cadetblue")

#
```
Les résultats présentent un tableau montrant la fréquence des termes co-occurrents avec le terme "кудрово" (Kudrovo) dans différents textes ou documents. Chaque terme est accompagné du pourcentage de ses termes co-occurrents (en tant que fréquence relative) et du pourcentage de la relation des termes co-occurrents avec ce terme (en tant que rapport de fréquence).

Le terme "метро" (métro) apparaît dans 31,08% des cas avec "кудрово". Cela indique que "кудрово" en tant que quartier peut être lié au métro, peut-être qu'il y a une station de métro dans ce quartier ou qu'il a de bonnes liaisons de transport avec le métro.

Le terme "станция" (station) apparaît dans 45,16% des cas avec "кудрово". Cela indique également la présence possible d'une gare ferroviaire ou d'autres types de stations de transport dans le quartier "кудрово".

"Транспорт" (transport) apparaît dans 49,56% des cas avec "кудрово", ce qui confirme son importance dans le contexte du quartier et peut indiquer divers moyens de transport et infrastructures dans ce quartier.

Le terme "строительство" (construction) apparaît dans 74,80% des cas avec "кудрово". Cela peut indiquer une activité de construction importante ou un développement dans le quartier, peut-être de nombreux nouveaux projets et chantiers.

"Линии" (lignes) apparaît dans 17,45% des cas avec "кудрово". Ce terme peut se référer à différents types de lignes, telles que des itinéraires de transport ou des lignes de communication.

"Километры" (kilomètres) apparaît dans 13,51% des cas avec "кудрово", ce qui peut également être lié à différentes mesures de distance ou d'emplacement dans le quartier.

"Эксплуатация" (exploitation) apparaît dans 9,49% des cas avec "кудрово", cela pourrait se rapporter à l'exploitation de différents objets ou systèmes dans ce quartier.

"Улицы" (rues) apparaît dans 32,08% des cas avec "кудрово". Cela peut indiquer la présence d'un réseau de rues développé ou une activité active liée aux rues et aux routes dans le quartier.

Dans l'ensemble, les résultats mettent en évidence certaines thématiques clés et caractéristiques liées au quartier "кудрово". Il est important de noter que des valeurs spécifiques peuvent être soigneusement étudiées dans leur contexte pour une interprétation plus précise et une meilleure compréhension des aspects importants liés à "кудрово".


```{r}
tab5 <-cooc_terms(frenchdtmsmo_d2l,"koudrovo",
             p = 0.1,
  n = 25,
  sparsity = 0.98,
  min_occ = 100
)

kable(tab5,
      caption = "**Cooccurrences for the term Kudrovo**",
      align="lcc", escape = FALSE) %>%
  kable_styling(bootstrap_options = c("striped", "hover"), full_width = FALSE)%>%
  column_spec(3, background = "#8968CD")
```


### Les termes co-occurrents avec le terme "санктпетербург" (Saint-Petersbourg) :

```{r cooccurrences2, echo=FALSE, message=FALSE, warning=FALSE}
tab5 <-cooc_terms(dtmsmo_d2l,"спб",
             p = 0.1,
  n = 25,
  sparsity = 0.98,
  min_occ = 100
)

kable(tab5,
      caption = "**Cooccurrences for the term Saint-Petersbourg**",
      align="lcc", escape = FALSE) %>%
  kable_styling(bootstrap_options = c("striped", "hover"), full_width = FALSE)%>%
  column_spec(3, background = "cadetblue")

#
```

```{r}

tab5 <-cooc_terms(frenchdtmsmo_d2l,"spb",
             p = 0.1,
  n = 25,
  sparsity = 0.98,
  min_occ = 100
)

kable(tab5,
      caption = "**Cooccurrences for the term Saint-Petersbourg**",
      align="lcc", escape = FALSE) %>%
  kable_styling(bootstrap_options = c("striped", "hover"), full_width = FALSE)%>%
  column_spec(3, background = "#8968CD")

```



Les résultats présentent les termes co-occurrents avec le terme "санктпетербург" (Saint-Pétersbourg). Chaque terme est accompagné du pourcentage de ses co-occurrences (en tant que fréquence relative) et du pourcentage de la relation des co-occurrences avec ce terme (en tant que rapport de fréquence).

"Градостроительная компания" (société d'aménagement urbain) apparaît dans 9,63% des cas avec "санктпетербург". Cela peut faire référence à une entreprise ou une organisation portant le nom "GK" et étant associée à Saint-Pétersbourg. "девелопер" (développeur) apparaît dans 17,44% des cas avec "санктпетербург". Cela peut indiquer la présence et l'importance des développeurs immobiliers dans le contexte de Saint-Pétersbourg. "директор" (directeur) apparaît dans 23,37% des cas avec "санктпетербург". Cela peut faire référence aux directeurs d'entreprises, d'organisations ou d'institutions basés à Saint-Pétersbourg. "жилье" (logement) apparaît dans 42,20% des cas avec "санктпетербург". Cela peut indiquer que le terme "санктпетербург" est fréquemment associé au secteur du logement ou de l'immobilier. "квартиры" (appartements) apparaît dans 43,93% des cas avec "санктпетербург". Cela peut faire référence à l'importance des appartements ou du marché immobilier des appartements dans le contexte de Saint-Pétersbourg. "meétres carres" apparaît dans 33,28% des cas avec "санктпетербург". Il peut s'agir d'un terme spécifique lié à Saint-Pétersbourg, qui est fréquemment co-occurrence avec le terme "санктпетербург". "линии" (lignes) apparaît dans 17,99% des cas avec "санктпетербург". Cela peut faire référence à différentes lignes, peut-être des lignes de transport, des itinéraires ou des voies de communication dans la ville. "метро" (métro) apparaît dans 32,14% des cas avec "санктпетербург". Cela peut indiquer l'importance du système de métro dans la ville de Saint-Pétersbourg. "метрополитен" (métropolitain) apparaît dans 10,72% des cas avec "санктпетербург". Cela fait probablement référence au système de métro de Saint-Pétersbourg. "недвижимость" (immobilier) apparaît dans 26,10% des cas avec "санктпетербург". Cela confirme l'importance de l'immobilier dans le contexte de Saint-Pétersbourg. "объем" (volume) apparaît dans 19,53% des cas avec "санктпетербург". Cela peut se référer à différents volumes d'activités, peut-être liés à l'immobilier ou à d'autres aspects de la ville. "покупатели" (acheteurs) apparaît dans 24,20% des cas avec "санктпетербург". Cela peut indiquer que Saint-Pétersbourg est souvent associée à des acheteurs ou des consommateurs. "продажи" (ventes) apparaît dans 22,67% des cas avec "санктпетербург". Cela peut indiquer l'importance des ventes ou du commerce dans le contexte de la ville. "проект" (projet) apparaît dans 89,52% des cas avec "санктпетербург". Cela peut indiquer que le terme "санктпетербург" est fréquemment utilisé dans le contexte de différents projets. "развитие" (développement) apparaît dans 35,64% des cas avec "санктпетербург". Cela peut faire référence à l'importance du développement et de la croissance dans le contexte de la ville. "рынок" (marché) apparaît dans 45,25% des cas avec "санктпетербург". Cela indique l'importance du marché ou de l'économie dans le contexte de la ville. "смольный" (Smolny) apparaît dans 8,99% des cas avec "санктпетербург". Cela peut faire référence au Smolny, qui est un bâtiment historique important à Saint-Pétersbourg. "спрос" (demande) apparaît dans 18,61% des cas avec "санктпетербург". Cela peut indiquer l'importance de la demande dans le contexte de la ville. "станция" (station) apparaît dans 46,00% des cas avec "санктпетербург". Cela peut faire référence à différentes stations de transport ou de communication dans la ville. "трамвай" (tramway) apparaît dans 8,07% des cas avec "санктпетербург". Cela indique que le terme "санктпетербург" est souvent lié au système de tramway de la ville. "транспорт" (transport) apparaît dans 50,93% des cas avec "санктпетербург". Cela confirme l'importance du transport dans le contexte de la ville. "цены" (prix) apparaît dans 27,73% des cas avec "санктпетербург". Cela peut indiquer que les prix ou le coût des choses sont fréquemment associés à Saint-Pétersbourg. Enfin, les termes "всеволожск" (Vsevolozhsk) et "кудрово" (Kudrovo) ont également été examinés. Ils apparaissent respectivement dans 15,05% et 58,74% des cas avec "санктпетербург". Ces pourcentages indiquent que ces termes sont souvent co-occurrences avec "санктпетербург", ce qui peut indiquer des des relations avec la ville de Saint-Pétersbourg.

Les résultats de l'analyse des termes co-occurents avec "санктпетербург" (Saint-Pétersbourg) permettent de tirer les conclusions suivantes :

Importance du secteur immobilier : Les termes tels que "жилье" (logement) et "квартиры" (appartements) apparaissent fréquemment avec "санктпетербург", indiquant l'importance du secteur immobilier dans le contexte de la ville.

Référence à des acteurs et institutions : Les termes "директор" (directeur) et "градостроительная компания" (société d'aménagement urbain) indiquent la présence d'entreprises et d'organisations liées à Saint-Pétersbourg.

Transports et mobilité : Les termes "метро" (métro), "трамвай" (tramway) et "транспорт" (transport) mettent en évidence l'importance du système de transport et de mobilité dans la ville.

Projets de développement : Le terme "проект" (projet) est largement associé à "санктпетербург", ce qui suggère que la ville est fréquemment liée à divers projets de développement.

Économie et marché : Les termes "рынок" (marché) et "спрос" (demande) indiquent l'importance de l'économie et des échanges commerciaux dans le contexte de la ville.

Sites historiques importants : Le terme "смольный" (Smolny) fait référence à un bâtiment historique important à Saint-Pétersbourg.

Relation avec les villes voisines : Les termes "всеволожск" (Vsevolozhsk) et "кудрово" (Kudrovo) ont des co-occurrences significatives avec "санктпетербург", ce qui suggère des relations avec des villes voisines.

En résumé, l'analyse des co-occurrences des termes avec "санктпетербург" met en évidence des thèmes importants tels que l'immobilier, les transports, le développement de projets et l'économie, reflétant ainsi les aspects clés liés à la ville de Saint-Pétersbourg.

## Révision lexicaleс par chaque journal étudié

```{r lexical, echo=FALSE, message=FALSE, warning=FALSE}
tab8 <- lexical_summary(dtmsmo2l, corpus2, "Газета", unit = c("document", "global"))

kable(tab8,
      caption = "**Lexical review**",
      align="lcc", escape = FALSE) %>%
  kable_styling(bootstrap_options = c("striped", "hover"), full_width = FALSE)%>%
  row_spec(3, bold = T, background = "cadetblue")
```

Ces résultats représentent une revue lexicale qui couvre diverses sources de nouvelles et de journaux. Analysons certaines des principales mesures et tirons des conclusions :

Nombre de termes (Number of terms) : Chaque source de nouvelles contient des termes uniques, et le nombre total de termes peut être un indicateur de la diversité des caractéristiques lexicales. Par exemple, les sources "Российская Газета" et "РБК" ont le plus grand nombre de termes, ce qui peut indiquer une large couverture thématique dans leurs nouvelles.

Nombre de termes uniques (Number of unique terms) : Il s'agit du nombre de termes qui n'apparaissent que dans une seule source de nouvelles. Une source avec un grand nombre de termes uniques peut être plus diversifiée dans son lexique. Par exemple, la source "Коммерсант" a un nombre relativement faible de termes uniques, ce qui peut indiquer une thématique plus spécifique de ses nouvelles.

Pourcentage de termes uniques (Percent of unique terms) : Cette mesure reflète la proportion de termes uniques par rapport au nombre total de termes. Les sources ayant un pourcentage plus élevé de termes uniques ont généralement des sujets plus spécialisés ou uniques. La source "Бумага" a le pourcentage le plus élevé de termes uniques, ce qui peut indiquer une spécificité unique dans les nouvelles qu'elle publie.

Nombre d'hapax (Number of hapax legomena) : Il s'agit du nombre de termes qui n'apparaissent qu'une seule fois dans le texte. Une source avec un grand nombre d'hapax peut être plus diversifiée et contenir de nombreux termes uniques. Par exemple, la source "СПбДневник" a un nombre élevé d'hapax, ce qui peut indiquer une diversité de la composition lexicale de ses nouvelles.

De cette revue lexicale, on peut conclure que chaque source de nouvelles possède son propre lexique unique et peut couvrir différentes thématiques. Certaines sources, comme "Российская Газета" et "РБК", présentent un large éventail de sujets d'actualité, tandis que d'autres, comme "Коммерсант" et "СПбДневник", peuvent être plus spécialisées. Ces résultats peuvent aider à l'analyse ultérieure et à la compréhension des caractéristiques particulières de chaque source de nouvelles.


## L'Analyse Factorielle des Correspondances (AFC)

### AFC (terms)

L'Analyse Factorielle des Correspondances (AFC) permet d'étudier les liens entre les variables catégorielles dans de vastes ensembles de données, tels que des documents textuels. Lorsque nous traitons un corpus d'articles, chaque article peut être considéré comme un document, et chaque terme (mot ou expression) comme une variable catégorielle.

L'AFC permet de représenter les liens entre les articles et les termes dans un espace de plus petite dimension, ce qui facilite la visualisation des données. Cela peut aider à identifier les thèmes communs, les regroupements de termes ou d'articles, ainsi que les schémas dans les données.

De plus, l'AFC est utilisée pour regrouper les articles et les termes en fonction de leurs similarités. Cela permet de découvrir des groupes d'articles ayant des thèmes ou des contenus similaires, ainsi que des groupes de termes qui apparaissent souvent ensemble.

L'AFC peut également mettre en évidence les termes les plus importants et fréquents dans le corpus d'articles, ce qui peut être utile pour une analyse et une interprétation ultérieures des données. De même, elle peut aider à identifier les articles qui contribuent le plus aux thèmes communs ou aux variations dans les données. Cela peut être utile pour déterminer les articles les plus pertinents ou significatifs dans le corpus.

En résumé, l'AFC représente un puissant outil d'analyse pour les corpus d'articles, permettant de mettre en évidence les caractéristiques, les schémas et la structure des données, et facilitant l'interprétation des résultats.

Dans l'ACM (Analyse des Correspondances Multiples) du corpus d'articles sur Kudrovo, 35 documents ont été exclus en raison de l'absence de toute occurrence des termes conservés dans la matrice document-terme finale.

De plus, les variables portant les noms "numéro", "Zagolovok" (titre), "doc_id" et "doc_n" ont également été exclues car elles contiennent plus de 100 niveaux. Cela signifie que ces variables ont un nombre trop élevé de valeurs différentes et ne sont probablement pas adaptées pour être incluses dans l'analyse car elles pourraient entraîner une complexité excessive ou rendre l'interprétation des résultats difficile.

Le résultat obtenu est le suivant.


```{r restle, eval=FALSE, message=FALSE, warning=FALSE, include=FALSE}
library(explor)
library(tm)
library(FactoMineR)
resTLE<-corpus_ca(corpus_d,dtmsmo_d2,sparsity=0.98)
res <- explor::prepare_results(resTLE)
explor::CA_var_plot(res, xax = 1, yax = 2, lev_sup = FALSE, var_sup = FALSE,
    var_sup_choice = , var_hide = "Row", var_lab_min_contrib = 1, col_var = "Position",
    symbol_var = "Type", size_var = "Contrib", size_range = c(10, 300), labels_size = 10,
    point_size = 56, transitions = TRUE, labels_positions = NULL, xlim = c(-2.6,
        4.05), ylim = c(-2.75, 3.91))
```

```{r restle, eval=FALSE, message=FALSE, warning=FALSE, include=FALSE}
library(explor)
library(tm)
library(FactoMineR)
resTLE<-corpus_ca(corpus_d,frenchdtmsmo_d2l,sparsity=0.98)
res <- explor::prepare_results(resTLE)
explor::CA_var_plot(res, xax = 1, yax = 2, lev_sup = FALSE, var_sup = FALSE,
    var_sup_choice = , var_hide = "Row", var_lab_min_contrib = 1, col_var = "Position",
    symbol_var = "Type", size_var = "Contrib", size_range = c(10, 300), labels_size = 10,
    point_size = 56, transitions = TRUE, labels_positions = NULL, xlim = c(-2.6,
        4.05), ylim = c(-2.75, 3.91))
```
### Contribution à l'Axe 1
```{r contributive_docs1, echo=FALSE}
contributive_docs(corpus_d, resTLE, 2, ndocs = 10, nterms = 10)
  
```
Dans le contexte de l'Analyse Factorielle des Correspondances (AFC), le terme "Documents les plus contributifs" fait référence aux documents qui apportent la plus grande contribution à l'explication de la variation des données pour un axe spécifique. Les axes dans l'AFC représentent des facteurs qui expliquent les liens entre différentes variables ou catégories de données.

Lorsqu'on parle de "Documents les plus contributifs" sur un axe spécifique, cela signifie que ces documents ont une grande importance et informativité par rapport à cet axe. Ils contiennent des informations ou des caractéristiques qui ont un fort impact sur la distribution des données le long de cet axe. Ainsi, ces documents jouent un rôle clé dans l'explication des facteurs qui contribuent le plus à la répartition des variables ou des catégories de données sur cet axe dans l'AFC.

Les documents les plus significatifs du côté positif de l'axe 1 sont liés à la sécurité routière et à la prévention des accidents de la route. Ces documents contiennent des informations sur l'amélioration de la sécurité aux passages à niveau, l'organisation d'hébergements et de repas chauds pour les victimes, ainsi que des enquêtes sur les accidents de la route impliquant des minibus.

D'un autre côté, les documents du côté négatif de l'axe 1 sont liés au programme de réparation des routes et aux travaux en cours, qui peuvent temporairement restreindre la circulation. Ces documents reflètent davantage des informations sur les réparations et les restrictions qui ne sont pas aussi étroitement liées à la sécurité routière et à la prévention des accidents de la route que les documents du côté positif de l'axe 1.

Conclusion : Les documents du côté positif de l'axe 1 fournissent des informations plus importantes sur la sécurité routière et la prévention des accidents de la route, tandis que les documents du côté négatif de l'axe 1 sont davantage liés à la réparation des routes et aux restrictions temporaires de circulation.



### Contribution à l'Axe 2
```{r contributive_docs2, echo=FALSE}
contributive_docs(corpus_d, resTLE, 3, nterms = 10)
```

Les résultats de l'Analyse Factorielle des Correspondances (AFC) ont montré que du côté positif de l'axe 2 dans l'analyse du corpus d'articles sur Koudrovo, se trouvent des documents qui ont le plus d'influence sur cet axe. Ces documents abordent différents aspects liés aux accidents de la circulation (dorénavant DTC) et à la sécurité routière.

Plus précisément, ces documents contiennent des informations sur les aspects suivants :

L'amélioration de la sécurité aux passages à niveau.
L'organisation d'un abri et d'un repas chaud pour les personnes blessées.
Les conducteurs responsables d'accidents qui n'ont pas maîtrisé leur véhicule.
Les incidents impliquant des personnes blessées nécessitant l'intervention de la police et des services d'urgence.
Les accidents mortels impliquant des minibus et les enquêtes menées par les forces de l'ordre.
Les mesures d'enquête et les interrogatoires de témoins en relation avec les incidents.
La mention d'incidents survenus la veille au soir.
Ces résultats mettent en évidence l'importance de ces documents dans l'analyse du corpus d'articles sur la sécurité routière à Koudrovo, et fournissent des informations précieuses sur les événements liés aux DTC et à la sécurité routière dans cette région.

### Termes extrêmes de l'Axe 1
```{r extreme_docs2}
extreme_docs(corpus_d, resTLE, 2, ndocs = 10, nterms = 10)
```
Dans le contexte de l'Analyse Factorielle des Correspondances (AFC), le terme "Documents les plus extrêmes" se réfère aux documents qui exercent une influence majeure sur un axe d'analyse spécifique. Lors de l'AFC, les axes représentent les facteurs qui expliquent la relation entre différentes variables ou catégories de données. Chaque document (ou article) est représenté dans un espace multidimensionnel de variables, et sa position sur l'axe est déterminée par l'influence de différents facteurs.

Les documents les plus extrêmes du côté positif de l'axe 1 ont la plus grande corrélation ou similitude avec cet axe et exercent une influence significative sur sa direction. Cela signifie que ces documents sont fortement liés aux aspects spécifiques représentés par l'axe 1 et peuvent être essentiels pour comprendre les données dans ce contexte.

Les documents les plus extrêmes du côté positif de l'axe 1 sont liés à différents aspects de la sécurité routière et des accidents de la circulation. Ils couvrent les sujets suivants :

Sécurité aux passages à niveau : les documents indiquent la nécessité d'améliorer la sécurité aux passages à niveau, peut-être sur la base d'une analyse des incidents passés.

Hébergement et restauration pour les victimes : ces documents concernent probablement l'organisation de l'aide aux personnes impliquées dans des accidents ou des situations d'urgence.

Réparations routières et incidents liés aux accidents de la circulation : certains documents contiennent des informations sur les réparations planifiées ou en cours des routes, peut-être avec mention de zones accidentogènes.

Enquêtes sur les accidents mortels de la circulation et les infractions au code de la route : les documents sont liés aux enquêtes et aux vérifications impliquant les forces de l'ordre.

### Termes extrêmes de l'Axe 2
```{r extreme_doc3}
extreme_docs(corpus_d, resTLE, 3, ndocs = 10, nterms = 10)
```

Les documents du côté positif de l'axe 2 sont liés à la sécurité routière, à la prévention des accidents de la route et à l'aide aux victimes. Ils contiennent des informations sur différents aspects qui influencent cet axe et sont essentiels pour comprendre les données dans ce contexte.

D'autre part, les documents du côté négatif de l'axe 2 sont liés au programme de réparation des routes et aux restrictions temporaires de circulation en raison des travaux. Ces documents peuvent également avoir une incidence sur la répartition des données sur l'axe 2, mais ils présentent plutôt des informations sur les réparations et les restrictions qui ne sont pas aussi étroitement liées à la sécurité routière et aux accidents de la route que les documents du côté positif de l'axe 2.

```{r rainette1, message=FALSE, warning=FALSE, include=FALSE}
library(rainette)
library(quanteda)
dfm <- as.dfm(dtmsmo2l)
resrai <- rainette(dfm)
#dfm <- as.dfm(dtmsmo_2dl)
#rainette_explor(resrai, dfm)
```

```{r rainette1, message=FALSE, warning=FALSE, include=FALSE}
library(rainette)
library(quanteda)
#frenchdfm <- as.dfm(frenchdtmsmo2l)
#frenchresrai <- rainette(frenchdfm)
#dfm <- as.dfm(dtmsmo_2dl)
#rainette_explor(frenchresrai, frenchdfm)
```

```{r rainette1 plot, message=FALSE, warning=FALSE, echo=FALSE}
## Clustering description plot
rainette_plot(
  resrai, dfm, k = NULL,
  n_terms = 30,
  free_scales = TRUE,
  measure = c("chi2", "lr", "frequency", "docprop"),
  show_negative = TRUE,
  cluster_label = NULL,
  keyness_plot_xlab = NULL,
  text_size = 8
)
```


```{r rainette1 plot, eval=FALSE, message=FALSE, warning=FALSE, include=FALSE}
## Clustering description plot
rainette_plot(
  frenchresrai, dfm, k = NULL,
  n_terms = 30,
  free_scales = TRUE,
  measure = c("chi2", "lr", "frequency", "docprop"),
  show_negative = TRUE,
  cluster_label = NULL,
  keyness_plot_xlab = NULL,
  text_size = 8
)
```
Le programme a divisé toutes les données en 10 clusters. Les clusters sont des groupes de points de données qui se trouvent proches les uns des autres sur le graphique. Chaque cluster peut représenter un groupe ou une catégorie distincte d'éléments dans les données. Les clusters les plus importants sont le cluster 2 (638 éléments), le cluster 4 (490 éléments), le cluster 8 (487 éléments) et le cluster 9 (427 éléments).

Le premier cluster (n=1)  et le cluster 3 (n=61) contient des termes sans rapport avec le sujet du texte : des mots en anglais provenant d'annonces publicitaires et de bannières sur les sites web des journaux. Ces clusters n'ont donc pas non plus d'intérêt pour la recherche.

Le cluster 2 (n = 638) contient 638 articles. Les termes les plus fréquemment rencontrés dans ces articles sont : DPT (police routière), Conducteur, Blessé, Police, Homme, Urgence (événement d'urgence), Voiture, Arrêté, Incident, Témoins, Automobile, Criminel, Blessures, Été, Soirée, Enquêteur, Accident, Ministère de l'Intérieur, Hospitalisation, Policier, Découvert, Employée, Vérification, Femme, Camion, Collision, Suspect.Ces termes sont couramment utilisés pour décrire des articles liés à des accidents de la route impliquant des voitures, des conducteurs et la police.


Le Cluster 4 (n=490)contient les termes suivants : transport, oblast de Léningrad, autoroute, voie rapide, tramway, station, nœud, échangeur, ceinture périphérique est, routes, Kudrovo, ceinture périphérique ouest, gouverneur, route longitudinale, direction, fédérale, rues, routes, Saint-Pétersbourg, diamètre, nœud de transport intermodal, est, routes, routier, secteur, correspondance, construction, Drozdénko, Gazprom. Le cluster 4 contient des termes liés à l'infrastructure de transport et au développement du réseau de transport de la ville et de la région. On y retrouve des mots décrivant différents modes de transport, tels que les tramways et les voies rapides, ainsi que des éléments d'infrastructure tels que les stations, les intersections et les routes. Apparemment, ce cluster est axé sur les actualités et les mises à jour concernant le système de transport de Saint-Pétersbourg et de l'oblast de Léningrad, ainsi que sur les récits de projets et de développement de l'infrastructure du réseau de transport dans la région.

Le Cluster 5 (n=152) contient des termes liés au métro et à différents aspects de son fonctionnement. Dans ce cluster, on retrouve des mots décrivant les stations de métro, les lignes, les branches, les vestibules et autres constructions souterraines. On y trouve également des termes liés à la conception et au développement du métro, tels que les appels d'offres, les études géotechniques et les schémas. Les rapports sur les concours, les mises à jour et l'expansion du réseau de métro peuvent également être liés à ce cluster. Il semble que ce cluster soit intéressant pour l'analyse et l'étude des événements et du développement du métro à Saint-Pétersbourg et dans les environs.

Le Cluster 6 (n=46) contient les termes suivants : kilomètres, augmentation des prix, rénové, billet, passage, achèvement, pilote, électrique, IT, itinéraire, réparation, délais, exécutées, travaux, FAP, cartes, paiement, autobus, bus, bord, volume, coursier, caisses, remplacement, OPOL, Inonolov, utiliser, voie fluviale, organisations.

Le cluster 6 (n=46) contient des termes liés à l'infrastructure de transport et aux services dans la ville. Ce cluster comprend des mots liés aux transports en commun, tels que les bus, les billets de transport et les itinéraires. On y trouve également des termes liés à la réparation et à la mise à jour des véhicules de transport, tels que "отремонтированный" (rénové), "выполнены работы" (travaux effectués) et "объем ремонта" (volume des réparations).

Certains termes peuvent être liés à l'organisation et au paiement des services de transport en commun, tels que "оплата" (paiement), "карты" (cartes) et "организации" (organisations).

Les termes comme "километры" (kilomètres) et "пилотный" (pilote) pourraient indiquer un suivi et un contrôle des itinéraires.

Le cluster 6 pourrait également contenir des informations sur les délais, les coursiers (probablement liés aux transports de marchandises), les transports fluviaux et les postes de secours et d'accouchement.

Cependant, pour une compréhension plus précise du contenu du cluster et de son rôle dans le contexte général de l'étude, une analyse supplémentaire et une contextualisation des termes dans les articles peuvent être nécessaires.

Le cluster 7 (n=8) contient des termes tels que "svod pravil" (code de règles), "genplan" (plan directeur), "okhrannaya zona" (zone de protection), "stomatologiya" (dentisterie), "kabinet" (cabinet), "zapis'" (inscription), "gosudarstvennoye byudzhetnoye uchrezhdeniye zdravookhraneniya" (établissement budgétaire d'État pour les soins de santé), "shestv" (défilé), "kupel'" (bain), "adres" (adresse), "bessmertnyy" (immortel), "vremennyye zdaniya" (bâtiments temporaires), "polk" (régiment), "Lenin", "prazdnik" (fête), "patsiyent" (patient), "svyatoy" (saint), "munitsipal'nyy" (municipal), "Lenobl'" (oblast de Leningrad), "ulitsy" (rues), "bratskiy" (fraternel), "Luga" (Luga), "Marx", "vozlozheniye" (dépôt), "Karl", "poselok" (settlement), "Priozerskiy" (Priozersk), "klinika" (clinique), "zakhoroneny" (enterré).

Ces termes semblent être liés à différents aspects médicaux, architecturaux et historiques. Dans ce cluster, on trouve des termes liés aux services médicaux tels que "stomatologiya" (dentisterie), "kabinet" (cabinet) et "patsiyent" (patient).

On y retrouve également des termes qui pourraient être liés à des plans architecturaux et d'aménagement urbain, comme "genplan" (plan directeur) et "okhrannaya zona" (zone de protection).

Certains termes, tels que "shestv" (défilé), "prazdnik" (fête) et "vozlozheniye" (dépôt), pourraient être liés à des événements et des célébrations culturelles. En se basant sur le contenu du Cluster 7, on peut conclure que le mécanisme de regroupement a identifié un lien entre les termes liés à la santé (comme "stomatologiya" - stomatologie) et les commémorations nationales (telles que "bessmertnyy polk" - le régiment immortel, "vozlozheniye tsvetov k mogilam" - le dépôt de fleurs sur les tombes, etc.). Malgré cela, on peut conclure que la question de la santé est importante pour la ville.

Le Cluster 8 (n=487) contient des termes tels que Tosno, agglomération, formation administrative, locaux, problèmes, résidents, tribunal, équipes, stade, candidat, loi, ainsi que des termes marqués en rouge : quartier, complexe, volume, résidentiel, hypothèque, métro, Saint-Pétersbourg, demande, projet, ventes, prix, logement, appartements, station, marché.

Le Cluster 8 représente un ensemble de termes variés qui peuvent être liés à différents aspects de la vie urbaine et du développement. Les termes marqués en bleu, tels que "тосно" (Tosno), "поселение" (établissement), "административное образование" (unité administrative), "местные" (locaux), "проблемы" (problèmes), "жители" (résidents), "суд" (tribunal), "команды" (équipes), "стадион" (stade), "кандидат" (candidat) et "закон" (loi), peuvent indiquer des questions liées à la gestion locale, aux résidents et aux aspects juridiques de la ville.

Les termes marqués en rouge, tels que "квартал" (quartier), "комплекс" (ensemble immobilier), "объем" (volume), "жилые" (résidentielles), "ипотека" (hypothèque), "метро" (métro), "санкт-петербург" (Saint-Pétersbourg), "спрос" (demande), "проект" (projet), "продажи" (ventes), "цены" (prix), "жилье" (logement), "квартиры" (appartements), "станция" (station) et "рынок" (marché), sont liés aux aspects de l'infrastructure résidentielle et du marché immobilier de la ville.

Ainsi, le Cluster 8 peut refléter différents aspects de la vie urbaine, y compris la gestion locale, les questions juridiques, les événements sportifs, ainsi que le développement du secteur résidentiel et immobilier de la ville. La couleur rouge des barres indique que ces termes ont une valeur de clé négative (selon l'indicateur statistique), ce qui peut indiquer leur faible spécificité pour ce cluster. D'autre part, la couleur bleue des barres indique que les termes sont considérés comme les plus spécifiques et caractéristiques de ce cluster (avec une valeur de clé positive). Ce cluster suscite un intérêt pour des recherches et des analyses plus approfondies afin de déterminer plus précisément son contenu et les liens entre les termes.

Le Cluster 9 (n=427) contient les termes suivants : "рынок" (marché), "квартира" (appartement), "жилье" (logement), "квадратные метры" (mètres carrés), "цены" (prix), "недвижимость" (immobilier), "покупатель" (acheteur), "продажи" (ventes), "застройка" (construction), "жилые" (résidentielles), "спрос" (demande), "ипотека" (hypothèque), "жилой комплекс" (ensemble résidentiel), "директор" (directeur), "комфорт" (confort), "квартал" (quartier), "аренда" (location), "класс" (classe), "компания" (entreprise), "градостроительная компания" (société d'aménagement urbain), "объекты" (projets), "метры" (mètres), "девелопер" (promoteur immobilier), "сегмент" (segment) et "объем" (volume) (marqués en bleu), ainsi que "транспорт" (transport), "улицы" (rues), "Кудрово" (Koudrovo), "станция" (station) et "Ленобласть" (oblast de Léningrad) (marqués en rouge).

Le cluster 9 est principalement axé sur les termes liés au marché immobilier et à l'habitat. Les termes marqués en bleu peuvent indiquer des questions liées aux prix, aux ventes, aux acheteurs et aux dimensions des propriétés, ainsi qu'aux projets résidentiels, à la demande et aux hypothèques. Ces termes suggèrent que le cluster 9 pourrait être lié à l'analyse du marché immobilier résidentiel, des tendances d'achat, des projets de développement et des préférences des acheteurs en termes de taille et de confort des logements.

D'un autre côté, les termes marqués en rouge, tels que "транспорт" (transport), "улицы" (rues), "Кудрово" (Koudrovo), "станция" (station) et "Ленобласть" (oblast de Léningrad), pourraient indiquer que ce cluster est également lié aux aspects de l'accessibilité, de la localisation géographique et de l'infrastructure de transport dans la région de Léningrad.

En somme, le Cluster 9 semble être un groupe de termes diversifié, lié à l'immobilier résidentiel, à l'économie du marché immobilier, ainsi qu'à l'accessibilité et à la localisation géographique des propriétés dans la région de Léningrad. Une analyse plus approfondie de ces termes pourrait aider à mieux comprendre les relations et les implications de ce cluster dans le contexte de développement urbain et immobilier de la région.

Le Cluster 10 (n=35) contient les termes suivants : "баллы" (points), "рейтинг" (classement), "домофон" (interphone), "смс" (SMS), "долгострой" (construction inachevée), "билайн" (Beeline, nom d'une compagnie de télécommunications), "подмосковье" (région de Moscou), "тюмень" (Tioumen, nom d'une ville), "набрать" (composer un numéro), "рассылка" (diffusion), "мбит" (mégabit), "реут" (REUT, abréviation pour l'agence de presse russe), "геленджик" (Guelendjik, nom d'une ville), "занятие" (activité), "шкалы" (échelles), "дружелюбный" (convivial), "шахтинский" (chakhtinskii, nom géographique), "сек" (seconde), "каменский" (kamenskii, nom géographique), "порта" (port), "мтс" (MTS, nom d'une compagnie de télécommunications), "кисилев" (Kisselev, nom de famille), "строчка" (ligne), "ейск" (Eisk, nom d'une ville), "вымпел" (fanion), "город" (ville).

Le cluster 10 semble contenir un mélange de termes liés aux télécommunications, à la construction, aux activités géographiques et aux aspects techniques. Les termes tels que "баллы" (points) et "рейтинг" (classement) peuvent être liés à des évaluations ou à des notations, tandis que "домофон" (interphone) et "смс" (SMS) suggèrent des aspects de communication et de technologie.

D'autres termes tels que "долгострой" (construction inachevée), "подмосковье" (région de Moscou) et "тюмень" (Tioumen) peuvent être liés à des projets de construction ou à des aspects géographiques spécifiques.

En outre, la présence de termes géographiques tels que "геленджик" (Guelendjik), "шахтинский" (chakhtinskii), "каменский" (kamenskii), "ейск" (Eisk) indique que ce cluster pourrait être lié à des lieux spécifiques.

En résumé, le Cluster 10 semble être un groupe de termes hétérogène, lié à différents domaines tels que les télécommunications, la construction, la géographie et les aspects techniques.

En conclusion, l'analyse des 10 clusters a permis d'identifier plusieurs domaines d'intérêt dans les actualités de la région de Léningrad. Ces domaines comprennent les accidents de la route, l'infrastructure de transport, le métro, les services de transport en commun, la santé, le développement urbain, le marché immobilier, les aspects économiques et les services de télécommunications. Chaque cluster représente un groupe de termes liés à un sujet spécifique et peut être utilisé pour une analyse plus approfondie des tendances et des développements dans la région. 


