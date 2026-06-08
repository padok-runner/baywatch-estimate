# Item Types & MCO Base Rates

## Principes

L'effort MCO se ventile en deux couches :

1. **Substrate** — où le code tourne (cluster, VM, container service, hypervisor)
2. **Application** — ce qui tourne dessus (DB, cache, search, app custom)

Les deux se cumulent. Une MySQL self-hosted sur EC2 = 1 ligne substrate (la VM) + 1 ligne app self-hosted (le MySQL).

## Scaling sublinéaire (loi de puissance)

Quand un client a N ressources du **même type** et **même coefficient taille/complexité** (ex. 11 EC2 Debian medium), l'effort ne scale pas linéairement — patching, monitoring, backup s'orchestrent. Mais l'amortissement n'est pas non plus logarithmique : 1000 VMs génèrent plus d'incidents, de drift et de coordination que 10, même avec automation. On modélise l'effort par une **loi de puissance** :

```
scale(N) = N^k          avec k ≈ 0.8
```

Le coût par ressource décroît en `N^(k−1) = N^(−0.2)` : doux et régulier, sans seuil ni cassure. `k = 1` serait linéaire (aucune économie d'échelle) ; `k → 0` un coût fixe quel que soit N ; `k = 0.8` une économie d'échelle modérée — cohérente avec un parc managé où l'automation amortit mais où chaque ressource garde un coût marginal réel.

| N | scale(N) = N^0.8 | coût/ressource vs solo (N^−0.2) |
|---|---|---|
| 1 | 1.00 | 1.00 |
| 2 | 1.74 | 0.87 |
| 3 | 2.41 | 0.80 |
| 5 | 3.62 | 0.72 |
| 10 | 6.31 | 0.63 |
| 30 | 15.20 | 0.51 |
| 100 | 39.81 | 0.40 |
| 480 | 139.6 | 0.29 |
| 1000 | 251.2 | 0.25 |

> **Calibration de k.** k = 0.8 est ancré sur la cible « le coût par instance à 100 instances vaut ~40 % du coût d'une instance isolée ». C'est la *pente* (shape) ; le *niveau* absolu par type vient du `base_rate × TJM` de chaque item (inchangé dans cette révision). k est le paramètre maître — à recalibrer sur des engagements réels clos, pas à asséner comme une loi. Loi de puissance pure : l'amortissement démarre dès la 2ᵉ ressource (N=2 → 1.74, pas 2.0), contrairement au « 3 premières comptent plein » de l'ancienne formule sqrt.

### Fragmentation multi-tenants — même exposant

L'amortissement suppose que les N ressources sont opérables ensemble (même fenêtre de maintenance, même script de patching, même comms). **Dès que les ressources sont réparties sur plusieurs tenants** (SELAS, BUs, comptes clients indépendants avec change-management séparé), l'amortissement se casse partiellement : tickets de changement séparés, fenêtres de maintenance distinctes, validations multiples.

Comme la loi de puissance est concave, il suffit de **sommer par tenant** : la fragmentation tombe du même exposant k, sans paramètre supplémentaire.

```
scale(N, T) = Σ_tenant (N_t^k)              # exact, distribution quelconque
            = T^(1−k) × N^k                  # tenants de taille égale (cas courant)

T = nombre de tenants distincts sur le bucket (défaut 1)
cas particuliers :
  T = 1             : scale = N^k             (single-tenant — courbe ci-dessus)
  N_t = 1 ∀ tenant  : scale = T = N           (1 instance par tenant → aucun amortissement)
```

La fragmentation se résume à un facteur propre `× T^(1−k) = T^0.2` sur la courbe single-tenant :

| T | facteur T^0.2 |
|---|---|
| 1 | 1.00 |
| 5 | 1.38 |
| 10 | 1.58 |
| 20 | 1.82 |
| 30 | 1.97 |
| 100 | 2.51 |

Exemples combinés :

| Exemple bucket | N | T | N^0.8 (T=1) | scale(N, T) | Ratio |
|---|---|---|---|---|---|
| 480 VMs très petites en 1 tenant | 480 | 1 | 139.6 | 139.6 | 1.00× |
| 480 VMs très petites sur 30 SELAS | 480 | 30 | 139.6 | 1.97 × 139.6 = 275.6 | 1.97× |
| 30 stacks middleware (1 par SELAS) | 30 | 30 | 15.2 | = N = 30.0 | 1.97× |
| 100 VMs sur 10 comptes cloisonnés | 100 | 10 | 39.81 | 1.58 × 39.81 = 63.1 | 1.58× |
| 12 ADs consolidés en landing zone | 12 | 1 | 7.30 | 7.30 | 1.00× |

**Application :**
```
MCO_pour_bucket = base_rate × scale(N, T) × coefficient
```

Le scaling se fait au sein d'un même bucket `(type, coeff, tenants_spanned)`. Si les ressources d'un même `(type, coeff)` ont des `tenants_spanned` différents (ex. 480 VMs prod par-SELAS + 121 VMs non-prod consolidé), elles forment **deux buckets distincts** — leurs `scale` s'additionnent naturellement.

Le coût est ensuite distribué prorata du count par env au sein de chaque bucket, puis le SLA s'applique par env (cf. `service-levels.md`).

### Quand déclarer T > 1

| Situation | T |
|---|---|
| Single-tenant (client mono-entité) | 1 (défaut) |
| Multi-tenant explicite : N SELAS, N BUs, N filiales avec change-management indépendants | N |
| Landing zone ou shared services consolidés en multi-tenant | 1 (consolidation = un seul tenant opérationnel) |
| Plateforme unique (cluster K8s, lake) qui sert N tenants | 1 (la plateforme est un seul tenant opérationnel ; les tenants applicatifs au-dessus sont une autre dimension) |
| Per-environnement (prod / non-prod / shared) | T compte les tenants au sein de l'env, pas les envs (la séparation par env est déjà gérée par la distribution prorata) |

> **Important** — la fragmentation multi-tenant adresse le cloisonnement **interne au client** (un client avec 30 SELAS). Elle ne s'applique pas à la mutualisation côté équipe (le pool Mutualisé sert 20 clients, mais c'est une économie côté TJM, pas une pénalité côté client).

