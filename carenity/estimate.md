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

**Verdict :** infra ultra-stable, mais le scaling sublinéaire de l'abaque (`multiplier(N) = min(N,3) + sqrt(max(N/3,1)) - 1`, racine carrée piecewise) intègre déjà l'amortissement de l'orchestration sur les ressources identiques sans aplatir abusivement comme le ferait du log10. Pas de discount appliqué.

Pas de FTE communiqué → pas de cross-check quantitatif possible. Si le client peut fournir une répartition FTE, on pourra valider que `MCO_déductive ≈ FTE × 20`.

---

## Synthèse

### Phase d'initialisation (one-shot)

Plateforme construite par Theodo : **Non**

> Monitoring et système d'agents IA sont **toujours** présents. Audit et remédiation prioritaire **n'apparaissent que si** la plateforme n'a pas été construite par Theodo.

- **Audit** : 5 j/h Lead Ops — cartographie ressources, qualité, résilience, sécurité, observabilité.
- **Remédiation prioritaire** (cible ROSE/YAMAS) : 4 j/h — docs, durcissement résilience, gaps qualité.
- **Mise en place du monitoring** : 1 j/h — métriques, alerting, dashboards, sondes, runbooks (réutilisation stack standard).
- **Mise en place du système d'agents IA** : omise (hors périmètre Carenity, décision SA/client documentée).

