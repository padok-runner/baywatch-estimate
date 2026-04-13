# Estimate — CISAC (CIS-Net 2)

**Date:** 2026-04-13
**Based on:** qualification.md (2026-04-13)
**Dispositif:** Semi-dédié

---

## Hypothèses de travail

| #   | Hypothèse                      | Information manquante                                       | Valeur retenue                                                                 | Justification                                                                                                  | Impact si l'hypothèse est fausse                                                                             |
| --- | ------------------------------ | ----------------------------------------------------------- | ------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| H1  | Sizing Container Apps          | CPU/RAM des Container Apps individuelles non fourni         | Coeff 0.5 (very low)                                                           | Confirmé par le SA — service managé Azure, faible effort ops unitaire                                          | Faible — coeff déjà au minimum, pas de risque de sous-estimation significative                               |
| H2  | Volume de données DBs          | Volume PostgreSQL et MongoDB en Go non connu                | Coefficients retenus tels quels (PostgreSQL medium 1, MongoDB high 2)          | Sizing des instances Azure documenté dans le TAD (B2s / M40) — cohérent avec ces coefficients                  | ±300€/mois si le volume réel pousse un coefficient à la catégorie supérieure ou inférieure                   |
| H3  | Historique tickets / incidents | Infra pas encore en prod — aucune donnée empirique          | Calibration basée sur le profil d'infra (Azure PaaS, 6 envs, 16 microservices) | Approche standard pour les nouvelles plateformes sans historique                                               | ±2 000€/mois — l'estimation calibrée pourrait varier de ±30% selon le volume réel d'incidents post-lancement |
| H4  | FTEs actuels Castelis          | Nombre d'ETPs non communiqué                                | Pas de cross-check empirique possible                                          | Infra gérée par un prestataire externe — donnée non accessible                                                 | Pas d'impact direct sur le prix, mais empêche la validation croisée                                          |
| H5  | Inventaire envs légers         | DEV, INTEG, TEST, TRAINING non détaillés                    | ~25% de PROD par env (Container Apps + DBs réduites + services Azure minimaux) | Environnements activables/désactivables à la demande, pas de VMs dédiées                                       | ±1 500€/mois — 4 envs × incertitude sur l'inventaire                                                         |
| H6  | RTO/RPO                        | Non défini — DRP/BCP marqués TBD dans le TAD                | Pas d'engagement DR spécifique inclus dans le MCO de base                      | Sections TBD → le DR sera traité comme une évolution quand défini                                              | Impact potentiel sur le périmètre si le client attend du DR opérationnel dès le démarrage                    |
| H7  | Volume ETL / SFTP              | Volume de fichiers EDI/XML et nombre de sociétés non connus | Coefficients ETL et SFTP maintenus tels quels (high 2 et low 0.8)              | Profil standard pour une plateforme B2B d'échange de données musicales                                         | ±200€/mois si le volume réel est significativement différent                                                 |
| H8  | Évolutions mensuelles          | Backlog d'évolutions non chiffré en j/h                     | 5 j/h/mois                                                                     | Backlog principalement technique (monitoring, IaC, DR, sécurité) — équivalent à ~1 chantier technique par mois | ±2 000€/mois — directement proportionnel au volume réel d'évolutions                                         |

> **⚠ Sensibilité** : Les hypothèses H3 (calibration MCO) et H8 (évolutions) représentent ensemble ±4 000€/mois, soit ±25% du prix final. Il est recommandé de revalider le MCO après 3 mois d'exploitation et de préciser le backlog d'évolutions avant contractualisation.

---

## Analyse de réalisme

### Signaux empiriques disponibles

| Signal                | Valeur                                                                                             | Source        |
| --------------------- | -------------------------------------------------------------------------------------------------- | ------------- |
| Historique de tickets | **Aucun** — infra pas encore en prod                                                               | TAD           |
| FTEs actuels          | **Non connu** — infra gérée par Castelis                                                           | Qualification |
| Maturité infra        | **Faible** — mise en prod 27/04, monitoring/DR/sécurité TBD                                        | TAD           |
| Complexité techno     | **Moyenne-haute** — 16 microservices (NestJS + Java + Python), 2 DBs managées, services Azure PaaS | TAD           |
| Taille équipe dev     | **Petite** (2-5 devs) — volume de déploiements modéré                                              | Qualification |
| Profil Azure          | **PaaS-heavy** — Container Apps, DBs managées, services managés → surface d'incident réduite       | TAD           |

