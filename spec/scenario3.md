# Récupérer des données
* Chercher des données sur open data paris
  * [Recherche open data paris](http://fr.lmgtfy.com/?q=open+data+paris)
  * Menu "Les données"
  * Thème : Mobilité espace public
  * Mot clé : emplacement
  * Choisir: Stationnement sur voie publique - emplacements

# Charger les données
* Ajouter un data 'source' et charger depuis l'[url github](https://raw.githubusercontent.com/evanliomain/presentation_vega/master/data/stationnement-voie-publique-emplacements.csv)
* Spécifier le format dsv, delimiter ;
* Formatter en nombre les champs Largeur, Longueur, Nbre_de_places

# Question
Quelle est la répartition du type de stationnement sur Paris ?

# Aggréger les données
* Créer une donnée enrichie
  * Calculer la surface de chaque stationnement
* Créer une donnée table et aggréger sur Regime_particulier
  * Sommer Nbre_de_places et Surface
  * Filtrer pour enlever les records sans place (Nbre_de_places == 0)
* Calculer les angles pour réaliser un camembert
  * Ajouter une source de donnée pie avec sa transformation
  * Aggréger sur le champ Nbre_de_places
  * trier

# Ajouter un camembert
* Ajouter un scale de couleur category20
* Ajouter une mark arc sur la data pie
  * enter
    * Placer x et y au centre de la zone de travail
    * Colorier avec la scale
  * update
    * Ajouter startAngle et endAngle en passe plat
    * Ajouter outerRadius à `width / 3`
* Ajouter une légende sur la couleur (fill: colorScale)

* Jouer en remplaçant dans la construction de pie, le champ Nbre_de_places par Surface

# Convertir le camembert en radial-camembert
* Supprimer la légende (sinon le placement se fait mal)
* Ajouter une scale logarithmique 'radialScale'
  * sur le Nbre_de_places
  * vers la range : `[width / 8, width / 3]`

# Question
Quel stationnement est le plus efficace ?

# Convertir en scatter plot
* Réaliser un scatter plot
  * En abscisse la Surface
  * En ordonné le nombre de place
  * la couleur en fonction du type
Pour ce faire
* Supprimer la data pie, la mark et la radialScale
* Ajouter une xScale linéaire sur Surface
* Ajouter une yScale linéaire sur Nbre_de_places
* Placer les points, les colorier
* Ajouter les axes x et y : Surface / Nombre de place
* Observer
* Remplacer les scale x et y de `linear` à `log`

# Et en vrai ?
* Calculer la Surface par nombre de place (`Surface_par_nb_place`)
* Remplacer Surface par Surface_par_nb_place
* Repasser xScale en linear
* Légende axe x Surface par nombre de place

# Question
Quels arrondissements favorisent quel type de stationnement ?

* Regrouper les données en 7 catégories :
  * Vélos
  * Velib
  * Motos
  * Voiture
  * Handicapé
  * Autolib
  * ZL
  * Autres

Pour se faire, récupérer toutes les catégories en tapant dans la console
`VEGA_DEBUG.view.data('table').map(x => x.Regime_particulier)`

Associer la catégorie dans la data `enrichie`
```javascript
datum.t === 'Motos' || datum.t === 'Vélos' || datum.t === 'Velib' ? datum.t : datum.t === 'GIG-GIC Elargie' || datum.t === 'GIG-GIC Standard' ? 'Handicapé' : datum.t === 'ZL periodique' || datum.t === 'ZL permanente' ? 'ZL' : datum.t === 'Rotatif' || datum.t === 'Mixte' || datum.t === 'Gratuit' || datum.t === 'Taxi gaine autorisée' || datum.t === 'Taxi gaine interdite' ? 'Voiture' : datum.t === 'Autolib2' || datum.t === 'Autolib1' ? 'Autolib' : 'Autres'
```

* Réaggréger les données sur `category` et `Arrondissement` et filter les données sur le `Nbre_de_places`
```json
    {
      "name": "category",
      "source": "enrichie",
      "transform": [
        {
          "type": "aggregate",
          "groupby": [
            "category", "Arrondissement"
          ],
          "fields": [
            "Nbre_de_places",
            "Surface"
          ],
          "ops": [
            "sum",
            "sum"
          ],
          "as": [
            "Nbre_de_places",
            "Surface"
          ]
        },
        {
          "type": "filter",
          "expr": "datum.Nbre_de_places !== 0"
        }
      ]
    }
```
* Ajouter une donnée normalisante
```json
{
  "name": "general",
  "source":"category",
  "transform": [
    {"type": "formula", "expr": "datum.category", "as": "stk1"},
    {"type": "formula", "expr": "datum.Arrondissement", "as": "stk2"},
    {"type": "formula", "expr": "datum.Nbre_de_places", "as": "size"}
  ]
}
```
* Adapter le code du diagramme de Sankey présenté pour Kibana : https://nyurik.github.io/Vega-Sankey-Graph-for-Kibana
  * Ajouter les nodes
    ```json
    {
      "name": "nodes",
      "source": "general",
      "transform": [
        {
          "type": "filter",
          "expr": "!groupSelector || groupSelector.stk1 == datum.stk1 || groupSelector.stk2 == datum.stk2"
        },
        {
          "type": "formula",
          "as":"key",
          "expr": "datum.stk1+datum.stk2"

        },
        {
          "type": "fold",
          "fields":["stk1", "stk2"],
          "as": ["stack", "grpId"]
        },
        {
          "type": "formula",
          "expr": "datum.stack == 'stk1' ? datum.stk1+datum.stk2 : datum.stk2+datum.stk1",
          "as": "sortField"
        },
        {
          "type": "stack",
          "groupby": ["stack"],
          "sort": {"field": "sortField", "order": "descending"},
          "field": "size"
        },
        {
          "type": "formula",
          "expr": "(datum.y0+datum.y1)/2",
          "as": "yc"
        }
      ]
    }
    ```
  * Ajouter les scales:
    ```json
    {
      "name": "x",
      "type": "band",
      "range": "width",
      "domain": ["stk1", "stk2"],
      "paddingOuter": 0.05,
      "paddingInner": 0.95
    },
    {
      "name": "y",
      "type": "linear",
      "range": "height",
      "domain": {"data": "nodes", "field": "y1"}
    },
    {
      "name": "color",
      "type": "ordinal",
      "range": "category",
      "domain": {"data": "general", "field": "stk1"}
    },
    {
      "name": "color2",
      "type": "ordinal",
      "range": "category",
      "domain": {"data": "general", "field": "stk2"}
    },
    {
      "name": "stackNames",
      "type": "ordinal",
      "range": ["Type de stationnement", "Arrondissement"],
      "domain": ["stk1", "stk2"]
    }
    ```
  * Ajouter les données `groups`, `destinationNodes`, `edges`
    ```json
    {
      "name": "edges",
      "source": "nodes",
      "transform": [
        {"type": "filter", "expr": "datum.stack == 'stk1'"},
        {
          "type": "lookup",
          "from": "destinationNodes",
          "key": "key",
          "fields": ["key"],
          "as": ["target"]
        },
        {
          "type": "linkpath",
          "orient": "horizontal",
          "shape": "diagonal",
          "sourceX": {"expr": "scale('x', 'stk1') + bandwidth('x')"},
          "sourceY": {"expr": "scale('y', datum.yc)"},
          "targetX": {"expr": "scale('x', 'stk2')"},
          "targetY": {"expr": "scale('y', datum.target.yc)"}
        },
        {
          "type": "formula",
          "expr": "range('y')[0]-scale('y', datum.size)",
          "as": "strokeWidth"
        },
        {
          "type": "formula",
          "expr": "datum.size/domain('y')[1]",
          "as": "percentage"
        }
      ]
    }
    ```
  * Ajouter les signals `groupHover` et `groupSelector`
    ```json
    {
      "name": "groupHover",
      "value": {},
      "on": [
        {
          "events": "@groupMarkSource:mouseover,@groupMarkDest:mouseover",
          "update": "{stk1:datum.stack=='stk1' && datum.grpId, stk2:datum.stack=='stk2' && datum.grpId}"
        },
        {"events": "mouseout", "update": "{}"}
      ]
    },
    {
      "name": "groupSelector",
      "value": false,
      "on": [
        {
          "events": "@groupMarkSource:click!,@groupMarkDest:click!",
          "update": "{stack:datum.stack, stk1:datum.stack=='stk1' && datum.grpId, stk2:datum.stack=='stk2' && datum.grpId}"
        },
        {
          "events": [
            {"type": "click", "markname": "groupReset"},
            {"type": "dblclick"}
          ],
          "update": "false"
        }
      ]
    }
    ```
  * Ajouter les axes
    ```json
    {
      "orient": "bottom",
      "scale": "x",
      "encode": {
        "labels": {
          "update": {
            "text": {"scale": "stackNames", "field": "value"}
          }
        }
      }
    },
    {"orient": "left", "scale": "y"}
    ```
  * Enfin, ajouter les marks
    ```json
    {
      "type": "path",
      "name": "edgeMark",
      "from": {"data": "edges"},
      "clip": true,
      "encode": {
        "update": {
          "stroke": [
            {
              "test": "groupSelector && groupSelector.stack=='stk1'",
              "scale": "color2",
              "field": "stk2"
            },
            {"scale": "color", "field": "stk1"}
          ],
          "strokeWidth": {"field": "strokeWidth"},
          "path": {"field": "path"},
          "strokeOpacity": {
            "signal": "!groupSelector && (groupHover.stk1 == datum.stk1 || groupHover.stk2 == datum.stk2) ? 0.9 : 0.3"
          },
          "zindex": {
            "signal": "!groupSelector && (groupHover.stk1 == datum.stk1 || groupHover.stk2 == datum.stk2) ? 1 : 0"
          },
          "tooltip": {
            "signal": "datum.stk1 + ' → ' + datum.stk2 + '    ' + format(datum.size, ',.0f') + '   (' + format(datum.percentage, '.1%') + ')'"
          }
        },
        "hover": {
          "strokeOpacity": {"value": 1}
        }
      }
    },
    {
      "type": "rect",
      "name": "groupMarkSource",
      "from": {"data": "groups"},
      "encode": {
        "enter": {
          "fill": {"scale": "color", "field": "grpId"},
          "width": {"scale": "x", "band": 1},
          "cursor": {"value": "pointer"}
        },
        "update": {
          "x":  {"field": "scaledX"},
          "y":  {"field": "scaledY0"},
          "y2": {"field": "scaledY1"},
          "fillOpacity": {"value": 0.6},
          "tooltip": {
            "signal": "datum.grpId + '   ' + format(datum.total, ',.0f') + '   (' + format(datum.percentage, '.1%') + ')'"
          }
        },
        "hover": {
          "fillOpacity": {"value": 1}
        }
      }
    },
    {
      "type": "rect",
      "name": "groupMarkDest",
      "from": {"data": "groups"},
      "encode": {
        "enter": {
          "fill": {"scale": "color2", "field": "grpId"},
          "width": {"scale": "x", "band": 1},
          "cursor": {"value": "pointer"}
        },
        "update": {
          "x":  {"field": "scaledX"},
          "y":  {"field": "scaledY0"},
          "y2": {"field": "scaledY1"},
          "fillOpacity": {"value": 0.6},
          "tooltip": {
            "signal": "datum.grpId + '   ' + format(datum.total, ',.0f') + '   (' + format(datum.percentage, '.1%') + ')'"
          }
        },
        "hover": {
          "fillOpacity": {"value": 1}
        }
      }
    },
    {
      "type": "text",
      "from": {"data": "groups"},
      "interactive": false,
      "encode": {
        "update": {
          "x": {
            "signal": "scale('x', datum.stack) + (datum.rightLabel ? bandwidth('x') + 8 : -8)"
          },
          "yc": {"signal": "(datum.scaledY0 + datum.scaledY1)/2"},
          "align": {"signal": "datum.rightLabel ? 'left' : 'right'"},
          "baseline": {"value": "middle"},
          "fontWeight": {"value": "bold"},
          "text": {"signal": "abs(datum.scaledY0-datum.scaledY1) > 13 ? datum.grpId : ''"}
        }
      }
    },
    {
      "type": "group",
      "data": [
        {
          "name": "dataForShowAll",
          "values": [{}],
          "transform": [{"type": "filter", "expr": "groupSelector"}]
        }
      ],
      "encode": {
        "enter": {
          "xc": {"signal": "width/2"},
          "y": {"value": 30},
          "width": {"value": 80},
          "height": {"value": 30}
        }
      },
      "marks": [
        {
          "type": "group",
          "name": "groupReset",
          "from": {"data": "dataForShowAll"},
          "encode": {
            "enter": {
              "cursor":{"value": "pointer"},
              "cornerRadius": {"value": 6},
              "fill": {"value": "#f5f5f5"},
              "stroke": {"value": "#c1c1c1"},
              "strokeWidth": {"value": 2},
              "height": {
                "field": {"group": "height"}
              },
              "width": {
                "field": {"group": "width"}
              }
            },
            "update": {
              "opacity": {"value": 0.5}
            },
            "hover": {
              "opacity": {"value": 1}
            }
          },
          "marks": [
            {
              "type": "text",
              "interactive": false,
              "encode": {
                "enter": {
                  "xc": {
                    "field": {"group": "width"},
                    "mult": 0.5
                  },
                  "yc": {
                    "field": {"group": "height"},
                    "mult": 0.5,
                    "offset": 2
                  },
                  "align": {"value": "center"},
                  "baseline": {"value": "middle"},
                  "fontWeight": {"value": "bold"},
                  "text": {"value": "Show All"}
                }
              }
            }
          ]
        }
      ]
    }
    ```
  * Observer le diagramme de sankey
  * En changeant l'origine du champ `size` de la donnée `general` de `datum.Nbre_de_places` en `datum.Surface`, on peut analyser quelle surface les arrondissements alloue au stationnement. Pour changer plus facilement, ajouter un signal avec input pour choisir entre les 2 champs :
    ```json
    { "name": "sizeField",
      "value": "Nbre_de_places",
      "bind": {"input": "radio", "options": ["Nbre_de_places", "Surface"]}
    }
    ```
    Puis modifier la donnée `general` pour utiliser ce signal :
    ```json
    {"type": "formula", "expr": "datum[sizeField]", "as": "size"}
    ```

Chaque arrondissement possède une superficie différente, pour pouvoir vraiment comparer la place alloué, il faut relativisé la surface alloué par rapport à la superficie de chaque arrondissement

* Ajouter la source de donnée `superficie`
```json
{
  "name": "superficie",
  "url": "https://raw.githubusercontent.com/evanliomain/presentation_vega/master/data/arrondissement-paris-superficie.csv",
  "format": {
    "type": "dsv",
    "delimiter": ";",
    "parse": {
      "Superficie": "number"
    }
  }
}
```
* Ajouter un lookup pour récupérer la superficie de l'arrondissement dans les données enrichie
    ```json
    {
      "type": "lookup",
      "from": "superficie",
      "key": "Numero",
      "fields": ["Arrondissement"],
      "as": ["target"]
    },
    {
      "type": "formula", "as": "ratio", "expr": "datum.Surface / datum.target.Superficie"
    }
    ```
* Ajouter la somme des `ratio` à l'aggregation de `category`
* Enfin, ajouter `ratio` en valeur possible du signal `sizeField`
