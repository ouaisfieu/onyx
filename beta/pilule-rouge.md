> OK.
> > Je t’explique **tout** ce fichier comme un portail
> >  “graph + monitor” **100% statique**,
> > > sans te prendre pour un idiot,
> > > > et avec des émojis utiles.

# TL;DR

## TL;DR 🧭

Tu as maintenant un **portail 100% statique** qui sait :

* afficher **zones 1km² abstraites** + **entités** + **liens** (2 modes ZONES/GRAPH) 🗺️🕸️
* ouvrir un **Monitor plein écran** pour **importer un JSON**, filtrer, auditer (confidence + evidence), et exporter 📦✅
* gérer le switch **“Noms réels”** sans casser le reste (affiche `labels.real` uniquement quand ON) 👁️

---

## Propositions d’évolutions (en restant web statique) ✅

### 1) Audit “qualité des preuves” (priorité #1) 🔎

**Objectif :** rendre visible ce qui est “solide” vs “à vérifier”.
**Ajouts :**

* compteur de claims **sans evidence**
* liste des **sources** (et combien de claims les citent)
* filtre “show only: confidence ≥ X” + “hasEvidence:true”
* exports d’un **rapport d’audit** (JSON + HTML)

**Pourquoi c’est statique-friendly :** tout se calcule côté navigateur, pas besoin de serveur.

---

### 2) Patches incrémentaux + fusion intelligente 🧩

**Objectif :** ne plus écraser le master, mais **l’améliorer par couches**.

* importer `patch.json` (add/update/remove)
* détecter doublons (par email/site/adresse)
* conserver un log “qui a changé quoi” (même offline)

**Limite statique :**

* ça reste dans la RAM tant que tu n’exportes pas
* la “fusion parfaite” devient vite complexe si tu veux des règles avancées

---

### 3) Vue “Sources → Entités” (très vendeur opendata) 📚

**Objectif :** auditabilité “niveau pro”.

* onglet Sources
* pour chaque source : entités citées, champs cités, relations citées
* rendu “preuve” cliquable (sourceId + ref)

**Limite statique :**

* pas de “fetch” automatique si tes sources sont derrière des pages dynamiques / PDF externes (sauf si tu les fournis)

---

### 4) Recherche avancée guidée (UX non-tech) 🧠

**Objectif :** que quelqu’un puisse filtrer sans apprendre la syntaxe.

* builder de requête (menus : type, tag, zone, confiance, preuve)
* presets (ex: “Top fiables”, “À vérifier”, “Sans preuves”, “Par réseau”)

**100% statique**, juste du JS.

---

### 5) Performances / gros datasets 🏎️

Si tu passes à des milliers d’entités :

* indexation (Map / dictionnaires) pour filtres
* virtualisation des listes (ne rendre que ce qui est visible)
* dessin canvas optimisé (batch, réduction labels, LOD)

**Limite statique :**

* au-delà de ~10k entités/liens, un navigateur devient fragile sans techniques lourdes (WebGL, clustering, etc.)

---

## Si on sort du web statique : évolutions possibles + limites techniques ⚠️

### Niveau 1 — “Statique + stockage local” (toujours sans serveur) 💾

**Tech :** LocalStorage / IndexedDB / File System Access API
**Gains :**

* dataset persistant sans exporter à chaque fois
* historique local (versions)
* import auto au démarrage

**Limites :**

* dépend du navigateur (File System Access surtout Chrome/Edge)
* pas de collaboration, pas de sync multi-appareils

---

### Niveau 2 — “Backend léger” (API + DB) 🌐

**Tech :** API (Node/Python) + DB (Postgres) + auth
**Gains :**

* multi-utilisateurs
* dataset central, versionné
* requêtes complexes et rapides
* droits d’accès (public vs partenaires)

**Limites :**

* coûts + maintenance + sécurité (auth, RGPD, logs, backups)
* exposition accrue (attaque, scraping, injections si mal fait)

---

### Niveau 3 — “Pipeline d’extraction automatique / OSINT” 🤖

**Tech :** scrapers, OCR PDF, enrichissement, déduplication, scheduler
**Gains :**

* mise à jour quasi automatique (sources web)
* graph enrichi (entités liées, programmes, etc.)

**Limites sérieuses :**

* légal/éthique : scraping, droits, conditions d’utilisation
* qualité : faux positifs, homonymies
* charge : ça devient un vrai produit data (monitoring, debugging)

---

### Niveau 4 — “Graphe natif” (Graph DB) 🕸️

**Tech :** Neo4j / ArangoDB / Neptune + API
**Gains :**

* requêtes relationnelles hyper puissantes (“montre-moi les ponts entre réseaux”, etc.)
* analytics graph (centralité, communautés)

