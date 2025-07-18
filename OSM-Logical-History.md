---
title: OSM Logical History
css: 'style.css'
---

# OSM
# Logical History
<hr/>

## SotM FR 2025

Frédéric Rodrigo <f.rodrigo@teritorio.fr>

Teritorio, 2025 - CC BY-SA

---

## Contexte du besoin
- Clearance : synchronisation d'extraits OSM
  - -> si modifications de qualité
- LoCha : changements locaux
- Évaluation sémantique de la qualité

---

### Problèmes
## Schéma OSM & Diff
- OSM : modèle de données mêlant tags et nature des objets (nwr)
- Diff : historique technique et non sémantique
- Absence d’identifiants permanents

----

### Problèmes
## Changeset
- Absence de cohérence dans modifications
- Non transactionnel (pas de modification à date précise)

---

## Objectifs

Correspondance entre deux versions d’OSM

![](include/03-split-way.svg)

![](include/04-split-semantic.svg)

---

## Préparation des données OSM
- Calcul des géométries complètes
- Exclusion des objets sans modification

---

## Rapprochement

----

### Rapprochement
## par références

- Références métiers (tags `ref` et `ref:`)
- Rapprochement si un seul objet possède exactement le même ensemble de références

----

### Rapprochement
## par similarité
- Distances de similarité entre tous les objets avant / après
- Rapprochement des objets av/ap selon la plus petite distance
- Suppression des objets appariés
- Itération tant que possible

---

## Notion de
## similarité / distance

Comment comparer deux objets OSM ?
- Géométrie
- Tags

----

### Distance géométrique [0 ; 1]
## Point
- Distance euclidienne
- Distance non linéaire pour mieux discriminer les petites distances
- Définie si < 200 m

----

### Distance géométrique [0 ; 1]
## Ligne / Polygone
Buffer pour comparer les way / gommer les petites différences

→ Tout est converti en polygones

----

### Distance géométrique [0 ; 1]
## Polygone
- Si inclusion ⊂ : distance 0
- Si intersection ∩ : distance de Jaccard [0 ; 1] `∩/∪`
- Sinon : distance euclidienne [0,5 ; 1]
  - Pénalité de 0,5

---

### Distance
## sémantique / tags [0 ; 1]

Natures de tags
1. Hiérarchie de concepts
2. Attributs complémentaires

----

## Hiérarchie de concepts
## [0 ; 0,5]
- Si pas de clé en commun : non comparable
- Clé / Valeur par nature : distance 1
  - `amenity=recycling` / `amenity=driving_school`
- Valeur par niveau : distance 0,5
  - `highway=motorway` / `highway=unclassified`
- → Moyenne

----

<table><thead>
  <tr>
    <th>Avant</th>
    <th>Après</th>
    <th>Distance</th>
  </tr></thead>
<tbody>
  <tr>
    <td>shop=bakery</td>
    <td>shop=bakery</td>
    <td>0</td>
  </tr>
  <tr>
    <td>craft=blacksmith</td>
    <td>craft=bakery</td>
    <td>1</td>
  </tr>
  <tr>
    <td></td>
    <td>craft=tiler</td>
    <td>∅</td>
  </tr>
  <tr>
    <td>highway=residential</td>
    <td>highway=service</td>
    <td>0,5</td>
  </tr>
</tbody>
</table>

----

## Attributs complémentaires
## [0 ; 0,5]
- Pour chaque clé av/ap, distance de Levenshtein des valeurs, normalisée [0 ; 1]
- Clé sans correspondance : 1
- → Moyenne

----

<table><thead>
  <tr>
    <th>Avant</th>
    <th>Après</th>
    <th>Distance</th>
  </tr></thead>
<tbody>
  <tr>
    <td>name=Paul</td>
    <td>name=Paul</td>
    <td>0</td>
  </tr>
  <tr>
    <td>name=Amandine 1900</td>
    <td>name=Amandine</td>
    <td>0,38</td>
  </tr>
  <tr>
    <td></td>
    <td>addr:street=Avenue Tassigny</td>
    <td>1</td>
  </tr>
</tbody>
</table>

----

## Distance

Si comparable, d < 200m, concepts en commun

<img src="include/05-distance.svg" alt="drawing" width="600em"/>

Somme pondérée des distances
- 50 % : géométrie
- 25 % : concept
- 25 % : attributs

---

## Correspondances partielles

![](include/03-split-way.svg)

Si inclusion géométrique, découpage de l’objet :
1. Partie en correspondance
2. Partie disponible pour d’autres correspondances

---

## Simplifications

Pour les objets non mis en correspondance :

- Fusion des objets partiels avec l'objet d'origine
- Mise en correspondance par ID OSM
  - Ex. changement de sémantique : banque → pizzeria

---

## Implémentation
Projet OpenStreetMap Logical History

API pour le calcul d’historique sémantique

https://github.com/teritorio/openstreetmap-logical-history

<!--
https://github.com/teritorio/openstreetmap-logical-history-component
-->

---

# Conclusion

Historique plus lisible

→ Meilleure évaluation des changements

→ Analyse de haut niveau de la qualité des changements OSM dans Clearance


----

## Takeaway

![](include/qr-code.svg)
