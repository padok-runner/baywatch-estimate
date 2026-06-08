# Estimation Infogérance Cloud

## Étapes

### 1. Collecter les informations client

Utiliser le [guide commercial](questions.md) pour poser les bonnes questions au client. Consigner les réponses dans `<client>/input.md`.

### 2. Qualification (`/qualify`)

Lancer le skill `/qualify`. Il parcourt les réponses, pose les questions complémentaires et génère `<client>/qualification.md`.

### 3. Estimation (`/estimate`)

Lancer le skill `/estimate`. Il calcule les jours/homme par mois et produit le chiffrage dans `<client>/estimate.md`.

### 4. Review par un Solutions Architect

Faire relire la qualification et l'estimation par un SA avant envoi au client.

## Améliorations futures

### Périmètre & pricing

- Ajouter le chiffrage de la **réversibilité** (sortie de prestation / transfert).
- Lister clairement les **hypothèses de design-to-cost**.
- Récupérer le **budget client** — pour information uniquement : ne doit pas influencer le sizing, mais indique si on est en posture de design-to-cost.

### Collecte & qualification

- Enrichir `questions.md` : demander l'**inventaire complet des ressources** (fournir des scripts AWS/k8s pour l'export), le **code IaC**, l'**extract des tickets des 6 derniers mois**, et les questions de **niveau de service / compliance**.
- Si les informations sont insuffisantes, l'agent doit **échouer proprement** et demander des compléments (ou proposer une enveloppe au temps homme) plutôt que de deviner.

### Terminologie & cohérence

- "MCO" → "support" ?
- "consommé" plutôt que "temps passé" ?
- Harmoniser la **langue** (français / anglais) dans les skills et les sorties.
