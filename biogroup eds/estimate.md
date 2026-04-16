# Estimate — BioGroup EDS

**Date:** 2026-04-16
**Based on:** qualification.md (2026-04-16)
**TJM:** 863€ (blended Ops / Lead Ops / DM)
**Dispositif:** Semi-dédié (10-100 j/h/mois)

---

## Hypothèses de travail

| # | Hypothèse | Information manquante | Valeur retenue | Justification | Impact si l'hypothèse est fausse |
|---|-----------|----------------------|----------------|---------------|----------------------------------|
| H1 | Valeurs `n` et `x` de l'inventaire (Bulles sécurisées, Transit Zone) | Nombre réel de bulles et datasets BigQuery associés | `n = 3`, `x = 2` (hypothèse médiane) | Un EDS gère typiquement 2-5 projets confidentiels | ±1 500€/mois si `n` passe à 10+ bulles (chaque bulle = +cluster + +datasets) |
| H2 | Interprétation "3 environnements" | Structure exacte dev/preprod/prod vs prod/non-prod/shared | Modèle retenu : **Prod + Non-prod + Shared** (Shared Services hébergé dans un env dédié) | Cohérent avec la topologie Shared Services project = observabilité + CI/CD | ±2 000€/mois si la structure réelle est dev+preprod+prod (3 envs lourds au lieu de 2 lourds + 1 shared) |
| H3 | Sizing CPU/RAM/RPS par cluster et par app | Métriques Prometheus / requests K8s non fournis | Coef 1 (medium) pour prod, 0.5-0.8 pour non-prod | Sizing standard pour un EDS de laboratoire à ce stade de maturité | ±15% sur le MCO si le sizing réel est Large (>64 CPU) sur plusieurs clusters |
| H4 | Volume BigQuery (Go ingérés / mois) | Volumétrie des datasets non communiquée | Coef 0.5 par projet BigQuery (managed, faible effort ops unitaire) | Service 100% managé GCP, surface d'incident minimale | +1 000€/mois si les datasets dépassent 500 Go (passage coef 5) et nécessitent tuning partitionnement |
| H5 | Historique tickets / incidents | Infra pas encore en prod | Calibration basée sur profil EDS multi-cluster K8s auto-hébergé (LGTM, Vault, Airflow) | Approche standard pour nouvelle plateforme | ±2 500€/mois — la calibration MCO peut varier de ±30% post-MEP |
| H6 | HDS + plage Standard | Astreinte hors heures ouvrées non confirmée | Plage Standard maintenue, overhead HDS/ISO intégré au buffer SLA | Cohérent avec la qualification (Prod Gold Standard) | +3 000€/mois si client demande astreinte Étendue ou Complète a posteriori |
| H7 | Backlog évolutions 6 mois | Non chiffré précisément ("MEP + stabilisation") | 2.5 j/h/mois | Équivalent à ~1 chantier de stabilisation par mois (monitoring, DR, durcissement HDS) | ±1 000€/mois selon le rythme réel de stabilisation |
| H8 | Stack observabilité LGTM auto-hébergée | Volume de métriques / logs non connu | Prometheus + Loki coef 2, Grafana + Tempo coef 1 | Complexité élevée par défaut pour du self-hosted | +500€/mois si l'ingestion logs/metrics dépasse les seuils |

> **⚠ Sensibilité** : Les hypothèses H1 (bulles sécurisées), H2 (interprétation des envs) et H5 (calibration MCO) représentent ensemble ±5 000€/mois, soit ±40% du prix final. **Recommandations** : organiser un atelier de cadrage pour fixer H1/H2 avant contractualisation et prévoir une clause de revoyure MCO à 3 mois post-MEP.

---

## Analyse de réalisme

### Signaux empiriques disponibles

| Signal | Valeur | Source |
|--------|--------|--------|
| Historique de tickets | **Aucun** — infra pas encore en prod | Qualification |
| FTEs actuels | **Non communiqué** | Qualification |
| Maturité infra | **Faible** — mise en prod en cours, stabilisation à faire | Qualification |
| Complexité techno | **Haute** — 4 clusters GKE, stack LGTM self-hosted, Vault, Airflow, Metabase OSS+Enterprise, HDS + ISO 27001 | Qualification |
| Taille équipe dev | **Petite** (1-5 devs) | Qualification |
| Profil cloud | **S3ns / GCP managé** — mais beaucoup de self-hosting (observabilité, IAM, ETL) | Qualification |

### Calibration empirique