### Calibration empirique

| Composante                  | j/h/mois | Raisonnement                                                                                                                                                                                       |
| --------------------------- | -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| MCO réactif                 | 3.0      | Nouvelle plateforme, complexité moyenne. Estimation ~6 incidents/mois × 0.5 j/h. Azure PaaS réduit la surface d'incidents infrastructure.                                                          |
| MCO proactif                | 9.0      | 3× le réactif. Inclut : monitoring (setup + review), patching VMs/images, mises à jour sécurité, vérification backups, performance monitoring. Facteur 3× car Azure PaaS (pas de K8s self-hosted). |
| Buffer SLA Gold + astreinte | 1.5      | Gold (+1.0) + astreinte Étendue (+0.5). Overhead pour garantir les temps de réponse SLA et la couverture hors heures ouvrées standard.                                                             |
| **Total calibré MCO**       | **13.5** |                                                                                                                                                                                                    |

### Comparaison déductive vs calibrée

| Approche               | MCO j/h/mois | Total j/h/mois (MCO+Gov+Evol) | Prix/mois (Étendue) | Écart    |
| ---------------------- | ------------ | ----------------------------- | ------------------- | -------- |
| Déductive pure         | 22.10        | 27.77                         | 25 539€             | —        |
| **Calibrée (retenue)** | **13.50**    | **19.17**                     | **17 895€**         | **-30%** |

**Décision :** La déductive surestime le MCO de 64% par rapport à la calibrée. L'écart s'explique principalement par la multiplication mécanique des ressources × environnements dans le modèle déductif, qui ne reflète pas l'effort réel sur une infrastructure Azure PaaS-heavy où :

- Les services managés (Container Apps, DBs, Service Bus) réduisent significativement l'effort ops unitaire
- Les environnements légers (DEV/INTEG/TEST/TRAINING) nécessitent très peu de maintenance proactive
- Le tooling et les pipelines sont partagés entre environnements

**L'estimation calibrée (13.5 j/h/mois MCO) est retenue** car elle reflète mieux l'effort réel attendu. La déductive (22.10 j/h/mois) sert de plafond de référence.

---

## Synthèse

|                         | PROD                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         | NON-PROD                                                                                                                                                                                                                          | Transverse                                                                                                    |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| **Nom des envs**        | prod                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         | preprod, dev, integ, test, training                                                                                                                                                                                               | —                                                                                                             |
| **Inventaire**          | **Servers :**<br>1 VM Standard B4ls v2 (4 vCPU, 8GB)<br>**Bases de données :**<br>1 PostgreSQL 16.10 (Azure DB, B2s)<br>1 MongoDB 8.0 (Cosmos DB, M40)<br>**Applications off-the-shelf :**<br>Azure Container Apps (16 µservices)<br>Azure API Manager<br>Azure Service Bus, Azure Search, Redis<br>Azure Key Vault, App Configuration<br>Container Registry, NSG<br>**Applications custom :**<br>CN2 Web Platform (9 NestJS)<br>CN2 ETL Pipeline (5 Java SpringBoot)<br>CN2 dsearch (Python)<br>SFTP Server | **Servers :**<br>1 VM B4ls v2 (preprod uniquement)<br>**Bases de données :**<br>PostgreSQL + MongoDB par env (sizing réduit)<br>**Applications :**<br>Container Apps + services Azure (légers)<br>CN2 Apps (déploiement dev/test) | Azure Blob Storage (partagé)<br>Azure Monitoring<br>GitLab CI<br>SonarQube / ESLint                           |
| **Services**            | Maintien : Gestion des demandes de service, des incidents, des problèmes, des changements de version, de la continuité, de la surveillance et astreintes                                                                                                                                                                                                                                                                                                                                                     | Maintien : Gestion des demandes de service, des incidents, des problèmes, des changements de version, de la continuité                                                                                                            | Gouvernance : COPROD mensuel, Audits semestriels ROSE et LEAF<br>Evolutions : Gestion des Changements mineurs |
| **Niveaux de services** | Gold                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         | Bronze                                                                                                                                                                                                                            | -                                                                                                             |
| **Plages de service**   | _3 scénarios ci-dessous_                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     | Standard                                                                                                                                                                                                                          | -                                                                                                             |
| **Dispositif**          | Ops : 12.8 j/mois, Lead Ops : 4.3 j/mois, Delivery Manager : 2.0 j/mois                                                                                                                                                                                                                                                                                                                                                                                                                                      |                                                                                                                                                                                                                                   |                                                                                                               |