**Total initialisation : 10 j/h — 10 315€ HT (one-shot, payée une seule fois en début d'engagement)**

> Cette enveloppe est indépendante du prix mensuel récurrent ci-dessous.

---

### Tableau de synthèse mensuelle

|                                                            | Production                                                                                                                                              | Non-production                                                                                | Shared / Infra                              | Transverse                                                  |
| ---------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- | ------------------------------------------- | ----------------------------------------------------------- |
| **Nom des envs**                                           | prod                                                                                                                                                    | recette                                                                                       | shared                                      | —                                                           |
| **Inventaire**                                             | 5 EC2, 2 RDS MySQL 8, 1 MySQL 5 self-hosted, 1 Opensearch, 1 Redis, 8 apps custom                                                                       | 3 EC2, 1 RDS, 1 Opensearch, 1 Redis, 1 MySQL 5 self-hosted                                    | 3 EC2 (bastion, packages, déploiement)      | Organisation AWS, Terraform IaC                             |
| **Services au Forfait socle** *(engagés mensuellement)*    | Capacité réservée 24/7 — astreinte (immobilisation)                                                                                                     | —                                                                                             | —                                           | Gouvernance : COPROD trimestriel + Audits ROSE/YAMAS/LEAF   |
| **Services au Temps passé** *(facturés à la consommation)* | MCO : Gestion des incidents, demandes, problèmes, changements de version, continuité, surveillance, interventions d'astreinte, patching, monitoring drift | MCO : Gestion des incidents, demandes, problèmes, changements de version, continuité          | MCO : Gestion des incidents, continuité    | Évolutions : Gestion des changements mineurs                |
| **Niveaux de services**                                    | Gold                                                                                                                                                    | Bronze                                                                                        | Bronze                                      | —                                                           |
| **Plages de service**                                      | Complète (24/7)                                                                                                                                         | Standard                                                                                      | Standard                                    | —                                                           |

#### Prix mensuel €HT — Forfait socle + carnet temps passé

> Profil Carenity : 5 tickets / 12 mois, 1 incident / 12 mois, infra LAMP stable. Le forfait MCO classique ferait payer une enveloppe de 4.5 j/h que l'historique ne justifie pas. Le modèle proposé aligne **toute** la facturation MCO sur la consommation réelle. Le socle (gouvernance + immobilisation) couvre la cadence contractuelle, les audits HDS et la capacité 24/7.

| Mode                       | Périmètre                                                                                                                 | j/h/mois  | Montant €HT/mois |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------- | --------- | ---------------- |
| **Forfait socle**          | Gouvernance (0.4 — COPROD + ROSE + YAMAS + LEAF) + Immobilisation (Complète × Mutualisé)                                  | 0.4       | 1 345€           |
| **Temps passé MCO**        | MCO (toutes catégories : incidents, demandes, problèmes, changements, patching, monitoring). Facturé à la consommation réelle (TJM blended 863€). | sur conso | Sur consommation |
| **Temps passé Évolutions** | À la demande (TJM Ops 750€ / Lead Ops 1 200€ / DM 850€)                                                                   | —         | Sur consommation |
|                            | **Mensuel socle seul** (0 j/h MCO consommé)                                                                               | **0.4**   | **1 345€**       |
|                            | **Mensuel espéré** (1.5 j/h MCO/mois — historique 5 tickets/12 mois)                                                      | **1.9**   | **~2 640€**      |

**Référence enveloppe déductive (pour info — non plafonnée contractuellement) : 4.5 j/h MCO/mois → 5 229€/mois → 62 748€/an, équivalent à un Forfait étendu classique.**

**Estimations annuelles attendues :**
- **Annuel socle seul** : 16 140€ HT *(uniquement socle, 0 j/h MCO consommé sur l'année)*
- **Annuel espéré** : ~31 700€ HT *(basé sur l'historique : ~1.5 j/h MCO consommé/mois en moyenne)*

> **Conditions** : engagement contractuel ≥2 ans (remise -3% applicable, -8% si 3 ans). Le socle (immobilisation + gouvernance) protège le revenu sur les mois calmes ; la consommation MCO suit l'usage réel sans plancher ni plafond. Si la consommation dérive structurellement au-dessus de l'enveloppe déductive, revue contractuelle conjointe (avenant Forfait ou élargissement enveloppe).

---

## Annexe A — Détail du calcul

### MCO bucket-by-bucket (avec scaling sublinéaire)

`multiplier(N) = min(N, 3) + sqrt(max(N/3, 1)) - 1`

| Bucket (item type, coeff) | N | base | mult(N) | coeff | MCO base | Distribution par env (prorata count) | MCO après SLA |
|---|---|---|---|---|---|---|---|
| Public managed VM, 0.8 | 11 (5 prod + 3 rec + 3 sh) | 0.1 | 3.915 | 0.8 | 0.313 | prod 0.142 / rec 0.085 / sh 0.085 | 0.157 + 0.085 + 0.085 = **0.327** |
| Managed off-the-shelf, 1.0 | 3 (2 RDS prod + 1 OS prod) | 0.3 | 3.000 | 1.0 | 0.900 | prod 0.900 | 0.900 × 1.10 = **0.990** |
| Managed off-the-shelf, 0.8 | 4 (1 Redis prod + 1 RDS rec + 1 OS rec + 1 Redis rec) | 0.3 | 3.155 | 0.8 | 0.757 | prod 0.189 / rec 0.568 | 0.208 + 0.568 = **0.776** |
| Self-hosted off-the-shelf, 1.0 | 1 (MySQL 5 prod) | 0.6 | 1.000 | 1.0 | 0.600 | prod 0.600 | 0.600 × 1.10 = **0.660** |
| Self-hosted off-the-shelf, 0.8 | 1 (MySQL 5 recette) | 0.6 | 1.000 | 0.8 | 0.480 | rec 0.480 | 0.480 × 1.0 = **0.480** |
| Custom application, 1.0 | 1 (MPA Legacy) | 0.25 | 1.000 | 1.0 | 0.250 | prod 0.250 | 0.250 × 1.10 = **0.275** |
| Custom application, 0.8 | 5 (3 MPA CI + 2 SPA) | 0.25 | 3.291 | 0.8 | 0.658 | prod 0.658 | 0.658 × 1.10 = **0.724** |
| Custom application, 0.5 | 2 (Jobqueues + SSO) | 0.25 | 2.000 | 0.5 | 0.250 | prod 0.250 | 0.250 × 1.10 = **0.275** |

### MCO summary

| | j/h/mois |
|---|---|
| Sum buckets après SLA | 4.507 |
| **Total MCO (arrondi tenth)** | **4.5** |

### Gouvernance

| Activité    | Fréquence   | Effort  | j/h/mois (précis) |
|-------------|-------------|---------|-------------------|
| COPROD      | trimestriel | 0.5 j/h | 0.17              |
| ROSE        | semestriel  | 0.5 j/h | 0.08              |
| YAMAS (HDS) | semestriel  | 0.5 j/h | 0.08              |
| LEAF (FinOps / Green IT) | semestriel | 0.5 j/h | 0.08   |
| **Total**   |             |         | **0.42 → 0.4**    |

### Total quantité

| Catégorie    | j/h/mois |
|--------------|----------|
| MCO          | 4.5      |
| Gouvernance  | 0.4      |
| Évolutions   | 0        |
| **Total**    | **4.9**  |

### Prix — Forfait socle + carnet temps passé

| Ligne                                       | j/h/mois | TJM  | Montant         |
|---------------------------------------------|----------|------|-----------------|
| Gouvernance (COPROD + ROSE + YAMAS + LEAF)  | 0.4      | 863€ | 345€            |
| Immobilisation (Complète × Mutualisé)       | —        | —    | 1 000€          |
| **Forfait socle**                           | **0.4**  |      | **1 345€**      |
| MCO consommé (facturé à la réalité, TJM blended 863€) | sur conso | 863€ | Sur consommation |
| **Total mensuel — socle seul** (0 j/h consommé) | **0.4** |     | **1 345€**      |
| **Total mensuel — espérance** (1.5 j/h consommé, historique 5 tickets/12 mois) | 1.9 |  | **~2 640€**     |

**Annuel attendu** : socle seul **16 140€** | espérance **~31 700€**.

> Référence déductive `/estimate` (non plafonnée contractuellement) : MCO 4.5 j/h × 863€ + Gouv 0.4 j/h × 863€ + Immo 1 000€ = 5 229€/mois. Cette enveloppe sert de dimensionnement capacitaire et de point de comparaison avec un Forfait classique. Le modèle proposé aligne la facturation MCO sur la consommation réelle.

---

## Annexe B — Initialisation (one-shot)

| Composante | Sizing | j/h | TJM | Montant €HT |
|------------|--------|-----|-----|-------------|
| Audit (Lead Ops) | Small (custom, ajusté à 5 j/h) | 5 | 1 200€ | 6 000€ |
| Remédiation prioritaire | Light (ajustée à 4 j/h) | 4 | 863€ | 3 452€ |
| Mise en place du monitoring | Custom (sous-Simple, réutilisation stack standard) | 1 | 863€ | 863€ |
| Mise en place système d'agents IA | Omis (hors périmètre) | 0 | — | 0€ |
| **Total initialisation** | | **10** | | **10 315€** |

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

- **HDS applicable** — Audit YAMAS inclus dans le socle. Périmètre HDS exact à confirmer.
- **Engagement requis** — Le modèle proposé exige un engagement contractuel **≥2 ans** (remise -3%) ou **≥3 ans** (-8%). Sans engagement multi-annuel, repli sur un Forfait classique à 5 229€/mois (cf. `shared/pricing-rules.md`).
- **Évolutions** — Les 3 migrations potentielles 2026 (MySQL 8.4, Redis 8.4, Debian 13) non incluses. Si confirmées : ~20-30 j/h en temps passé (863€/jour).
- **Pourquoi ce modèle pour Carenity** — Historique ultra-stable (5 tickets/12 mois, 1 incident/12 mois) ne justifie pas de payer une enveloppe MCO de 4.5 j/h chaque mois. La consommation réelle attendue est ~1.5 j/h/mois → ~31 700€/an au lieu de 62 748€/an. Si l'infra se déstabilise, la facturation suit la consommation réelle (la référence déductive 4.5 j/h/mois sert de jalon de comparaison ; au-delà d'une dérive structurelle, revue contractuelle conjointe).
- **Méthodologie** — Le prix repose sur l'abaque déductive (`item × multiplier(N) × coeff × SLA`) avec scaling sublinéaire intégré pour les ressources identiques. Pas de discount empirique appliqué — le scaling capture déjà l'amortissement automation.
- **Self-hosted vs managed** — Les MySQL 5 (self-hosted) sont valorisés à 0.6 j/h base (vs 0.3 pour le managed RDS), reflétant l'overhead de patching DB manuel et tuning. Le cumul VM substrate + app self-hosted est intentionnel (la VM couvre l'OS, l'app couvre le moteur DB).
