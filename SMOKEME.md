Les “labels” **marchent**, mais dans ton code ils sont **bridés** : ils ne s’affichent que si **(mode === "GRAPH") OU (zoom >= 0.95)**. Donc en ZONES + zoom un peu bas → tu crois que le switch ne fait rien. C’est exactement ici dans ton fichier  :

```js
const canLabel = showLabels && (mode==="GRAPH" || view.z>=0.95);
```

### Fix simple (labels = ON ⇒ toujours visibles)

Cherche ce bloc dans `draw()` et remplace-le comme suit :

```js
// AVANT
const canLabel = showLabels && (mode==="GRAPH" || view.z>=0.95);

// APRÈS (labels = toujours visibles si activés)
const canLabel = showLabels;
```

### Bonus (si tu veux garder un seuil, mais “qui a du sens”)

Tu peux mettre un seuil plus permissif et indépendant du mode :

```js
const canLabel = showLabels && view.z >= 0.70;
```

### Option “propre” (éviter le surchargement en ZONES)

Garde l’affichage mais **limite** les labels quand y’en a trop :

Remplace ton bloc label par ceci :

```js
if (canLabel){
  const nm = ui.reveal.checked
    ? (e.display.real_name||e.display.masked_name||e.id)
    : (e.display.masked_name||e.id);

  // limite auto: en ZONES, on n'affiche pas les labels si on est trop zoom-out
  if(mode==="ZONES" && view.z < 0.85) {
    // rien
  } else {
    ctx.fillStyle="rgba(234,241,255,.80)";
    ctx.font=`${Math.max(10,11*view.z)}px ${getComputedStyle(document.documentElement).getPropertyValue('--mono')||'monospace'}`;
    ctx.fillText(nm, s.x+10*view.z, s.y-8*view.z);
  }
}
```

Si tu appliques le **fix simple**, ton switch “Labels” redevient “bête et méchant” : ON = visible, OFF = invisible, sans surprise.



...

OK. On va partir d’un principe simple : **on ne stocke pas “des infos”**, on stocke **des “affirmations” (claims) + leurs preuves (sources) + un score de confiance**. Comme ça, ton fichier peut évoluer sans jamais te mélanger le sûr et le douteux.

Je me base ici sur tes sources déjà fournies, surtout `lieux_de_liens_et_plus.md`  (et le PDF “chatgpt” qui liste aussi des ressources/URLs) .

---

## 1) Modèle de données “propre” (évolutif + auditabilité)

### A. Les 3 objets indispensables

* **sources** : d’où ça vient (fichier, page, date, extrait/repère)
* **entities** : lieux / orgas / réseaux / programmes / personnes (si tu veux, mais mieux en “role” non nominatif)
* **relations** : liens typés entre entités (porte/parraine/associe/finance/territoire/etc.)

### B. Stocker des “claims” plutôt que des champs “bruts”

Chaque entité a une liste de champs, et **chaque champ est une claim** :

* `value` : la donnée
* `confidence` : 0 → 1
* `evidence` : références vers une ou plusieurs sources + repères

➡️ Résultat : tu peux afficher “seulement ce qui est ≥0.9” ou au contraire “tout mais coloré”.

---

## 2) Types de relations à prévoir (pour les liens “pointus”)

Tu vas vite gagner en puissance si tu normalises 15–20 types max :

* `IS_A` (type/subtype)
* `LOCATED_IN_ADMIN` (commune, code postal) *(pas GPS)*
* `HAS_ZONE_HINT` (zone 1km² abstraite, superposable)
* `OPERATED_BY` (lieu → organisation porteuse)
* `CO_OPERATED_BY` (lieu → co-porteurs)
* `SUPPORTED_BY` (soutien, accompagnement)
* `FUNDED_BY` (financeur / programme)
* `PART_OF_NETWORK` (lieu → réseau)
* `NETWORK_COVERS` (réseau → territoire)
* `ASSOCIATED_WITH` (relation faible, “associé à”)
* `HAS_ACTIVITY` (lieu → activité)
* `HAS_SCHEDULE` (lieu → horaires)
* `HAS_CONTACT_POINT` (lieu → email/tel/site)
* `REFERENCES` (source → entité/claim)

Exemples dans ta source :

