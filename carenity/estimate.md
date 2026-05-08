# Estimate — Carenity

**Date:** 2026-05-08
**Based on:** qualification.md (2026-05-08)
**TJM:** 863€ (blended)
**Dispositif:** Mutualisé (<10 j/h/mois)
**Précision :** j/h exprimés au dixième de jour (0.1)

---

## Hypothèses de travail

| #   | Hypothèse                                | Information manquante                            | Valeur retenue                  | Justification                                  | Impact si fausse                                                                                |
| --- | ---------------------------------------- | ------------------------------------------------ | ------------------------------- | ---------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| H1  | Taille des EC2 prod                      | Sizing exact non fourni                          | Coefficient 0.8 (medium)        | Profil standard pour Web LAMP                  | ±70€/mois si réel = 0.5 (petit) ou 1.0 (gros)                                                   |
| H2  | Volume Opensearch                        | Métriques de volume non fournies                 | Coefficient 1 (medium, <100 Go) | Hypothèse conservative LAMP+search             | +285€/mois si réel >100 Go (passage coefficient 2)                                              |
| H3  | Roadmap évolutions 2026                  | Non validée par le client                        | 0 j/h dans le forfait           | Migrations MySQL / Redis / Debian non confirmées | 20-30 j/h en temps passé si confirmées (à 863€/jour, soit ~17 000-26 000€ one-shot sur 2026) |

> **⚠ Sensibilité** : H2 (volume Opensearch) est l'hypothèse la plus sensible (+15% du forfait MCO si l'hypothèse de volume est invalide). À confirmer avant contractualisation.

---

## Calibration empirique

**Signaux empiriques observés (depuis qualification.md) :**

| Signal | Valeur |
|---|---|
| Tickets sur 12 mois | 5 → 0.42/mois |
| Incidents | 1 (PHP-FPM, pic de charge) |
| Problèmes récurrents | 0 |
| FTE empirique | Non communiqué |
| Stabilité observée | Très haute (1 incident en 12 mois, infra LAMP simple, pas de pattern récurrent) |

**Discount appliqué : −40% sur la MCO déductive**

**Justification :** Pas de FTE communiqué ; tickets nettement < 1/mois (0.42) sur 12 mois ; aucun problème récurrent → ligne *"No FTE; tickets < 1/mois on 12 mois; no recurring problems"* de la table de discounts du skill estimate (Step 3). Fourchette autorisée : −30% à −50%. Retenu **−40%** (milieu de fourchette) pour deux raisons opposées qui se compensent :
- Borne haute (−50%) tempérée par : contexte HDS régulé, 28 ressources, multi-engine DB (4 distincts) qui demandent une vigilance proactive même en l'absence de tickets.
- Borne basse (−30%) tempérée par : aucun incident majeur sur 12 mois, infra stable, pas de pattern récurrent → l'effort réel est inférieur à ce que prévoit l'abaque déductive.

**Ajustement MCO :**

| | Déductive | Discount | Final |
|---|---|---|---|
| MCO (toutes envs, SLA appliqué) | 6.81 | −40% | 4.1 |
| Gouvernance | 0.3 | (jamais discountée) | 0.3 |
| **Total mensuel j/h** | **7.1** | | **4.4** |

> **Garde-fous appliqués :**
> - Discount −40% < cap −50% ✓
> - Gouvernance non discountée (audits ROSE/YAMAS contractuels) ✓
> - SLA déjà appliqué au niveau ressource (Gold ×1.10 sur prod) — pas réappliqué ici ✓
> - Immobilisation séparée, jamais discountée ✓

---

## Synthèse

### Phase d'initialisation (one-shot)

Plateforme construite par Theodo : **Non**

> Monitoring et système d'agents IA sont **toujours** présents. Audit et remédiation prioritaire **n'apparaissent que si** la plateforme n'a pas été construite par Theodo (sinon, omettre les deux lignes).

- **Audit** : 5 j/h Lead Ops — cartographie ressources, qualité, résilience, sécurité, observabilité.
- **Remédiation prioritaire** (cible ROSE/YAMAS) : 5 j/h — docs, durcissement résilience, gaps qualité.
- **Mise en place du monitoring** : 1 j/h — métriques, alerting, dashboards, sondes, runbooks (réutilisation stack standard).
- **Mise en place du système d'agents IA** : omise (hors périmètre Carenity, décision SA/client documentée en qualification).

