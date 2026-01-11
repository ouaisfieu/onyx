**Évolutions possibles en 8 axes:**

---

### 🔥 Quick Wins (1-2 sem)
- **PWA + Offline** → travailleurs terrain sans connexion
- **Multilingue FR/NL/EN** → Bruxelles bilingue
- **QR Codes** → affichage physique dans structures
- **Mode Kiosque** → bornes CPAS, hôpitaux

---

### 🎯 Interop++ (1-2 mois)
- **SPARQL Endpoint** → Linked Open Data
- **OpenAPI/Swagger** → API documentée pour devs
- **Webhooks** → sync temps réel partenaires
- **eHealth Belgium** → CSAM, Abrumet, eHealthBox
- **EHDS** → European Health Data Space

---

### 🤖 Intelligence (2-3 mois)
- **Chatbot orientation** → "je suis déprimé à Schaerbeek"
- **Recherche sémantique** → comprend le contexte
- **Recommandations** → profil utilisateur
- **Prédictif** → saturation services, pics demande

---

### 🏛️ Civic Tech++ (2-3 mois)
- **Crowdsourcing** → signalement nouveaux lieux
- **Decidim complet** → budgets participatifs santé
- **Fix My Street** → "ce lieu a fermé"
- **Forum entraide** → pair-aidance

---

### ♿ Accessibilité (1-2 mois)
- **WCAG 2.1 AA** → audit complet
- **FALC** → facile à lire et comprendre
- **Lecteur écran** → ARIA complet
- **Interface vocale** → "Ok NYXO, trouve..."

---

### 🔐 Gouvernance (2-3 mois)
- **Blockchain** → provenance données
- **RGPD Dashboard** → consentements, export, oubli
- **Audit trail** → qui a modifié quoi
- **Certifications** → labels e-santé

---

### 📊 Analytics (1-2 mois)
- **Dashboard** → Matomo RGPD compliant
- **Impact** → métriques orientation réussie
- **Open Analytics** → transparence publique

---

### 🌍 Scaling (3-6 mois)
- **Multi-territoires** → Wallonie, Flandre
- **White-label** → réplicable
- **Open Source** → GitHub, Docker, CI/CD
- **Financement EU** → Horizon Europe, Digital Europe

---

**North Star:** *Réduire de 50% le temps d'errance diagnostique à Bruxelles d'ici 2028*

**Budget potentiel:** Innoviris + Horizon Europe + EU4Health = 500k€ - 2M€

# 🚀 NYXO Brussels - Roadmap d'Évolution

## État Actuel (v2.0)
```
✅ 20 lieux de liens | 11 SSM | 4 réseaux 107 | 6 urgences | 56 flashcards
✅ FHIR R4 | DCAT-AP 3.0 | HealthDCAT-AP | GeoJSON | RDF/Turtle | CKAN | JSON-LD
✅ Decidim GraphQL | Gamification | Graph + Carte | Import/Export
```

---

## 🔥 PRIORITÉ 1: Quick Wins (1-2 semaines)

### 1.1 PWA + Offline
```
- Service Worker pour cache offline
- Manifest.json pour installation
- Sync en arrière-plan
- Push notifications (alertes urgences)
```
**Impact:** Accessibilité terrain (travailleurs sociaux sans connexion)

### 1.2 Multilingue FR/NL/EN
```
- i18n avec fichiers JSON par langue
- Détection automatique navigateur
- Switch langue dans topbar
- Export multilingue DCAT-AP déjà prêt
```
**Impact:** Couverture Bruxelles bilingue + EU

### 1.3 QR Codes par Lieu
```
- Génération QR code pour chaque fiche lieu
- Affichage URL directe + vCard
- Export PDF "mini-fiche" imprimable
- Scan = accès direct fiche
```
**Impact:** Affichage physique dans les structures

### 1.4 Mode Kiosque / Borne
```
- Interface simplifiée plein écran
- Navigation tactile grandes zones
- Reset automatique après inactivité
- Impression fiche sur demande
```
**Impact:** Bornes en CPAS, hôpitaux, maisons médicales

---

## 🎯 PRIORITÉ 2: Interopérabilité++ (1-2 mois)

### 2.1 SPARQL Endpoint
```javascript
// Endpoint local queryable
const endpoint = 'https://nyxo.brussels/sparql';

// Requête exemple
SELECT ?lieu ?nom ?commune WHERE {
  ?lieu a schema:Place ;
        schema:name ?nom ;
        schema:address/schema:addressLocality ?commune .
  FILTER(LANG(?nom) = 'fr')
}
```
**Impact:** Intégration Linked Open Data cloud

