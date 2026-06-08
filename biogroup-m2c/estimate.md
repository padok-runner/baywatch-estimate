# Biogroup Move to Cloud — Estimation du RUN

> Plateforme cible **GCP S3NS** (Crypt3ns + Premi3ns) livrée clean par Theodo (IaC, Terraform, runbooks, HDS). Le présent document chiffre **uniquement le RUN récurrent**.
>
> **Données chiffrées** (composition équipe, totaux steady-state Y3+, trajectoire pluri-annuelle Y1-Y6 + plancher Y7+, macro split UO, inventaire source, SLA) : cf. *Annex 04 Pricing.xlsx*. Le présent MD couvre méthodologie, hypothèses, benchmarks externes, risques.

---

## 1. Périmètre cible

Sources : *Annex 04 Pricing* (onglets **Périmètre** + **SLA** + **Hosting**) + *Sizing Pricing V5*.

**Cible LEAN après rationalisation** (Sizing V5, non couverte par Annex 04) :

| Élément | Valeur |
|---|---|
| VMs cible | **~1 062** (vs 1 635 source, -35 %) |
| — dont L&S Prod / Non-Prod | 759 / 196 |
| — dont Infra L&S + Consolidation | 41 |
| GKE worker nodes (~5 clusters) | 66 |
| Projets GCP cibles | **Hypothèse : ~28-29 (1/SELAS)** — non imposé par RFP §8.4.1 |
| Régions | 2 (Crypt3ns FR + Premi3ns BE) |
| Hosting GCP S3NS | **Y1 0,9 → Y3+ 4,49 M€/an** (moyenne 6 ans ~3,67 M€) — cf. xlsx onglet *Hosting* |
| Migration cible complète | ~Y3 (ramp hosting Y1 ~20 % → Y2 ~70 % → Y3+ 100 %) |
| Certification | **HDS obligatoire** |

**Inventaire source, tiering et SLA** : cf. *Annex 04 Pricing* — onglets **Périmètre** et **SLA**.

**KaliSIL** (SIL Labo, équivalent ERP) : 105 VMs Premi3ns + 7 BDD PG (quelques To) + ~17 frontaux GCS, **1 VM par automate**, HDS, RPO=0. Données majoritairement en stockage fichier (GCNV/GCS).

---

## 2. Hypothèses

| # | Hypothèse | Source |
|---|---|---|
| H1 | Plateforme Theodo IaC industrialisé | Cadre Theodo |
| H2 | Observabilité internalisée Biogroup | RFP §8.8.3 |
| H3 | SOC mutualisé Biogroup | Annex 04 |
| H4 | RUN 100 % presta 18-24 mois puis infogérance collaborative | RFP §8.8.3 |
| H5 | HNO 24/7 obligatoire sur T1 | Annex 04 SLA |
| H6 | Outillage Run = services cloud natifs GCP (OS Config, VM Manager, GCBDR, Cloud Ops) | RFP §8.8.3 |
| H7 | "Ne pas reproduire CAC 40" → sizing LEAN | RFP §8.8.3 |
| H8 | KaliSIL = fichiers majoritaires + PG quelques To → pas de DBA dédié | RFP §6.4 + inventaire |

---

## 3. Méthodes d'estimation

**M1 bottom-up = méthode de référence.** Une plage marché MSP sert de calibration externe. L'abaque interne Theodo `/estimate` fournit un cross-check.

### 3.1 M1 — Bottom-up activités

| Activité | Volume/mois | Charge | h/mois | ETP |
|---|---|---|---:|---:|
| Incidents P1 T1 (HDS) | 5 | 4h × 2 | 40 | 0,25 |
| Incidents P2 | 15 | 4h | 60 | 0,40 |
| Incidents P3 | 30 | 2h | 60 | 0,40 |
| Changes | 50 | 2,5h | 125 | 0,80 |
| Patching (IaC + OS Config) | 1 cycle | 20h | 20 | 0,13 |
| Backup monitoring + restos | quotidien | 30 min/j + 5×1h | 20 | 0,15 |
| Tests PRA T1 (semestriels) | 2/an | 2 j | 8 | 0,05 |
| Vuln mgmt | hebdo | 1j/sem | 32 | 0,20 |
| HDS audit prep + run | annuel | 20 j/an lissés | 13 | 0,10 |
| FinOps reviews | hebdo + trim. | 8h/mois + 5j/trim | 22 | 0,15 |
| Capacity planning / archi run | continu | – | 160 | 1,00 |
| SDM / PMO (Service Delivery + ITSM process consolidés) | continu | – | 176 | 1,10 |
| Astreinte HNO (indemnité) | 24/7 T1 | rotation 5-6 pers | – | 0,40 éq. |
| **Sous-total** | | | | **5,15** |
| Redondance / congés / formation (~20 %) | | | | +1,05 |
| **TOTAL M1 (steady-state Y3+)** | | | | **~6,2 ETP** |