| Composante | j/h/mois | Raisonnement |
|------------|----------|--------------|
| MCO réactif | 1.5 | Nouvelle plateforme EDS. Estimation ~15-20 tickets/an × 1 j/h en moyenne. BigQuery managé réduit les incidents data. |
| MCO proactif | 6.0 | ~4× le réactif. Multi-cluster K8s, stack LGTM auto-hébergée (Grafana/Loki/Prometheus/Tempo), Vault, Airflow ETL, Metabase. Patching, monitoring config, veille sécurité HDS. |
| Buffer SLA Gold + HDS + ISO 27001 | 1.5 | Gold (+1.0) + overhead HDS (auditabilité, chiffrement, habilitations) + ISO 27001 (processus revue sécurité). Réparti sur les 3 envs. |
| **Total calibré MCO** | **9.0** | |

### Comparaison déductive vs calibrée

| Approche | MCO j/h/mois | Total j/h/mois (MCO+Gov+Evol) | Prix/mois | Écart |
|----------|--------------|-------------------------------|-----------|-------|
| Déductive pure | 17.80 (SLA-ajusté) | 21.05 | 18 166€ | — |
| **Calibrée (retenue)** | **9.50 (SLA-ajusté)** | **12.75** | **11 004€** | **-39%** |

**Décision :** La déductive surestime le MCO de ~87% par rapport à la calibrée. L'écart vient principalement de :

- **Dédoublonnage logique** : cert-manager / vault-secrets-operator / alloy déployés dans chaque cluster = 1 seul effort opérationnel réel (configuration GitOps centralisée), pas N.
- **BigQuery entièrement managé** : le coût ops unitaire est minime (IAM, quotas), très inférieur à un off-the-shelf auto-hébergé.
- **Stack LGTM partagée** : Grafana/Loki/Prometheus/Tempo vivent sur le cluster Shared Services uniquement, pas dupliquée par env.

**L'estimation calibrée (9.0 j/h/mois MCO) est retenue.** La déductive (17.80 j/h/mois) sert de plafond de référence. Une clause de revoyure à 3 mois post-MEP est recommandée pour ajuster selon le volume réel de tickets.

---

## Synthèse

|  | PROD | NON-PROD | SHARED | Transverse |
|---|---|---|---|---|
| **Nom des envs** | prod | non-prod | shared | — |
| **Inventaire** | **Servers :**<br>3 GKE clusters (Data Orchestration, Data Exploration, Bulles sécurisées)<br>**Datasets BigQuery :**<br>Data Warehouse (6), Data Ident (2), Bulles sécurisées (n), Transit Zone (2)<br>**Applications off-the-shelf :**<br>Airflow, Authentik, Keycloak<br>Cloud-native PG, Envoy Gateway<br>Metabase OSS + Enterprise<br>**Resources GCP :**<br>GCS buckets, KMS keys, IPs statiques, Private Service Endpoints | **Servers :**<br>Miroir de prod avec sizing réduit (3 GKE clusters dimensionnés staging)<br>**Applications :**<br>Mêmes off-the-shelf avec volumes réduits<br>**Datasets BigQuery :**<br>Sous-ensembles de test | **Servers :**<br>1 GKE cluster Shared Services<br>2 VMs Interco<br>**Applications off-the-shelf (Shared Services) :**<br>ArgoCD (GitOps)<br>Stack observabilité : Grafana, Loki, Prometheus, Tempo, Alloy<br>Vault (secrets central)<br>Authentik (IAM central)<br>Cloud-native PG (metadata)<br>**Landing zone :**<br>Transit Zone, Interco, VPC peering | Organisation GCP / S3ns, IAM fédéré, stratégie de chiffrement KMS, audit trail |
| **Services** | Maintien : Gestion des demandes de service, des incidents, des problèmes, des changements de version, de la continuité, de la surveillance | Maintien : Gestion des demandes de service, des incidents, des problèmes, des changements de version, de la continuité | Maintien : Gestion des demandes de service, des incidents, des problèmes, des changements de version, de la continuité | Gouvernance : COPROD mensuel, Audits semestriels ROSE, YAMAS (HDS) et LEAF<br>Évolutions : Gestion des changements mineurs |
| **Niveaux de services** | Gold | Bronze | Bronze | — |
| **Plages de service** | Standard | Standard | Standard | — |
| **Dispositif** | Ops : 8.5 j/mois, Lead Ops : 2.9 j/mois, Delivery Manager : 1.4 j/mois | | | |

#### Prix mensuel €HT

| Mode | Périmètre | j/h/mois | Montant €HT/mois |
|------|-----------|----------|-------------------|
| **Temps passé** | MCO (9.5 j/h SLA-ajusté) + Gouvernance (0.75 j/h) | 10.25 | 8 846€ |
| **Temps passé** | Évolutions (à la demande, TJM blended 863€ ou Ops 750€ / Lead Ops 1 200€ / DM 850€) | 2.5 | 2 158€ |
|  | Immobilisation (semi-dédié × Standard) | — | 0€ |
|  | **Total mensuel** | **12.75** | **11 004€** |

**Total annuel : 132 048€ HT**

#### Option Forfait (MCO + Gouvernance, contingence moyenne +20%)