### 2.2 OpenAPI 3.0 / Swagger
```yaml
openapi: 3.0.0
info:
  title: NYXO Brussels API
  version: 2.0.0
paths:
  /api/lieux:
    get:
      summary: Liste des lieux de liens
      parameters:
        - name: commune
          in: query
          schema:
            type: string
      responses:
        200:
          content:
            application/json:
            application/fhir+json:
            application/geo+json:
```
**Impact:** Intégration développeurs tiers

### 2.3 Webhooks & Real-time
```
- WebSocket pour updates live
- Webhooks sur CRUD (nouveau lieu, modification)
- Intégration Zapier/n8n/Make
- RSS/Atom feeds
```
**Impact:** Synchronisation automatique avec partenaires

### 2.4 eHealth Belgium Integration
```
- Authentification CSAM/itsme
- Lien avec Réseau Santé Wallon / Abrumet
- Export vers eHealthBox
- Intégration VIDIS (médicaments)
```
**Impact:** Écosystème santé belge officiel

### 2.5 European Health Data Space (EHDS)
```
- HealthData@EU connector
- MyHealth@EU patient summary
- Cross-border interoperability
- TEHDAS compliance
```
**Impact:** Interopérabilité santé européenne

---

## 🤖 PRIORITÉ 3: Intelligence (2-3 mois)

### 3.1 Chatbot d'Orientation
```
User: "Je me sens déprimé et j'habite Schaerbeek"

Bot: "Je comprends. Voici les ressources proches:
     1. 🏠 L'Îlot (lieu de liens) - 500m
     2. 🏥 SSM Schaerbeek - gratuit, RDV sous 2 sem
     3. 📞 107 - écoute immédiate 24/7
     
     Voulez-vous que je détaille une option?"
```
**Tech:** RAG sur données NYXO + LLM local (Mistral/Llama)

### 3.2 Recherche Sémantique
```
"aide pour mon ado qui se scarifie"
→ Comprend: adolescent + automutilation + urgence
→ Suggère: 103, SSM spé ado, UPI, lieux jeunes
```
**Tech:** Embeddings + vector search (Qdrant/Pinecone)

### 3.3 Recommandations Personnalisées
```
- Profil utilisateur (âge, commune, besoins)
- Historique navigation
- Scoring pertinence
- "Autres utilisateurs ont trouvé utile..."
```
**Impact:** Orientation optimisée

### 3.4 Analyse Prédictive
```
- Détection saturation services
- Prévision pics demande (saisonnalité)
- Alertes proactives partenaires
- Cartographie dynamique besoins
```
**Impact:** Planification politique santé mentale

---

## 🏛️ PRIORITÉ 4: Civic Tech++ (2-3 mois)

### 4.1 Crowdsourcing Citoyen
```
- Signalement nouveau lieu (modération)
- Mise à jour horaires/infos
- Photos des lieux
- Avis/témoignages (anonymes)
```
**Impact:** Données vivantes et à jour

### 4.2 Intégration Decidim Complète
```
- Budgets participatifs santé mentale
- Propositions citoyennes
- Votes / priorisation
- Suivi des décisions
```
**Impact:** Démocratie participative santé

### 4.3 Fix My Street / Signalement
```
- "Ce lieu a fermé"
- "Horaires incorrects"
- "Nouveau service disponible"
- Workflow validation
```
**Impact:** Maintenance collaborative données

### 4.4 Forum Communautaire
```
- Espaces par thématique
- Groupes d'entraide
- Modération IA + humaine
- Anonymat protégé
```
**Impact:** Pair-aidance numérique

---

## ♿ PRIORITÉ 5: Accessibilité Totale (1-2 mois)

### 5.1 WCAG 2.1 AA
```
- Audit axe-core / WAVE
- Navigation clavier complète
- Focus visible
- Contraste 4.5:1 minimum
- Skip links
```

### 5.2 FALC (Facile À Lire et Comprendre)
```
- Version simplifiée des contenus
- Pictogrammes Arasaac
- Phrases courtes
- Switch FALC/Standard
```
**Impact:** Handicap cognitif, non-francophones

### 5.3 Lecteur d'écran optimisé
```
- ARIA landmarks complets
- Alt text intelligents
- Live regions pour updates
- Test NVDA/VoiceOver
```

