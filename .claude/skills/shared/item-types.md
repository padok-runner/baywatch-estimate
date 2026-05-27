# Item Types & MCO Base Rates

## Principes

L'effort MCO se ventile en deux couches :

1. **Substrate** — où le code tourne (cluster, VM, container service, hypervisor)
2. **Application** — ce qui tourne dessus (DB, cache, search, app custom)

Les deux se cumulent. Une MySQL self-hosted sur EC2 = 1 ligne substrate (la VM) + 1 ligne app self-hosted (le MySQL).

## Scaling linéaire

L'effort MCO scale **linéairement** avec le nombre de ressources d'un même bucket `(type, coeff)`. Une parc de 1000 VMs génère 1000× l'effort d'une VM isolée — l'automation amortit le coût marginal **par ressource** (capturé dans le `base_rate` calibré), pas le total.

```
MCO_pour_N_ressources_de_même_type_et_coeff = base_rate × N × coefficient
```

| N | multiplier | Lecture |
|---|---:|---|
| 1 | 1 | ressource isolée |
| 10 | 10 | parc petit |
| 100 | 100 | parc moyen |
| 1000 | 1000 | parc large |

> **Pourquoi pas d'amortissement sublinéaire ?** Sur un parc legacy multi-tenants, multi-OS, avec exceptions par site, l'amortissement orchestration est faible. Les `base_rate` sont calibrés pour un parc déjà industrialisé (IaC, OS Config, monitoring centralisé) — l'effort par ressource est déjà bas. Toute amortissement sublinéaire supplémentaire (sqrt, log) sous-estime systématiquement les missions à fort volume.

Le scaling se fait **globalement** (toutes envs confondues) au sein d'un même bucket. Le coût est ensuite distribué prorata du count par env, puis le SLA s'applique par env (cf. `service-levels.md`).

---

## Substrate layer

### Managed K8s cluster (EKS, GKE, AKS, OVH MKS)
- **Base :** 0.25 j/h/mois pour 1 cluster
- **Couvre :** coordination upgrades cluster (control plane piloté provider) ; review RBAC, network policies, ingress ; version tracking add-ons (CNI, CSI, autoscaler) ; surveillance santé cluster, capacity nodes. Pas de patching control plane (provider-managed).
- **Scaling :** rare d'avoir N>1, mais linéaire (× N) si multi-cluster.

### Self-hosted K8s cluster
- **Base :** 0.5 j/h/mois pour 1 cluster
- **Couvre :** tout ce que le managed couvre, **plus** patching control plane (apiserver, etcd, scheduler) ; patching node OS ; operations etcd (snapshots, restores) ; networking layer manuel (CNI, MetalLB...).
- **Scaling :** linéaire (× N).

### Public cloud managed VM (EC2, GCE, Azure VM)
- **Base :** 0.1 j/h/mois pour 1 VM
- **Couvre :** patching OS mensuel (sécurité + minor) ; reboots planifiés ; monitoring système (CPU, mem, disk, réseau) ; surveillance certificats TLS ; vérif backup VM ; logs review hebdo.
- **Scaling :** linéaire (× N) au sein de (même OS, même coeff). Ex. 11 EC2 Debian coeff 0.8 = `0.1 × 11 × 0.8 = 0.88 j/h/mois`.

### Private cloud managed VM (VMware, OpenStack)
- **Base :** 0.1 j/h/mois pour 1 VM
- **Couvre :** identique au public, plus coordination avec l'équipe hypervisor pour les opérations.
- **Scaling :** linéaire (× N).

### Public cloud managed container service (ECS, Fargate, Cloud Run)
- **Base :** 0.05 j/h/mois pour 1 service
- **Couvre :** review service definition (CPU/mem, scaling rules) ; surveillance auto-scaling ; pas d'OS, pas de runtime à patcher.
- **Scaling :** linéaire (× N).

### Hypervisor (private cloud)
- **Base :** 0.5 j/h/mois pour 1 hypervisor
- **Couvre :** patching hypervisor, capacity planning, networking, storage backend, monitoring physique.
- **Scaling :** linéaire (× N).

---

## Application layer

### Managed off-the-shelf application (RDS, ElastiCache, MSK, Opensearch managed, etc.)
- **Base :** 0.3 j/h/mois pour 1 instance
- **Couvre :** tracking minor versions disponibles ; coordination maintenance windows ; review parameter groups ; slow query review (DB) ; vérif backup automatisé ; pas de patching infra (provider-managed).
- **Scaling :** linéaire (× N) au sein de (même engine, même coeff).

### Self-hosted off-the-shelf application (MySQL on VM, Postgres on VM, Kafka self-managed, ClickHouse self-managed, etc.)
- **Base :** 0.6 j/h/mois pour 1 instance
- **Couvre :** identique au managed, **plus** patching applicatif manuel (sécurité + version) ; configuration backup manuelle + tests restauration ; replication setup, failover, slot management ; tuning manuel (buffer pool, cache, indexing).
- **Scaling :** linéaire (× N).

> Le self-hosted **se cumule avec la ligne substrate** (la VM ou le container qui héberge). Ex. MySQL 5 on EC2 = 0.1 (VM) + 0.6 (self-hosted off-the-shelf) = 0.7 j/h. Vs RDS managed = 0.3 j/h. Différentiel honnête entre managé et self-hosted.

### Custom application (apps maintenues par le client)
- **Base :** 0.25 j/h/mois pour 1 app
- **Couvre :** surveillance déploiements (succès/rollbacks) ; alert tuning (anti-bruit) ; investigation logs/traces sur incidents applicatifs ; coordination releases avec équipe dev client.
- **Scaling :** linéaire (× N).

---

## Coefficients taille/complexité

Voir `coefficients.md`. Le coefficient s'applique sur le scaling linéaire :

```
MCO = base × N × coefficient
```

Quand un type a à la fois server-size et application-complexity, prendre le **plus haut** des deux.

## Calibration

Ces base rates sont à recalibrer périodiquement contre des clients dont le prix est connu et accepté. Si v2 sort un prix > ±15% du prix accordé pour ≥2 clients existants, **ajuster les bases**, pas la formule.
