# Estimate — Carenity

**Date:** 2026-05-08
**Based on:** qualification.md (2026-05-08)
**TJM:** 863€ (blended)
**Dispositif:** Mutualisé (<10 j/h/mois)
**Précision :** j/h exprimés au dixième de jour (0.1)

---

## Hypothèses de travail

| #   | Hypothèse               | Valeur retenue                  | Impact si fausse                                  |
| --- | ----------------------- | ------------------------------- | ------------------------------------------------- |
| H1  | Taille des EC2 prod     | Coefficient 0.8 (medium)        | ±190€/mois                                        |
| H2  | Volume Opensearch       | Coefficient 1 (medium, <100 Go) | +475€/mois si >100 Go                             |
| H3  | Roadmap évolutions 2026 | 0 j/h dans le forfait           | 20-30 j/h en temps passé si migrations confirmées |

---

## Analyse de réalisme

Le modèle déductif produit **6.81 j/h MCO/mois**. L'historique réel montre :

| Signal empirique     | Valeur                        |
| -------------------- | ----------------------------- |
| Tickets sur 12 mois  | ~5                            |
| Incidents            | 1 (PHP-FPM, pic de charge)    |
| Problèmes récurrents | 0                             |
| Évolutions infra     | 2-3 tickets (Opensearch, IAM) |

**Constat :** le déductif surestime l'effort réel. Cette infra LAMP est stable, peu complexe (pas de K8s, pas de microservices), avec un historique d'incidents quasi nul.

**Calibration empirique :**

| Composante                                       | j/h/mois | Raisonnement                                                       |
| ------------------------------------------------ | -------- | ------------------------------------------------------------------ |
| MCO réactif (incidents, demandes)                | 0.4      | 5 tickets/an ÷ 12 ≈ 0.42                                           |
| MCO proactif (monitoring, patching, veille sécu) | 1.3      | ×3 du réactif (LAMP simple) → 0.42 × 3 = 1.25                      |
| Buffer SLA Gold (coeff ×1.10 sur prod)           | 0.1      | ×1.10 sur portion prod (~70% du MCO) ; astreinte couverte par immo |
| Gouvernance                                      | 0.3      | COPROD trimestriel + 2 audits semestriels (ROSE, YAMAS)            |
| **Total calibré**                                | **2.1**  |                                                                    |

> **Note rounding** : MCO proactif utilise le multiplicateur LAMP simple (×3), pas le multiplicateur K8s/microservices (×5). Le buffer SLA Gold applique le coefficient ×1.10 sur la portion prod du MCO, pas un forfait arrondi à 1 j/h ; l'astreinte 24/7 est déjà couverte par l'immobilisation Complète (1 000€).

**Comparaison :**

| Approche               | j/h/mois | Prix/mois  | Écart |
| ---------------------- | -------- | ---------- | ----- |
| Déductive pure         | 7.23     | 7 239€     | —     |
| **Calibrée (retenue)** | **2.1**  | **2 812€** | -61%  |

Le prix retenu est basé sur l'estimation calibrée : le déductif sert de plafond, l'empirique de plancher. Sur cette infra ultra-stable (5 tickets/an), l'écart au déductif est important — c'est attendu et défendable.

---

## Synthèse

### Phase d'initialisation (one-shot)

Plateforme construite par Theodo : **Non**

> Monitoring et système d'agents IA sont **toujours** présents. Audit et remédiation prioritaire **n'apparaissent que si** la plateforme n'a pas été construite par Theodo (sinon, omettre les deux lignes).

- **Audit** : 5 j/h Lead Ops — cartographie ressources, qualité, résilience, sécurité, observabilité.
- **Remédiation prioritaire** (cible ROSE/YAMAS) : 5 j/h — docs, durcissement résilience, gaps qualité.
- **Mise en place du monitoring** : 1 j/h — métriques, alerting, dashboards, sondes, runbooks (réutilisation stack standard).
- **Mise en place du système d'agents IA** : omise (hors périmètre Carenity).

**Total initialisation : 11 j/h — 11 178€ HT (one-shot, payée une seule fois en début d'engagement)**

> Cette enveloppe est indépendante du prix mensuel récurrent ci-dessous.

---

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
| **Forfait**     | MCO + Gouvernance + Immobilisation                                  | 2.1      | 2 812€           |
| **Temps passé** | Évolutions (à la demande, TJM Ops 750€ / Lead Ops 1 200€ / DM 850€) | —        | Sur consommation |
|                 | **Total forfait**                                                   | **2.1**  | **2 812€**       |

**Total annuel : 33 744€ HT**

---

## Annexe — Détail du calcul

### MCO calibré

| Composante    | Détail                                                                              | j/h/mois |
| ------------- | ----------------------------------------------------------------------------------- | -------- |
| MCO réactif   | Incidents + demandes (5 tickets/an ÷ 12 ≈ 0.42)                                     | 0.4      |
| MCO proactif  | Monitoring, patching OS/DB, veille sécu (×3 du réactif LAMP simple → 1.25)          | 1.3      |
| Buffer SLA    | Coefficient Gold ×1.10 sur portion prod (~70% du MCO) ; astreinte couverte par immo | 0.1      |
| **Total MCO** |                                                                                     | **1.8**  |

### Gouvernance

| Activité    | Fréquence   | Effort  | j/h/mois (précis) |
| ----------- | ----------- | ------- | ----------------- |
| COPROD      | trimestriel | 0.5 j/h | 0.17              |
| ROSE        | semestriel  | 0.5 j/h | 0.08              |
| YAMAS (HDS) | semestriel  | 0.5 j/h | 0.08              |
| **Total**   |             |         | **0.33 → 0.3**    |

### Prix

| Ligne                                 | j/h/mois | Montant     |
| ------------------------------------- | -------- | ----------- |
| MCO                                   | 1.8      | 1 553€      |
| Gouvernance                           | 0.3      | 259€        |
| Immobilisation (Complète × Mutualisé) | —        | 1 000€      |
| **Total mensuel**                     | **2.1**  | **2 812€**  |
| **Total annuel**                      |          | **33 744€** |

### Référence déductive (plafond)

Pour mémoire, le modèle déductif pur donnait 7.23 j/h/mois (7 239€/mois). Le détail ressource par ressource est disponible dans l'historique de qualification.

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
