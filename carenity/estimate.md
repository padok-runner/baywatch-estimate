# Estimate — Carenity

**Date:** 2026-05-09
**Based on:** qualification.md (2026-05-08)
**TJM:** 863€ (blended)
**Dispositif:** Mutualisé (<10 j/h/mois)
**Précision :** j/h exprimés au dixième de jour (0.1)

---

## Hypothèses de travail

| #   | Hypothèse                      | Information manquante                | Valeur retenue                  | Justification                          | Impact si fausse                                                                  |
| --- | ------------------------------ | ------------------------------------ | ------------------------------- | -------------------------------------- | --------------------------------------------------------------------------------- |
| H1  | Taille des EC2 prod            | Sizing exact non fourni              | Coefficient 0.8 (medium)        | Profil standard pour Web LAMP          | ±100€/mois si réel = 0.5 (petit) ou 1.0 (gros)                                    |
| H2  | Volume Opensearch              | Métriques de volume non fournies     | Coefficient 1 (medium, <100 Go) | Hypothèse conservative LAMP+search     | +250€/mois si réel >100 Go (passage coefficient 2)                                |
| H3  | Roadmap évolutions 2026        | Non validée par le client            | 0 j/h dans le forfait           | Migrations non confirmées              | 20-30 j/h en temps passé si confirmées (~17 000-26 000€ one-shot sur 2026)        |

> **⚠ Sensibilité** : H2 est l'hypothèse la plus sensible. À confirmer avant contractualisation.

---

## Cross-check empirique

> Ce bloc est un **sanity check**, pas un ajustement. La méthode déductive (abaque + scaling sublinéaire) produit le chiffre final tel quel.

**Signaux empiriques observés (depuis qualification.md) :**

| Signal | Valeur |
|---|---|
| Tickets sur 12 mois | 5 → 0.42/mois |
| Incidents | 1 (PHP-FPM, pic de charge) |
| Problèmes récurrents | 0 |
| FTE empirique | Non communiqué |
| Stabilité observée | Très haute (1 incident en 12 mois, infra LAMP simple, pas de pattern récurrent) |

**Verdict :** infra ultra-stable, mais le scaling sublinéaire de l'abaque (`multiplier(N) = min(N,3) + log10(max(N/3,1))`) intègre déjà l'amortissement de l'orchestration sur les ressources identiques. Pas de discount appliqué.

Pas de FTE communiqué → pas de cross-check quantitatif possible. Si le client peut fournir une répartition FTE, on pourra valider que `MCO_déductive ≈ FTE × 20`.

---

## Synthèse

### Phase d'initialisation (one-shot)

Plateforme construite par Theodo : **Non**

> Monitoring et système d'agents IA sont **toujours** présents. Audit et remédiation prioritaire **n'apparaissent que si** la plateforme n'a pas été construite par Theodo.

- **Audit** : 5 j/h Lead Ops — cartographie ressources, qualité, résilience, sécurité, observabilité.
- **Remédiation prioritaire** (cible ROSE/YAMAS) : 5 j/h — docs, durcissement résilience, gaps qualité.
- **Mise en place du monitoring** : 1 j/h — métriques, alerting, dashboards, sondes, runbooks (réutilisation stack standard).
- **Mise en place du système d'agents IA** : omise (hors périmètre Carenity, décision SA/client documentée).

