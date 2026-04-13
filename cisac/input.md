# Sizing Run Infra

# Questions de qualification - Infogérance Cloud

![image.png](Sizing%20Run%20Infra/image.png)

![image.png](Sizing%20Run%20Infra/image%201.png)

## A. Maturité de l’infrastructure cloud

- [x]  **L’infrastructure cloud cible est-elle déjà en production depuis plus de 6 mois ?**
    
    Non → mise en prod le 27.04
    

## B. Inventaire des ressources (demande client)

> **Action client requise** : fournir l’inventaire exhaustif des ressources cloud/on-prem à migrer.
> 
> - [x]  **Sur quel cloud est hébergé l’infra ? Azure**
> - [x]  **Fournissez un export d’inventaire (RVTools, CMDB, ou équivalent) avec pour chaque ressource :**
> - Type de ressource (VM, cluster K8s, hyperviseur, etc.)
> - Nombre de vCPUs, RAM, stockage
> - OS et version
> - Cloud cible (public cloud managé, private cloud managé)
> - Environnement (production, pré-production, développement, recette, etc.)

> **Informations extraites du Technical Architecture Document (TAD) :**
> 
> 
> **VMs :**
> 
> | Nom | Taille | vCPU | RAM | Disk 1 | Disk 2 | OS | Env |
> | --- | --- | --- | --- | --- | --- | --- | --- |
> | vm-cisnet2-apps-preprod | Standard B4ls v2 | 4 | 8 GB | SSD Premium LRS 30GB | HDD Standard LRS 200GB | Linux Ubuntu 24.04 (64 bits) | Preprod |
> | vm-cisnet2-apps-prod | Standard B4ls v2 | 4 | 8 GB | SSD Premium LRS 30GB | HDD Standard LRS 200GB | Linux Ubuntu 24.04 (64 bits) | Prod |
> 
> **Bases de données :**
> 
> | Service | Technologie | Tier/SKU | vCores | RAM | Stockage | Env |
> | --- | --- | --- | --- | --- | --- | --- |
> | PostgreSQL (Azure DB for PostgreSQL) | PostgreSQL 16.10 | Standard_B2s | 2 | 4 GB | SSD Premium 256GB | Preprod |
> | MongoDB (Azure Cosmos DB for MongoDB 8.0) | MongoDB 8.0 | M40 | 4 | 16 GB | 32 GB | Preprod |
> | PostgreSQL (Azure DB for PostgreSQL) | PostgreSQL 16.10 | Standard_B2s | 2 | 4 GB (subject to change) | SSD Premium 256GB | Prod |
> | MongoDB (Azure Cosmos DB for MongoDB 8.0) | MongoDB 8.0 | M10 | 1 | 2 GB | 32 GB | Prod |
> 
> **Autres services Azure :**
> 
> - Azure Container Apps (héberge les microservices NestJS et Java SpringBoot)
> - Azure API Manager (authentification et throttling API)
> - Azure Service Bus (messaging asynchrone inter-services)
> - Azure Blob Storage (stockage objets partagé entre environnements)
> - Azure Key Vault (gestion secrets/certificats) — instances multiples par app et par env
> - Azure App Configuration (paramétrage centralisé)
> - Azure Search (moteur de recherche)
> - Azure Monitoring (supervision, logs, alertes)
> - Azure Container Registry (acrcisnet2preprod / acrcisnet2prod)
> - NSG (Network Security Groups) — preprod et prod
> 
> **Cloud cible** : Azure public cloud managé (région France Central)
> 
- [x]  **Combien d’environnements distincts avez-vous ?** (prod, non-prod, shared services, landing zone)
    
    > **5 environnements identifiés** :
    > 
    - DEV
    - INTEG (test CISAC)
    - TEST (test sociétés/UAT)
    - PREPROD
    - PROD
    - TRAINING

Resource Groups : rg-cisnet2-dev, rg-cisnet2-integ, rg-cisnet2-preprod, rg-cisnet2-prod
Note : les environnements peuvent être activés/désactivés à la demande pour réduire les coûts.

- [x]  **Avez-vous des clusters Kubernetes managés ou self-hosted ? Combien ?**
    
    > **Non.** Pas de Kubernetes. L’infrastructure utilise **Azure Container Apps** (serverless containers managés) pour héberger les microservices, et des **VMs** pour les ETL/jobs.
    > 
