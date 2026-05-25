# Daily Rates & Team Composition

## Daily Rates (TJM par rôle)

### Core run team

| Role             | TJM    |
| ---------------- | ------ |
| Ops              | 750€   |
| Lead Ops         | 1 200€ |
| Delivery Manager | 850€   |

### Specialization roles (v3, déclarés via qualification)

Pour les engagements où le profil client justifie une capacité spécialiste **dédiée** (HDS, multi-tenant régulé, K8s complexe, multi-cloud avec cadence FinOps), la qualification déclare une liste `specializations[]`. Chaque rôle ajoute une baseline en j/h/mois facturée à son **propre TJM** (pas au TJM blended).

| Role | TJM | j/h/mois défaut | Déclencheur typique |
| ---- | --- | --- | --- |
| SecOps Lead | 1 400€ | 5.0 | Périmètre HDS, données critiques, multi-tenant régulé |
| FinOps Lead | 1 200€ | 2.5 | Multi-cloud, >20 VMs, cadence LEAF lourde |
| K8s Specialist | 1 100€ | 5.0 | K8s managé avec ≥10 nodes ou multi-cluster |
| HDS Officer / Compliance Lead | 1 400€ | 2.5 | HDS + cadence audit > semestriel, multi-SELAS HDS |

Le sizing par défaut peut être surchargé en qualification avec justification. Le j/h spécialiste est **additionnel** au MCO core ; il n'est pas affecté par `tenancy_penalty` ni `year_1_ramp` (déjà dimensionné pour le profil).

> **Note clients Mutualisé.** Les spécialisations mutualisées au niveau du pool d'équipe (SecOps partagé entre 20 clients) **ne sont pas** déclarées par client — elles sont absorbées dans le TJM blended du pool. Les `specializations[]` v3 ne sont déclarées que pour les engagements qui justifient une capacité dédiée.

## Standard Team Ratio (core run)

For every **1 day** of Ops work:

| Role             | Days     | Cost       |
| ---------------- | -------- | ---------- |
| Ops              | 1.00     | 750€       |
| Lead Ops         | 0.34     | 408€       |
| Delivery Manager | 0.16     | 136€       |
| **Total**        | **1.50** | **1 294€** |

**Blended TJM** = 1 294€ / 1.50 = **863€/jour**

Le blended TJM s'applique au **MCO core**, à la **gouvernance**, aux **évolutions**, et au **monitoring/AI agent setup** (init). L'audit (init) reste facturé au TJM Lead Ops. Les spécialistes v3 ont leur propre TJM (table ci-dessus).