* Le lieu “Tarot” (Babel’zin) est “Lieu de liens du SSM Le Grès” + horaires + équipe + réseau + activités 
* “L’Entr’Act” a “Organisation porteuse : Centre L’Orée + WOPS …” 
* Les réseaux Psy 107 (RÉZONE, NORWEST, HERMES+, BRUXELLES-EST) ont territoires, siège, contacts, et “lieux de liens associés” 
* Organismes de coordination (LBSM, PBSM-BPGG, Brusano, Vivalis) avec rôles et contacts 

---

## 3) Convention “noms réels” vs “noms publics” (pour ton switch UI)

Tu veux pouvoir masquer/afficher : donc **ne jamais dépendre d’un seul champ `name`**.

* `labels.public` : codename (ex: “Tarot”, “Astro”, “Lieu_07”)
* `labels.real` : nom réel (facultatif)
* `privacy.realNamePolicy` : `"hiddenByDefault"`

Ensuite ton toggle “Noms réels” se contente de choisir quel label afficher.

---

## 4) Exemple de fichier master JSON (structure + premiers objets utiles)

> Ci-dessous : **un squelette importable** + **un noyau minimal** (Tarot + un lieu lié à Astro + réseaux Psy 107 + coordos).
> Je laisse “Astro” comme **entité codée** (sans nommer). La relation “co-portage” existe parce que la source dit “Centre L’Orée + WOPS …”  (et on mappe WOPS → Astro côté jeu).

