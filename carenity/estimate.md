# Estimate — Carenity

**Date:** 2026-03-30
**Based on:** qualification.md (2026-03-30)
**TJM:** 863€ (blended)
**Dispositif:** Mutualisé (<10 j/h/mois)

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

| Composante                                       | j/h/mois | Raisonnement                                            |
| ------------------------------------------------ | -------- | ------------------------------------------------------- |
| MCO réactif (incidents, demandes)                | 0.5      | ~5 tickets/an × ~1 j/h moyen                            |
| MCO proactif (monitoring, patching, veille sécu) | 2.5      | ~4× le réactif, standard pour infra stable              |
| Buffer SLA Gold + astreinte 24/7                 | 1.0      | Overhead d'engagement de service                        |
| Gouvernance                                      | 0.33     | COPROD trimestriel + 2 audits semestriels (ROSE, YAMAS) |
| **Total calibré**                                | **4.33** |                                                         |

**Comparaison :**

| Approche               | j/h/mois | Prix/mois  | Écart |
| ---------------------- | -------- | ---------- | ----- |
| Déductive pure         | 7.23     | 7 239€     | —     |
| **Calibrée (retenue)** | **4.33** | **4 737€** | -34%  |

Le prix retenu est basé sur l'estimation calibrée : le déductif sert de plafond, l'empirique de plancher.

---

## Synthèse

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
| **Forfait**     | MCO + Gouvernance + Immobilisation                                  | 4.33     | 4 737€           |
| **Temps passé** | Évolutions (à la demande, TJM Ops 750€ / Lead Ops 1 200€ / DM 850€) | —        | Sur consommation |
|                 | **Total forfait**                                                   | **4.33** | **4 737€**       |

**Total annuel : 56 844€ HT**

---

## Annexe — Détail du calcul

### MCO calibré

| Composante    | Détail                                                             | j/h/mois |
| ------------- | ------------------------------------------------------------------ | -------- |
| MCO réactif   | Incidents + demandes de service (~5 tickets/an)                    | 0.50     |
| MCO proactif  | Monitoring, patching OS/DB, veille sécurité, mises à jour mineures | 2.50     |
| Buffer SLA    | Overhead Gold 24/7 (astreinte, engagement de temps de réponse)     | 1.00     |
| **Total MCO** |                                                                    | **4.00** |

### Gouvernance

| Activité    | Fréquence   | Effort  | j/h/mois |
| ----------- | ----------- | ------- | -------- |
| COPROD      | trimestriel | 0.5 j/h | 0.17     |
| ROSE        | semestriel  | 0.5 j/h | 0.08     |
| YAMAS (HDS) | semestriel  | 0.5 j/h | 0.08     |
| **Total**   |             |         | **0.33** |

### Prix

| Ligne                                 | j/h/mois | Montant     |
| ------------------------------------- | -------- | ----------- |
| MCO                                   | 4.00     | 3 452€      |
| Gouvernance                           | 0.33     | 285€        |
| Immobilisation (Complète × Mutualisé) | —        | 1 000€      |
| **Total mensuel**                     | **4.33** | **4 737€**  |
| **Total annuel**                      |          | **56 844€** |

### Référence déductive (plafond)

Pour mémoire, le modèle déductif pur donnait 7.23 j/h/mois (7 239€/mois). Le détail ressource par ressource est disponible dans l'historique de qualification.

---

## Notes

- **HDS applicable** — Audit YAMAS inclus. Périmètre HDS exact à confirmer.
- **Engagement 1 an** — Pas de remise multi-annuelle.
- **Évolutions** — Les 3 migrations potentielles 2026 (MySQL, Redis, Debian) non incluses. Si confirmées : ~20-30 j/h en temps passé (863€/jour).
