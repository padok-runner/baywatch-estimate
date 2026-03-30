# Qualification — Carenity

**Date:** 2026-03-30
**Approach:** Both (Empirical + Deductive)
**Solutions Architect:** —

## Client Context

Carenity est une plateforme santé avec une stack LAMP (PHP CodeIgniter / PHP Slim + VueJS) hébergée sur AWS. L'équipe dev/IT est réduite (2 développeurs seniors, 1 PO, 1 CTO) et l'infrastructure est stable avec très peu d'incidents. Le client recherche une infogérance cloud avec exigences HDS, couvrant la production en 24/7 avec SLA Gold.

### Deductive Data
- **Dev team size:** 4 (2 développeurs seniors, 1 PO, 1 CTO)
- **Evolution backlog (non validé — roadmap interne en attente):**
  - Migration RDS MySQL 8.0 → 8.4 (recette et prod)
  - Migration Elasticache Redis 6.2 → 8.4 (recette et prod)
  - Migration OS Debian 11 → 13 (instances EC2 recette et prod)

### Empirical Data
- **IaC access:** Non (Terraform + GitHub Actions utilisés par le client, mais pas d'accès pour nous)
- **Ticket history (12 derniers mois):**
  - Total tickets: ~5 (volume très faible)
  - Incidents: 1 (indisponibilité PHP-FPM suite à pic de charge)
  - Service requests: 0
  - Changes/évolutions: quelques tickets (migration Opensearch, changement profil IAM pour Terraform)
  - Problèmes récurrents: 0
  - Average resolution time: non communiqué
  - Recurring patterns: aucun pattern récurrent identifié
- **Current FTEs:** Non communiqué (répartition MCO/build/gouvernance inconnue)
- **Empirical estimate:** Non dérivable — les données FTE ne sont pas disponibles
- **Known inefficiencies/gaps:** Déploiements de code lancés manuellement via EC2 dédiée (procédures automatisées mais déclenchement manuel)

## Resource Inventory

### Environment: Production
**Plage horaire:** Complète (24/7)
**SLA:** Gold (coefficient 1.10)

| Resource | Type | Cloud | Size | Complexity | Coeff |
|----------|------|-------|------|------------|-------|
| EC2 Prod #1 | Public Cloud Managed Virtual Machine | Public | Moyenne (<32 CPU, <50 GB) | medium | 0.8 |
| EC2 Prod #2 | Public Cloud Managed Virtual Machine | Public | Moyenne (<32 CPU, <50 GB) | medium | 0.8 |
| EC2 Prod #3 | Public Cloud Managed Virtual Machine | Public | Moyenne (<32 CPU, <50 GB) | medium | 0.8 |
| EC2 Prod #4 | Public Cloud Managed Virtual Machine | Public | Moyenne (<32 CPU, <50 GB) | medium | 0.8 |
| EC2 Prod #5 | Public Cloud Managed Virtual Machine | Public | Moyenne (<32 CPU, <50 GB) | medium | 0.8 |
| RDS MySQL 8 #1 (27 Go) | Off-the-shelf application | Public | 60 Go total, AAS 1-4 | medium | 1 |
| RDS MySQL 8 #2 | Off-the-shelf application | Public | 60 Go total, AAS 1-4 | medium | 1 |
| Opensearch | Off-the-shelf application | Public | <100 Go | medium | 1 |
| Elasticache Redis | Off-the-shelf application | Public | Petit cache | low | 0.8 |
| MySQL 5 self-hosted | Off-the-shelf application | Public | Legacy, faible volume | medium | 1 |
| MPA Legacy PHP5/MySQL5 | Custom application | Public | — | medium | 1 |
| MPA CodeIgniter #2 | Custom application | Public | — | low | 0.8 |
| MPA CodeIgniter #3 | Custom application | Public | — | low | 0.8 |
| MPA CodeIgniter #4 | Custom application | Public | — | low | 0.8 |
| SPA VueJS/Slim #1 | Custom application | Public | — | low | 0.8 |
| SPA VueJS/Slim #2 | Custom application | Public | — | low | 0.8 |
| App Infra - Jobqueues | Custom application | Public | — | very low | 0.5 |
| App Infra - SSO/Auth | Custom application | Public | — | very low | 0.5 |

### Environment: Recette (non-prod)
**Plage horaire:** Standard (Lundi-Vendredi, 09h30-18h30)
**SLA:** Bronze (coefficient 1)

| Resource | Type | Cloud | Size | Complexity | Coeff |
|----------|------|-------|------|------------|-------|
| EC2 Recette #1 | Public Cloud Managed Virtual Machine | Public | Moyenne | medium | 0.8 |
| EC2 Recette #2 | Public Cloud Managed Virtual Machine | Public | Moyenne | medium | 0.8 |
| EC2 Recette #3 | Public Cloud Managed Virtual Machine | Public | Moyenne | medium | 0.8 |
| RDS MySQL 8 Recette | Off-the-shelf application | Public | <15 Go | low | 0.8 |
| Opensearch Recette | Off-the-shelf application | Public | Petit | low | 0.8 |
| Elasticache Redis Recette | Off-the-shelf application | Public | Petit | low | 0.8 |
| MySQL 5 self-hosted Recette | Off-the-shelf application | Public | Legacy | low | 0.8 |

### Environment: Shared / Infra
**Plage horaire:** Standard (Lundi-Vendredi, 09h30-18h30)
**SLA:** Bronze (coefficient 1)

| Resource | Type | Cloud | Size | Complexity | Coeff |
|----------|------|-------|------|------------|-------|
| EC2 Bastion | Public Cloud Managed Virtual Machine | Public | Petite | low | 0.8 |
| EC2 Packages/Librairies | Public Cloud Managed Virtual Machine | Public | Petite | low | 0.8 |
| EC2 Déploiement | Public Cloud Managed Virtual Machine | Public | Petite | low | 0.8 |

## Service Commitments Summary

| Environment | Plage horaire | SLA | SLA Coeff |
|------------|---------------|-----|-----------|
| Production | Complète (24/7) | Gold | 1.10 |
| Recette (non-prod) | Standard | Bronze | 1 |
| Shared / Infra | Standard | Bronze | 1 |

## Informations manquantes

| Information manquante | Impact sur l'estimation | Comment l'obtenir |
|-----------------------|------------------------|-------------------|
| Mapping EC2 → applications en prod | Impossible de savoir si certaines EC2 portent plusieurs apps ou si certaines sont sous-utilisées — peut affecter le sizing des VMs | Demander le détail au client ou accéder à la console AWS |
| Répartition FTE MCO/build/gouvernance | Pas d'estimation empirique possible — on se base uniquement sur l'approche déductive | Demander au client la répartition du temps de l'équipe |
| Taille exacte des instances EC2 | Coefficient de sizing approximatif (hypothèse moyenne) — peut varier de ±30% | Accéder à la console AWS ou demander les types d'instances |
| Volume de données Opensearch | Complexité classée medium par défaut — pourrait être higher si gros volumes | Demander les métriques au client |
| Temps moyen de résolution des incidents | Pas de baseline pour évaluer l'efficacité de la réponse actuelle | Demander au client ou accéder à leur outil de ticketing |
| Accès IaC (Terraform) | Impossible de vérifier la configuration exacte et d'évaluer la dette technique infra | Demander l'accès au repo GitHub |
| Roadmap évolutions 2026 non validée | Les 3 migrations (MySQL, Redis, Debian) ne sont pas confirmées — impact significatif sur le volume d'évolutions | Attendre la validation de la roadmap interne du client |

## Constraints & Notes

- **HDS requis** — Audit YAMAS applicable. Le périmètre HDS exact est à discuter (MySQL5 + app legacy pourraient être exclus).
- **Engagement : 1 an** — Pas de remise multi-annuelle.
- **Déploiements manuels** — Les déploiements de code sont déclenchés manuellement via une EC2 dédiée avec procédures automatisées. L'infra (Terraform) est déployée automatiquement via GitHub Actions.
- **Méthodologie SCRUM** — L'équipe dev travaille en agile.
- **Application legacy PHP5/MySQL5** — À décommissionner à terme (pas de deadline). Sera potentiellement exclue du périmètre HDS.
- **Volume d'incidents très faible** — 1 seul incident en 12 mois, infrastructure stable.