```json
{
  "meta": {
    "schemaVersion": "0.1.0",
    "scope": "bruxelles",
    "geopolicy": {
      "no_precise_geolocation": true,
      "zonesKm2": true,
      "overlapAllowed": true
    },
    "confidencePolicy": {
      "1.0": "copie directe d'une source fournie",
      "0.8": "source fournie mais ambiguë / multi-interprétation possible",
      "0.6": "inférence raisonnable (à valider)",
      "0.3": "piste / à vérifier"
    }
  },

  "sources": [
    {
      "id": "src_md_ldlplus_2024_2025",
      "kind": "markdown",
      "title": "Lieux de liens et plus (Bruxelles)",
      "path": "lieux_de_liens_et_plus.md",
      "notes": "Synthèse structurée des lieux, réseaux et organismes."
    },
    {
      "id": "src_pdf_ldl_chatgpt",
      "kind": "pdf",
      "title": "Lieux de liens (PDF) + ressources",
      "path": "lieux-de-liens-chatgpt.pdf",
      "notes": "Liste aussi des URLs de référence."
    }
  ],

  "entities": [
    {
      "id": "ent_tarot",
      "type": "place",
      "subtype": "lieu_de_liens",
      "labels": { "public": "Tarot", "real": "Babel'zin" },
      "privacy": { "realNamePolicy": "hiddenByDefault" },
      "tags": ["bas_seuil", "participatif", "sante_mentale", "bruxelles_est"],
      "zoneHints": ["zone_1km2_auderghem_01"],
      "fields": [
        {
          "key": "address",
          "value": "Chaussée de Wavre 1688, 1160",
          "confidence": 1.0,
          "evidence": [{ "sourceId": "src_md_ldlplus_2024_2025", "ref": "Babel'zin / Adresse" }]
        },
        {
          "key": "phone",
          "value": "0492/44.88.70",
          "confidence": 1.0,
          "evidence": [{ "sourceId": "src_md_ldlplus_2024_2025", "ref": "Babel'zin / Téléphone" }]
        },
        {
          "key": "email",
          "value": "babelzin.legres@gmail.com",
          "confidence": 1.0,
          "evidence": [{ "sourceId": "src_md_ldlplus_2024_2025", "ref": "Babel'zin / Email" }]
        },
        {
          "key": "website",
          "value": "babelzin.be",
          "confidence": 1.0,
          "evidence": [{ "sourceId": "src_md_ldlplus_2024_2025", "ref": "Babel'zin / Site web" }]
        },
        {
          "key": "schedule_2024_2025",
          "value": {
            "mon": "10h-16h",
            "tue": "14h-20h",
            "wed": "11h-17h",
            "thu": "10h-16h",
            "fri": "14h-20h",
            "sat": "fermé",
            "sun": "14h-17h"
          },
          "confidence": 1.0,
          "evidence": [{ "sourceId": "src_md_ldlplus_2024_2025", "ref": "Babel'zin / Horaires 2024-2025" }]
        },
        {
          "key": "activities",
          "value": ["ateliers créatifs", "percussions", "multimédia", "ciné-club", "informaticien public", "théâtre", "radio", "cuisine", "sorties", "poulailler communautaire"],
          "confidence": 1.0,
          "evidence": [{ "sourceId": "src_md_ldlplus_2024_2025", "ref": "Babel'zin / Activités" }]
        },
        {
          "key": "team",
          "value": ["Alex", "Noémie", "Juan", "Benoît", "Stéphane B. (coordinateur)"],
          "confidence": 1.0,
          "evidence": [{ "sourceId": "src_md_ldlplus_2024_2025", "ref": "Babel'zin / Équipe actuelle" }]
        }
      ]
    },

    {
      "id": "ent_astro",
      "type": "org",
      "subtype": "asbl",
      "labels": { "public": "Astro", "real": null },
      "privacy": { "realNamePolicy": "hiddenByDefault" },
      "tags": ["top_priority", "reseau"],
      "fields": [
        {
          "key": "note",
          "value": "Entité codée (mapping côté jeu). Liée à un lieu via co-portage.",
          "confidence": 0.8,
          "evidence": [{ "sourceId": "src_md_ldlplus_2024_2025", "ref": "L'Entr'Act / Organisation porteuse mentionne WOPS" }]
        }
      ]
    },

    {
      "id": "ent_place_weekend_01",
      "type": "place",
      "subtype": "lieu_de_liens",
      "labels": { "public": "Lieu_weekend_01", "real": "L'Entr'Act" },
      "privacy": { "realNamePolicy": "hiddenByDefault" },
      "tags": ["weekend", "bas_seuil", "bruxelles_est"],
      "zoneHints": ["zone_1km2_schaerbeek_01"],
      "fields": [
        {
          "key": "address",
          "value": "Avenue de Roodebeek 273, 1030",
          "confidence": 1.0,
          "evidence": [{ "sourceId": "src_md_ldlplus_2024_2025", "ref": "L'Entr'Act / Adresse" }]
        },
        {
          "key": "email",
          "value": "contact@lentract.be",
          "confidence": 1.0,
          "evidence": [{ "sourceId": "src_md_ldlplus_2024_2025", "ref": "L'Entr'Act / Email" }]
        },
        {
          "key": "schedule",
          "value": "Samedis, dimanches et jours fériés 10h-17h; 1 vendredi/mois 18h-21h",
          "confidence": 1.0,
          "evidence": [{ "sourceId": "src_md_ldlplus_2024_2025", "ref": "L'Entr'Act / Horaires" }]
        },
        {
          "key": "specificity",
          "value": "Ouvert quand les autres lieux sont fermés (week-ends).",
          "confidence": 1.0,
          "evidence": [{ "sourceId": "src_md_ldlplus_2024_2025", "ref": "L'Entr'Act / Spécificité" }]
        }
      ]
    },

    {
      "id": "net_bruxelles_est",
      "type": "network",
      "subtype": "psy107_antenne",
      "labels": { "public": "Réseau_Est", "real": "BRUXELLES-EST (Psy 107)" },
      "privacy": { "realNamePolicy": "hiddenByDefault" },
      "tags": ["psy107", "territorial"],
      "fields": [
        {
          "key": "territory",
          "value": ["Schaerbeek", "Evere", "Etterbeek", "Auderghem", "WSP", "WSL", "Haren"],
          "confidence": 1.0,
          "evidence": [{ "sourceId": "src_md_ldlplus_2024_2025", "ref": "BRUXELLES-EST / Territoire" }]
        },
        {
          "key": "email",
          "value": ["info@p107bxl-est.be", "info@107bru.be"],
          "confidence": 1.0,
          "evidence": [{ "sourceId": "src_md_ldlplus_2024_2025", "ref": "BRUXELLES-EST / Email" }]
        }
      ]
    },

    {
      "id": "org_lbsm",
      "type": "org",
      "subtype": "federation",
      "labels": { "public": "Coord_01", "real": "Ligue Bruxelloise pour la Santé Mentale (LBSM)" },
      "privacy": { "realNamePolicy": "hiddenByDefault" },
      "tags": ["coordination", "publication_guide"],
      "fields": [
        {
          "key": "address",
          "value": "Rue Mercelis 39, 1050",
          "confidence": 1.0,
          "evidence": [{ "sourceId": "src_md_ldlplus_2024_2025", "ref": "LBSM / Adresse" }]
        },
        {
          "key": "email",
          "value": "info@lbsm.be",
          "confidence": 1.0,
          "evidence": [{ "sourceId": "src_md_ldlplus_2024_2025", "ref": "LBSM / Email" }]
        }
      ]
    }
  ],

  "relations": [
    {
      "id": "rel_tarot_part_of_net_est",
      "type": "PART_OF_NETWORK",
      "from": "ent_tarot",
      "to": "net_bruxelles_est",
      "confidence": 1.0,
      "evidence": [{ "sourceId": "src_md_ldlplus_2024_2025", "ref": "BRUXELLES-EST / Lieux de liens associés inclut Babel'zin" }]
    },
    {
      "id": "rel_weekend_place_part_of_net_est",
      "type": "PART_OF_NETWORK",
      "from": "ent_place_weekend_01",
      "to": "net_bruxelles_est",
      "confidence": 1.0,
      "evidence": [{ "sourceId": "src_md_ldlplus_2024_2025", "ref": "BRUXELLES-EST / Lieux de liens associés inclut L'Entr'Act" }]
    },
    {
      "id": "rel_astro_co_operates_weekend_place",
      "type": "CO_OPERATED_BY",
      "from": "ent_place_weekend_01",
      "to": "ent_astro",
      "confidence": 0.9,
      "evidence": [{ "sourceId": "src_md_ldlplus_2024_2025", "ref": "L'Entr'Act / Organisation porteuse: Centre L'Orée + WOPS (mappé → Astro)" }]
    }
  ],

  "zones": [
    { "id": "zone_1km2_auderghem_01", "label": "Zone 1km² (Auderghem) #01" },
    { "id": "zone_1km2_schaerbeek_01", "label": "Zone 1km² (Schaerbeek) #01" }
  ]
}
```

