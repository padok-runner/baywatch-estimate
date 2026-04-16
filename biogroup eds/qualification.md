# Qualification — BioGroup EDS

**Date :** 2026-04-16
**Approche :** Déductive (infra non live depuis +6 mois)
**Solutions Architect :** —

## Client Context

BioGroup est un réseau de laboratoires de biologie médicale. Le périmètre **EDS** (Entrepôt de Données de Santé) vise à centraliser, structurer et exploiter les données de santé issues de l'activité des laboratoires à des fins de pilotage, recherche et exploration data/IA.

La plateforme s'appuie sur **S3ns** (Google Cloud souverain, partenariat Thalès-Google) et combine data warehouse BigQuery, clusters GKE pour les workloads applicatifs (orchestration Airflow, self-service analytics Metabase, IAM Authentik/Keycloak, secret management Vault) et une landing zone réseau (Transit Zone + Interco).

La manipulation de données de santé impose **HDS + RGPD renforcé + ISO 27001**.

### Deductive Data (always present)
- **Dev team size :** 1-5 personnes (petite équipe).
- **Evolution backlog (6 mois) :** Mise en production + stabilisation. Priorités : fiabilisation de l'exploitation, durcissement sécurité (HDS), monitoring/alerting, DR & backup, automatisation IaC.

### Empirical Data (if applicable)
Non applicable — infrastructure pas encore en production depuis plus de 6 mois.

## Resource Inventory

L'inventaire a été fourni par le client sous forme d'un CSV (cf. [input.md](input.md)). Il décrit **8 projets** déployés sur **3 environnements** (Prod / Non-prod / Shared). Chaque ligne du tableau ci-dessous représente le volume **par environnement** sauf mention contraire.

Cloud : **Public Cloud Managed (S3ns / GCP)** pour tous les projets.

---

### Environment: Production (prod)

**Plage horaire :** Standard (Lun-Ven 9h30-18h30)
**SLA :** Gold (coef 1.10)

| Resource | Type | Cloud | Size | Complexity |
|----------|------|-------|------|------------|
| Cluster GKE — Data Orchestration | Public Cloud Managed K8s Cluster | public | <32 CPU, <50 GB | low (0.8) |
| Cluster GKE — Shared Services | Public Cloud Managed K8s Cluster | public | <32 CPU, <50 GB | medium (1) |
| Cluster GKE — Data Exploration | Public Cloud Managed K8s Cluster | public | <32 CPU, <50 GB | medium (1) |
| Cluster GKE — Bulles sécurisées | Public Cloud Managed K8s Cluster | public | variable (n) | medium (1) |
| 2 VMs — Interco | Public Cloud Managed VM | public | <16 CPU, <10 GB | very low (0.5) |
| 10+n BigQuery datasets (Data Warehouse, Bulles sécurisées, Data Ident, Transit Zone) | Off-the-shelf app | public | n Go (à préciser) | medium (1) |
| 11+n GCS buckets (tous projets) | Public Cloud Managed VM (proxy storage) | public | faible | very low (0.5) |
| 7 clés KMS | Public Cloud Managed VM (proxy sec) | public | faible | very low (0.5) |
| 4 IP statiques | infrastructure réseau | public | — | very low (0.5) |
| 11+ Private Service Endpoints | infrastructure réseau | public | — | low (0.8) |
| **Applications Kubernetes déployées (off-the-shelf)** | | | | |
| Airflow (Data Orchestration) | Off-the-shelf application | public | medium | high (2) |
| ArgoCD (Shared Services) | Off-the-shelf application | public | small | low (0.8) |
| Authentik (3 clusters) | Off-the-shelf application | public | small | medium (1) |
| Keycloak (Data Exploration) | Off-the-shelf application | public | small | medium (1) |
| Cert-manager (3 clusters) | Off-the-shelf application | public | small | very low (0.5) |
| Cloud-native PG (3 clusters) | Off-the-shelf application | public | <50 GB | high (2) |
| Envoy Gateway (2 clusters) | Off-the-shelf application | public | small | low (0.8) |
| Vault (Shared Services) | Off-the-shelf application | public | small | medium (1) |
| Vault Secrets Operator (3 clusters) | Off-the-shelf application | public | small | very low (0.5) |
| Alloy (3 clusters) | Off-the-shelf application | public | small | low (0.8) |
| Grafana / Loki / Prometheus / Tempo (Shared Services, stack LGTM) | Off-the-shelf application | public | medium | high (2) |
| Metabase OSS + Enterprise (Data Exploration) | Off-the-shelf application | public | small-medium | medium (1) |

---

### Environment: Non-production (non-prod)

**Plage horaire :** Standard (Lun-Ven 9h30-18h30)
**SLA :** Bronze (coef 1)

