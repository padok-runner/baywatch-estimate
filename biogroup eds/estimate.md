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
| H9 | Mode des clusters GKE | Confirmé client : **GKE Autopilot** | Coef 0.5 (very low) sur tous les clusters | Autopilot = Google gère nodes, scaling, patching OS, sécurité. Effort ops résiduel = workloads, network policies, IAM | -1 200€/mois vs GKE Standard. Si bascule en mode Standard a posteriori, repasser coef à 1 → +1 200€/mois |

> **⚠ Sensibilité** : Les hypothèses H1 (bulles sécurisées), H2 (interprétation des envs) et H5 (calibration MCO) représentent ensemble ±4 500€/mois, soit ±45% du prix final. **Recommandations** : organiser un atelier de cadrage pour fixer H1/H2 avant contractualisation et prévoir une clause de revoyure MCO à 3 mois post-MEP.

---

## Analyse de réalisme

### Signaux empiriques disponibles

| Signal | Valeur | Source |
|--------|--------|--------|
| Historique de tickets | **Aucun** — infra pas encore en prod | Qualification |
| FTEs actuels | **Non communiqué** | Qualification |
| Maturité infra | **Faible** — mise en prod en cours, stabilisation à faire | Qualification |
| Complexité techno | **Moyenne-haute** — 4 clusters GKE **Autopilot** (nodes managés Google), stack LGTM self-hosted, Vault, Airflow, Metabase OSS+Enterprise, HDS + ISO 27001 | Qualification |
| Taille équipe dev | **Petite** (1-5 devs) | Qualification |
| Profil cloud | **S3ns / GCP managé** + GKE Autopilot — surface de maintenance node-level supprimée. Self-hosting concentré sur les workloads (observabilité, IAM, ETL) | Qualification |

### Calibration empirique