**Total initialisation : 11 j/h — 11 178€ HT (one-shot, payée une seule fois en début d'engagement)**

> Cette enveloppe est indépendante du prix mensuel récurrent ci-dessous.

---

### Tableau de synthèse mensuelle

|                         | Production                                                                                                 | Non-production                                                                 | Shared / Infra                         | Transverse                            |
| ----------------------- | ---------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------ | -------------------------------------- | ------------------------------------- |
| **Nom des envs**        | prod                                                                                                       | recette                                                                        | shared                                 | —                                     |
| **Inventaire**          | 5 EC2, 2 RDS MySQL 8, 1 MySQL 5 self-hosted, 1 Opensearch, 1 Redis, 8 apps custom                          | 3 EC2, 1 RDS, 1 Opensearch, 1 Redis, 1 MySQL 5 self-hosted                     | 3 EC2 (bastion, packages, déploiement) | Organisation AWS, Terraform IaC       |
| **Services**            | Gestion des incidents, demandes, problèmes, changements de version, continuité, surveillance et astreintes | Gestion des incidents, demandes, problèmes, changements de version, continuité | Gestion des incidents, continuité      | COPROD trimestriel, Audits ROSE/YAMAS |
| **Niveaux de services** | Gold                                                                                                       | Bronze                                                                         | Bronze                                 | —                                     |
| **Plages de service**   | Complète (24/7)                                                                                            | Standard                                                                       | Standard                               | —                                     |

#### Prix mensuel €HT

| Mode            | Périmètre                                                           | j/h/mois | Montant €HT/mois |
| --------------- | ------------------------------------------------------------------- | -------- | ---------------- |
| **Forfait**     | MCO + Gouvernance + Immobilisation                                  | 4.8      | 5 142€           |
| **Temps passé** | Évolutions (à la demande, TJM Ops 750€ / Lead Ops 1 200€ / DM 850€) | —        | Sur consommation |
|                 | **Total forfait**                                                   | **4.8**  | **5 142€**       |

**Total annuel : 61 704€ HT**

---

## Annexe A — Détail du calcul

### MCO bucket-by-bucket (avec scaling sublinéaire)

`multiplier(N) = min(N, 3) + log10(max(N/3, 1))`

| Bucket (item type, coeff) | N | base | mult(N) | coeff | MCO base | Distribution par env (prorata count) | MCO après SLA |
|---|---|---|---|---|---|---|---|
| Public managed VM, 0.8 | 11 (5 prod + 3 rec + 3 sh) | 0.1 | 3.564 | 0.8 | 0.285 | prod 0.130 / rec 0.078 / sh 0.078 | 0.143 + 0.078 + 0.078 = **0.298** |
| Managed off-the-shelf, 1.0 | 3 (2 RDS prod + 1 OS prod) | 0.3 | 3.000 | 1.0 | 0.900 | prod 0.900 | 0.900 × 1.10 = **0.990** |
| Managed off-the-shelf, 0.8 | 4 (1 Redis prod + 1 RDS rec + 1 OS rec + 1 Redis rec) | 0.3 | 3.125 | 0.8 | 0.750 | prod 0.188 / rec 0.563 | 0.206 + 0.563 = **0.769** |
| Self-hosted off-the-shelf, 1.0 | 1 (MySQL 5 prod) | 0.6 | 1.000 | 1.0 | 0.600 | prod 0.600 | 0.600 × 1.10 = **0.660** |
| Self-hosted off-the-shelf, 0.8 | 1 (MySQL 5 recette) | 0.6 | 1.000 | 0.8 | 0.480 | rec 0.480 | 0.480 × 1.0 = **0.480** |
| Custom application, 1.0 | 1 (MPA Legacy) | 0.25 | 1.000 | 1.0 | 0.250 | prod 0.250 | 0.250 × 1.10 = **0.275** |
| Custom application, 0.8 | 5 (3 MPA CI + 2 SPA) | 0.25 | 3.222 | 0.8 | 0.644 | prod 0.644 | 0.644 × 1.10 = **0.708** |
| Custom application, 0.5 | 2 (Jobqueues + SSO) | 0.25 | 2.000 | 0.5 | 0.250 | prod 0.250 | 0.250 × 1.10 = **0.275** |

### MCO summary

| | j/h/mois |
|---|---|
| Sum buckets après SLA | 4.455 |
| **Total MCO (arrondi tenth)** | **4.5** |

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
| MCO          | 4.5      |
| Gouvernance  | 0.3      |
| Évolutions   | 0        |
| **Total**    | **4.8**  |

### Prix

| Ligne                                 | j/h/mois | TJM    | Montant     |
|---------------------------------------|----------|--------|-------------|
| MCO                                   | 4.5      | 863€   | 3 884€      |
| Gouvernance                           | 0.3      | 863€   | 259€        |
| **Sous-total**                        | **4.8**  |        | **4 142€**  |
| Immobilisation (Complète × Mutualisé) | —        |        | 1 000€      |
| **Total mensuel**                     |          |        | **5 142€**  |
| **Total annuel**                      |          |        | **61 704€** |

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
- **Évolutions** — Les 3 migrations potentielles 2026 (MySQL 8.4, Redis 8.4, Debian 13) non incluses. Si confirmées : ~20-30 j/h en temps passé (863€/jour).
- **Méthodologie** — Le prix repose sur l'abaque déductive (`item × multiplier(N) × coeff × SLA`) avec scaling sublinéaire intégré pour les ressources identiques. Pas de discount empirique appliqué — le scaling capture déjà l'amortissement automation.
- **Self-hosted vs managed** — Les MySQL 5 (self-hosted) sont valorisés à 0.6 j/h base (vs 0.3 pour le managed RDS), reflétant l'overhead de patching DB manuel et tuning. Le cumul VM substrate + app self-hosted est intentionnel (la VM couvre l'OS, l'app couvre le moteur DB).