- [x]  **Avez-vous des hyperviseurs dédiés ? Combien ?**
    
    > **Non.** Pas d’hyperviseurs dédiés. L’infrastructure est 100% cloud Azure (VMs managées + Container Apps).
    > 

## C. Infrastructure live (si +6 mois en production)

> *Section peu applicable car l’infra n’est pas encore en production depuis +6 mois (mise en prod le 27.04).*
> 
- [ ]  **Pouvez-vous nous donner accès au code d’infrastructure (Terraform, Ansible, etc.) ?**
    
    > Le TAD mentionne un workflow IaC cible avec **Git, GitLab CI, Terraform** et Terraform Workspaces (Dev, Test, Prod). Cependant, en phase MVP, l’infrastructure est provisionnée via **Azure CLI (az) scripts** et non Terraform. 
    **→ A confirmer : accès au repo GitLab contenant les scripts az / futurs fichiers Terraform ?**
    > 
- [ ]  **Pouvez-vous fournir un export des tickets des 6 derniers mois** (incidents, demandes de service, changements) ?
    
    > **→ NON DISPONIBLE dans le TAD. A demander au client.**
    > 
- [ ]  **Combien d’ETPs sont actuellement mobilisés pour maintenir la plateforme ?**
    
    > **→ NON DISPONIBLE dans le TAD.** Stakeholders identifiés : Client, Developer, Infra Delivery Manager, IT Ops Director (noms non renseignés). **A demander au client.**
    > 
- [ ]  **Quel est le backlog d’évolutions anticipé sur les 6 prochains mois ?**
    
    > **→ NON DISPONIBLE dans le TAD. A demander au client.**
    > 

## D. Infrastructure non live (migration / build en cours)

- [ ]  **Quelle est la taille des équipes de développement ?**
    
    > **→ NON DISPONIBLE dans le TAD. A demander au client.** Le document a été préparé par Castelis (Hunik Group / PMP Strategy).
    > 
- [ ]  **Quel est le backlog d’évolutions anticipé sur les 6 prochains mois ?**
    
    > **→ NON DISPONIBLE dans le TAD.** De nombreuses sections sont marquées “TBD” dans le TAD (Monitoring, DR, Security, Backup details), ce qui suggère un backlog technique significatif restant.
    > 

## E. Applications et middleware

- [x]  **Quelles applications off-the-shelf exploitez-vous ?** (ex. MySQL, Elasticsearch, Kafka, ArgoCD, Vault, etc.)
    - Pour chacune : quelle complexité ? (faible, moyenne, haute, très haute)
    
    | Application off-the-shelf | Usage | Complexité estimée |
    | --- | --- | --- |
    | PostgreSQL 16.10 (Azure DB for PostgreSQL) | Base de données relationnelle principale | Haute++ |
    | MongoDB 8.0 (Azure Cosmos DB) | Stockage NoSQL (données métier non-relationnelles) | Haute |
    | Azure Service Bus | Messaging asynchrone inter-microservices (queues & topics) | Moyenne |
    | Azure API Manager | Authentification API, throttling, routage | Haute |
    | Azure Search | Moteur de recherche | Moyenne |
    | Azure Key Vault | Gestion des secrets et certificats | Faible |
    | Azure App Configuration | Paramétrage centralisé des apps | Faible |
    | Azure Blob Storage | Stockage objets (documents, logs, fichiers config) | Faible |
    | Azure Container Apps | Hébergement des microservices containerisés | Haute |
    | Azure Container Registry | Registre d’images Docker | Faible |
    | Redis (mentionné comme cache in-memory) | Cache applicatif | Moyenne |
    | GitLab CI | CI/CD pipeline (lint, tests, build, deploy) | Moyenne |
    | SonarQube / ESLint | Analyse qualité code | Faible |

**→ A confirmer avec le client : versions exactes et complexité perçue côté exploitation.**

