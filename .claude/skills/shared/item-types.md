# Item Types & MCO Base Rates

## Principes

L'effort MCO se ventile en deux couches :

1. **Substrate** — où le code tourne (cluster, VM, container service, hypervisor)
2. **Application** — ce qui tourne dessus (DB, cache, search, app custom)

Les deux se cumulent. Une MySQL self-hosted sur EC2 = 1 ligne substrate (la VM) + 1 ligne app self-hosted (le MySQL).

## Scaling sublinéaire

Quand un client a N ressources du **même type** et **même coefficient taille/complexité** (ex. 11 EC2 Debian medium), l'effort ne scale pas linéairement — patching, monitoring, backup s'orchestrent. Mais l'amortissement n'est pas non plus logarithmique : 1000 VMs génèrent plus d'incidents, de drift et de coordination que 10, même avec automation. Formule (racine carrée piecewise, dans l'esprit Erlang C / Universal Scalability Law) :

```
multiplier(N) = min(N, 3) + sqrt(max(N/3, 1)) - 1

Équivalent piecewise :
  multiplier(N) = N                  pour N ≤ 3
                = 2 + sqrt(N/3)      pour N > 3
```

| N | multiplier | Lecture |
|---|---|---|
| 1 | 1.00 | full effort, ressource isolée |
| 2 | 2.00 | quasi-linéaire, peu d'amortissement |
| 3 | 3.00 | seuil d'orchestration (linéaire jusqu'ici) |
| 5 | 3.29 | sqrt kicks in |
| 10 | 3.83 | |
| 30 | 5.16 | |
| 100 | 7.77 | |
| 1000 | 20.26 | l'orchestration amortit mais ne s'aplatit pas |

Linéaire jusqu'à 3 ressources (chaque nouvelle compte plein, on n'a pas encore amorti l'orchestration), puis racine carrée au-delà. La √(N/3) suit le pattern Erlang C : l'effort opérationnel scale moins que linéairement mais ne s'aplatit pas comme du log10 — un parc de 1000 VMs reste un parc qui génère des incidents, du drift, des fenêtres de maintenance.

### Pénalité de fragmentation multi-tenants (v3)

L'amortissement de la formule sqrt suppose que les N ressources sont opérables ensemble (même fenêtre de maintenance, même script de patching, même comms). **Dès que les ressources sont réparties sur plusieurs tenants** (SELAS, BUs, comptes clients indépendants), l'amortissement se casse : T tickets de changement séparés, T fenêtres de maintenance, T validations.

Pour chaque bucket, la qualification déclare `tenants_spanned T` (défaut 1) :

```
m_T(N, T) = sqrt(T) × multiplier(N / sqrt(T))

cas particuliers :
  T = 1                  : m_T = multiplier(N)            (comportement v2 inchangé)
  T = N (1 ressource/tenant) : m_T = sqrt(N) × m(sqrt(N)) (proche de m(N) mais ré-amorti)
  T quelconque, N grand  : m_T ≈ sqrt(T) × sqrt(N/(3T))   (l'effort grossit en √(T) × √(N))
```

Interprétation : on imagine le bucket découpé en `sqrt(T)` sous-pools parallèles de taille `N/sqrt(T)` chacun. Chaque sous-pool amortit en interne ; la coordination entre sous-pools suit la même économie sqrt (l'équipe apprend une fois, applique plusieurs fois).

| Exemple bucket | N | T | m(N) | m_T(N, T) | Ratio |
|---|---|---|---|---|---|
| 480 VMs très petites en 1 tenant | 480 | 1 | 14.65 | 14.65 | 1.00× |
| 480 VMs très petites sur 30 SELAS | 480 | 30 | 14.65 | 5.48 × 8.11 = 44.46 | 3.03× |
| 30 stacks middleware (1 par SELAS) | 30 | 30 | 5.16 | 5.48 × 3.35 = 18.37 | 3.56× |
| 12 ADs consolidés en landing zone | 12 | 1 | 4.00 | 4.00 | 1.00× |
| 7 BDDs consolidées par SELAS-group | 7 | 7 | 3.53 | 2.65 × 2.65 = 7.00 | 1.98× |

**Application v3 :**
```
MCO_pour_bucket = base_rate × m_T(N, T) × coefficient
```

Le scaling se fait au sein d'un même bucket `(type, coeff, tenants_spanned)`. Si les ressources d'un même `(type, coeff)` ont des `tenants_spanned` différents selon l'env (ex. 480 VMs prod par-SELAS + 121 VMs non-prod consolidé), elles forment **deux buckets distincts**.

Le coût est ensuite distribué prorata du count par env au sein de chaque bucket, puis le SLA s'applique par env (cf. `service-levels.md`).

### Quand déclarer T > 1

| Situation | T |
|---|---|
| Single-tenant (client mono-entité) | 1 (défaut) |
| Multi-tenant explicite : N SELAS, N BUs, N filiales avec change-management indépendants | N |
| Landing zone ou shared services consolidés en multi-tenant | 1 (consolidation = un seul tenant opérationnel) |
| Plateforme unique (cluster K8s, lake) qui sert N tenants | 1 (la plateforme est un seul tenant opérationnel ; les tenants applicatifs au-dessus sont une autre dimension) |
| Per-environnement (prod / non-prod / shared) | T compte les tenants au sein de l'env, pas les envs (la séparation par env est déjà gérée par la distribution prorata) |

> **Important** — la pénalité multi-tenant adresse la fragmentation **interne au client** (un client avec 30 SELAS). Elle ne s'applique pas à la mutualisation côté équipe (le pool Mutualisé sert 20 clients, mais c'est une économie côté TJM, pas une pénalité côté client).