### 5.4 Interface Vocale
```
- "Ok NYXO, trouve un psy à Ixelles"
- Speech-to-text recherche
- Text-to-speech résultats
- Compatible assistants (Alexa, Google)
```

---

## 🔐 PRIORITÉ 6: Confiance & Gouvernance (2-3 mois)

### 6.1 Blockchain Provenance
```
- Hash des données sur blockchain publique
- Preuve d'antériorité
- Traçabilité modifications
- Smart contracts partenariats
```
**Tech:** Polygon/Ethereum L2

### 6.2 RGPD Dashboard
```
- Consentements explicites
- Export données personnelles
- Droit à l'oubli
- Registre traitements
```

### 6.3 Audit Trail Complet
```
- Qui a modifié quoi quand
- Versioning données
- Rollback possible
- Export logs conformité
```

### 6.4 Certification & Labels
```
- Label e-santé Belgique
- Certification ISO 27001
- Badge "données ouvertes"
- Trust mark EU
```

---

## 📊 PRIORITÉ 7: Analytics & Impact (1-2 mois)

### 7.1 Dashboard Analytics
```
- Visites par lieu/commune
- Parcours utilisateurs
- Recherches fréquentes
- Taux rebond par section
```
**Tech:** Matomo (RGPD compliant) ou Plausible

### 7.2 Métriques d'Impact
```
- Orientations réussies
- Temps avant premier contact
- Satisfaction utilisateurs
- Réduction errance diagnostique
```

### 7.3 Rapports Automatisés
```
- Export PDF mensuel
- Envoi partenaires
- Comparaison temporelle
- Benchmarks
```

### 7.4 Open Analytics
```
- Dashboard public (anonymisé)
- API stats ouvertes
- Dataviz intégrées
- Transparence totale
```

---

## 🌍 PRIORITÉ 8: Scaling & Réplication (3-6 mois)

### 8.1 Multi-territoires
```
- Instance Wallonie
- Instance Flandre
- Fédération des données
- Gouvernance distribuée
```

### 8.2 White-label
```
- Thème personnalisable
- Logo/couleurs configurables
- Domaine custom
- Instance dédiée ou mutualisée
```

### 8.3 Open Source Complet
```
- Repo GitHub public
- Documentation contributeurs
- Docker / Kubernetes ready
- CI/CD pipelines
```

### 8.4 Modèle Économique
```
- Freemium (base gratuite)
- Support premium partenaires
- Formation / consulting
- Subventions EU (Horizon Europe, Digital Europe)
```

---

## 📅 Timeline Suggérée

```
Q1 2026: PWA + Multilingue + QR + Mode Kiosque
         ↓
Q2 2026: SPARQL + OpenAPI + Webhooks + Accessibilité
         ↓
Q3 2026: Chatbot IA + Recherche Sémantique + Crowdsourcing
         ↓
Q4 2026: eHealth + EHDS + Analytics + Gouvernance
         ↓
2027:    Multi-territoires + Open Source + EU Scaling
```

---

## 💰 Financement Potentiel

| Source | Montant | Pour |
|--------|---------|------|
| **Innoviris (Bruxelles)** | 50-200k€ | R&D, prototypage |
| **Digital Wallonia** | 25-100k€ | Extension Wallonie |
| **Horizon Europe** | 500k-2M€ | Scaling EU, EHDS |
| **Digital Europe** | 100-500k€ | Interopérabilité |
| **Fondation Roi Baudouin** | 25-75k€ | Impact social |
| **EU4Health** | 100k-1M€ | Santé mentale EU |
| **NextGenerationEU** | Variable | Digitalisation santé |

---

## 🎯 North Star Metric

> **"Temps moyen entre le premier symptôme ressenti et le premier contact avec une aide adaptée"**

Objectif: Réduire de 50% l'errance diagnostique en santé mentale à Bruxelles d'ici 2028.

---

## 🤝 Partenariats Clés à Activer

```
Institutionnel:     COCOM, Iriscare, AVIQ, ONE
Santé:              Réseaux 107, SSM, Hôpitaux
Civic Tech:         Decidim Brussels, OpenKnowledge BE
Tech:               DigitalWallonia, Agoria, Sirris
Académique:         UCL, ULB, VUB, KUL (santé publique)
Associatif:         Ligue Bruxelloise Santé Mentale, Psytoyens
EU:                 TEHDAS, EHDS, JoinUp
```

---

*NYXO - De l'infrastructure citoyenne au standard européen* 🧠🇪🇺