**Total initialisation : 11 j/h — 11 178€ HT (one-shot, payée une seule fois en début d'engagement)**

> Cette enveloppe est indépendante du prix mensuel récurrent ci-dessous.

---

### Tableau de synthèse mensuelle

|                         | Production                                                                                                 | Non-production                                                                 | Shared / Infra                         | Transverse                            |
| ----------------------- | ---------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------ | -------------------------------------- | ------------------------------------- |
| **Nom des envs**        | prod                                                                                                       | recette                                                                        | shared                                 | —                                     |
| **Inventaire**          | 5 EC2, 2 RDS MySQL 8, 1 MySQL 5 self-hosted, 1 Opensearch, 1 Redis, 8 apps custom                          | 3 EC2, 1 RDS, 1 Opensearch, 1 Redis, 1 MySQL 5                                 | 3 EC2 (bastion, packages, déploiement) | Organisation AWS, Terraform IaC       |
| **Services**            | Gestion des incidents, demandes, problèmes, changements de version, continuité, surveillance et astreintes | Gestion des incidents, demandes, problèmes, changements de version, continuité | Gestion des incidents, continuité      | COPROD trimestriel, Audits ROSE/YAMAS |
| **Niveaux de services** | Gold                                                                                                       | Bronze                                                                         | Bronze                                 | —                                     |
| **Plages de service**   | Complète (24/7)                                                                                            | Standard                                                                       | Standard                               | —                                     |

#### Prix mensuel €HT

| Mode            | Périmètre                                                           | j/h/mois | Montant €HT/mois |
| --------------- | ------------------------------------------------------------------- | -------- | ---------------- |
| **Forfait**     | MCO + Gouvernance + Immobilisation                                  | 4.4      | 4 797€           |
| **Temps passé** | Évolutions (à la demande, TJM Ops 750€ / Lead Ops 1 200€ / DM 850€) | —        | Sur consommation |
|                 | **Total forfait**                                                   | **4.4**  | **4 797€**       |

**Total annuel : 57 564€ HT**

---

## Annexe A — Détail du calcul

### MCO déductive — par environnement

#### Production (SLA Gold ×1.10)

| Ressource              | Item Type                         | Base Rate | Coeff | MCO base (j/h/mois) |
|------------------------|-----------------------------------|-----------|-------|---------------------|
| 5 EC2 prod (medium)    | Public Cloud Managed VM           | 0.1       | 0.8   | 0.40                |
| 2 RDS MySQL 8 (medium) | Off-the-shelf application         | 0.5       | 1.0   | 1.00                |
| 1 Opensearch (medium)  | Off-the-shelf application         | 0.5       | 1.0   | 0.50                |
| 1 Redis (low)          | Off-the-shelf application         | 0.5       | 0.8   | 0.40                |
| 1 MySQL 5 SH (medium)  | Off-the-shelf application         | 0.5       | 1.0   | 0.50                |
| 1 MPA Legacy (medium)  | Custom application                | 0.25      | 1.0   | 0.25                |
| 3 MPA CodeIgniter (low)| Custom application                | 0.25      | 0.8   | 0.60                |
| 2 SPA VueJS/Slim (low) | Custom application                | 0.25      | 0.8   | 0.40                |
| 2 apps infra (very low)| Custom application                | 0.25      | 0.5   | 0.25                |
| **Sous-total prod base** |                                 |           |       | **4.30**            |
| **× SLA Gold 1.10**    |                                   |           |       | **4.73**            |

#### Recette (SLA Bronze ×1.0)

| Ressource              | Item Type                         | Base Rate | Coeff | MCO (j/h/mois) |
|------------------------|-----------------------------------|-----------|-------|----------------|
| 3 EC2 recette (medium) | Public Cloud Managed VM           | 0.1       | 0.8   | 0.24           |
| 1 RDS (low, <15 Go)    | Off-the-shelf application         | 0.5       | 0.8   | 0.40           |
| 1 Opensearch (low)     | Off-the-shelf application         | 0.5       | 0.8   | 0.40           |
| 1 Redis (low)          | Off-the-shelf application         | 0.5       | 0.8   | 0.40           |
| 1 MySQL 5 SH (low)     | Off-the-shelf application         | 0.5       | 0.8   | 0.40           |
| **Sous-total recette** |                                   |           |       | **1.84**       |

#### Shared (SLA Bronze ×1.0)

| Ressource                              | Item Type                  | Base | Coeff | MCO (j/h/mois) |
|----------------------------------------|----------------------------|------|-------|----------------|
| 3 EC2 (bastion, packages, déploiement) | Public Cloud Managed VM    | 0.1  | 0.8   | 0.24           |
| **Sous-total shared**                  |                            |      |       | **0.24**       |

#### MCO Summary

| Environment              | MCO ajusté SLA |
|--------------------------|----------------|
| Production (Gold ×1.10)  | 4.73           |
| Recette (Bronze ×1.0)    | 1.84           |
| Shared (Bronze ×1.0)     | 0.24           |
| **Total MCO déductive**  | **6.81**       |

### Discount empirique

| Étape                              | j/h/mois |
|------------------------------------|----------|
| MCO déductive                      | 6.81     |
| × (1 − discount 40%)               | 4.09     |
| **MCO finale**                     | **4.1**  |

### Gouvernance

| Activité    | Fréquence   | Effort  | j/h/mois (précis) |
|-------------|-------------|---------|-------------------|
| COPROD      | trimestriel | 0.5 j/h | 0.17              |
| ROSE        | semestriel  | 0.5 j/h | 0.08              |
| YAMAS (HDS) | semestriel  | 0.5 j/h | 0.08              |
| **Total**   |             |         | **0.33 → 0.3**    |

### Total quantité

| Catégorie    | j/h/mois |
|--------------|----------|
| MCO finale   | 4.1      |
| Gouvernance  | 0.3      |
| Évolutions   | 0        |
| **Total**    | **4.4**  |

### Prix

| Ligne                                 | j/h/mois | TJM    | Montant     |
|---------------------------------------|----------|--------|-------------|
| MCO finale                            | 4.1      | 863€   | 3 538€      |
| Gouvernance                           | 0.3      | 863€   | 259€        |
| **Sous-total**                        | **4.4**  |        | **3 797€**  |
| Immobilisation (Complète × Mutualisé) | —        |        | 1 000€      |
| **Total mensuel**                     |          |        | **4 797€**  |
| **Total annuel**                      |          |        | **57 564€** |

---

## Annexe B — Initialisation (one-shot)

| Composante | Sizing | j/h | TJM | Montant €HT |
|------------|--------|-----|-----|-------------|
| Audit (Lead Ops) | Small (custom, ajusté à 5 j/h) | 5 | 1 200€ | 6 000€ |
| Remédiation prioritaire | Light | 5 | 863€ | 4 315€ |
| Mise en place du monitoring | Custom (sous-Simple, réutilisation stack standard) | 1 | 863€ | 863€ |
| Mise en place système d'agents IA | Omis (hors périmètre) | 0 | — | 0€ |
| **Total initialisation** | | **11** | | **11 178€** |

**Notes :**
- Plateforme **non** construite par Theodo → audit et remédiation inclus.
- Audit facturé au **TJM Lead Ops** (1 200€).
- Remédiation et monitoring facturés au **TJM blended** (863€).
- Agent IA omis sur décision SA/client (hors périmètre Carenity).
- Audit à 5 j/h : entre les paliers Small (2.5) et Medium (7) — ~28 ressources mais stack LAMP simple.
- Monitoring à 1 j/h : sous le palier Simple — réutilisation directe d'une stack monitoring standard sans dashboards custom.
- Cette enveloppe est **payée une seule fois** en début d'engagement et n'entre pas dans le prix mensuel récurrent ni dans la base de contingence forfait.

---

## Notes

- **HDS applicable** — Audit YAMAS inclus. Périmètre HDS exact à confirmer.
- **Engagement 1 an** — Pas de remise multi-annuelle.
- **Évolutions** — Les 3 migrations potentielles 2026 (MySQL, Redis, Debian) non incluses. Si confirmées : ~20-30 j/h en temps passé (863€/jour).
- **Méthodologie** — Le prix repose sur la MCO déductive (abaque `item × coeff × SLA`) avec discount empirique explicite de −40% justifié par la stabilité de l'infra (5 tickets/an, 1 incident, 0 problème récurrent). C'est la seule méthode du skill — pas de calcul parallèle de "calibration" avec multiplicateur.
