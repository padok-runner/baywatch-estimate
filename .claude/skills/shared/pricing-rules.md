# Pricing Rules

## Dispositif d'équipe

| Dispositif | Quantité par client    | Description                                                             | Exemples                  |
| ---------- | ---------------------- | ----------------------------------------------------------------------- | ------------------------- |
| Mutualisé  | < 10 jour-homme/mois   | Equipe ~7 personnes pour ~20 clients                                    | Ontex, Relevanc, Beneteau |
| Semi-dédié | 10-100 jour-homme/mois | Equipe ~7 personnes pour <5 clients. Le client a un tech dédié au moins | GTT, BNPP, ANS            |
| Dédié      | > 100 jour-homme/mois  | Equipe ~7 personnes pour 1 client                                       | BPI, LCL, ANS (futur)     |

## Modes d'engagement

### 1. Temps passé (par défaut)

- Enveloppe de jours-homme staffée et reportable sur le mois suivant uniquement (M+1).
- Priorités dans l'enveloppe : incidents > problèmes > versions > changements mineurs.
- S'adapte aux besoins du client, au budget et aux aléas.

### 2. Forfait

- Enveloppe + contingence pour risque :
  - Pas d'incertitude : +0%
  - Incertitude faible : +10%
  - Incertitude moyenne : +20%
  - Incertitude haute : +30 à 40%
- **Le forfait n'est PAS sur les jours d'évolution** (uniquement MCO + gouvernance).

## Remises multi-annuelles

| Durée d'engagement | Remise |
| ------------------ | ------ |
| 2 ans              | -3%    |
| 3 ans ou plus      | -8%    |

- Possibilité de rompre le contrat à partir de 6 mois de collaboration avec préavis de 3 mois.

## Immobilisation

Les frais d'immobilisation dépendent du dispositif et de la plage horaire. Voir `service-levels.md` pour le tableau complet.

## HDS & Nearshore

- **HDS** : proposé par défaut en France (équipes Cloud plus développées en France).
- **Nearshore** : si le client souhaite réduire les coûts → en discuter avec Hugo/Lila/Manu.

## Abaques gouvernance & audits

### COPROD

| Dispositif | Fréquence     | Effort estimé       |
| ---------- | ------------- | ------------------- |
| Mutualisé  | 1 / trimestre | 0.5 j/h par session |
| Semi-dédié | 1 / mois      | 0.5 j/h par session |
| Dédié      | 1 / semaine   | 0.5 j/h par session |

### COPIL

| Dispositif       | Fréquence     | Effort estimé       |
| ---------------- | ------------- | ------------------- |
| Dédié uniquement | 1 / trimestre | 0.5 j/h par session |

### Audits

| Audit                    | Fréquence    | Effort estimé     |
| ------------------------ | ------------ | ----------------- |
| ROSE (Qualité)           | 1 / semestre | 0.5 j/h par audit |
| YAMAS (HDS / ISO 27001)  | 1 / semestre | 0.5 j/h par audit |
| LEAF (FinOps / Green IT) | 1 / semestre | 0.5 j/h par audit |

> YAMAS applicable uniquement si périmètre HDS.

## Facteur de conversion FTE

- 1 FTE = 20 j/h/mois (base : ~20 jours ouvrés par mois)

## Formule de prix finale

```
Prix mensuel = (Total j/h MCO × coeff SLA × TJM) + Gouvernance + Evolutions + Immobilisation

Où :
- Total j/h MCO = somme de (item_rate × coeff_size_complexity) pour chaque ressource par env
- Gouvernance = selon abaques ci-dessus (COPIL + COPROD + audits), dépend du dispositif
- Evolutions = estimées comme du build
- coeff SLA = voir `service-levels.md` — par environnement
- TJM = taux journalier moyen (selon grille Theodo)
- Immobilisation = selon dispositif × plage horaire

Si forfait :
  Prix forfait = (MCO + Gouvernance) × (1 + contingence%) — hors évolutions

Si multi-annuel :
  Prix final = Prix × (1 - remise%)
```
