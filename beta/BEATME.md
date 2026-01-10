Parfait. Voilà la **roadmap 4 sprints (S0 → S3)**, **concrète**, **réaliste**, alignée avec tes contraintes (web statique, opendata, santé mentale Bruxelles), avec **livrables clairs** et **critères de réussite vérifiables**.
Pas de bullshit, pas de promesses creuses.

---

# 🧭 Roadmap produit — 4 sprints

## 🎯 Vision cible (à garder en tête)

> **Devenir la référence opendata bruxelloise sur les lieux, réseaux et liens en santé mentale**, avec :

* un **dataset auditable** (preuves, confiance),
* un **portail lisible par des humains**,
* et un **socle réutilisable** (ARG, recherche, plaidoyer, partenariats).

---

## 🟦 S0 — Stabilisation & socle (fondations solides)

### Objectif

Arrêter l’hémorragie de bugs, figer les règles du jeu, sécuriser la base.

### Livrables

1. **Portail statique stable**

   * Monitor plein écran fonctionnel
   * Import `.json` (master + UI)
   * Export UI + export master (best effort)
   * Toggles fiables :

     * Labels
     * Noms réels
     * Modes ZONES / GRAPH

2. **Schéma MASTER ↔ UI documenté**

   * (le `.md` que tu as maintenant fait partie de S0)

3. **Dataset master v0**

   * Extraction **complète** depuis `lieux_de_liens_et_plus.md`
   * Tous les lieux listés dans la source
   * Tous les réseaux Psy 107
   * Tous les organismes de coordination
   * Claims + evidence + confidence partout où possible

### Critères de réussite (objectifs, pas subjectifs)

* ✅ Import du master **sans erreur**
* ✅ Switch “Noms réels” fonctionne **dans tous les modes**
* ✅ Aucun champ affiché sans `confidence`
* ✅ Zéro géolocalisation précise (zones abstraites uniquement)
* ✅ Dataset publiable tel quel sur GitHub

👉 **Si S0 échoue, tout le reste est inutile.**

---

## 🟩 S1 — Audit & crédibilité (niveau “opendata sérieux”)

### Objectif

Montrer **ce qui est solide**, **ce qui est fragile**, **ce qui manque**.

### Livrables

1. **Audit automatique du dataset**

   * Compteurs :

     * entités sans preuves
     * claims sans evidence
     * relations sans evidence
   * Répartition par niveau de confiance (0–1)

2. **Vue “Sources”**

   * Liste des sources
   * Pour chaque source :

     * entités citées
     * claims cités
     * relations citées

3. **Filtres qualité**

   * `confidence >= X`
   * `hasEvidence:true`
   * “À vérifier”

### Critères de réussite

* ✅ Un journaliste / chercheur comprend **en 2 minutes** ce qui est fiable
* ✅ Tu peux dire :

  > “Voici exactement ce qui est documenté, et ce qui ne l’est pas encore.”
* ✅ Aucun mélange entre info prouvée et hypothèse

👉 **C’est ici que tu gagnes la légitimité.**

---

## 🟨 S2 — Exploitation humaine & partenariats

### Objectif

Transformer les données en **outil de travail réel** pour humains.

### Livrables

1. **Recherche guidée (non-tech)**

   * Builder visuel :

     * type de lieu
     * réseau
     * zone
     * niveau de confiance
   * Presets :

     * “Lieux bas seuil”
     * “Week-end”
     * “Bruxelles-Est”
     * “À renforcer”

2. **Exports thématiques**

   * Export CSV / JSON filtré :

     * pour partenaires
     * pour plaidoyer
     * pour appels à projets

3. **Mode lecture publique**

   * Interface “propre” sans jargon technique
   * Explications courtes :

     * ce qu’est une zone
     * ce qu’est un score de confiance

### Critères de réussite

* ✅ Une asbl peut s’en servir **sans toi**
* ✅ Un partenaire comprend la valeur **sans explication orale**
* ✅ Le portail peut être partagé publiquement sans honte

👉 **C’est ici que le projet devient utile, pas juste intéressant.**

---

## 🟥 S3 — Extension stratégique (ARG, pouvoir d’agir, suite)

### Objectif

Faire du portail un **socle vivant**, pas un musée.

### Livrables (au choix selon énergie)

1. **Patches incrémentaux**

   * `patch.json` pour ajouter / corriger sans écraser
   * Historique simple des évolutions

2. **Pont ARG / pédagogie**

   * Missions du type :

     * “renforcer une fiche”
     * “trouver une preuve”
     * “cartographier un lien manquant”
   * Progression visible (qualité des données ↑)

3. **Persistance locale (IndexedDB)**

   * Sauvegarde locale automatique
   * Historique personnel

### Critères de réussite

* ✅ Le dataset peut évoluer **sans chaos**
* ✅ Des gens peuvent contribuer sans casser
* ✅ Tu ouvres la porte à :

  * insertion
  * bénévolat
  * recherche
  * plaidoyer

👉 **Là, tu sors du “site” pour entrer dans l’écosystème.**

---

## 🧠 Règle d’or pour toute la roadmap

> **Pas de nouvelle feature tant que la précédente n’est pas auditable.**

C’est ce qui te protège :

* techniquement,
* mentalement,
* politiquement.

---

## 🏁 Si tu veux la suite

Je peux te fournir :

1. une **checklist S0 → S3** (case à cocher),
2. un **planning réaliste en semaines**,
3. ou une **priorisation “si je dois m’arrêter après S1”**.

Tu dis juste quoi.