| Composante | j/h/mois | Raisonnement |
|------------|----------|--------------|
| MCO réactif | 1.5 | Nouvelle plateforme EDS. Estimation ~15-20 tickets/an × 1 j/h en moyenne. BigQuery managé + GKE Autopilot réduisent les incidents infra. |
| MCO proactif | 4.5 | ~3× le réactif (réduit grâce à GKE Autopilot : pas de patching nodes, pas d'autoscaler à tuner, pas de capacity planning). Couvre : monitoring config LGTM, mises à jour applicatives, veille sécurité HDS, gestion Vault/IAM. |
| Buffer SLA Gold + HDS + ISO 27001 | 1.5 | Gold (+1.0) + overhead HDS (auditabilité, chiffrement, habilitations) + ISO 27001 (processus revue sécurité). Réparti sur les 3 envs. |
| **Total calibré MCO** | **7.5** | |

### Comparaison déductive vs calibrée

| Approche | MCO j/h/mois | Total j/h/mois (MCO+Gov+Evol) | Prix/mois | Écart |
|----------|--------------|-------------------------------|-----------|-------|
| Déductive pure | 16.97 (SLA-ajusté) | 20.22 | 17 450€ | — |
| **Calibrée (retenue)** | **7.91 (SLA-ajusté)** | **11.16** | **9 631€** | **-45%** |

**Décision :** La déductive surestime le MCO. L'écart vient principalement de :

- **GKE Autopilot** : Google gère intégralement les nodes (provisioning, scaling, patching OS, sécurité). L'effort ops cluster se limite à la gestion des workloads, network policies et IAM — d'où un coef 0.5 (very low) sur les clusters.
- **Dédoublonnage logique** : cert-manager / vault-secrets-operator / alloy déployés dans chaque cluster = 1 seul effort opérationnel réel (configuration GitOps centralisée), pas N.
- **BigQuery entièrement managé** : le coût ops unitaire est minime (IAM, quotas), très inférieur à un off-the-shelf auto-hébergé.
- **Stack LGTM partagée** : Grafana/Loki/Prometheus/Tempo vivent sur le cluster Shared Services uniquement, pas dupliquée par env.

**L'estimation calibrée (7.5 j/h/mois MCO) est retenue.** La déductive (16.97 j/h/mois) sert de plafond de référence. Une clause de revoyure à 3 mois post-MEP est recommandée pour ajuster selon le volume réel de tickets.

---

## Synthèse

|  | PROD | NON-PROD | SHARED | Transverse |
|---|---|---|---|---|
| **Nom des envs** | prod | non-prod | shared | — |
| **Inventaire** | **Servers :**<br>3 GKE **Autopilot** clusters (Data Orchestration, Data Exploration, Bulles sécurisées)<br>**Datasets BigQuery :**<br>Data Warehouse (6), Data Ident (2), Bulles sécurisées (n), Transit Zone (2)<br>**Applications off-the-shelf :**<br>Airflow, Authentik, Keycloak<br>Cloud-native PG, Envoy Gateway<br>Metabase OSS + Enterprise<br>**Resources GCP :**<br>GCS buckets, KMS keys, IPs statiques, Private Service Endpoints | **Servers :**<br>Miroir de prod avec sizing réduit (3 GKE Autopilot clusters dimensionnés staging)<br>**Applications :**<br>Mêmes off-the-shelf avec volumes réduits<br>**Datasets BigQuery :**<br>Sous-ensembles de test | **Servers :**<br>1 GKE **Autopilot** cluster Shared Services<br>2 VMs Interco<br>**Applications off-the-shelf (Shared Services) :**<br>ArgoCD (GitOps)<br>Stack observabilité : Grafana, Loki, Prometheus, Tempo, Alloy<br>Vault (secrets central)<br>Authentik (IAM central)<br>Cloud-native PG (metadata)<br>**Landing zone :**<br>Transit Zone, Interco, VPC peering | Organisation GCP / S3ns, IAM fédéré, stratégie de chiffrement KMS, audit trail |
| **Services** | Maintien : Gestion des demandes de service, des incidents, des problèmes, des changements de version, de la continuité, de la surveillance | Maintien : Gestion des demandes de service, des incidents, des problèmes, des changements de version, de la continuité | Maintien : Gestion des demandes de service, des incidents, des problèmes, des changements de version, de la continuité | Gouvernance : COPROD mensuel, Audits semestriels ROSE, YAMAS (HDS) et LEAF<br>Évolutions : Gestion des changements mineurs |
| **Niveaux de services** | Gold | Bronze | Bronze | — |
| **Plages de service** | Standard | Standard | Standard | — |
| **Dispositif** | Ops : 7.4 j/mois, Lead Ops : 2.5 j/mois, Delivery Manager : 1.2 j/mois | | | |

#### Prix mensuel €HT

| Mode | Périmètre | j/h/mois | Montant €HT/mois |
|------|-----------|----------|-------------------|
| **Temps passé** | MCO (7.91 j/h SLA-ajusté) + Gouvernance (0.75 j/h) | 8.66 | 7 474€ |
| **Temps passé** | Évolutions (à la demande, TJM blended 863€ ou Ops 750€ / Lead Ops 1 200€ / DM 850€) | 2.5 | 2 158€ |
|  | Immobilisation (semi-dédié × Standard) | — | 0€ |
|  | **Total mensuel** | **11.16** | **9 631€** |

**Total annuel : 115 572€ HT**

#### Option Forfait (MCO + Gouvernance, contingence moyenne +20%)

Phase de mise en production + stabilisation = incertitude **moyenne** (+20%).

| Mode | Périmètre | j/h/mois | Montant €HT/mois |
|------|-----------|----------|-------------------|
| **Forfait** | (MCO 7.91 + Gouvernance 0.75) × 1.20 | 10.39 | 8 968€ |
| **Temps passé** | Évolutions | 2.5 | 2 158€ |
|  | **Total mensuel forfait** | **12.89** | **11 126€** |

**Total annuel forfait : 133 512€ HT**

---

## Annexe A — Détail du calcul

### MCO déductif (plafond de référence)

#### Environment: PROD (SLA Gold, coef 1.10)

| Resource | Item Type | Base Rate | Coeff | MCO (j/h/mois) |
|----------|-----------|-----------|-------|----------------|
| 3 clusters GKE **Autopilot** (DataOrch/DataExp/Bulles) | Public Cloud Managed K8s | 0.25 | 0.5 (Autopilot) | 0.38 |
| BigQuery (DataWH + DataIdent + Bulles) | Off-the-shelf app | 0.5 | 0.5 | 0.75 |
| Airflow (DataOrch) | Off-the-shelf app | 0.5 | 2 (high) | 1.00 |
| Authentik + Keycloak (IAM) | Off-the-shelf app | 0.5 | 1 | 1.00 |
| Cloud-native PG | Off-the-shelf app | 0.5 | 2 (high) | 1.00 |
| Envoy Gateway | Off-the-shelf app | 0.5 | 0.8 | 0.40 |
| Metabase OSS + Enterprise | Off-the-shelf app | 0.5 × 2 | 1 | 1.00 |
| Landing zone prod (GCS/KMS/IP/endpoints) | infra | 0.5 | — | 0.30 |
| **Subtotal PROD brut** | | | | **5.83** |
| **× SLA Gold 1.10** | | | | **6.41** |

#### Environment: NON-PROD (SLA Bronze, coef 1.00)

| Resource | Item Type | Base Rate | Coeff | MCO (j/h/mois) |
|----------|-----------|-----------|-------|----------------|
| 3 clusters GKE **Autopilot** (réduits) | Public Cloud Managed K8s | 0.25 | 0.5 (Autopilot) | 0.38 |
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
| 1 cluster GKE **Autopilot** Shared Services | Public Cloud Managed K8s | 0.25 | 0.5 (Autopilot) | 0.13 |
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
| **Subtotal SHARED** | | | | **6.63** |
| **× SLA Bronze 1.00** | | | | **6.63** |

#### MCO déductif — Summary

| Environment | MCO brut | SLA Coeff | MCO ajusté |
|-------------|----------|-----------|------------|
| PROD | 5.83 | 1.10 | 6.41 |
| NON-PROD | 3.93 | 1.00 | 3.93 |
| SHARED | 6.63 | 1.00 | 6.63 |
| **Total MCO déductif** | | | **16.97** |

> _Économie GKE Autopilot vs hypothèse initiale (GKE Standard) : -0.53 j/h/mois sur le total déductif (-0.41 PROD + -0.12 SHARED)._

### MCO calibré (retenu)

| Composante | Détail | j/h/mois |
|------------|--------|----------|
| MCO réactif | ~15-20 tickets/an × 1 j/h moyen | 1.50 |
| MCO proactif | Multi-cluster K8s **Autopilot** (Google gère nodes) + LGTM self-hosted + Vault + Airflow + Metabase + HDS hardening (~3× réactif) | 4.50 |
| Buffer SLA | Gold prod (+1.0) + HDS + ISO 27001 overhead réparti sur 3 envs | 1.50 |
| **Total MCO calibré (avant SLA-adjust)** | | **7.50** |
| SLA-ajustement (pondéré sur répartition prod/non-prod/shared) | 0.55 × 1.10 + 0.25 × 1.0 + 0.20 × 1.0 | ×1.055 |
| **Total MCO calibré ajusté** | | **7.91** |

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
| MCO (calibré, SLA-ajusté) | 7.91 |
| Gouvernance | 0.75 |
| Évolutions | 2.50 |
| **Total** | **11.16** |

### Dispositif

- **Total 11.16 j/h/mois** → seuil 10-100 → **Semi-dédié** (proche du seuil mutualisé)
- Répartition : Ops 7.4 j/mois (66%) + Lead Ops 2.5 j/mois (23%) + DM 1.2 j/mois (11%)

### Prix (temps passé)

| Ligne | j/h/mois | TJM | Montant |
|-------|----------|-----|---------|
| MCO (ajusté SLA) | 7.91 | 863€ | 6 826€ |
| Gouvernance | 0.75 | 863€ | 647€ |
| Évolutions | 2.50 | 863€ | 2 158€ |
| **Subtotal** | | | **9 631€** |
| Immobilisation (semi-dédié × Standard) | — | — | 0€ |
| **Total mensuel** | **11.16** | | **9 631€** |
| **Total annuel** | | | **115 572€** |

### Prix (forfait +20%)

| Ligne | j/h/mois | TJM | Montant |
|-------|----------|-----|---------|
| MCO + Gouvernance | 8.66 | 863€ | 7 474€ |
| Contingence moyenne (+20%) | | | +1 495€ |
| Évolutions (hors forfait, temps passé) | 2.50 | 863€ | 2 158€ |
| Immobilisation | — | — | 0€ |
| **Total mensuel forfait** | **12.89** | | **11 126€** |
| **Total annuel forfait** | | | **133 512€** |

### Remise multi-annuelle

- Engagement 1 an → **pas de remise** applicable.
- Si engagement 3 ans est validé (mention "renouvelable si ça se passe bien") : remise **-8%** → 106 326€ annuel (temps passé) ou 122 831€ annuel (forfait).

---

## Notes

- **GKE Autopilot** — Mode confirmé client. Google opère intégralement les nodes (provisioning, scaling, patching OS, hardening sécurité). L'effort ops résiduel se concentre sur les workloads, network policies et IAM. Économie d'environ -1 500€/mois vs un déploiement GKE Standard équivalent.
- **HDS applicable** — Audit YAMAS inclus dans la gouvernance. Périmètre HDS exact à valider (certification client ou hébergement chez un prestataire HDS-certifié).
- **ISO 27001** — Processus revue sécurité intégré au MCO via audits ROSE et YAMAS. Cadence renforcée possible via évolutions si besoin d'audits externes.
- **Nearshore** — Non évalué ici (France par défaut). À discuter avec Hugo / Lila / Manu si le client souhaite optimiser le TJM.
- **Revoyure à 3 mois post-MEP** recommandée pour calibrer l'enveloppe sur données réelles (tickets, incidents, FTE effectifs).
- **Tension plage Standard + HDS** — Si le client demande a posteriori une plage Étendue ou Complète (astreinte hors heures), prévoir +3 000€/mois (immobilisation semi-dédié Étendue = 1 000€ + uplift MCO).
- **Engagement 1 an** — Pas de levier tarifaire multi-annuel à ce stade.
- **Bascule mutualisé envisageable** — Total 11.16 j/h/mois, proche du seuil mutualisé (10 j/h). Si évolutions tombent à <1 j/h/mois post-stabilisation, le dispositif peut basculer en mutualisé (immobilisation 500€/mois + équipe ~7 personnes pour ~20 clients).