Miroir allégé de la production : mêmes projets, mêmes types de ressources, volumes identiques à ce stade (aucune information contraire dans l'inventaire). Le sizing / complexity peut être réduit d'un cran pour les workloads critiques (ex. cloud-native PG : coef 1 au lieu de 2).

| Resource | Type | Cloud | Size | Complexity |
|----------|------|-------|------|------------|
| Mêmes ressources que prod | idem | public | inférieur d'un cran | réduit d'un cran |

---

### Environment: Shared services

**Plage horaire :** Standard (Lun-Ven 9h30-18h30)
**SLA :** Bronze (coef 1)

Regroupe l'outillage transverse : CI/CD (ArgoCD), observabilité (stack LGTM : Loki, Grafana, Tempo, Prometheus, Alloy), IAM (Authentik, Vault). Dans le CSV, le projet "Shared Services" contient déjà la majorité de ces outils.

| Resource | Type | Cloud | Size | Complexity |
|----------|------|-------|------|------------|
| Cluster GKE Shared Services | Public Cloud Managed K8s Cluster | public | <32 CPU, <50 GB | medium (1) |
| Stack observabilité (LGTM + Alloy) | Off-the-shelf application (×5) | public | medium | high (2) pour logging/monitoring |
| ArgoCD | Off-the-shelf application | public | small | low (0.8) |
| Authentik + Vault + Vault Secrets Operator | Off-the-shelf application (×3) | public | small | medium (1) |
| Cert-manager + Cloud-native PG | Off-the-shelf application (×2) | public | small | medium (1) |
| 4 buckets GCS, 2 KMS keys, 1 IP | infra réseau / stockage | public | — | very low (0.5) |

---

## Service Commitments Summary

| Environment   | Plage horaire | SLA    | SLA Coeff |
|---------------|---------------|--------|-----------|
| Production    | Standard      | Gold   | 1.10      |
| Non-production| Standard      | Bronze | 1.00      |
| Shared services| Standard     | Bronze | 1.00      |

**Contrainte plages horaires :** 1 seul groupe de plages horaires (Standard partout) → conforme à la limite de 2 groupes max.

## Informations manquantes

| Information manquante | Impact sur l'estimation | Comment l'obtenir |
|-----------------------|------------------------|-------------------|
| **Valeurs `n` et `x` dans l'inventaire** (BigQuery datasets Bulles sécurisées, GCS buckets Transit Zone, endpoints Bulles sécurisées) | Le volume total de ressources peut varier significativement selon le nombre réel de bulles sécurisées provisionnées → marge d'incertitude ±20% sur le MCO | Demander au client le nombre cible de bulles sécurisées actives + datasets associés |
| **Sizing CPU/RAM/RPS par cluster GKE et par app K8s** | Coefficient de complexité approximé par catégorie générale (low/medium/high). Peut faire varier le MCO de ±30% | Export des configurations Kubernetes (requests/limits) + métriques Prometheus une fois la plateforme live |
| **Volume des datasets BigQuery (Go ingérés par mois)** | Complexité Data Warehouse sous-évaluée si >500 Go → passage coef 1 → 5 | Export `INFORMATION_SCHEMA.TABLE_STORAGE` ou estimation prévisionnelle métier |
| **Interprétation exacte des "3 environnements"** | Structure réelle des environnements (Dev/Preprod/Prod vs Prod/Non-prod/Shared) impacte la répartition des coefficients SLA | Atelier de cadrage avec le client pour valider la topologie |
| **Plages horaires réelles d'utilisation** | Un laboratoire peut avoir besoin d'astreinte étendue si traitement nocturne ou multi-sites → upgrade plage Standard → Étendue = +effort astreinte | Confirmer les horaires d'activité réels des laboratoires et les batchs nocturnes |
| **Classification de criticité métier** | Validation du SLA Gold en prod (pourrait justifier Platine) | Atelier de criticité avec le métier / DSI BioGroup |
| **Stratégie IaC cible (Terraform ?)** | Impact sur l'effort d'automatisation des changements | Accès au repo GitLab/GitHub et confirmation du modèle cible |
| **Stratégie de backup et DR (RTO/RPO)** | Impact direct sur la gestion de la continuité et le SLA | Atelier sécurité / continuité d'activité |
| **Interlocuteurs côté client** (DSI, CTO, RSSI, DPO) | Nécessaires pour la gouvernance (COPIL, COPROD, audits) | Demande directe au sponsor BioGroup |
| **Budget cible / enveloppe** | Valider que le chiffrage est aligné avec les attentes | Demande directe |
| **Applications custom (non listées)** | Le CSV ne liste que les apps off-the-shelf K8s. Y a-t-il des applications métier custom à opérer ? | Atelier d'architecture applicative |

## Constraints & Notes

- **Certification HDS requise** → audit YAMAS obligatoire dans la gouvernance ; attention aux exigences de traçabilité, chiffrement, et habilitations.
- **RGPD renforcé** (données de santé) + **ISO 27001** → processus de revue sécurité régulier, DPO impliqué, audits de compliance.
- **Engagement 1 an** (renouvelable 3 ans si satisfaction) — pas de levier de négociation pluriannuel initial.
- **Stack observabilité auto-hébergée** (Grafana, Loki, Prometheus, Tempo, Alloy) → à opérer nous-mêmes, coefficient de complexité plus élevé que du managed.
- **Environnement shared services + transit zone + landing zone** présents → bonne maturité cloud, mais charge d'administration réseau (4 IPs, 11+ endpoints, VPC peering).
- **HDS en Standard :** à noter que la plage horaire Standard peut être en tension avec les exigences opérationnelles HDS (gestion d'incidents critiques hors heures de bureau). À ré-adresser avec le client si besoin d'astreinte.