---

## Substrate layer

### Managed K8s cluster (EKS, GKE, AKS, OVH MKS)
- **Base :** 0.25 j/h/mois pour 1 cluster
- **Couvre :** coordination upgrades cluster (control plane piloté provider) ; review RBAC, network policies, ingress ; version tracking add-ons (CNI, CSI, autoscaler) ; surveillance santé cluster, capacity nodes. Pas de patching control plane (provider-managed).
- **Scaling :** rare d'avoir N>1, mais scale(N, T) si multi-cluster.

### Self-hosted K8s cluster
- **Base :** 0.5 j/h/mois pour 1 cluster
- **Couvre :** tout ce que le managed couvre, **plus** patching control plane (apiserver, etcd, scheduler) ; patching node OS ; operations etcd (snapshots, restores) ; networking layer manuel (CNI, MetalLB...).
- **Scaling :** scale(N, T).

### Public cloud managed VM (EC2, GCE, Azure VM)
- **Base :** 0.1 j/h/mois pour 1 VM
- **Couvre :** patching OS mensuel (sécurité + minor) ; reboots planifiés ; monitoring système (CPU, mem, disk, réseau) ; surveillance certificats TLS ; vérif backup VM ; logs review hebdo.
- **Scaling :** scale(N, T) au sein de (même OS, même coeff). Ex. 11 EC2 Debian coeff 0.8 = `0.1 × 6.81 × 0.8 = 0.545 j/h` (vs 0.88 linéaire).

### Private cloud managed VM (VMware, OpenStack)
- **Base :** 0.1 j/h/mois pour 1 VM
- **Couvre :** identique au public, plus coordination avec l'équipe hypervisor pour les opérations.
- **Scaling :** scale(N, T).

### Public cloud managed container service (ECS, Fargate, Cloud Run)
- **Base :** 0.05 j/h/mois pour 1 service
- **Couvre :** review service definition (CPU/mem, scaling rules) ; surveillance auto-scaling ; pas d'OS, pas de runtime à patcher.
- **Scaling :** scale(N, T).

### Hypervisor (private cloud)
- **Base :** 0.5 j/h/mois pour 1 hypervisor
- **Couvre :** patching hypervisor, capacity planning, networking, storage backend, monitoring physique.
- **Scaling :** scale(N, T).

---

## Application layer

### Managed off-the-shelf application (RDS, ElastiCache, MSK, Opensearch managed, etc.)
- **Base :** 0.3 j/h/mois pour 1 instance
- **Couvre :** tracking minor versions disponibles ; coordination maintenance windows ; review parameter groups ; slow query review (DB) ; vérif backup automatisé ; pas de patching infra (provider-managed).
- **Scaling :** scale(N, T) au sein de (même engine, même coeff).

### Self-hosted off-the-shelf application (MySQL on VM, Postgres on VM, Kafka self-managed, ClickHouse self-managed, etc.)
- **Base :** 0.6 j/h/mois pour 1 instance
- **Couvre :** identique au managed, **plus** patching applicatif manuel (sécurité + version) ; configuration backup manuelle + tests restauration ; replication setup, failover, slot management ; tuning manuel (buffer pool, cache, indexing).
- **Scaling :** scale(N, T).

> Le self-hosted **se cumule avec la ligne substrate** (la VM ou le container qui héberge). Ex. MySQL 5 on EC2 = 0.1 (VM) + 0.6 (self-hosted off-the-shelf) = 0.7 j/h. Vs RDS managed = 0.3 j/h. Différentiel honnête entre managé et self-hosted.

### Custom application (apps maintenues par le client)
- **Base :** 0.25 j/h/mois pour 1 app
- **Couvre :** surveillance déploiements (succès/rollbacks) ; alert tuning (anti-bruit) ; investigation logs/traces sur incidents applicatifs ; coordination releases avec équipe dev client.
- **Scaling :** scale(N, T).

---

## Coefficients taille/complexité

Voir `coefficients.md`. Le coefficient s'applique **après** le scaling :

```
MCO = base × scale(N, T) × coefficient
```

Quand un type a à la fois server-size et application-complexity, prendre le **plus haut** des deux.

## Calibration

Ces base rates (et l'exposant k) sont à recalibrer périodiquement contre des clients dont le prix est connu et accepté. Si le calcul sort un prix > ±15 % du prix accordé pour ≥2 clients existants, **ajuster les bases** (ou k), pas la structure de la formule.
