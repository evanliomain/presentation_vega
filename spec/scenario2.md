# Récupérer des données
* Chercher des données sur
  * http://data.un.org/
  * Environment Statistics Database, UNSD
  * Total population served by municipal waste collection
  * http://data.un.org/Data.aspx?d=ENV&f=variableID%3a1878

# Créer un multi-line chart
Question : Quelle est l'évolution de la collecte de déchets par le monde ?

* Créer une coquille de spec vega
* Ajouter les données `source`
* Créer une source de donnée `table`
* Ajouter une échelle lineaire ('xScale') sur `year`, avec zero non compris
* Ajouter un axe sur xScale
* Ajouter une échelle lineaire ('yScale') sur `value`
* Ajouter un axe sur yScale
* Ajouter une échelle ordinal ('colorScale') sur `area`, avec une range "scheme": "category20"
* Ajouter une légende fill sur colorScale, à droite

* Ajouter une mark group avec
  * une facet regroupé sur area
  * une sous mark line et définir x, y, stroke avec les échelles et données associées
