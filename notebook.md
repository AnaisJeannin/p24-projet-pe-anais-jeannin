 # NOTEBOOK 

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
```

```python
cities = pd.read_csv('cities.csv')
countries = pd.read_csv('countries.csv')
weather = pd.read_csv('daily-weather-cities.csv')
```

## Etude de la table daily-weather-cities.csv

```python
weather.head() #on regarde quelles sont les informations dans la table
```

```python
print('On étudie les villes :' , weather['city_name'].unique()) #on regarde combien de villes sont concernées par les données
Pays = ['Vienna', 'Brussels', 'Paris', 'Berlin']
print('Les saisons présentes dans la table sont :' , weather['season'].unique())
```

On peut commencer par tracer quelques boxplots des colonnes de températures min et max en regroupant par villes. Dans l'ordre : celui pour toutes saisons confondues, celui pour l'Hiver, celui pour le Printemps, celui pour l'Ete et celui pour l'Automne.

```python
weather.boxplot(['min_temp_c', 'max_temp_c'], by='city_name')
```

```python
weather_Winter = weather.loc[weather['season'] == 'Winter'] #création sous-tableau pour l'hiver
weather_Winter.boxplot(['min_temp_c', 'max_temp_c'], by = 'city_name')
```

```python
weather_Spring = weather.loc[weather['season'] == 'Spring'] #création sous-tableau pour le printemps
weather_Spring.boxplot(['min_temp_c', 'max_temp_c'], by = 'city_name')
```

```python
weather_Summer = weather.loc[weather['season'] == 'Summer'] #création sous-tableau pour l'été
weather_Summer.boxplot(['min_temp_c', 'max_temp_c'], by = 'city_name')
```

```python
weather_Autumn= weather.loc[weather['season'] == 'Autumn'] #création sous-tableau pour l'automne
weather_Autumn.boxplot(['min_temp_c', 'max_temp_c'], by = 'city_name')
```

Les résultats de ces premières exploitations ne tiennent pas compte de l'évolution temporelle des données. On peut essayer de regarder par ville comment la température a évolué au cours des années. 

```python
weather_Vienna = weather.loc[weather['city_name'] == 'Vienna'] #création sous-tableau pour la ville de Vienne
weather_Vienna_Winter = weather_Vienna.loc[weather_Vienna['season'] == 'Winter']
plt.plot(weather_Vienna_Winter['date'],weather_Vienna_Winter['min_temp_c'])
```

```python
weather_Vienna_Winter['snow_depth_mm'].plot(kind = 'bar')
```

Les derniers graphiques sont longs à tracer et pas très lisible. Il faudrait chercher une autre solution pour y voir plus clair.


## Etude de la table cities.csv

```python
cities.head()
```

Localisation des villes

```python
cities.plot.scatter(x='longitude', y ='latitude')
```

Le data set a l'air assez complet car on voit presque la carte du monde se tracer. On remarque déjà que certaines régions sont soit très peu peuplées soit laissées de côté par l'étude (ex: Amazonie).


On peut regarder si les villes d'un pays sont plus représentées que celles des autres pays.

```python
nb_villes_par_pays = pd.DataFrame(cities['country'].value_counts(), index = cities['country'].unique())
nb_villes_par_pays = nb_villes_par_pays.sort_values(by = 'count', ascending = False)
nb_villes_par_pays.head(30).plot(kind = 'bar')
```

On trace le graphique seulement pour les 30 pays dont les villes sont les plus représentées afin que la légende reste lisible.
On peut s'apercevoir que pour certains pays, on a beaucoup plus de villes que pour d'autres pays.

```python
nb_villes_par_pays.boxplot()
```

Les villes représentées sur le diagramme en barre sont des exceptions. Pour la plupart des pays, on a moins de 10 villes.

```python
print('Le nombre moyen de villes par pays dans ces données est:' , nb_villes_par_pays.mean())
```

La Russie est le pays pour lequel on a le plus de villes. On peut donc s'intéresser à son cas. Regardons si les villes se situent à des endroits très différents ou non.

```python
cities_Russia = cities.loc[cities['country'] == 'Russia']
cities_Russia.plot.scatter(x='longitude', y='latitude')
print('La latitude moyenne pour les villes en Russie de ce data set est:',cities_Russia['latitude'].mean(),'et la longitude moyenne:', cities_Russia['longitude'].mean())
```




On ne voit pas de schéma particulier appparaitre. Si ce n'est que beaucoup de villes se situent à une longitude entre 40 et 60 et à une latitude entre 50 et 60.
A priori, les conditions météorologiques et géographiques n'interviennent pas trop dans le "choix" du lieu de développement de villes.
On peut faire la même étude pour un territoire accidenté, par exemple le Chilie, pour voir si un schéma apparaît cette fois-ci.

```python
cities_Chile = cities.loc[cities['country'] == 'Chile']
cities_Chile.plot.scatter(x='longitude', y='latitude')
print('La latitude moyenne pour les villes en Chile de ce data set est:',cities_Chile['latitude'].mean(),'et la longitude moyenne:', cities_Chile['longitude'].mean())
```

L'étude n'est pas beaucoup plus concluante.


## Etude de la table countries.csv

```python
countries.head()
```

On peut accéder rapidement au pays le plus peuplé et à celui le plus grand.

```python
countries = countries.sort_values(by = 'population', ascending = False)
print('le pays le plus peuplé est', countries.iloc[0,0])
countries = countries.sort_values(by = 'area', ascending = False)
print('le pays le plus grand est', countries.iloc[0,0])
```

On peut remarquer que si on affiche la table countries.csv après chaque sort_values, elle est classée du pays le plus peuplé (ou grand dans le deuxième cas) au moins peuplé (grand).


On peut accéder à la densité moyenne en population des différents pays et donc classer les pays du plus au moins dense en population.

```python
countries.set_index('country', inplace = True)
countries['densité'] = countries['population']/countries['area']
countries = countries.sort_values(by = 'densité', ascending = False)
print('les pays les plus denses sont :', countries['densité'].head(10))
```

```python
print(countries['area'].isna().value_counts())
print(countries['densité'].isna().value_counts())
print(countries['population'].isna().value_counts())
```

On vérifie ainsi qu'on a les données pour la majorité des pays et que donc les observations veulent dire quelque chose.

On peut aussi vouloir des données plus générales concernant la densité en population.

```python
countries.boxplot('densité')
print("Ce graphique est trop applati pour qu'on lise quoique ce soit.")
print('valeur moyenne de la densité en population', countries['densité'].mean())
```

Une solution pour avoir un boxplot lisible est de représenter le logarithme de la densité.

```python
countries['densité log'] = np.log(countries['densité'])
countries.boxplot('densité log')
```

On peut aussi vouloir connaitre le pays le plus peuplé et le plus grand de chaque continent. On rangera ces informations dans des dictionnaires.

```python
pays_le_plus_peuplé_par_continent = {}
pays_le_plus_grand_par_continent = {}
#print(countries['continent'].unique()) problème à cause de la valeur nan.
liste_continents = ['Asia', 'Europe', 'Africa', 'Oceania', 'North America','South America', 'Antarctica'] #je crée donc une liste des continents pour éviter le problème

for continent in liste_continents  :
        countries_by_continent = countries.loc[countries['continent'] == continent]
        countries_by_continent = countries_by_continent.sort_values(by = 'population', ascending = False)
        pays_le_plus_peuplé_par_continent[continent] = countries_by_continent.iloc[0, 0]
        countries_by_continent = countries_by_continent.sort_values(by = 'area', ascending = False)
        pays_le_plus_grand_par_continent[continent] = countries_by_continent.iloc[0, 0]
    
print('Pays le plus peuplé par continent', pays_le_plus_peuplé_par_continent)
print('Pays le plus grand par contient', pays_le_plus_grand_par_continent)
    
```

On peut regarder où sont situées les capitales.

```python
countries.plot.scatter(x='capital_lng' , y ='capital_lat')
```