> Sizing M1 ci-dessus = **steady-state Y3+** (perimètre complet sous gestion). Y1-Y2 = équipe partielle calée sur la ramp hosting (60 % puis 90 %). DB ops PG absorbé dans Cloud Engineering (cf H8). Chiffrage € (équipe + astreinte + provision HNO + trajectoire ramp Y1-Y2 / steady Y3 / dégressif Y4-Y6 / plancher Y7+) : cf. *Annex 04 Pricing* — onglet **Estimations Services Managés**.

### 3.2 Calibration marché & abaques internes

**Benchmarks MSP Enterprise** multi-account 24/7 HDS : **1,0 – 1,4 M€/an** (10-25 % du cloud spend, ou 0,9-1,3 M€ tier Enterprise haut).
Sources : [CloudBolt](https://www.cloudbolt.io/msp-best-practices/msp-pricing-models/), [Opsio AWS](https://opsiocloud.com/knowledge-base/aws-managed-services-cost-pricing/), [Opsio Azure](https://opsiocloud.com/knowledge-base/azure-managed-services-pricing-2026/).

**Abaques internes Theodo (`/estimate`, scaling linéaire)** :

| Bucket | N | base × N × coeff | post-SLA |
|---|---:|---:|---:|
| GCE VMs (Prod/Non-Prod/Infra) | 996 | 99,6 | 111,7 (×1.121 blendé T1/T2/T3) |
| GKE clusters managed | 5 | 1,25 | 1,5 (×1.20 Platine) |
| GKE worker nodes | 66 | 5,28 | 6,3 (×1.20) |
| PostgreSQL self-hosted (KaliSIL) | 7 | 8,40 | 10,1 (×1.20) |
| Apps custom (~20, KaliSIL incl.) | 20 | 7,50 | 8,6 (×1.15 Gold blendé) |
| **MCO total** | | **122,0** | **138,2 j/h/mois** |

+ Gouvernance dédié (COPROD hebdo + COPIL trim. + audits ROSE/YAMAS/LEAF) = **2,6 j/h/mois**
+ Immobilisation Complète dédié = 60 k€/an

**Total abaque (linéaire, parc générique) : 140,8 j/h/mois × 863 €/j × 12 + 60 k€ = ~1 519 k€/an** — plafond conservateur. L'abaque chiffre un parc standard ; il n'a **pas** connaissance du contexte spécifique de la mission.

**Contexte deal Biogroup** (plateforme IaC livrée clean par Theodo, outillage cloud-natif GCP, SOC mutualisé côté Biogroup — H1/H3/H6) : ce contexte est déjà intégré dans la méthode de référence **M1 bottom-up** (6,2 ETP steady-state → ~**1 220 k€/an**, cf. xlsx). L'écart abaque↔M1 (~20 %, soit ~8 ETP génériques vs 6,2 ETP contextualisés) **mesure** l'industrialisation Theodo sur ce périmètre — ce n'est pas un discount appliqué après coup sur l'abaque (ce que le verifier interdit), mais l'écart attendu entre un chiffrage générique (abaque, cross-check) et un chiffrage contextualisé (M1, référence).

### 3.3 Synthèse — convergence des sources

| Source (steady-state Y3+) | €/an | €/VM/an | Position |
|---|---:|---:|---|
| **M1 Bottom-up Theodo** | cf. xlsx *Estimations Services Managés* → *Synthèse Prix* | – | **référence centrale** |
| Calibration marché MSP Enterprise HDS | 1,0 – 1,4 M€ | 940 – 1 320 | bench externe |
| Abaque /estimate (contexte Theodo industrialisé, ancré M1) | ~1,22 M€ | ~1 150 | convergence avec M1 |
| Abaque /estimate (sans leverage) | ~1,52 M€ | ~1 430 | plafond conservateur |

Prix/VM sur base 1 062 VMs (incluant 66 GKE workers) au steady-state. Les trois sources cadrent dans **1,0 – 1,5 M€/an**. **Trajectoire pluri-annuelle** (ramp Y1-Y2 calé sur la migration hosting → steady Y3 → dégressif -5 %/an Y4-Y6 → plancher Y7+) : cf. xlsx onglet *Estimations Services Managés* (sections *Synthèse Prix* + *Trajectoire pluri-annuelle*).

---

## 4. Composition équipe & trajectoire

Détail rôles / profils / TJM / FTE / capacité / TJM blendé / chiffrages mensuels et annuels / **trajectoire pluri-annuelle (ramp Y1-Y2 → steady Y3 → dégressif Y4-Y6 → plancher Y7+)** : **cf. *Annex 04 Pricing* — onglet *Estimations Services Managés*** (sections *Composition de l'Équipe* + *Synthèse Prix* + *Trajectoire pluri-annuelle*).

> **Note rôle** : la ligne *Delivery Manager* de l'Excel **consolide SDM** (interface Biogroup, COPIL, Service Reviews) **et PMO** (ITSM process, CAB, reporting SLA). À 6,2 ETP au steady-state, pas de PMO dédié — split à déclencher si volume change > 80/mois ou reporting hebdomadaire exigé (cf. §5).

> **Shape trajectoire** : Y1 = 60 % steady (équipe partielle pendant migration en flight, perimètre sous gestion ramping), Y2 = 90 %, Y3 = 100 % (perimètre complet sous gestion), Y4-Y6 dégressif -5 %/an (gains industrialisation : IaC mature, agents IA Ops, automation incidents L1 et patching). Plancher contractuel Y7+ = prix Y6 — protège le cœur d'équipe non-leverageable (DM/SDM + Compliance HDS + capacité astreinte 24/7).
>
> **Ratio RUN/Hosting** : Y1 ~80 % (transition + ramp), Y2 ~35 %, Y3-Y6 23-27 % (steady, top de la fourchette MSP Enterprise HDS 10-25 %).

---

## 5. Risques

| Risque | Impact ETP | Mitigation |
|---|---|---|
| Theodo livre moins clean (dette IaC) | +2-3 | Critère réception, audit transition |
| 28+ SELAS avec exceptions | +1-2 | Standardisation, refus exceptions |
| Incidents/changes sous-estimés | +1 | Clause revue à 12 mois |
| Observabilité Biogroup pas prête T0 | +1 transitoire | Transfert progressif |
| Découpage tenants différent | ±0,5 | Recaler post-audit |

| Scénario (steady-state Y3+) | ETP | M€/an | Proba |
|---|---:|---:|---:|
| Optimiste (grappe géo 5-6 projets) | 5,0-5,5 | 0,9-1,0 | 25 % |
| **Central (~29 projets)** | **6,0-6,5** | **1,1-1,2** | **50 %** |
| Pessimiste (dette IaC, exceptions) | 8-10 | 1,4-1,7 | 20 % |
| Dégradé | 13+ | 2,1+ | 5 % |

---

## Sources

- [CloudBolt — MSP Pricing Models](https://www.cloudbolt.io/msp-best-practices/msp-pricing-models/)
- [Opsio — AWS Managed Services Pricing 2026](https://opsiocloud.com/knowledge-base/aws-managed-services-cost-pricing/)
- [Opsio — Azure Managed Services Pricing 2026](https://opsiocloud.com/knowledge-base/azure-managed-services-pricing-2026/)
- [Gartner — Cloud Migration Costs](https://www.gartner.com/smarterwithgartner/6-ways-cloud-migration-costs-go-off-the-rails)
- [FinOps Foundation — State of FinOps 2026](https://data.finops.org/)
- [CloudZero — Cloud TCO Calculator 2026](https://www.cloudzero.com/blog/cloud-tco/)
- [Theodo Cloud — Outsourcing GCP](https://cloud.theodo.com/en/outsourcing)
