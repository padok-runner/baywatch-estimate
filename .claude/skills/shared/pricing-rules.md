# Pricing Rules

## Dispositif d'équipe

| Dispositif | Quantité par client    | Description                                                             | Exemples                  |
| ---------- | ---------------------- | ----------------------------------------------------------------------- | ------------------------- |
| Mutualisé  | < 10 jour-homme/mois   | Equipe ~7 personnes pour ~20 clients                                    | Ontex, Relevanc, Beneteau |
| Semi-dédié | 10-100 jour-homme/mois | Equipe ~7 personnes pour <5 clients. Le client a un tech dédié au moins | GTT, BNPP, ANS            |
| Dédié      | > 100 jour-homme/mois  | Equipe ~7 personnes pour 1 client                                       | BPI, LCL, ANS (futur)     |

## Modes d'engagement

Deux modes de facturation, à combiner selon la configuration de périmètre choisie (voir section suivante).

### 1. Temps passé

- Enveloppe de jours-homme staffée et reportable sur le mois suivant uniquement (M+1).
- **Facturation à la consommation réelle**, plafonnée par l'enveloppe.
- Priorités dans l'enveloppe : incidents > problèmes > versions > changements mineurs.
- S'adapte aux besoins du client, au budget et aux aléas.

### 2. Forfait

- **Facturation engagée** : le client paie l'enveloppe en totalité, qu'il la consomme ou non.
- Enveloppe + contingence pour risque :
  - Pas d'incertitude : +0%
  - Incertitude faible : +10%
  - Incertitude moyenne : +20%
  - Incertitude haute : +30 à 40%
- **Le forfait n'est jamais appliqué aux évolutions** (par construction, hors-périmètre contingence).

## Configurations de périmètre (quel scope dans quel mode ?)

Le périmètre alloué à chaque mode est un **choix de configuration**, pas une règle fixe. Trois configurations courantes :

### Configuration A — Forfait étendu (par défaut)

- **Forfait** : MCO + Gouvernance + Immobilisation
- **Temps passé** : Évolutions uniquement
- **Profil client** : recherche la prédictibilité maximale du budget mensuel. Aucune incertitude sur le montant récurrent.

### Configuration B — Forfait socle + carnet temps passé

- **Forfait socle** : Gouvernance + Audits + Immobilisation. Couvre la cadence contractuelle (ceremonies, audits HDS/ROSE/LEAF) et la capacité réservée (24/7 si plage Étendue/Complète).
- **Temps passé** : **MCO (toutes catégories : incidents, demandes, problèmes, changements, patching, monitoring)** + Évolutions. Le client paie ce qu'il consomme, plafonné par l'enveloppe MCO déductive de `/estimate`.
- **Profil client** : infra stable, historique d'incidents faible, sensible au prix d'entrée. Permet un montant mensuel plancher significativement plus bas, avec montant réel proportionnel à la consommation.
- **Réservée aux clients engagés ≥2 ans** (voir règles de protection ci-dessous).

### Configuration C — Temps passé pur

- **Forfait** : Immobilisation uniquement (capacité réservée 24/7 si plage Étendue/Complète).
- **Temps passé** : tout le reste (MCO + Gouvernance + Évolutions).
- **Profil client** : rare. La gouvernance et les audits HDS ont une cadence contractuelle qui s'accommode mal d'une facturation pure consommation. Réservée à des contextes très spécifiques (PoC, audit-only, etc.).

## Règles de protection pour Configurations B et C

Le modèle repose sur la consommation pure côté MCO. La protection principale vient du **forfait socle** (immobilisation + gouvernance) et de l'**engagement multi-annuel**.

1. **Engagement contractuel ≥2 ans** : indispensable. La remise multi-annuelle (-3% pour 2 ans, -8% pour 3+ ans) s'applique normalement.

2. **Immobilisation comme protection capacitaire** : l'immobilisation mensuelle couvre la capacité réservée (24/7 si plage Étendue/Complète, slot équipe en mutualisé). C'est elle qui protège le revenu sur les mois calmes — pas un plancher artificiel sur le MCO.

3. **TJM facturé à la consommation** : MCO temps passé facturé au TJM blended (cf. `daily-rates.md`) ou au TJM par rôle pour les évolutions, selon ce qui est convenu contractuellement.

> Pas de plancher mensuel ni de plafond contractuel sur la consommation MCO. Si le client consomme zéro un mois, il paie le socle. Si la consommation explose, elle est facturée intégralement — ce qui doit déclencher une revue avec le client (peut-être faut-il passer en avenant Forfait, ou élargir l'enveloppe d'évolutions).

> Configuration A par défaut. Proposer Configuration B explicitement si (a) infra stable historiquement (<2 incidents/trimestre), (b) client en compétition sur le prix d'entrée, (c) engagement ≥2 ans acceptable.

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
