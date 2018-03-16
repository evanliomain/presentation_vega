# Récupérer des données
* Chercher des données sur open data ratp
* Choisir [trafic-annuel-entrant-par-station-du-reseau-ferre-2016](https://data.ratp.fr/explore/dataset/trafic-annuel-entrant-par-station-du-reseau-ferre-2016/api/?rows=369&sort=-rang&dataChart=eyJxdWVyaWVzIjpbeyJjb25maWciOnsiZGF0YXNldCI6InRyYWZpYy1hbm51ZWwtZW50cmFudC1wYXItc3RhdGlvbi1kdS1yZXNlYXUtZmVycmUtMjAxNiIsIm9wdGlvbnMiOnt9fSwiY2hhcnRzIjpbeyJ0eXBlIjoiYXJlYXNwbGluZXJhbmdlIiwiZnVuYyI6IkNPVU5UIiwieUF4aXMiOiJyYW5nIiwic2NpZW50aWZpY0Rpc3BsYXkiOnRydWUsImNvbG9yIjoiI2ZjOGQ2MiIsImNoYXJ0cyI6W3siZnVuYyI6Ik1JTiIsInlBeGlzIjoicmFuZyJ9LHsiZnVuYyI6Ik1BWCIsInlBeGlzIjoicmFuZyJ9XX1dLCJ4QXhpcyI6InJlc2VhdSIsIm1heHBvaW50cyI6NTAsInNvcnQiOiIifV0sInRpbWVzY2FsZSI6IiJ9)
* Récupérer les données via l'[api](https://data.ratp.fr/api/records/1.0/search/?dataset=trafic-annuel-entrant-par-station-du-reseau-ferre-2016&rows=369&sort=-rang&facet=reseau&facet=station&facet=ville)
* Nettoyer les données avec [jmespath](http://jmespath.org/) et `records[].fields.{value:trafic,label:station,rank:rang}`

# Créer un bar chart
Question : Quelle est la répartition des stations suivant leur trafic ?

* Créer une coquille de spec vega
* Ajouter les données `source`
* Créer une source de donnée `table`
  * ajouter un champ `quantity` égale à 1
  * trier sur `value`
  * faire des tas de données avec `bin` sur `value`, entre 100 et 10 000 000, maximum 10 paquets
  * aggréger sur `bin0` et compter (`sum`) les record (`quantity`) dans le champ `nb`
* Ajouter une échelle x ('xScale') sur `bin0`
* Ajouter une échelle y ('yScale') sur `nb`
* Ajouter 2 axes liés à ces 2 échelles
  * Formater l'axe x avec `,.0s` dans `encode.labels.update.text`
  * Ajouter labelBound pour ne pas avoir trop de valeur
* Ajouter les bars
  * y = yScale(nb)
  * y2 = yScale(0)

# Créer une répartition de point
* Séparer la source table en pack/table
* Dans pack, ajouter un champ `quantity` égale à 1
* Créer une `stack` des données `pack` sur `quantity`, groupBy `bin0`, offset `center`
* Enlever l'axe y
* Remplacer les `rect` par des cercles :
  * x = xScale(bin0)
  * y = yScale(y0)
* Ajouter un petit effet `hover` en changant la couleur du cercle en rouge

# Créer un Wordcloud
* Désactiver l'option autoparse car l'algo wordcloud est gourmand
* Supprimer les `transform`, `scales`, `axes` et `marks`
* Ajouter un signal `rotate` de valeur
* Ajouter un champ `rotate` de valeur `[-rotate, 0, rotate][~~(random() * 3)]`
* Effectuer un `wordcloud`
  * "size": [{"signal": "width"}, {"signal": "height"}],
  * "text":{"field":"label"},
  * "fontSize": {"field": "value"},
  * "fontSizeRange": [10, 56],
  * "fontWeight": "normal",
  * "rotate": {"field": "rotate"}
* Ajouter une `mark` text avec :
  * "text":{"field":"label"},
  * "align": {"value": "center"},
  * "baseline": {"value": "alphabetic"},
  * "x": {"field": "x"},
  * "y": {"field": "y"},
  * "angle": {"field": "angle"},
  * "fontSize": {"field": "fontSize"},
  * "fontWeight": {"field": "weight"}
* Jouer avec height et width général pour afficher + ou - de station