- [x]  **Quelles applications custom (développées en interne) sont déployées sur l’infrastructure ?**
    
    > **Plateforme CIS-Net 2 (CN2)** — plateforme modulaire microservices pour centraliser, gérer et exposer les oeuvres musicales et leurs métadonnées. Architecture multi-couches :
    > 
    > 
    > **Microservices NestJS (Node.js) :**
    > 
    > - app-cisnet2-front (Frontend Next.js)
    > - app-cisnet2-user (BFF - Backend For Frontend)
    > - app-cisnet2-business (logique métier)
    > - app-cisnet2-code (service code)
    > - app-cisnet2-search (service recherche)
    > - app-cisnet2-aisearch (recherche IA)
    > - app-cisnet2-manager
    > - app-cisnet2-notif (notifications)
    > - app-cisnet2-submit (soumission MWI)
    > 
    > **Microservices Java SpringBoot (ETL) :**
    > 
    > - app-cisnet2-etl (ETL principal)
    > - app-cisnet2-parser
    > - app-cisnet2-preparer
    > - app-cisnet2-listener
    > - app-cisnet2-soc (notifications sociétés)
    > 
    > **Autre :**
    > 
    > - app-cisnet2-dsearch (Python — recherche)
    > - SFTP server (échange de fichiers avec les sociétés)
    > 
    > **Intégrations externes :** ISWC API, OpenAI (embeddings IA), systèmes des sociétés membres CISAC
    > 
- [ ]  **Pour chaque application critique, quel est le volume de requêtes (RPS), la taille CPU/RAM ?**
    
    > **→ NON DISPONIBLE dans le TAD.** Seules les tailles des VMs et bases de données sont documentées (cf. section B). Les ressources CPU/RAM des Container Apps individuelles ne sont pas spécifiées. **A demander au client.**
    > 

## F. Criticité et niveaux de service (SLA)

- [ ]  **Comment classez-vous vos applications par criticité ?** (critique / haute / moyenne / basse)
    
    > **→ NON DISPONIBLE dans le TAD.** Pas de classification de criticité formalisée dans le document. **A demander au client.**
    > 
- [ ]  **Quel est le niveau de SLA attendu par environnement ?**
    - Bronze (standard), Silver, Gold, Platine ?
    - Par défaut : Bronze pour le hors-production, sauf demande contraire
    
    > **→ NON DISPONIBLE dans le TAD.** Le document ne mentionne aucun SLA cible. **A demander au client.**
    > 
- [ ]  **Quelles applications nécessitent une haute disponibilité 24/7 ?**
    
    > **→ NON DISPONIBLE explicitement.** Cependant, CN2 est une plateforme utilisée par les sociétés membres CISAC à l’international (échange de données MWI via SFTP + API), ce qui suggère un besoin potentiel de disponibilité étendue. Le Disaster Recovery Plan et le Business Continuity Plan sont marqués “TBD” dans le TAD. **A demander au client.**
    > 

## G. Plages horaires de service

- [ ]  **Quel est le profil d’utilisation de vos applications ?**
    - Application interne France → Standard (Lun-Ven 9h30-18h30)
    - Application métier étendue → Étendue (Lun-Sam 8h-22h)
    - Application critique 24/7 → Complète (7j/7, 24h/24, fériés inclus)

Les 3 options sont à chiffrer

- [ ]  **Y a-t-il des contraintes spécifiques de couverture horaire ?** (utilisateurs internationaux, batch nocturnes, etc.)

Oui, mais pas de pression de temps réel.

## H. Conformité et contraintes réglementaires

- [ ]  **~~Êtes-vous soumis à la certification HDS (Hébergeur de Données de Santé) ?~~**
- [ ]  **Avez-vous d’autres contraintes réglementaires ?** (SecNumCloud, RGPD renforcé, ISO 27001, etc.)
    
    > **→ NON**
    > 
- [ ]  **Des audits de compliance réguliers sont-ils nécessaires ?**
    
    > **→ NON**
    > 

## I. Mode d’engagement et budget

- [ ]  **Préférez-vous un engagement au temps passé (enveloppe JH/mois) ou au forfait ?**
    - Temps passé (par défaut) : enveloppe reportable sur le mois suivant
    - Forfait : enveloppe + contingence risque (+0% à +40% selon incertitude)
    
    > **→ Pas fermé au temps passé mais nécessité d’avoir une enveloppe bien cappée avec un système d’alerting budgétaire très clair**
    > 