---

## Substrate layer

### Managed K8s cluster (EKS, GKE, AKS, OVH MKS)
- **Base :** 0.25 j/h/mois pour 1 cluster
- **Couvre :** coordination upgrades cluster (control plane piloté provider) ; review RBAC, network policies, ingress ; version tracking add-ons (CNI, CSI, autoscaler) ; surveillance santé cluster, capacity nodes. Pas de patching control plane (provider-managed).
- **Scaling :** rare d'avoir N>1, mais multiplier(N) si multi-cluster.

### Self-hosted K8s cluster
- **Base :** 0.5 j/h/mois pour 1 cluster
- **Couvre :** tout ce que le managed couvre, **plus** patching control plane (apiserver, etcd, scheduler) ; patching node OS ; operations etcd (snapshots, restores) ; networking layer manuel (CNI, MetalLB...).
- **Scaling :** multiplier(N).

### Public cloud managed VM (EC2, GCE, Azure VM)
- **Base :** 0.1 j/h/mois pour 1 VM
- **Couvre :** patching OS mensuel (sécurité + minor) ; reboots planifiés ; monitoring système (CPU, mem, disk, réseau) ; surveillance certificats TLS ; vérif backup VM ; logs review hebdo.
- **Scaling :** multiplier(N) au sein de (même OS, même coeff). Ex. 11 EC2 Debian coeff 0.8 = `0.1 × 3.92 × 0.8 = 0.313 j/h` (vs 0.88 linéaire).

### Private cloud managed VM (VMware, OpenStack)
- **Base :** 0.1 j/h/mois pour 1 VM
- **Couvre :** identique au public, plus coordination avec l'équipe hypervisor pour les opérations.
- **Scaling :** multiplier(N).

### Public cloud managed container service (ECS, Fargate, Cloud Run)
- **Base :** 0.05 j/h/mois pour 1 service
- **Couvre :** review service definition (CPU/mem, scaling rules) ; surveillance auto-scaling ; pas d'OS, pas de runtime à patcher.
- **Scaling :** multiplier(N).

### Hypervisor (private cloud)
- **Base :** 0.5 j/h/mois pour 1 hypervisor
- **Couvre :** patching hypervisor, capacity planning, networking, storage backend, monitoring physique.
- **Scaling :** multiplier(N).

---

## Application layer

### Managed off-the-shelf application (RDS, ElastiCache, MSK, Opensearch managed, etc.)
- **Base :** 0.3 j/h/mois pour 1 instance
- **Couvre :** tracking minor versions disponibles ; coordination maintenance windows ; review parameter groups ; slow query review (DB) ; vérif backup automatisé ; pas de patching infra (provider-managed).
- **Scaling :** multiplier(N) au sein de (même engine, même coeff).

### Self-hosted off-the-shelf application (MySQL on VM, Postgres on VM, Kafka self-managed, ClickHouse self-managed, etc.)
- **Base :** 0.6 j/h/mois pour 1 instance
- **Couvre :** identique au managed, **plus** patching applicatif manuel (sécurité + version) ; configuration backup manuelle + tests restauration ; replication setup, failover, slot management ; tuning manuel (buffer pool, cache, indexing).
- **Scaling :** multiplier(N).

> Le self-hosted **se cumule avec la ligne substrate** (la VM ou le container qui héberge). Ex. MySQL 5 on EC2 = 0.1 (VM) + 0.6 (self-hosted off-the-shelf) = 0.7 j/h. Vs RDS managed = 0.3 j/h. Différentiel honnête entre managé et self-hosted.

### Custom application (apps maintenues par le client)
- **Base :** 0.25 j/h/mois pour 1 app
- **Couvre :** surveillance déploiements (succès/rollbacks) ; alert tuning (anti-bruit) ; investigation logs/traces sur incidents applicatifs ; coordination releases avec équipe dev client.
- **Scaling :** multiplier(N).

---

## Coefficients taille/complexité

Voir `coefficients.md`. Le coefficient s'applique **après** le multiplier :

```
MCO = base × multiplier(N) × coefficient
```

Quand un type a à la fois server-size et application-complexity, prendre le **plus haut** des deux.

## Calibration

Ces base rates sont à recalibrer périodiquement contre des clients dont le prix est connu et accepté. Si v2 sort un prix > ±15% du prix accordé pour ≥2 clients existants, **ajuster les bases**, pas la formule.
