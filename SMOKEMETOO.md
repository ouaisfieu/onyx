## crtl-v ne répond plus

Tu veux une justification honnête. 
Voilà, sans langue de bois, 
et avec ce qui relève **de moi** vs ce qui relève **du cadre** vs ce qui relève **de ta direction produit**.

## Ce que j’ai mal fait (moi)

### 1) J’ai laissé dériver le “contrat” de base

Tu avais un cahier des charges stable :

* web statique ✅
* mobile first ✅
* 2 modes visuellement distincts ✅
* labels lisibles ✅
* toggle “noms réels” fiable ✅
* monitor plein écran + import JSON ✅

Et au lieu de figer une **architecture minimale** et de l’améliorer **sans casser**, j’ai fait des itérations qui modifiaient trop de pièces à la fois (UI + rendu + schéma JSON + interactions). Résultat : régressions.

### 2) Je n’ai pas verrouillé la compatibilité dataset ↔ UI

Le bug “noms réels identiques” vient presque toujours d’un truc simple :

* soit le dataset n’a pas de champ “real”
* soit l’UI ne lit pas le bon champ
* soit une conversion “master → UI” supprime/écrase la valeur

J’aurais dû imposer un mapping unique dès le départ :

* `labels.public` et `labels.real` (master)
* converti en `display.masked_name` et `display.real_name` (UI)
  et **ne plus jamais changer** ça.

### 3) J’ai raté la lisibilité (c’est ton point principal)

Un graphe illisible, c’est un graphe qui affiche trop :

* trop de labels (tous en même temps)
* trop tôt (même zoom)
* sans stratégie de priorité (hover/selected/quota)

Ce que j’aurais dû faire dès V1 :

* Labels **SMART par défaut** (selected + hover + quota)
* Labels “ALL” en mode debug seulement (Alt+clic ou option cachée)

### 4) Je n’ai pas protégé l’expérience contre tes données “parasites”

Ton screenshot montre des nœuds avec du texte genre “Perspectives & recommandations…”, ce qui ressemble à des **sections de documents** / “concepts” qui n’ont rien à faire dans une vue “lieux de liens” si l’objectif est de cartographier des entités opérationnelles.

J’aurais dû filtrer / séparer :

* dataset “opérationnel” (places/org/network/service)
* dataset “documentation” (concept/doc/annexe)
  et ne pas les mélanger dans la même vue par défaut.

---

## Ce que le cadre technique rend “facile à casser” (pas une excuse, mais un fait)

### 1) Web statique + canvas = tu es ton propre moteur

Pas de lib (d3/cytoscape) = c’est performant et maîtrisable, mais :

* le moindre refactor peut casser une interaction
* l’UI et le rendu doivent être hyper disciplinés

### 2) Import JSON libre = risque permanent de schéma divergent

Si tu acceptes “n’importe quel JSON” :

* tu dois gérer 3–4 variantes
* sinon ça “marche” mais mal (ex: noms réels vides → toggle inutile)

---

## Là où tu diriges peut-être mal (oui je te le dis)

Tu as raison sur l’ambition, mais il y a **2 points** qui sabotent la stabilité :

### 1) “Je veux tout, tout de suite, et ça doit rester simple”

Le paradoxe :

* plus tu veux un niveau de détail “NSA”
* plus tu dois accepter une phase où l’UI montre *moins* mais *mieux*, et où le détail est dans le Monitor / fiche entité.

Sinon la carte devient un nuage de texte et tu perds l’utilité principale.

👉 Recommandation : la carte doit montrer **structure + relations**, pas le dossier complet.
Le “détail NSA” vit dans :

* fiche entité
* tableau des claims
* export
* filtrage

### 2) “On ne géolocalise pas mais on veut des zones 1km²”

C’est jouable, mais ça demande une règle claire :

* soit tu assignes chaque entité à 1 zone abstraite (propre, lisible)
* soit tu autorises multi-zones (réaliste, mais visuellement chaotique)

👉 Recommandation : **1 zone principale** + “zoneHints secondaires” en métadonnées (pas visibles par défaut).

---

## La vraie cause de ta rage (et tu as raison)

Tu ne t’énerves pas parce que “le graph est moche”.
Tu t’énerves parce que tu as perdu :

* du temps
* de la confiance
* du contrôle

Et ça arrive quand chaque itération casse un truc existant.

---

## Ce que j’aurais dû faire comme méthode (et ce que je te propose comme règle maintenant)

### Règle 1 — “Zéro régression”

À chaque modif, on teste obligatoirement :

* bouton Monitor ouvre/ferme ✅
* import .json ✅
* labels on/off ✅
* noms réels on/off ✅
* mode graph/zones visuellement différent ✅

Si une de ces 5 choses casse : on n’ajoute rien d’autre.

### Règle 2 — “Le graphe n’affiche jamais tout”

Par défaut :

* labels : SMART
* détails : dans Monitor
* la carte reste lisible

### Règle 3 — “Deux datasets, pas un”

* `master.json` (détail + sources + evidence)
* `ui.json` (résumé + ids + types + zones + liens)
  Le portail statique peut convertir master → ui, mais **l’affichage** utilise un format stable.

---

## Ce que tu peux faire pour m’aider à ne plus te faire perdre de temps

Sans me redonner 50 consignes en rafale (je les ai), juste ceci :

* **Ton objectif principal de la carte :** “voir le réseau” ou “voir le dossier” ?

  * Si c’est “réseau” → labels minimal + liens mis en avant.
  * Si c’est “dossier” → ce n’est plus une carte, c’est un explorateur (table + filtres + détails).

Si tu réponds “réseau”, je verrouille définitivement :

* labels SMART
* graph vs zones ultra distinct
* et les détails uniquement dans monitor.

Si tu réponds “dossier”, je te dis cash : **le canvas n’est pas la bonne vue principale**. On met une **table** comme vue par défaut, et la carte devient secondaire.

Voilà ma justification. Et oui : j’ai merdé sur la discipline de versioning et sur la lisibilité par défaut.
