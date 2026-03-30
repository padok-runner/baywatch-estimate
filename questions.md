# Guide Commercial — Qualification Infogérance Cloud

Informations à collecter auprès du client pour le skill `/qualify`.

---

## 1. Contexte et conformité

- Nom du client
- Solutions Architect assigné
- Activité du client (secteur, produit/service)
- Motivation pour l'infogérance (manque de ressources, conformité, montée en charge...)
- **Conformité / certifications** : HDS, SecNumCloud, ISO 27001, RGPD... _(à clarifier en priorité — peut modifier le périmètre)_
- Infra en production depuis plus de 6 mois ? → détermine l'approche empirique ou déductive

---

## 2. Équipe

- Taille de l'équipe dev/IT (développeurs, PO, CTO, ops...)
- Mode de déploiement actuel (CI/CD automatisé, manuel, outils utilisés)

---

## 3. Inventaire des ressources _(obligatoire)_

Pour chaque ressource, collecter :

| Champ | Exemples |
|---|---|
| Nom | "EC2 Prod #1", "RDS MySQL principale" |
| Type | VM, cluster K8s, hyperviseur, BDD, app custom, app tierce |
| Cloud | Public (AWS, GCP, Azure) ou privé |
| Taille | CPU, RAM, volume de données (Go), RPS, noeuds... |
| Complexité | very low / low / medium / high / very high |
| Environnement | prod, recette, dev, shared, landing zone |

Vérifier que tous les environnements sont couverts (prod, non-prod, services partagés, landing zone).

Ne pas oublier : instances de calcul, bases de données, applications hébergées, infra partagée (bastion, CI/CD, monitoring).

---

## 4. Niveaux de service

Par environnement. Défauts : non-prod = Bronze/Standard, prod = Gold.
Max 2 groupes de plages horaires.

| Plage horaire | Horaires | Usage typique |
|---|---|---|
| Standard | Lun-Ven, 09h30-18h30 | App interne France |
| Étendue | Lun-Sam, 08h-22h | B2B multi-fuseaux |
| Complète | 24/7 jours fériés inclus | B2C critique, santé |

| SLA | Usage typique |
|---|---|
| Bronze | Non-prod, non critique |
| Silver | App interne, impact modéré |
| Gold | Client-facing, impact business |
| Platine | Infrastructure vitale |

---

## 5. Données empiriques _(si infra +6 mois)_

### Tickets (6-12 derniers mois)

- Nombre total
- Répartition : incidents, demandes de service, changements, problèmes
- Temps moyen de résolution
- Patterns récurrents

### Effectifs (FTE)

- Nombre de personnes sur la plateforme
- Répartition : MCO/run, évolutions/build, gouvernance

### Accès IaC

- Outil utilisé (Terraform, Pulumi, CloudFormation...)
- Accès au repo possible ?

---

## 6. Évolutions prévues

- Backlog à 6 mois (migrations, changements d'architecture, nouveaux services...)
- Roadmap validée ou prévisionnelle ?

---

## 7. Contraintes complémentaires

- Résidence de données / localisation géographique
- SLAs contractuels existants envers les clients du client
- Durée d'engagement souhaitée (1 an, 2 ans, 3+)

---

## Informations manquantes

Noter ce que le client ne peut pas fournir : ce qui manque, l'impact sur l'estimation, comment l'obtenir plus tard. Le skill `/qualify` génère une section dédiée.

