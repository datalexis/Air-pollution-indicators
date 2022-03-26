# OVERVIEW #

This work takes part inside a european project in order to determine elements available to make a outdoor pollution map at an european level. 
 
Experts has build an index for this work in relation with the international air quality index. This index give a air quality classes (good, fair, moderate, poor, very poor, extremely poor) based on pollutant-specific ranges for each 5 key pollutants (particles < 2.5 µm, particles < 10 µm, nitrogen dioxide, ozone, sulfur dioxide).

In addition, they have selected radon potential as indicator for soil pollution.

Air quality consultants need data analysis skills to structure and vizualize data they have.


## EXIGENCES ##

- multi-pollutants
- using open source databases
- regularly updated

## DATA SOURCES ##

Radon database (FR_radon.csv) : https://www.data.gouv.fr/fr/datasets/connaitre-le-potentiel-radon-de-ma-commune/#resources
French postal code (FR_laposte_hexasmal.csv) : https://datanova.laposte.fr/explore/dataset/laposte_hexasmal

Air quality database (StatisticTable.csv) : https://www.eea.europa.eu/data-and-maps/data/airbase-the-european-air-quality-database-8

More recent data are available on https://www.eea.europa.eu/data-and-maps/data/aqereporting-9

### DATA EXTRACTION ###

The public metadata files from https://www.eea.europa.eu/data-and-maps/data/aqereporting-8#tab-european-data have been used to get the key statistics for every measurements stations in the concerned countries and for each measured pollutants. Some of the stations recover data daily, other hourly. For each sampling, the mean and the median are available. A Python code was used to get for every year, station and pollutant these statistics and agregate them into a single .csv file. This file  were the input data used to build the interactive map described in the next sub-section.  

### DATA WRANGLING ###

Une fois que les données brutes ont été extraites par le code Python, l’étape suivante est le data wrangling. Le data wrangling est le processus qui permet de découvrir, structurer, nettoyer, enrichir et valider les données brutes afin de pouvoir publier les résultats dans un format adapté à l’analyse des données. Pour réaliser cette étape, nous utilisons des outils ETL (Extract-Transform-Load) qui sont des logiciels spécialisés dans la mise en forme des données. L’outil ETL utilisé dans ce projet est Tableau Prep qui est disponible avec Tableau Software.

Pour ce projet, les données brutes extraites sont répartis dans plusieurs fichiers selon leur catégorie (caractéristiques des stations météo, données enregistrées par les stations météo)  et leur pays (France, Espagne, Portugal). Nous commençons par fusionner les fichiers de même catégorie des différents pays ensemble et à supprimer les colonnes inutiles dans les jeux de données.

A partir des données enregistrées par les stations météo, nous calculons la valeur moyenne de chaque polluant EAQI par station météo et par année. Nous classons ensuite ces valeurs selon les étiquettes de l’indicateur EAQI (GOOD, POOR, etc.). 

A partir de ces valeurs, nous récupérons la plus défavorable par station météo et par année pour en tirer la valeur de l’index EAQI. Ces informations sont ensuite ajoutées dans le jeu de données des caractéristiques des stations météo.

Nous fusionnons la liste des communes françaises avec le jeu de données du potentiel radon par commune. Le problème principal apparu lors du data wrangling est que l’index EAQI est relié à une année mais pas le potentiel radon. Partant du principe que le potentiel radon n’évolue pas d’une année sur l’autre, nous affectons le potentiel radon de chaque commune à chaque année rattachée à l’index EAQI et nous ajoutons ces données dans le jeu de données des caractéristiques des stations météo en les reliant via le code postal de la commune.

A l’issue de ce travail, nous obtenons donc un fichier avec l’ensemble des stations météo et leurs caractéristiques (coordonnées, valeurs annuelles EAQI, potentiel radon) et un fichier avec les données mesurées par ces stations.

### DATAVIZUALIZATION ###

Avec Tableau Software, nous importons ces deux fichiers et nous les mettons en relation à travers leurs colonnes communes : station météo et année. 

Dans une première vue, nous créons une première couche en format carte sur laquelle nous affichons les communes avec une couleur correspondant à leur potentiel radon. Nous créons ensuite une deuxième couche en format cercle, par-dessus la première , qui indique la localisation des stations météo sur la carte et avec l’index EAQI en code couleur des cercles et le nombre de polluants EAQI détectés comme taille des cercles. Nous ajoutons enfin l’année comme filtre sur la carte.

Dans une seconde vue, nous utilisons le jeu de données enregistrées par les stations météos pour afficher dans un histogramme la valeur mesurées de chaque polluant. Nous créons pour l’occasion une famille permettant de distinguer les polluants en deux groupes : les polluants de l’EAQI et les autres. Le code couleur de l’histogramme reprend ensuite le même code couleur basé sur l’index EAQI que sur la carte.

Dans la première vue, nous paramétrons pour que la seconde vue s’affiche en infobulle lorsque nous passons notre curseur sur les stations météo affichées sur la carte. Tableau réalise ensuite automatiquement le même filtrage sur la seconde vue.

Nous créons enfin un tableau de bord qui permet de mettre en forme le titre du document et ses filtres.
