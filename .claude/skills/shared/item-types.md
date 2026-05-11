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

**Application :**
```
MCO_pour_N_ressources_de_même_type_et_coeff = base_rate × multiplier(N) × coefficient
```

Le scaling se fait **globalement** (toutes envs confondues) au sein d'un même bucket `(type, coeff)`. Le coût est ensuite distribué prorata du count par env, puis le SLA s'applique par env (cf. `service-levels.md`). Cela reflète le bénéfice automation/orchestration partagé entre envs.

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