Phase de mise en production + stabilisation = incertitude **moyenne** (+20%).

| Mode | Périmètre | j/h/mois | Montant €HT/mois |
|------|-----------|----------|-------------------|
| **Forfait** | (MCO 9.5 + Gouvernance 0.75) × 1.20 | 12.30 | 10 615€ |
| **Temps passé** | Évolutions | 2.5 | 2 158€ |
|  | **Total mensuel forfait** | **14.80** | **12 773€** |

**Total annuel forfait : 153 276€ HT**

---

## Annexe A — Détail du calcul

### MCO déductif (plafond de référence)

#### Environment: PROD (SLA Gold, coef 1.10)

| Resource | Item Type | Base Rate | Coeff | MCO (j/h/mois) |
|----------|-----------|-----------|-------|----------------|
| 3 clusters GKE (DataOrch/DataExp/Bulles) | Public Cloud Managed K8s | 0.25 | 1.0 | 0.75 |
| BigQuery (DataWH + DataIdent + Bulles) | Off-the-shelf app | 0.5 | 0.5 | 0.75 |
| Airflow (DataOrch) | Off-the-shelf app | 0.5 | 2 (high) | 1.00 |
| Authentik + Keycloak (IAM) | Off-the-shelf app | 0.5 | 1 | 1.00 |
| Cloud-native PG | Off-the-shelf app | 0.5 | 2 (high) | 1.00 |
| Envoy Gateway | Off-the-shelf app | 0.5 | 0.8 | 0.40 |
| Metabase OSS + Enterprise | Off-the-shelf app | 0.5 × 2 | 1 | 1.00 |
| Landing zone prod (GCS/KMS/IP/endpoints) | infra | 0.5 | — | 0.30 |
| **Subtotal PROD brut** | | | | **6.20** |
| **× SLA Gold 1.10** | | | | **6.82** |

#### Environment: NON-PROD (SLA Bronze, coef 1.00)

| Resource | Item Type | Base Rate | Coeff | MCO (j/h/mois) |
|----------|-----------|-----------|-------|----------------|
| 3 clusters GKE (réduits) | Public Cloud Managed K8s | 0.25 | 0.5 | 0.38 |
| BigQuery (datasets test) | Off-the-shelf app | 0.5 | 0.5 | 0.50 |
| Airflow | Off-the-shelf app | 0.5 | 1 | 0.50 |
| Authentik + Keycloak | Off-the-shelf app | 0.5 | 0.8 | 0.80 |
| Cloud-native PG | Off-the-shelf app | 0.5 | 1 | 0.50 |
| Envoy Gateway | Off-the-shelf app | 0.5 | 0.5 | 0.25 |
| Metabase OSS + Enterprise | Off-the-shelf app | 0.5 × 2 | 0.8 | 0.80 |
| Landing zone non-prod | infra | 0.5 | — | 0.20 |
| **Subtotal NON-PROD** | | | | **3.93** |
| **× SLA Bronze 1.00** | | | | **3.93** |

#### Environment: SHARED (SLA Bronze, coef 1.00)

| Resource | Item Type | Base Rate | Coeff | MCO (j/h/mois) |
|----------|-----------|-----------|-------|----------------|
| 1 cluster GKE Shared Services | Public Cloud Managed K8s | 0.25 | 1 | 0.25 |
| 2 VMs Interco | Public Cloud Managed VM | 0.1 | 0.5 | 0.10 |
| BigQuery Transit Zone | Off-the-shelf app | 0.5 | 0.5 | 0.25 |
| ArgoCD | Off-the-shelf app | 0.5 | 0.8 | 0.40 |
| Authentik | Off-the-shelf app | 0.5 | 1 | 0.50 |
| Vault | Off-the-shelf app | 0.5 | 1 | 0.50 |
| Cloud-native PG (metadata) | Off-the-shelf app | 0.5 | 2 | 1.00 |
| Grafana | Off-the-shelf app | 0.5 | 1 | 0.50 |
| Loki | Off-the-shelf app | 0.5 | 2 | 1.00 |
| Prometheus | Off-the-shelf app | 0.5 | 2 | 1.00 |
| Tempo | Off-the-shelf app | 0.5 | 1 | 0.50 |
| Alloy | Off-the-shelf app | 0.5 | 0.8 | 0.40 |
| Landing zone shared (Transit + Interco) | infra | 0.5 | — | 0.35 |
| **Subtotal SHARED** | | | | **6.75** |
| **× SLA Bronze 1.00** | | | | **6.75** |

#### MCO déductif — Summary

| Environment | MCO brut | SLA Coeff | MCO ajusté |
|-------------|----------|-----------|------------|
| PROD | 6.20 | 1.10 | 6.82 |
| NON-PROD | 3.93 | 1.00 | 3.93 |
| SHARED | 6.75 | 1.00 | 6.75 |
| **Total MCO déductif** | | | **17.50** |