> _Note : les j/h indiqués sont les jours travaillés (MCO 13.5 + Gov 0.67 = 14.17). Le montant MCO intègre le coefficient SLA Gold (×1.10) sur la part PROD, d'où un montant supérieur à 14.17 × TJM._

### Prix mensuel €HT — Scénario 1 : Standard (Lun-Ven 9h30-18h30)

| Mode            | Périmètre                               | j/h/mois  | Montant €HT/mois |
| --------------- | --------------------------------------- | --------- | ---------------- |
| **Temps passé** | MCO (13.5 j/h) + Gouvernance (0.67 j/h) | 14.17     | 12 580€          |
| **Temps passé** | Évolutions                              | 5.00      | 4 315€           |
|                 | Immobilisation (Standard × Semi-dédié)  | —         | 0€               |
|                 | **Total**                               | **19.17** | **16 895€**      |

**Total annuel : 202 740€ HT**

### Prix mensuel €HT — Scénario 2 : Étendue (Lun-Sam 8h-22h) ⭐ Recommandé

| Mode            | Périmètre                               | j/h/mois  | Montant €HT/mois |
| --------------- | --------------------------------------- | --------- | ---------------- |
| **Temps passé** | MCO (13.5 j/h) + Gouvernance (0.67 j/h) | 14.17     | 12 580€          |
| **Temps passé** | Évolutions                              | 5.00      | 4 315€           |
|                 | Immobilisation (Étendue × Semi-dédié)   | —         | 1 000€           |
|                 | **Total**                               | **19.17** | **17 895€**      |

**Total annuel : 214 740€ HT**

_Tarif HNO (heures non ouvrées) : TJM × 1.5 = 1 295€/jour — facturé en sus sur intervention effective hors plage Standard._

### Prix mensuel €HT — Scénario 3 : Complète (7j/7, 24h/24)

| Mode            | Périmètre                               | j/h/mois  | Montant €HT/mois |
| --------------- | --------------------------------------- | --------- | ---------------- |
| **Temps passé** | MCO (13.5 j/h) + Gouvernance (0.67 j/h) | 14.17     | 12 580€          |
| **Temps passé** | Évolutions                              | 5.00      | 4 315€           |
|                 | Immobilisation (Complète × Semi-dédié)  | —         | 2 500€           |
|                 | **Total**                               | **19.17** | **19 395€**      |

**Total annuel : 232 740€ HT**

_Tarif HNO (heures non ouvrées) : TJM × 3 = 2 589€/jour — facturé en sus sur intervention effective hors plage Standard._

---

## Annexe A — Detailed Calculation

### MCO Breakdown (Déductif)

#### Environment: PROD

**SLA:** Gold (coeff 1.10)
**Plage:** Étendue (recommandée)

| Resource                    | Item Type                 | Base Rate | Coeff | MCO (j/h/mois) |
| --------------------------- | ------------------------- | --------- | ----- | -------------- |
| vm-cisnet2-apps-prod        | Public Cloud Managed VM   | 0.10      | 0.5   | 0.05           |
| PostgreSQL 16.10 (Azure DB) | Off-the-shelf application | 0.50      | 1     | 0.50           |
| MongoDB 8.0 (Cosmos DB M40) | Off-the-shelf application | 0.50      | 2     | 1.00           |
| Azure Container Apps        | Off-the-shelf application | 0.50      | 0.5   | 0.25           |
| Azure API Manager           | Off-the-shelf application | 0.50      | 2     | 1.00           |
| Azure Service Bus           | Off-the-shelf application | 0.50      | 1     | 0.50           |
| Azure Search                | Off-the-shelf application | 0.50      | 1     | 0.50           |
| Redis                       | Off-the-shelf application | 0.50      | 1     | 0.50           |
| Azure Key Vault             | Off-the-shelf application | 0.50      | 0.8   | 0.40           |
| Azure App Configuration     | Off-the-shelf application | 0.50      | 0.8   | 0.40           |
| Azure Container Registry    | Off-the-shelf application | 0.50      | 0.8   | 0.40           |
| NSG                         | Off-the-shelf application | 0.50      | 0.5   | 0.25           |
| CN2 Web Platform (9 NestJS) | Custom application        | 0.25      | 2     | 0.50           |
| CN2 ETL Pipeline (5 Java)   | Custom application        | 0.25      | 2     | 0.50           |
| CN2 dsearch (Python)        | Custom application        | 0.25      | 0.8   | 0.20           |
| SFTP Server                 | Custom application        | 0.25      | 0.8   | 0.20           |
| **Subtotal PROD**           |                           |           |       | **6.65**       |

