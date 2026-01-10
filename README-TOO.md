Je vais être clair et factuel.

### Pourquoi “Noms réels” ne s’affichait plus

1. **Le dataset importé n’avait plus de `labels.real` / `display.real_name`**, ou bien ces champs avaient changé de nom / étaient vides.
   👉 Si tu importes un JSON “master” où tu as `labels: { public, real }`, et que l’UI attendait `display.real_name`, alors le toggle ne peut rien afficher.

2. **Ton toggle était branché sur le mauvais champ** (ex: `entity.name` au lieu de `entity.labels.real`), ou sur un champ qui n’existe plus après conversion.

Dans le portail que je t’ai donné, je corrige ça en **supportant les deux schémas** :

* MASTER → `labels.public / labels.real`
* UI → `display.masked_name / display.real_name`

et en convertissant MASTER → UI automatiquement.

---

### Pourquoi “le niveau de détail” était devenu “moins bon”

Deux causes typiques :

1. **On a “aplati” les claims** (fields/claims) lors d’une conversion précédente (perte de `confidence` et `evidence`).
2. **On affichait seulement quelques champs** (UI) au lieu d’afficher **tous les claims** + leurs preuves.

Dans ma version, l’onglet Détails affiche :

* `key`
* `value` (objets JSON inclus)
* `confidence`
* `evidence` (sourceId + ref)
  et garde tout dans le JSON.

---

### Pourquoi ton import était “inaccessible” (encaissement)

Ce que tu décris (“encaissement”) correspond à un **panneau / overlay qui mange les clics** ou à un **layout non mobile-first** où le container de l’import sort de l’écran et devient inutilisable.

Dans la version que je t’ai donnée :

* le **Monitor prend tout l’écran** (donc pas de panneau qui se superpose “à moitié”),
* il y a un **vrai bouton “Importer .json”** (input file caché, déclenché par bouton),
* et le canvas est **désactivé quand le monitor est ouvert** (pas de conflit de clics).

---

### Pourquoi “Labels” ne fonctionnaient plus

En général, c’est soit :

* le toggle n’appelle plus de `render()`/`draw()` (oubli après refactor),
* soit on a introduit une condition qui court-circuite l’affichage (ex : le mode cache les labels),
* soit une erreur JS stoppe la boucle de rendu.

Dans la version actuelle :

* `Labels` agit directement sur le dessin (pas sur les données),
* pas besoin de recharger / ré-importer.

---

### Pourquoi “Graph” et “Zones” semblaient identiques

Parce que dans ta dernière itération :

* les deux modes utilisaient **le même rendu** (mêmes alphas, mêmes forces, mêmes objets visibles),
* donc visuellement rien ne différenciait.

Dans ma version :

* **ZONES** : zones très visibles + liens faibles
* **GRAPH** : liens très visibles + zones faibles
  et la physique (springs) est plus forte en GRAPH.

---

### Ce que je te garantis dans le code que je viens de donner

* Toggle Monitor **réparé** (bouton + FAB mobile + ESC + clic sur scrim)
* Monitor **plein écran**
* Import fichier `.json` **fonctionnel**
* Compatibilité import **UI et MASTER**
* “Noms réels” affiche quelque chose **si** le dataset contient réellement des noms réels
* Détails **claims + evidence + confidence** visibles dans le Monitor

---

### Ce que je ne peux PAS “justifier” sans ton contexte exact

Je ne peux pas “prouver” ce qui cassait précisément **dans tes fichiers précédents** sans relire la dernière version exacte du HTML que tu as en prod (celle qui “a cassé”).
Mais les symptômes que tu décris correspondent **pile** aux 4 causes ci-dessus (schéma JSON divergent, champs perdus, overlay qui capture les clics, toggles non reliés au rendu).

Si tu veux que je te pointe **ligne par ligne** ce qui cassait dans *ton* fichier actuel : donne-moi le contenu de `index (27).html` ou confirme que je dois le prendre tel quel depuis `/mnt/data/index (27).html` et je te fais l’autopsie technique.
