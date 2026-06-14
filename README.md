# 🌊 Widget Température Lac

Générateur de widgets [Scriptable](https://scriptable.app/) (iOS) affichant la
**température de l'eau**, la **température de l'air** et le **vent** pour les
plages des lacs romands.

👉 **[Ouvrir le générateur](https://alexhaed.github.io/widget-temperature-lac/)**

## Comment ça marche

1. Sur la page, choisis un **lac**, une **plage** et une **fréquence** de mise à jour.
2. Clique sur **Générer le script** et copie le code produit.
3. Installe [Scriptable](https://apps.apple.com/app/scriptable/id1405459188)
   (gratuit), crée un nouveau script et colle le code.
4. Ajoute un widget Scriptable à ton écran d'accueil et sélectionne ce script.

Le site est **100 % statique** : le formulaire génère simplement un script, et
tous les appels API se font ensuite depuis ton iPhone (aucun backend, aucune
clé API).

## Fonctionnalités du widget

- **Température de l'eau** avec un triangle de tendance (▲ se réchauffe,
  ▼ se refroidit, ▶ stable).
- **Température de l'air** et **vent** (vitesse + flèche pointant vers où le
  vent se dirige).
- **Titre** = nom de la plage, **heure** de la dernière mise à jour réussie.
- **Cache de secours** : si une API échoue, la dernière valeur connue reste
  affichée.

## Sources de données

| Donnée | API | Détail |
|---|---|---|
| Température de l'eau | [Alplakes](https://www.alplakes.eawag.ch/) (Eawag) | Modèle hydrodynamique par lac |
| Air & vent | [open-meteo.com](https://open-meteo.com/) | Conditions actuelles |

La température de l'eau est mesurée sur un **point au large** de chaque plage
(coordonnées pré-calculées dans `index.html`).

## Lacs couverts

| Lac | Modèle Alplakes |
|---|---|
| Léman | `delft3d-flow` / `geneva` |
| Neuchâtel | `mitgcm` / `neuchatel` |
| Bienne | `delft3d-flow` / `biel` |
| Morat | `delft3d-flow` / `murten` |
| Joux | `delft3d-flow` / `joux` |

Chaque lac a son propre modèle ; la liste des modèles/lacs disponibles vient de
`https://alplakes-api.eawag.ch/simulations/metadata`.

## Détails techniques

- **Interpolation** : Alplakes ne fournit un point que toutes les 3 h. Le script
  récupère une fenêtre encadrant l'heure actuelle et **interpole linéairement**
  entre le point précédent et le suivant (et en déduit la tendance en °C/h).
- **Rafraîchissement** : `refreshAfterDate` n'est qu'une *indication* ; iOS
  décide du moment réel et limite le nombre de rafraîchissements par jour.
- **`.nojekyll`** : GitHub Pages est servi sans Jekyll (site purement statique).

### Pièges connus

- Le modèle de script est stocké dans un `<textarea>`. Les `&` des URLs y sont
  écrits `&amp;` pour ne **pas** être décodés en entités HTML lors de la lecture
  (sinon `&current` devient le symbole `¤` et casse l'appel open-meteo).
- Les coordonnées des plages doivent tomber **dans l'eau** : un point sur la
  terre renvoie `null` côté Alplakes.

## Ajouter une plage

Dans `index.html`, ajoute une entrée `{ name, lat, lng }` au bon lac. Vérifie
d'abord que le point renvoie une température (et non `null`) :

```bash
curl -s "https://alplakes-api.eawag.ch/simulations/point/<modèle>/<lac>/<début>/<fin>/0/<lat>/<lng>?variables=temperature"
```

Format des dates : `YYYYmmddHHMM` (UTC).