#### Environment: PREPROD

**SLA:** Bronze (coeff 1.00)
**Plage:** Standard

| Resource                    | Item Type                 | Base Rate | Coeff | MCO (j/h/mois) |
| --------------------------- | ------------------------- | --------- | ----- | -------------- |
| vm-cisnet2-apps-preprod     | Public Cloud Managed VM   | 0.10      | 0.5   | 0.05           |
| PostgreSQL 16.10 (Azure DB) | Off-the-shelf application | 0.50      | 1     | 0.50           |
| MongoDB 8.0 (Cosmos DB M10) | Off-the-shelf application | 0.50      | 2     | 1.00           |
| Azure Container Apps        | Off-the-shelf application | 0.50      | 0.5   | 0.25           |
| Azure API Manager           | Off-the-shelf application | 0.50      | 2     | 1.00           |
| Azure Service Bus           | Off-the-shelf application | 0.50      | 1     | 0.50           |
| Azure Search                | Off-the-shelf application | 0.50      | 1     | 0.50           |
| Redis                       | Off-the-shelf application | 0.50      | 1     | 0.50           |
| Azure Key Vault             | Off-the-shelf application | 0.50      | 0.8   | 0.40           |
| Azure App Configuration     | Off-the-shelf application | 0.50      | 0.8   | 0.40           |
| Azure Container Registry    | Off-the-shelf application | 0.50      | 0.8   | 0.40           |
| NSG                         | Off-the-shelf application | 0.50      | 0.5   | 0.25           |
| CN2 Web Platform (9 NestJS) | Custom application        | 0.25      | 2     | 0.50           |
| CN2 ETL Pipeline (5 Java)   | Custom application        | 0.25      | 2     | 0.50           |
| CN2 dsearch (Python)        | Custom application        | 0.25      | 0.8   | 0.20           |
| SFTP Server                 | Custom application        | 0.25      | 0.8   | 0.20           |
| **Subtotal PREPROD**        |                           |           |       | **6.65**       |

#### Environment: NON-PROD léger (DEV, INTEG, TEST, TRAINING)

**SLA:** Bronze (coeff 1.00)
**Plage:** Standard

| Resource                   | Item Type                 | Base Rate | Coeff | MCO (j/h/mois) |
| -------------------------- | ------------------------- | --------- | ----- | -------------- |
| Azure Container Apps       | Off-the-shelf application | 0.50      | 0.5   | 0.25           |
| PostgreSQL (si dédié)      | Off-the-shelf application | 0.50      | 0.8   | 0.40           |
| MongoDB (si dédié)         | Off-the-shelf application | 0.50      | 1     | 0.50           |
| Azure services (groupés)   | Off-the-shelf application | 0.50      | 0.8   | 0.40           |
| CN2 Apps (groupées)        | Custom application        | 0.25      | 0.8   | 0.20           |
| **Subtotal par env léger** |                           |           |       | **1.75**       |
| **× 4 environnements**     |                           |           |       | **7.00**       |

#### Environment: Transverse

**SLA:** Bronze (coeff 1.00)
**Plage:** Standard

| Resource                | Item Type                 | Base Rate | Coeff | MCO (j/h/mois) |
| ----------------------- | ------------------------- | --------- | ----- | -------------- |
| Azure Blob Storage      | Off-the-shelf application | 0.50      | 0.8   | 0.40           |
| Azure Monitoring        | Off-the-shelf application | 0.50      | 1     | 0.50           |
| GitLab CI               | Off-the-shelf application | 0.50      | 1     | 0.50           |
| SonarQube / ESLint      | Off-the-shelf application | 0.50      | 0.8   | 0.40           |
| **Subtotal Transverse** |                           |           |       | **1.80**       |

#### MCO Summary (Déductif)

| Environment            | MCO (j/h/mois) | SLA Coeff | MCO ajusté (j/h prix) |
| ---------------------- | -------------- | --------- | --------------------- |
| PROD                   | 6.65           | 1.10      | 7.32                  |
| PREPROD                | 6.65           | 1.00      | 6.65                  |
| NON-PROD léger (×4)    | 7.00           | 1.00      | 7.00                  |
| Transverse             | 1.80           | 1.00      | 1.80                  |
| **Total MCO déductif** | **22.10**      |           | **22.77**             |

