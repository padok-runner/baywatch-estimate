# BioGroup EDS — Input Qualification

## Contexte

Source : `BioGroup Estimations Inventory.csv` (inventaire fourni par le client).

Nombre d'environnements : **3**

Cloud provider : **S3ns** (Google Cloud souverain, services GCP).

## Inventaire des ressources par projet (par environnement)

### Projet 1 — Data Orchestration

**Services S3ns :**

| Ressource | Service | Nombre |
| --- | --- | --- |
| Dataset BigQuery | BigQuery | 0 |
| Bucket GCS | Google Cloud Storage | 2 |
| Cluster Kubernetes | GKE | 1 |
| Clé KMS | KMS | 1 |
| IP statique | Google Compute Address | 1 |
| VM | Google Compute Instance | 0 |
| Endpoints | Private Service Endpoints | 5 |

**Applications Kubernetes :** Airflow, Alloy, Authentik, Cert manager, Cloud native PG, Envoy gateway, Vault secrets operator

---

### Projet 2 — Shared Services

**Services S3ns :**

| Ressource | Service | Nombre |
| --- | --- | --- |
| Dataset BigQuery | BigQuery | 0 |
| Bucket GCS | Google Cloud Storage | 4 |
| Cluster Kubernetes | GKE | 1 |
| Clé KMS | KMS | 2 |
| IP statique | Google Compute Address | 1 |
| VM | Google Compute Instance | 0 |
| Endpoints | Private Service Endpoints | (non renseigné) |

**Applications Kubernetes :** Alloy, ArgoCD, Authentik, Cert manager, Cloud native PG, Grafana, Loki, Prometheus, Tempo, Vault, Vault secrets operator

---

### Projet 3 — Data Exploration

**Services S3ns :**

| Ressource | Service | Nombre |
| --- | --- | --- |
| Dataset BigQuery | BigQuery | 0 |
| Bucket GCS | Google Cloud Storage | 1 |
| Cluster Kubernetes | GKE | 1 |
| Clé KMS | KMS | 1 |
| IP statique | Google Compute Address | 1 |
| VM | Google Compute Instance | 0 |
| Endpoints | Private Service Endpoints | 5 |

**Applications Kubernetes :** Alloy, Authentik, Cert manager, Cloud native PG, Envoy gateway, Keycloak, Metabase OOS, Metabase Enterprise, Vault secrets operator

---

### Projet 4 — Data Warehouse

**Services S3ns :**

| Ressource | Service | Nombre |
| --- | --- | --- |
| Dataset BigQuery | BigQuery | 6 |
| Bucket GCS | Google Cloud Storage | 0 |
| Cluster Kubernetes | GKE | 0 |
| Clé KMS | KMS | 0 |
| IP statique | Google Compute Address | 0 |
| VM | Google Compute Instance | 0 |
| Endpoints | Private Service Endpoints | 0 |

---

### Projet 5 — Bulles sécurisées

**Services S3ns :**

| Ressource | Service | Nombre |
| --- | --- | --- |
| Dataset BigQuery | BigQuery | n (variable) |
| Bucket GCS | Google Cloud Storage | 0 |
| Cluster Kubernetes | GKE | 1 |
| Clé KMS | KMS | 0 |
| IP statique | Google Compute Address | 1 |
| VM | Google Compute Instance | 0 |
| Endpoints | Private Service Endpoints | x (variable) |

---

### Projet 6 — Data Ident

**Services S3ns :**

| Ressource | Service | Nombre |
| --- | --- | --- |
| Dataset BigQuery | BigQuery | 2 |
| Bucket GCS | Google Cloud Storage | 0 |
| Cluster Kubernetes | GKE | 0 |
| Clé KMS | KMS | 1 |
| IP statique | Google Compute Address | 0 |
| VM | Google Compute Instance | 0 |
| Endpoints | Private Service Endpoints | 0 |

---

### Projet 7 — Transit Zone

**Services S3ns :**

| Ressource | Service | Nombre |
| --- | --- | --- |
| Dataset BigQuery | BigQuery | 2 |
| Bucket GCS | Google Cloud Storage | 4+n (variable) |
| Cluster Kubernetes | GKE | 0 |
| Clé KMS | KMS | 2 |
| IP statique | Google Compute Address | 0 |
| VM | Google Compute Instance | 0 |
| Endpoints | Private Service Endpoints | 0 |

---

### Projet 8 — Interco

**Services S3ns :**

| Ressource | Service | Nombre |
| --- | --- | --- |
| Dataset BigQuery | BigQuery | 0 |
| Bucket GCS | Google Cloud Storage | 0 |
| Cluster Kubernetes | GKE | 0 |
| Clé KMS | KMS | 0 |
| IP statique | Google Compute Address | 0 |
| VM | Google Compute Instance | 2 |
| Endpoints | Private Service Endpoints | 1 |

---

## Synthèse globale (par environnement)

| Ressource | Total (tous projets) |
| --- | --- |
| Datasets BigQuery | 10 + n |
| Buckets GCS | 11 + n |
| Clusters GKE | 4 |
| Clés KMS | 7 |
| IPs statiques | 4 |
| VMs | 2 |
| Private Service Endpoints | 11+ |

Multiplier par **3 environnements** pour obtenir le volume total estimé.

## Questions ouvertes (à clarifier avec le client)

- Activité / contexte métier de BioGroup EDS (laboratoire de biologie médicale ? données de santé ?)
- Exigences réglementaires : **HDS ?**, RGPD, ISO 27001
- Ancienneté de l'infrastructure en production (+6 mois ou pas)
- Taille de l'équipe Dev/IT (développeurs, PO, CTO, ops)
- Outillage CI/CD et IaC (Terraform ? GitOps ?)
- Plages horaires d'utilisation et SLA par environnement
- Volumétrie : RPS, taille des bases, ingestion BigQuery
- Valeurs « n » et « x » dans l'inventaire (projets Bulles sécurisées, Transit Zone)
- Roadmap d'évolution sur 6 mois
- Mode d'engagement et durée
