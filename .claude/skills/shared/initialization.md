# Phase d'initialisation (one-shot)

L'initialisation est une **enveloppe payée une seule fois** en début d'engagement. Elle prépare la plateforme pour les services managés et est **indépendante du prix mensuel récurrent**.

## Composantes

| Composante | Périmètre | Quand l'inclure | Rôle facturé |
|------------|-----------|-----------------|--------------|
| **Audit** | Cartographie ressources, qualité code/IaC, résilience, sécurité, observabilité | **Uniquement si plateforme NON construite par Theodo** | Lead Ops |
| **Remédiation prioritaire** | Docs manquantes, durcissement résilience, gaps cibles ROSE/YAMAS | **Uniquement si plateforme NON construite par Theodo** | Mix Lead Ops + Ops (TJM blended) |
| **Mise en place du monitoring** | Métriques, alerting, dashboards, sondes, runbooks | **Toujours** | Mix Lead Ops + Ops (TJM blended) |
| **Mise en place du système d'agents IA** | Déploiement agent ChatOps, intégrations métier, MCP custom | **Toujours** | Mix Lead Ops + Ops (TJM blended) |

## Abaques de sizing

### Audit (Lead Ops)

| Sizing | Critère | Effort |
|--------|---------|--------|
| Small | ≤10 ressources, infra simple | ~2.5 j/h |
| Medium | 10–50 ressources, complexité modérée | ~7–10 j/h |
| Large | 50+ ressources, multi-cluster K8s, microservices | ~15–20 j/h |

### Remédiation prioritaire

| Sizing | Critère | Effort |
|--------|---------|--------|
| Light | Docs partielles, quelques gaps résilience | ~5 j/h |
| Medium | Refonte docs, durcissement résilience significatif | ~15 j/h |
| Heavy | Chantier majeur pour atteindre ROSE/YAMAS | ~30+ j/h |

> Le sizing exact dépend des findings de l'audit ; capturer une hypothèse de cadrage à la qualification. Défaut si inconnu : Medium.

### Mise en place du monitoring

| Sizing | Critère | Effort |
|--------|---------|--------|
| Simple | Stack standard, alerting basique | ~2.5 j/h |
| Medium | Multi-env, dashboards custom, intégrations | ~7–10 j/h |
| Complex | Multi-cloud, SLO/SLI custom, observabilité avancée | ~15–20 j/h |

### Mise en place du système d'agents IA

| Sizing | Critère | Effort |
|--------|---------|--------|
| Simple | Agent ChatOps standard sur la plateforme | ~2.5 j/h |
| Medium | Workflows custom, intégrations métier | ~7–10 j/h |
| Complex | Multi-agents, MCP custom, intégration profonde | ~15–20 j/h |

## Règle de calcul du prix

```
Audit price          = audit_jh × TJM(Lead Ops)        # Lead Ops uniquement
Remediation price    = remediation_jh × blended_TJM
Monitoring price     = monitoring_jh × blended_TJM
AI agent price       = ai_agent_jh × blended_TJM

Total initialisation (j/h) = somme des composantes incluses
Total initialisation (€HT) = somme des prix ci-dessus
```

TJM Lead Ops et TJM blended : voir `daily-rates.md`.

## Règle d'affichage

- Bloc **"Phase d'initialisation (one-shot)"** placé **au-dessus** du tableau de synthèse mensuelle dans `estimate.md`.
- Si la plateforme a été construite par Theodo : audit et remédiation **omis** (lignes non affichées, prix = 0).
- Le total one-shot ne doit **jamais** être additionné au prix mensuel récurrent ni intégré dans la contingence forfait.
