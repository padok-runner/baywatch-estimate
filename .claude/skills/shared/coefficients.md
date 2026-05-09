# Size & Complexity Coefficients

## Coefficient table

| Coefficient | Server size                      | Application complexity               |
| ----------- | -------------------------------- | ------------------------------------ |
| 0.5         | <1 RPS or <16 CPU or <10 GB      | very low, e.g. ingress controller    |
| 0.8         | <10 RPS or <32 CPU or <50 GB     | low, e.g. argocd                     |
| 1           | <100 RPS or <64 CPU or <100 GB   | medium, e.g. vault                   |
| 2           | <1000 RPS or <128 CPU or <500 GB | high, e.g. Mysql Database            |
| 5           | 1000 RPS or >128 CPU or >500 GB  | very high, e.g. elasticsearch, kafka |

Use the **higher** of the two assessments (server size vs application complexity) when they differ.

## How to apply

Combined with the sublinear scaling from `item-types.md` :

```
MCO_for_N_resources_of_same_(type, coeff) = base_rate × multiplier(N) × coefficient

where multiplier(N) = min(N, 3) + log10(max(N/3, 1))
```

Group ressources by **(item type, coefficient)** before applying the formula. The scaling captures the marginal cost of identical-profile ressources at scale.

## Examples

- A small argocd instance on public cloud K8s:
  - Item: Managed off-the-shelf application → base 0.3 j/h/mois
  - Complexity: low (argocd) → coefficient 0.8
  - Count: 1 → multiplier 1.0
  - MCO = 0.3 × 1.0 × 0.8 = **0.24 j/h/mois**

- A large Kafka cluster (single instance) on private cloud:
  - Item: Self-hosted off-the-shelf application → base 0.6 j/h/mois
  - Complexity: very high (kafka, >500 GB) → coefficient 5
  - Count: 1 → multiplier 1.0
  - MCO = 0.6 × 1.0 × 5 = **3.0 j/h/mois**

- 11 EC2 Debian medium across 3 envs:
  - Item: Public cloud managed VM → base 0.1 j/h/mois
  - Coefficient: 0.8 (medium server <50 GB)
  - Count: 11 → multiplier = min(11,3) + log10(11/3) = 3 + 0.564 = 3.564
  - MCO base = 0.1 × 3.564 × 0.8 = **0.285 j/h/mois**
  - Then distributed prorata count per env, with SLA applied per env (Gold ×1.10 on prod portion, Bronze ×1.0 elsewhere).