#### MCO Summary (Calibré — retenu)

| Environment           | MCO (j/h/mois) | SLA Coeff | MCO ajusté (j/h prix) |
| --------------------- | -------------- | --------- | --------------------- |
| PROD                  | 4.06           | 1.10      | 4.47                  |
| PREPROD               | 4.06           | 1.00      | 4.06                  |
| NON-PROD léger (×4)   | 4.28           | 1.00      | 4.28                  |
| Transverse            | 1.10           | 1.00      | 1.10                  |
| **Total MCO calibré** | **13.50**      |           | **13.91**             |

_Répartition calibrée proportionnelle aux poids déductifs par environnement._

### Governance

| Activité              | Fréquence              | Effort/session | j/h/mois          |
| --------------------- | ---------------------- | -------------- | ----------------- |
| COPROD                | 1 / mois (semi-dédié)  | 0.5 j/h        | 0.50              |
| COPIL                 | N/A (dédié uniquement) | —              | 0                 |
| ROSE                  | 1 / semestre           | 0.5 j/h        | 0.08              |
| YAMAS                 | N/A (pas HDS)          | —              | 0                 |
| LEAF                  | 1 / semestre           | 0.5 j/h        | 0.08              |
| **Total Gouvernance** |                        |                | **0.67 j/h/mois** |

### Evolutions

- Estimated: **5.0 j/h/mois**
- Basis: Backlog principalement technique issu des sections TBD du TAD :
  - Migration IaC Azure CLI → Terraform (~15 j/h sur 6 mois)
  - Mise en place monitoring Centreon/Kibana (~10 j/h sur 6 mois)
  - Implémentation DRP/BCP (~5 j/h sur 6 mois)
  - Hardening sécurité (~5 j/h sur 6 mois)
  - Travaux divers (backup, optimisations) (~5 j/h sur 6 mois)
  - Total estimé : ~40 j/h sur 6 mois ≈ ~7 j/h/mois, arrondi conservativement à 5 j/h/mois car certains chantiers pourraient être hors scope MCO

### Total Quantity

| Category                   | j/h/mois  |
| -------------------------- | --------- |
| MCO (calibré)              | 13.50     |
| MCO ajusté SLA (prix)      | 13.91     |
| Governance                 | 0.67      |
| Evolutions                 | 5.00      |
| **Total (j/h travaillés)** | **19.17** |

### Price Breakdown (Scénario Étendue — recommandé)

| Line             | j/h/mois | TJM  | Montant     |
| ---------------- | -------- | ---- | ----------- |
| MCO (ajusté SLA) | 13.91    | 863€ | 12 002€     |
| Governance       | 0.67     | 863€ | 578€        |
| Evolutions       | 5.00     | 863€ | 4 315€      |
| **Subtotal**     |          |      | **16 895€** |

| Immobilisation | Étendue × Semi-dédié | 1 000€ |
| -------------- | -------------------- | ------ |

| **Total mensuel** |     | **17 895€**  |
| ----------------- | --- | ------------ |
| **Total annuel**  |     | **214 740€** |

### Comparaison des 3 scénarios de plage horaire

| Plage          | Immobilisation | Total mensuel | Total annuel | HNO (si intervention) |
| -------------- | -------------- | ------------- | ------------ | --------------------- |
| Standard       | 0€             | 16 895€       | 202 740€     | N/A                   |
| **Étendue** ⭐ | **1 000€**     | **17 895€**   | **214 740€** | **1 295€/h**          |
| Complète       | 2 500€         | 19 395€       | 232 740€     | 2 589€/h              |

---

## Notes

- **HDS :** Non applicable — pas de données de santé
- **Nearshore :** Non discuté dans cette proposition. Si réduction de coûts souhaitée, à évoquer avec Hugo/Lila/Manu. Le pricing nearshore n'est pas calculé ici.
- **Engagement :** 1 an initial. Si passage à 3 ans, remise de -8% applicable (économie de ~1 430€/mois sur le scénario Étendue).
- **Mode d'engagement :** Temps passé avec enveloppe cappée et alerting budgétaire, tel que demandé par le client.
- **Transition Castelis → Theodo :** Un plan de transition devra être défini. Le coût de transition (knowledge transfer, documentation, accès) n'est pas inclus dans cette estimation récurrente.
- **Revalidation recommandée :** Après 3 mois d'exploitation, revalider le MCO calibré avec les données réelles (volume de tickets, temps de résolution, charge proactive).