> _Note : arrondi à 17.80 dans le tableau comparatif pour inclure des contingents mineurs (landing zone VPC peering cross-env)._

### MCO calibré (retenu)

| Composante | Détail | j/h/mois |
|------------|--------|----------|
| MCO réactif | ~15-20 tickets/an × 1 j/h moyen | 1.50 |
| MCO proactif | Multi-cluster K8s + LGTM self-hosted + Vault + Airflow + Metabase + HDS hardening (~4× réactif) | 6.00 |
| Buffer SLA | Gold prod (+1.0) + HDS + ISO 27001 overhead réparti sur 3 envs | 1.50 |
| **Total MCO calibré (avant SLA-adjust)** | | **9.00** |
| SLA-ajustement (pondéré sur répartition prod/non-prod/shared) | 0.55 × 1.10 + 0.25 × 1.0 + 0.20 × 1.0 | ×1.055 |
| **Total MCO calibré ajusté** | | **9.50** |

### Gouvernance

| Activité | Fréquence (semi-dédié) | Effort/session | j/h/mois |
|----------|------------------------|----------------|----------|
| COPROD | 1 / mois | 0.5 j/h | 0.50 |
| COPIL | N/A (dédié uniquement) | — | 0.00 |
| ROSE (Qualité) | 1 / semestre | 0.5 j/h | 0.08 |
| YAMAS (HDS) | 1 / semestre | 0.5 j/h | 0.08 |
| LEAF (FinOps / Green IT) | 1 / semestre | 0.5 j/h | 0.08 |
| **Total Gouvernance** | | | **0.75** |

### Évolutions

- **Estimées : 2.5 j/h/mois**
- Basis : Backlog "MEP + stabilisation" sur 6 mois. ~1 chantier technique par mois (durcissement HDS, monitoring, DR, IaC Terraform).
- Mode : temps passé (évolutions sont hors forfait).

### Total quantité

| Category | j/h/mois |
|----------|----------|
| MCO (calibré, SLA-ajusté) | 9.50 |
| Gouvernance | 0.75 |
| Évolutions | 2.50 |
| **Total** | **12.75** |

### Dispositif

- **Total 12.75 j/h/mois** → seuil 10-100 → **Semi-dédié**
- Répartition : Ops 8.5 j/mois (66%) + Lead Ops 2.9 j/mois (23%) + DM 1.4 j/mois (11%)

### Prix (temps passé)

| Ligne | j/h/mois | TJM | Montant |
|-------|----------|-----|---------|
| MCO (ajusté SLA) | 9.50 | 863€ | 8 199€ |
| Gouvernance | 0.75 | 863€ | 647€ |
| Évolutions | 2.50 | 863€ | 2 158€ |
| **Subtotal** | | | **11 004€** |
| Immobilisation (semi-dédié × Standard) | — | — | 0€ |
| **Total mensuel** | **12.75** | | **11 004€** |
| **Total annuel** | | | **132 048€** |

### Prix (forfait +20%)

| Ligne | j/h/mois | TJM | Montant |
|-------|----------|-----|---------|
| MCO + Gouvernance | 10.25 | 863€ | 8 846€ |
| Contingence moyenne (+20%) | | | +1 769€ |
| Évolutions (hors forfait, temps passé) | 2.50 | 863€ | 2 158€ |
| Immobilisation | — | — | 0€ |
| **Total mensuel forfait** | **14.80** | | **12 773€** |
| **Total annuel forfait** | | | **153 276€** |

### Remise multi-annuelle

- Engagement 1 an → **pas de remise** applicable.
- Si engagement 3 ans est validé (mention "renouvelable si ça se passe bien") : remise **-8%** → 121 484€ annuel (temps passé) ou 141 014€ annuel (forfait).

---

## Notes

- **HDS applicable** — Audit YAMAS inclus dans la gouvernance. Périmètre HDS exact à valider (certification client ou hébergement chez un prestataire HDS-certifié).
- **ISO 27001** — Processus revue sécurité intégré au MCO via audits ROSE et YAMAS. Cadence renforcée possible via évolutions si besoin d'audits externes.
- **Nearshore** — Non évalué ici (France par défaut). À discuter avec Hugo / Lila / Manu si le client souhaite optimiser le TJM.
- **Revoyure à 3 mois post-MEP** recommandée pour calibrer l'enveloppe sur données réelles (tickets, incidents, FTE effectifs).
- **Tension plage Standard + HDS** — Si le client demande a posteriori une plage Étendue ou Complète (astreinte hors heures), prévoir +3 000€/mois (immobilisation semi-dédié Étendue = 1 000€ + uplift MCO).
- **Engagement 1 an** — Pas de levier tarifaire multi-annuel à ce stade.