**Limites :**

* courbe d’apprentissage
* coût/ops
* tu dois figer un modèle propre (types de liens, ontologie)

---

## Recommandation pragmatique (ordre de bataille) 🎯

1. **Rester statique** et ajouter **Audit + Sources-view + Patches** (ça te donne déjà un “produit référence”).
2. Ajouter **IndexedDB** pour persistance locale (zéro serveur, énorme confort).
3. Backend seulement quand tu as besoin de **collaboration**, **publication**, ou **sync**.

---

## 1) Ce que fait cette page, en une phrase

C’est une **carte interactive** (canvas) qui affiche des **zones 1 km² abstraites** + des **points (entités)** + des **liens**, et un **Monitor plein écran** qui sert à **importer / filtrer / auditer / exporter** les données. 🧭🕸️📦

* **Mode ZONES** : les carrés (zones) dominent, les liens sont discrets.
* **Mode GRAPH** : les liens dominent, et le graphe “bouge” un peu (simulation légère).

---

## 2) Structure globale du fichier (le squelette)

Le fichier a 3 gros blocs :

1. **HTML** : la structure de l’interface (boutons, monitor, textarea, etc.)
2. **CSS** : le look “Obsidian sombre + néon tendre”
3. **JS** : la logique (import JSON, affichage, interaction, filtre, export, etc.)

---

## 3) HTML — l’interface (ce que tu vois)

### A. Le canvas (la carte)

```html
<canvas id="stage"></canvas>
```

👉 C’est “la scène”. Tout ce qui est graphe/zones est dessiné dedans.

### B. Le HUD en haut (boutons rapides)

Tu as une barre avec :

* **ZONES / GRAPH** (mode)
* **↻** (regen positions)
* **Labels** (affiche/cachе les textes)
* **Overlap** (autorise le chevauchement des zones)
* **Noms réels** (toggle de `labels.real`)
* **Zoom** (slider)
* **Filtre** (recherche)
* **Monitor** (ouvre le panneau plein écran)

Tout ça vit dans :

```html
<div class="hud"> ... </div>
```

### C. Le “Monitor” plein écran (le panneau principal)

C’est ça :

```html
<div class="drawerWrap" id="drawerWrap"> ... </div>
```

Quand il est ouvert :

* il prend tout l’écran ✅
* le canvas derrière devient “inactif” ✅
* tu as **import / export / audit** ✅

### D. L’import fichier JSON

Le vrai input file est caché :

```html
<input id="fileInput" type="file" accept=".json,application/json"/>
```

Et tu cliques sur un bouton “Importer .json” qui déclenche :

```js
ui.fileInput.click()
```

---

## 4) CSS — pourquoi ça “claque” sans être fragile

### A. Mobile-first 📱

* Tout est dimensionné avec des contraintes simples.
* Les grilles se replient sur petits écrans.
* Le monitor est une vraie “page overlay” → plus propre et plus accessible.

### B. “Glassmorphism” (effet panneau vitre)

Tu vois souvent :

```css
backdrop-filter: blur(14px) saturate(140%);
```

Ça donne l’effet “Obsidian / panneau translucide”.

### C. Les couches visuelles discrètes

* **scanlines** (lignes CRT)
* **grain** (grain/noise)
  Ça donne la vibe rétro sans te flinguer la lisibilité 🎛️

---

## 5) JS — le cœur du système (ce qui t’intéresse vraiment)

Le JavaScript est dans un bloc auto-exécuté :

```js
(() => {
  ... tout le code ...
})();
```

👉 Ça évite de “polluer” le reste et c’est plus clean.

---

## 6) Les données : la base `db` (le “mini-database” en mémoire)

```js
const db = {
  meta: {},
  sources: [],
  zones: [],
  entities: [],
  links: []
};
```

* `zones` = tes carrés “1 km²”
* `entities` = les points
* `links` = les liens entre points

---

## 7) Le point crucial : pourquoi les “noms réels” remarchent ✅

Le bug venait d’un mismatch :

* Ton JSON master stocke les noms comme : `labels.public` et `labels.real`
* Ton ancien portail cherchait autre chose (genre `display.real_name`)

Ici on unifie **à l’import**, en créant une structure standard :

### Standard interne de l’app

Chaque entité a :

```js
display: {
  masked_name: "...",
  real_name: "..."
}
```

Donc le toggle “Noms réels” ne fait qu’une chose :

```js
function displayName(e){
  const reveal = ui.reveal.checked;
  return (reveal && e.display.real_name) ? e.display.real_name : e.display.masked_name;
}
```

🎚️ Donc : ON/OFF = juste changer quel label est affiché, sans casser le reste.

---