- [ ]  **Quelle durée d’engagement envisagez-vous ?**
    - 1 an (tarif standard)
    - 2 ans (levier de négociation possible)
    - 3 ans (levier de négociation possible)
    
    > **→ 1 an puis si ça se passe bien, 3ans peut être possible**
    > 
- [ ]  **Avez-vous une enveloppe budgétaire cible ?**
    
    > **→ @Alice Mathieu**
    > 

## J. Localisation et organisation

- [x]  **L’infogérance doit-elle être réalisée depuis la France ?** (par défaut : oui)
    - Si réduction de coûts souhaitée, option nearshore possible (à valider avec Hugo/Manu)
    
    > L’infrastructure est hébergée sur Azure **France Central**. CISAC est basé en France (Paris). **→ Par défaut oui, à confirmer.**
    > 
- [ ]  **Qui sont les interlocuteurs côté client ?** (DSI, CTO, responsable infrastructure, RSSI)
    
    > **PARTIELLEMENT.**
    > 
    > - Client (CISAC)
    > - Developer
    > - Infra Delivery Manager
    > - IT Ops Director
    > 
    > L’infra est actuellement gérée par **Castelis** (Hunik Group). **→ A obtenir : noms et contacts des interlocuteurs CISAC.**
    > 
- [ ]  **Utilisez-vous un outil ITSM existant ?** (ServiceNow, Jira Service Management, GLPI, etc.)
    
    > **→ NON DISPONIBLE dans le TAD. A demander au client.**
    > 

---

## Questions complémentaires à poser au client (CISAC)

### Priorité haute (indispensables pour le sizing)

1. **Inventaire complet des Container Apps** : Combien de Container Apps sont déployées par environnement ? Quelles sont les allocations CPU/RAM de chaque Container App ? Quelles sont les règles de scaling (min/max replicas) ?
2. **Volume de données et trafic** : Quel est le volume de requêtes API (RPS) en production ? Quel est le volume de données dans PostgreSQL et MongoDB (en Go) ? Combien de fichiers EDI/XML sont traités par jour via les ETL ?
3. **SLA et criticité** : Quelle classification de criticité pour CN2 ? Quel SLA cible (disponibilité %) pour la production ? Quel RTO/RPO attendu ?
4. **Plages horaires** : Les sociétés membres accèdent-elles 24/7 ou sur des créneaux définis ? Y a-t-il des batchs ETL planifiés la nuit ou le week-end ?
5. **Taille des équipes** : Combien de développeurs travaillent sur CN2 ? Combien d’ETP infra/ops actuellement chez Castelis ?
6. **Backlog technique** : Quelles sont les évolutions prévues sur les 6-12 prochains mois ? (migration Terraform, implémentation du DR, monitoring, sécurité — toutes les sections “TBD” du TAD)

### Priorité moyenne (nécessaires pour affiner la proposition)

1. **IaC** : Le passage de Azure CLI scripts à Terraform est-il prévu ? Si oui, quel calendrier ? Peut-on accéder au repo GitLab ?
2. **Monitoring actuel** : Centreon est mentionné pour le monitoring infra (TBD) et Kibana pour les logs applicatifs (TBD). Sont-ils déjà en place ou reste-t-il à les déployer ?
3. **Conformité/réglementaire** : CISAC a-t-il des exigences RGPD renforcées (données de sociétés internationales) ? ISO 27001 ? Audits de sécurité réguliers ?
4. **ITSM** : Quel outil de ticketing est utilisé aujourd’hui ? (Jira, ServiceNow, etc.)
5. **Budget** : Quelle enveloppe budgétaire est envisagée pour l’infogérance ? Durée d’engagement souhaitée ?
6. **Interlocuteurs** : Qui sont les contacts nommés côté CISAC (DSI, CTO, RSSI) ?

### Priorité basse (nice-to-have)

1. **OpenAI / IA** : L’intégration OpenAI (embeddings pour la recherche) est-elle en production ? Quel volume d’appels API ? Contraintes de données (les données musicales sont-elles envoyées à OpenAI) ?
2. **SFTP** : Combien de sociétés se connectent via SFTP ? Quel volume de transfert mensuel ?
3. **Environnements non-prod** : Les environnements DEV/INTEG/TEST sont-ils dimensionnés différemment de la preprod ? (le TAD ne détaille que preprod et prod)