Les “preuves” viennent directement de ton fichier de référence (Babel’zin, L’Entr’Act, réseaux, coordos).    

---

## 5) Ce qu’il reste à extraire (méthode “source → extraction → liens”)

À partir de `lieux_de_liens_et_plus.md`, tu peux générer (sans web) :

### A. Entités “lieux”

Pour chaque lieu : adresse / contacts / horaires / activités / philosophie / public cible / org porteuse / réseaux associés.
Exemples déjà très riches dans la source : “Tarot” , “L’Entr’Act” , “Le Delta” etc. 

### B. Entités “réseaux”

Les 4 antennes Psy 107 + leurs coordonnées + “lieux associés” (ça te donne un graphe quasi clé-en-main). 

### C. Entités “coordination / administration”

LBSM, PBSM-BPGG, Brusano, Vivalis : utiles pour “liens structurels” et “qui finance/agrée/coordonne”. 

### D. Relations (le vrai trésor)

* lieu → réseau (`PART_OF_NETWORK`)
* lieu → org porteuse (`OPERATED_BY`, `CO_OPERATED_BY`)
* réseau → communes couvertes (`NETWORK_COVERS`)
* org → rôle (`HAS_ROLE`)
* lieu → activités (`HAS_ACTIVITY`) *(c’est un pivot ARG énorme pour tes “missions bonus”)*

---

## 6) Règle d’or “vrai vs risqué”

* **Si c’est écrit explicitement dans la source** : confidence **1.0**
* **Si tu fais un mapping “jeu” (ex: WOPS → Astro)** : confidence **0.9** mais avec `evidence.ref` clair (pour audit)
* **Si tu infères** (ex: “ce lieu est plutôt orienté X”) : confidence **0.6** et tag `needs_review:true`

---

Si tu veux, à la prochaine étape je te fournis :

1. **le master JSON complet** (tous les lieux + les 4 réseaux + coordos) à partir de `lieux_de_liens_et_plus.md` 
2. **un 2e fichier “patch”** (format “import incremental”) pour ajouter de nouvelles sources sans casser l’historique.


---

...
> et je précise un truc :
> c'est un test pour savoir si tu es à la mesure :
> * - de la tâche
>   - * - des enjeux
>       - * - de mes attentes
* - de tes propres facultés -
* de ce qui nous attends pour la suite.
* * > tu peux abandonner maintenant
    > > > + ou te donner à fond
