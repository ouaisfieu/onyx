# Master vs UI — Documentation complète

## TL;DR
- **MASTER** = la **source de vérité** (opendata, audit, preuves, évolution long terme)
- **UI** = la **représentation interactive** (performance, lisibilité, manipulation)
- Le portail **convertit automatiquement MASTER → UI** à l’import
- L’export MASTER est un **best effort** (on peut perdre de la nuance)

---

## 1. Pourquoi deux formats ?

Ton projet a deux besoins **incompatibles dans un seul format** :

1. 📚 **Rigueur documentaire** (preuves, fiabilité, auditabilité, opendata)
2. ⚡ **Interactivité fluide** (canvas, filtres rapides, graph, mobile)

👉 Séparer MASTER et UI évite les compromis foireux.

---

## 2. Le format MASTER (vérité, audit, opendata)

### Rôle
- Référence officielle
- Publication opendata
- Audit externe possible
- Évolution lente et contrôlée

### Caractéristiques
- Orienté **preuves**, pas affichage
- Basé sur des **claims** (affirmations)
- Chaque info est **traçable**

### Structure globale
```json
{
  "meta": {},
  "sources": [],
  "entities": [],
  "relations": [],
  "zones": []
}
```

---

### 2.1 Sources
```json
{
  "id": "src_md_ldlplus_2024_2025",
  "kind": "markdown",
  "title": "Lieux de liens et plus",
  "path": "lieux_de_liens_et_plus.md"
}
```

➡️ Une source **ne dit rien toute seule**, elle sert à **prouver** des claims.

---

### 2.2 Entities (entités)

Une entité **ne contient pas des champs**, mais des **claims**.

```json
{
  "id": "ent_tarot",
  "type": "place",
  "subtype": "lieu_de_liens",
  "labels": {
    "public": "Tarot",
    "real": "Babel'zin"
  },
  "privacy": {
    "realNamePolicy": "hiddenByDefault"
  },
  "tags": ["sante_mentale", "bas_seuil"],
  "zoneHints": ["zone_1km2_01"],
  "fields": [
    {
      "key": "address",
      "value": "Chaussée de Wavre 1688, 1160",
      "confidence": 1.0,
      "evidence": [
        { "sourceId": "src_md_ldlplus_2024_2025", "ref": "Adresse" }
      ]
    }
  ]
}
```

### Ce que ça permet
- Distinguer **vrai / probable / douteux**
- Montrer **d’où vient chaque info**
- Mettre à jour sans écraser l’historique

---

### 2.3 Relations

Les liens sont aussi des objets documentés.

```json
{
  "id": "rel_tarot_part_of_net_est",
  "type": "PART_OF_NETWORK",
  "from": "ent_tarot",
  "to": "net_bruxelles_est",
  "confidence": 1.0,
  "evidence": [
    { "sourceId": "src_md_ldlplus_2024_2025", "ref": "Lieux associés" }
  ]
}
```

➡️ Un lien **peut être incertain** aussi.

---

## 3. Le format UI (visualisation, interaction)

### Rôle
- Affichage rapide
- Canvas fluide
- Manipulation humaine

### Caractéristiques
- Données **déjà digérées**
- Optimisé pour le JS temps réel
- Pas de logique d’audit profonde

---

### 3.1 Structure UI
```json
{
  "meta": {},
  "zones": [],
  "entities": [],
  "links": []
}
```

---

### 3.2 Entité UI

```json
{
  "id": "ent_tarot",
  "type": "place",
  "layer": "NEUTRE",
  "zone": "zone_1km2_01",
  "confidence": 0.95,
  "display": {
    "masked_name": "Tarot",
    "real_name": "Babel'zin"
  },
  "fields": {
    "address": {
      "value": "Chaussée de Wavre 1688, 1160",
      "confidence": 1.0,
      "evidence": [ ... ]
    }
  },
  "claims": [ ... ]
}
```

### Différences clés
| MASTER | UI |
|------|----|
| `labels.public` | `display.masked_name` |
| `labels.real` | `display.real_name` |
| `fields[]` (array) | `fields{}` (object) |
| relations | links |

---

## 4. Conversion MASTER → UI (automatique)

À l’import :
- les `labels` deviennent `display`
- les `fields[]` deviennent un objet clé → valeur
- la **confidence entité** = moyenne des claims
- les `zoneHints` deviennent une `zone`

➡️ Le portail **ne modifie jamais le MASTER**, il le **traduit**.

---

## 5. Export UI → MASTER (best effort)

À l’export :
- les `fields{}` sont reconvertis en `fields[]`
- les `links` deviennent `relations`
- les noms réels restent protégés par défaut

⚠️ Limite :
- si tu simplifies trop dans l’UI, tu peux perdre des nuances
- le MASTER original reste la référence

---

## 6. Bonnes pratiques (très important)

### ✅ Toujours faire
- Modifier le **MASTER** pour publier
- Utiliser l’UI pour explorer, vérifier, visualiser

### ❌ Éviter
- Éditer massivement le UI JSON à la main
- Utiliser l’export MASTER comme unique source sans vérification

---

## 7. Résumé mental (à garder en tête)

🧠 **MASTER = savoir documenté**  
👁️ **UI = vision manipulable**  

Le portail est un **traducteur**, pas un oracle.

---

Si tu veux, prochaine étape possible :
- un schéma visuel MASTER ↔ UI
- un guide "comment enrichir le master sans tout casser"
- une checklist qualité opendata (automatisable)