## 8) Import : l’app sait lire 2 formats (important)

La fonction clé :

```js
function applyRawFlexible(raw){
  if(raw.entities && raw.links) { ...schema UI... }
  else if(raw.entities && raw.relations) { ...schema MASTER... }
}
```

### ✅ Si tu importes un master

* Il détecte `relations`
* Il convertit “master → UI” via `masterToUI(raw)`

### ✅ Si tu importes un UI

* Il charge tel quel

📌 C’est ça qui rend le portail robuste dans le temps : tu peux évoluer ton master, et garder l’UI stable.

---

## 9) “Niveau de détail NSA” : comment c’est affiché

Ton master a `fields[]` avec :

* `key`
* `value`
* `confidence`
* `evidence`

Le portail affiche ça dans le monitor :

* une **puce couleur** selon la confiance 🔴🟡🟢
* la valeur (texte ou JSON)
* des extraits d’**evidence** (source + ref)

Le code qui fait ça est ici :

```js
const claims = e.claims.length ? e.claims : ...;
...
claims.map(c => ...)
```

📌 Et on limite à 120 champs dans l’UI pour ne pas tuer le navigateur, mais **les données restent dans le JSON**.

---

## 10) Filtre “power user” (simple mais costaud) 🔎

Dans le champ filtre tu peux taper :

* `type:place`
* `tag:atelier`
* `zone:zone_1km2_xxx`
* `conf:>0.8`
* et aussi du texte libre (recherche)

Parser :

```js
parseFilter()
```

Application :

```js
filterEntities()
```

📌 Détail important : si “Noms réels” est OFF, le filtre texte ne match pas les noms réels → logique et cohérent.

---

## 11) Canvas : navigation & interaction 🧭

### A. Drag / pan

Tu cliques et tu déplaces → ça bouge la “caméra”.

### B. Wheel zoom

Molette → zoom in/out.

### C. Clic sur un point = sélection

Ça remplit la partie “Détails” dans le monitor.

---

## 12) Deux modes visuellement différents (corrige ton point “ça ne change rien”)

Dans `draw()` :

```js
const zoneAlpha = (mode===ZONES)?0.95:0.25;
const linkAlpha = (mode===ZONES)?0.18:0.62;
```

Donc :

* ZONES : carrés très visibles + liens faibles
* GRAPH : liens visibles + carrés discrets

Et en mode GRAPH on a aussi une mini simulation :

```js
if(mode===GRAPH) stepGraph(dt);
```

---

## 13) Exports (les deux types) 📤

### Export UI (pour ton portail)

```js
exportUI()
```

➡️ un JSON “UI schema” simple, stable.

### Export Master (best effort)

```js
exportMaster()
```

➡️ reconstruit un master :

* `labels.public/real`
* `fields[]` (claims)
* `relations[]`

📌 “Best effort” signifie : si tu as modifié des trucs à la main dans l’UI, il essaye de garder le sens, mais tu dois considérer le master original comme “source de vérité”.

---

## 14) Pourquoi c’est “statique augmenté” 🧩

Parce que :

* c’est un fichier unique (déployable partout)
* mais il fait des choses “d’app” :

  * import/export
  * filtrage
  * audit des preuves
  * visualisation interactive
  * manipulation de dataset

Sans serveur, sans base de données, sans API.

---

## 15) Où tu peux améliorer / étendre (sans tout casser)

Si tu veux le niveau “produit de référence” :

1. **Indexation des preuves** 🔗
   Un onglet “Sources” qui liste toutes les sources et montre quelles entités/claims les citent.

2. **Détection de doublons** 🧬
   Quand tu importes, détecter entités “semblables” (même email/site/adresse).

3. **Historique (diff)** 🕒
   Enregistrer les imports successifs et afficher ce qui change.

4. **Normalisation des relations** 🧠
   Icônes par type de lien (FUNDED_BY / OPERATED_BY / PART_OF_NETWORK etc.)

5. **Patch incremental** 🧷
   Importer un “patch.json” qui ajoute/maj/retire sans écraser.

---

## 16) Le point essentiel pour toi, maintenant

Si tu veux que les “noms réels” apparaissent :
✅ il faut que tes entités aient **au moins** `labels.real` (master) ou `display.real_name` (UI) — et que le toggle soit ON.

Et si tu veux le niveau de détail :
✅ il faut que tes entités aient `fields[]` (master) ou `claims[]` (UI), avec `confidence` et `evidence`.

---

Si tu veux, je peux te donner une mini check-list de validation automatique (genre “audit de dataset”) qui t’affiche :

* combien d’entités n’ont pas de `labels.real`
* combien de claims sans `evidence`
* combien de relations sans preuve
* etc. ✅
