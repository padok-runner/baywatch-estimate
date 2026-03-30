# Size & Complexity Coefficients

## Coefficient table

| Coefficient | Server size                      | Application complexity               |
| ----------- | -------------------------------- | ------------------------------------ |
| 0.5         | <1 RPS or <16 CPU or <10 GB      | very low, e.g. ingress controller    |
| 0.8         | <10 RPS or <32 CPU or <50 GB     | low, e.g. argocd                     |
| 1           | <100 RPS or <64 CPU or <100 GB   | medium, e.g. vault                   |
| 2           | <1000 RPS or <128 CPU or <500 GB | high, e.g. Mysql Database            |
| 5           | 1000 RPS or >128 CPU or >500 GB  | very high, e.g. elasticsearch, kafka |

## How to apply

The coefficient is multiplied against the item base rate:

```
MCO per resource = item_base_rate × coefficient
```

Use the **higher** of the two assessments (server size vs application complexity) when they differ.

## Examples

- A small argocd instance on public cloud K8s:
  - Item: Off-the-shelf application → 0.5 j/h/mois
  - Complexity: low (argocd) → coefficient 0.8
  - MCO = 0.5 × 0.8 = 0.4 j/h/mois

- A large Kafka cluster on private cloud:
  - Item: Off-the-shelf application → 0.5 j/h/mois
  - Complexity: very high (kafka, >500 GB) → coefficient 5
  - MCO = 0.5 × 5 = 2.5 j/h/mois
