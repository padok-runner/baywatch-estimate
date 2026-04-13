# Qualification — CISAC (CIS-Net 2)

**Date:** 2026-04-13
**Approach:** Deductive
**Solutions Architect:** —

## Client Context

CISAC (Confédération Internationale des Sociétés d'Auteurs et Compositeurs) opère la plateforme **CIS-Net 2 (CN2)**, une plateforme modulaire microservices destinée à centraliser, gérer et exposer les oeuvres musicales et leurs métadonnées pour les sociétés membres à l'international. L'infrastructure est hébergée sur **Azure France Central** et n'est **pas encore en production** (mise en prod prévue le 27 avril 2026). L'infrastructure actuelle est gérée par **Castelis** (Hunik Group / PMP Strategy).

Le client recherche un partenaire pour l'infogérance cloud de cette plateforme, avec une enveloppe budgétaire bien cappée et un système d'alerting budgétaire clair.

### Deductive Data

- **Dev team size:** Petite (2-5 développeurs)
- **Evolution backlog:** Principalement technique — de nombreuses sections du TAD (Technical Architecture Document) sont marquées "TBD" :
  - Monitoring (Centreon, Kibana) : non encore déployé
  - Disaster Recovery Plan : TBD
  - Business Continuity Plan : TBD
  - Sécurité : TBD
  - Migration IaC : Azure CLI scripts → Terraform (planifié mais pas encore réalisé)
  - Backup : détails TBD
  - Peu d'évolutions fonctionnelles côté infrastructure anticipées

## Resource Inventory

### Environment: PROD

**Plage horaire:** Étendue (recommandée) — 3 scénarios à chiffrer (Standard / Étendue / Complète)
**SLA:** Gold (coeff 1.10)

| Resource | Type | Cloud | Size | Complexity |
|----------|------|-------|------|------------|
| vm-cisnet2-apps-prod | Public Cloud Managed VM | Public | 4 vCPU, 8GB RAM, SSD 30GB + HDD 200GB, Ubuntu 24.04 | S — coeff 0.5 |
| PostgreSQL 16.10 (Azure DB for PostgreSQL) | Off-the-shelf application | Public | Standard_B2s, 2 vCores, 4GB RAM, SSD Premium 256GB | Medium — coeff 1 |
| MongoDB 8.0 (Azure Cosmos DB) | Off-the-shelf application | Public | M40, 4 vCores, 16GB RAM, 32GB | High — coeff 2 |
| Azure Container Apps | Off-the-shelf application | Public | Héberge 16 microservices, sizing Container Apps TBD | Very low — coeff 0.5 |
| Azure API Manager | Off-the-shelf application | Public | Authentification, throttling, routage API | High — coeff 2 |
| Azure Service Bus | Off-the-shelf application | Public | Messaging asynchrone (queues & topics) | Medium — coeff 1 |
| Azure Search | Off-the-shelf application | Public | Moteur de recherche | Medium — coeff 1 |
| Redis | Off-the-shelf application | Public | Cache in-memory | Medium — coeff 1 |
| Azure Key Vault | Off-the-shelf application | Public | Gestion secrets/certificats, instances multiples | Low — coeff 0.8 |
| Azure App Configuration | Off-the-shelf application | Public | Paramétrage centralisé | Low — coeff 0.8 |
| Azure Container Registry (acrcisnet2prod) | Off-the-shelf application | Public | Registre d'images Docker | Low — coeff 0.8 |
| NSG (Network Security Groups) | Off-the-shelf application | Public | Règles de sécurité réseau | Very low — coeff 0.5 |
| CN2 Web Platform (9 services NestJS : front, user, business, code, search, aisearch, manager, notif, submit) | Custom application | Public | 9 microservices Node.js/Next.js sur Container Apps | High — coeff 2 |
| CN2 ETL Pipeline (5 services Java SpringBoot : etl, parser, preparer, listener, soc) | Custom application | Public | 5 microservices Java sur Container Apps + VM | High — coeff 2 |
| CN2 dsearch (Python) | Custom application | Public | 1 service Python sur Container Apps | Low — coeff 0.8 |
| SFTP Server | Custom application | Public | Échange de fichiers avec les sociétés membres | Low — coeff 0.8 |

### Environment: PREPROD

**Plage horaire:** Standard (Lun-Ven 9h30-18h30)
**SLA:** Bronze (coeff 1.00)

| Resource | Type | Cloud | Size | Complexity |
|----------|------|-------|------|------------|
| vm-cisnet2-apps-preprod | Public Cloud Managed VM | Public | 4 vCPU, 8GB RAM, SSD 30GB + HDD 200GB, Ubuntu 24.04 | S — coeff 0.5 |
| PostgreSQL 16.10 (Azure DB for PostgreSQL) | Off-the-shelf application | Public | Standard_B2s, 2 vCores, 4GB RAM, SSD Premium 256GB | Medium — coeff 1 |
| MongoDB 8.0 (Azure Cosmos DB) | Off-the-shelf application | Public | M10, 1 vCore, 2GB RAM, 32GB | High — coeff 2 |
| Azure Container Apps | Off-the-shelf application | Public | Héberge 16 microservices | Very low — coeff 0.5 |
| Azure API Manager | Off-the-shelf application | Public | Authentification, throttling, routage API | High — coeff 2 |
| Azure Service Bus | Off-the-shelf application | Public | Messaging asynchrone | Medium — coeff 1 |
| Azure Search | Off-the-shelf application | Public | Moteur de recherche | Medium — coeff 1 |
| Redis | Off-the-shelf application | Public | Cache in-memory | Medium — coeff 1 |
| Azure Key Vault | Off-the-shelf application | Public | Gestion secrets/certificats | Low — coeff 0.8 |
| Azure App Configuration | Off-the-shelf application | Public | Paramétrage centralisé | Low — coeff 0.8 |
| Azure Container Registry (acrcisnet2preprod) | Off-the-shelf application | Public | Registre d'images Docker | Low — coeff 0.8 |
| NSG (Network Security Groups) | Off-the-shelf application | Public | Règles de sécurité réseau | Very low — coeff 0.5 |
| CN2 Web Platform (9 services NestJS) | Custom application | Public | 9 microservices | High — coeff 2 |
| CN2 ETL Pipeline (5 services Java SpringBoot) | Custom application | Public | 5 microservices | High — coeff 2 |
| CN2 dsearch (Python) | Custom application | Public | 1 service | Low — coeff 0.8 |
| SFTP Server | Custom application | Public | Échange de fichiers | Low — coeff 0.8 |

### Environment: NON-PROD léger (DEV, INTEG, TEST, TRAINING)

**Plage horaire:** Standard (Lun-Ven 9h30-18h30)
**SLA:** Bronze (coeff 1.00)

Ces 4 environnements sont **très légers (~25% de PROD chacun)**. Ils comportent principalement des Container Apps et des services Azure minimaux. Pas de VMs dédiées documentées, DBs possiblement partagées ou de taille réduite. Les environnements peuvent être activés/désactivés à la demande pour réduire les coûts.

Resource groups identifiés : rg-cisnet2-dev, rg-cisnet2-integ. TEST et TRAINING n'ont pas de resource group dédié identifié.

Inventaire estimé par env léger (à confirmer) :

| Resource | Type | Cloud | Size | Complexity |
|----------|------|-------|------|------------|
| Azure Container Apps | Off-the-shelf application | Public | Microservices CN2 (subset ou complet) | Very low — coeff 0.5 |
| PostgreSQL (si dédié) | Off-the-shelf application | Public | Taille réduite | Low — coeff 0.8 |
| MongoDB (si dédié) | Off-the-shelf application | Public | Taille réduite | Medium — coeff 1 |
| Azure services (Service Bus, Key Vault, etc.) | Off-the-shelf application | Public | Instances minimales | Low — coeff 0.8 |
| CN2 Apps (groupées) | Custom application | Public | Déploiement dev/test | Low — coeff 0.8 |

### Environment: Transverse (ressources partagées)

**Plage horaire:** Standard
**SLA:** Bronze (coeff 1.00)

| Resource | Type | Cloud | Size | Complexity |
|----------|------|-------|------|------------|
| Azure Blob Storage | Off-the-shelf application | Public | Stockage objets partagé entre environnements | Low — coeff 0.8 |
| Azure Monitoring | Off-the-shelf application | Public | Supervision, logs, alertes (toutes envs) | Medium — coeff 1 |
| GitLab CI | Off-the-shelf application | Public | CI/CD pipeline (lint, tests, build, deploy) | Medium — coeff 1 |
| SonarQube / ESLint | Off-the-shelf application | Public | Analyse qualité code | Low — coeff 0.8 |

## Service Commitments Summary

| Environment | Plage horaire | SLA | SLA Coeff |
|-------------|---------------|-----|-----------|
| PROD | Étendue (recommandée) — 3 scénarios à chiffrer | Gold | 1.10 |
| PREPROD | Standard | Bronze | 1.00 |
| DEV | Standard | Bronze | 1.00 |
| INTEG | Standard | Bronze | 1.00 |
| TEST | Standard | Bronze | 1.00 |
| TRAINING | Standard | Bronze | 1.00 |
| Transverse | Standard | Bronze | 1.00 |

> **Note :** 2 groupes de plages horaires (Étendue pour PROD, Standard pour tout le reste) — conforme à la contrainte maximum de 2 groupes.

## Informations manquantes

| Information manquante | Impact sur l'estimation | Comment l'obtenir |
|-----------------------|------------------------|-------------------|
| Sizing CPU/RAM des Container Apps individuelles | Coefficient de sizing imprécis pour Container Apps — peut faire varier le MCO de ±20% | Accéder à la console Azure ou demander les specs au client |
| Volume de données PostgreSQL et MongoDB (en Go) | Impact le coefficient de taille des DBs | Demander au client ou accéder aux métriques Azure |
| Volume de requêtes API (RPS) en production | Impact le coefficient de taille de Container Apps et API Manager | Demander au client — non disponible car pas encore en prod |
| Historique de tickets / incidents | Impossible de calibrer empiriquement l'estimation | Pas disponible — infra pas encore en prod |
| Nombre d'ETPs actuels chez Castelis | Impossible de cross-checker l'effort estimé avec l'effort actuel | Demander au client ou à Castelis |
| Détail des environnements légers (DEV, INTEG, TEST, TRAINING) | Inventaire approximatif (~25% de PROD) — peut faire varier le MCO non-prod de ±50% | Accéder à la console Azure ou demander un export des resource groups |
| RTO/RPO attendu | Impact le design de la continuité et le DR — sections TBD dans le TAD | Demander au client |
| Outil ITSM existant | Impact la transition et le tooling de gestion des tickets | Demander au client |
| Budget cible | Pas d'ancrage pour la comparaison de prix | Demander à Alice Mathieu (mentionnée dans le questionnaire) |
| Interlocuteurs CISAC nommés | Nécessaire pour la mise en place de la gouvernance | Demander au client |
| Accès au repo GitLab | Nécessaire pour évaluer la maturité IaC et la complexité des déploiements | Demander au client |
| Volume de fichiers EDI/XML traités par les ETL | Impact le sizing des services ETL | Demander au client |
| Nombre de sociétés connectées via SFTP | Impact le sizing du service SFTP | Demander au client |

## Constraints & Notes

- **HDS :** Non applicable — pas de données de santé
- **Compliance :** Aucune contrainte spécifique (pas de SecNumCloud, pas de RGPD renforcé, pas d'ISO 27001)
- **Engagement :** 1 an initial, possibilité de passer à 3 ans si la collaboration se passe bien
- **Mode d'engagement :** Temps passé — enveloppe cappée avec alerting budgétaire
- **Plage horaire :** 3 scénarios à chiffrer (Standard, Étendue, Complète) — Étendue recommandée pour PROD
- **Localisation :** France (Azure France Central, CISAC basé à Paris)
- **Nearshore :** Non discuté — à évoquer si réduction de coûts souhaitée (contacter Hugo/Lila/Manu)
- **IaC :** Actuellement Azure CLI scripts, migration Terraform planifiée mais pas encore réalisée
- **Monitoring :** Centreon (infra) et Kibana (logs applicatifs) mentionnés dans le TAD mais marqués TBD — potentiellement à déployer
- **Intégrations externes :** ISWC API, OpenAI (embeddings IA), systèmes des sociétés membres CISAC via SFTP
- **Infra gérée par Castelis :** Transition à prévoir depuis Castelis vers Theodo
