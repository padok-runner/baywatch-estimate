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
