# Qualification — ANS / EvalCarbone SIH

**Date :** 2026-05-04
**Approche :** Empirique (avec cross-check déductif)
**Solutions Architect :** Emmanuel Lilette

## Client Context

L'**Agence du Numérique en Santé (ANS)** opère **EvalCarbone SIH**, un outil de mesure de l'empreinte environnementale du parc IT des établissements de santé et médico-sociaux français (cible ~10 000 entités). En service depuis avril 2024, l'application combine un **front NextJS spécifique** et le **moteur de calcul open source NumEcoEval** (porté par le Ministère de la Transition Écologique).

Le SI est hébergé sur la **Plateforme Cloud (PFC) ANS**, un cluster Kubernetes managé par Theodo dans une **prestation distincte** (bascule effective mars 2026). La présente prestation couvre uniquement les **9 conteneurs EvalCarbone SIH** (front + NumEcoEval) déployés sur ce cluster — d'où le qualificatif d'infogérance "light". Les conteneurs sont déployés via **Helm chart simple + ArgoCD**.

L'authentification utilisateurs s'appuie sur le **SI tiers Plage (opéré par l'ATIH)**. Aucune donnée personnelle n'est traitée. La sécurité est qualifiée au niveau **faible** (pas de HDS, pas de RIE, pas de SOC/SIEM). Le code est public sur [github.com/ansforge/Eval-Carbone-SIH](https://github.com/ansforge/Eval-Carbone-SIH).

**Pourquoi ANS a besoin de l'infogérance** : l'EdB (1.3) confirme qu'aucune infogérance n'est en place actuellement. La plateforme tourne en best-effort sans FTE dédié, ce qui s'est traduit par un incident critique non anticipé (saturation disque) sur les 6 derniers mois.

### Deductive Data (always present)
- **Dev team size :** 0 (en sommeil, pas de release prévue)
- **Evolution backlog :** Aucune. Seules les mises à jour images NumEcoEval (~1 majeure / an, mineures non systématiques) sont prévues, et elles sont intégrées au scope d'infogérance.

### Empirical Data
- **IaC access :** Partiel. Helm chart + application ArgoCD existent (structure simple connue), mais Theodo n'a pas encore d'accès direct au repo de déploiement → à confirmer en début de prestation.
- **Ticket history (6 derniers mois, source EdB) :**
  - Total tickets : ~3 événements significatifs
  - Breakdown :
    - Incidents : **1** (critique, saturation disque)
    - Service requests : 0
    - Changements : **1** (MEP — bascule prod sur PFC Cloud + création preprod, mars 2026)
    - Hotfix : **1** (correction liée à la saturation disque)
    - Évolutions : 0
    - Incidents sécurité : 0
  - Temps de résolution moyen : non renseigné dans l'EdB
  - Patterns récurrents : la saturation disque est le seul pattern identifié (1 occurrence ayant entraîné l'incident + le hotfix)
- **Current FTEs :** 0 FTE (mode best-effort, pas de contrat formel)
  - MCO/run : 0 FTE
  - Évolutions/build : 0 FTE
  - Gouvernance : 0 FTE
- **Empirical estimate (derived from FTEs) :**
  - MCO : ~0 j/h/mois (état actuel — non viable, gap reconnu par l'ANS)
  - Gouvernance : ~0 j/h/mois
  - Évolutions : ~0 j/h/mois
  - **Total empirique : ~0 j/h/mois — la baseline empirique confirme l'absence de couverture mais n'est pas exploitable comme cible. L'estimation est portée par la voie déductive.**
- **Known inefficiencies/gaps :**
  - Aucune supervision proactive : l'incident de saturation disque a été détecté tardivement
  - Pas de patch management automatisé (scan + patch semestriels uniquement)
  - Pas de runbook formalisé
  - Pas de point de gouvernance régulier
  - Documentation à jour sur GitHub mais pas de RACI opérationnel

## Resource Inventory

### Environment: Production
**Plage horaire :** Standard (Lundi-Vendredi 09h30-18h30, conforme JO/HO de l'EdB 2.2)
**SLA :** Bronze (conforme EdB 2.3 = 97% dispo, MT slide 12)

| Resource | Type | Cloud | Size | Complexity |
|----------|------|-------|------|------------|
| Frontend NextJS (EvalCarbone) | Custom application | Public (PFC ANS K8s managé) | <1 RPS, faible CPU/RAM | Low |
| Reverse proxy NGINX | Off-the-shelf application | Public (PFC ANS K8s managé) | <1 RPS | Very low |
| PostgreSQL (NumEcoEval) | Off-the-shelf application | Public (PFC ANS K8s managé) | 23 tables, <10 GB | High (DB) |
| Kafka (NumEcoEval) | Off-the-shelf application | Public (PFC ANS K8s managé) | Faible volume | Very high (kafka) |
| Zookeeper (NumEcoEval) | Off-the-shelf application | Public (PFC ANS K8s managé) | Faible volume | Low |
| api-rest-expositiondonneesentrees (Java) | Off-the-shelf application | Public (PFC ANS K8s managé) | <1 RPS | Low |
| api-rest-referentiels (Java) | Off-the-shelf application | Public (PFC ANS K8s managé) | <1 RPS | Low |
| api-event-donneesentrees (Java) | Off-the-shelf application | Public (PFC ANS K8s managé) | <1 RPS | Low |
| api-event-calculs (Java) | Off-the-shelf application | Public (PFC ANS K8s managé) | <1 RPS | Low |

### Environment: Préproduction / dev
**Plage horaire :** Standard
**SLA :** Bronze

Identique à la production : 9 conteneurs (mêmes images, même topologie, dimensionnement plus faible).

| Resource | Type | Cloud | Size | Complexity |
|----------|------|-------|------|------------|
| Frontend NextJS (EvalCarbone) | Custom application | Public (PFC ANS K8s managé) | <1 RPS | Low |
| Reverse proxy NGINX | Off-the-shelf application | Public (PFC ANS K8s managé) | <1 RPS | Very low |
| PostgreSQL (NumEcoEval) | Off-the-shelf application | Public (PFC ANS K8s managé) | <10 GB | High (DB) |
| Kafka (NumEcoEval) | Off-the-shelf application | Public (PFC ANS K8s managé) | Faible volume | Very high (kafka) |
| Zookeeper (NumEcoEval) | Off-the-shelf application | Public (PFC ANS K8s managé) | Faible volume | Low |
| api-rest-expositiondonneesentrees (Java) | Off-the-shelf application | Public (PFC ANS K8s managé) | <1 RPS | Low |
| api-rest-referentiels (Java) | Off-the-shelf application | Public (PFC ANS K8s managé) | <1 RPS | Low |
| api-event-donneesentrees (Java) | Off-the-shelf application | Public (PFC ANS K8s managé) | <1 RPS | Low |
| api-event-calculs (Java) | Off-the-shelf application | Public (PFC ANS K8s managé) | <1 RPS | Low |

> **Note scope** : le cluster K8s PFC ANS lui-même n'apparaît pas dans l'inventaire — il est managé par Theodo dans une **prestation distincte**. Cette prestation-ci ne gère que les 9 applications conteneurisées EvalCarbone SIH par environnement.

## Service Commitments Summary

| Environment | Plage horaire | SLA | SLA Coeff |
|------------|---------------|-----|-----------|
| Production | Standard | Bronze | 1.00 |
| Préproduction / dev | Standard | Bronze | 1.00 |

**Groupes de plages horaires :** 1 seul (Standard pour les 2 envs) — bien en deçà du maximum de 2.

## Phase d'initialisation (one-shot)

**Plateforme construite par Theodo :** Non

> Monitoring et système d'agents IA sont par défaut **toujours** présents. Audit et remédiation prioritaire sont **conditionnés** à "Plateforme construite par Theodo = Non". Dans cette prestation, le client a explicitement écarté la remédiation et le système d'agents IA (engagement court 3 mois renouvelable, scope volontairement allégé).

| Composante | Sizing retenu | Effort retenu (j/h) |
|------------|---------------|---------------------|
| Audit | Small | 2.5 |
| Remédiation prioritaire (cible ROSE/YAMAS) | Opt-out client | 0 |
| Mise en place du monitoring | Simple | 2.5 |
| Mise en place du système d'agents IA | Opt-out client | 0 |

**Total init retenu : 5 j/h.**

> La remédiation est par nature dépendante des findings de l'audit ; le client a fait le choix d'un opt-out à 0 j/h pour cette première période de 3 mois. Si l'audit révèle des gaps significatifs, une remédiation pourra être proposée en amendement ou au renouvellement.

## Informations manquantes

| Information manquante | Impact sur l'estimation | Comment l'obtenir |
|-----------------------|------------------------|-------------------|
| Accès direct au repo Helm/ArgoCD de déploiement EvalCarbone | Faible — la structure est confirmée simple par l'utilisateur, mais l'audit Small (2.5 j/h) suppose un accès rapide au code de déploiement. Si l'accès est tardif, l'audit peut glisser de quelques heures. | Demander à l'ANS l'accès au repo de déploiement dès la signature du bon de commande. |
| Historique post-bascule PFC limité (2 mois seulement, mars→mai 2026) | Moyen — l'historique 6 mois utilisé pour l'empirique inclut surtout du pré-bascule (infra OVH). Le comportement post-bascule peut différer (volumétrie, modes de défaillance). | Faire un point de revue à 3 mois (fin de la première période) avec les indicateurs réels post-bascule pour ajuster le renouvellement. |
| Temps de résolution moyen des tickets sur 6 mois | Faible — l'EdB ne renseigne pas ce KPI. Pour 3 événements seulement, ce n'est pas critique pour l'estimation. | À tracer via le futur ITSM dès le démarrage de la prestation. |
| Magnitude exacte de la remédiation post-audit | Moyen — opt-out à 0 j/h dans cette qualification (choix client). Si l'audit Small remonte des gaps significatifs (ex. saturation disque non monitorée, pas de backup automatisé), une remédiation devra être proposée hors-scope. | Livrer le rapport d'audit en fin de phase init et discuter d'un éventuel amendement. |
| Périmètre exact "système d'agents IA" pour cette prestation | Faible — opt-out à 0 j/h dans cette qualification. Pas de ChatOps ni d'intégrations IA prévues. | À reconsidérer au renouvellement (3 mois) si la prestation est étendue. |

## Constraints & Notes

### Compliance & sécurité
- **HDS NON requis** (EdB 1.5)
- **SecNumCloud NON requis** (EdB 1.6)
- Pas de **données personnelles** traitées (EdB 1.4 — auth via Plage/ATIH avec credentials utilisateurs uniquement)
- Pas de **raccordement RIE** (EdB 4.11)
- Pas de **SOC/SIEM** (EdB 7.5)
- Niveau de sécurité **FAIBLE** (EdB 7.1)
- Scan sécurité **semestriel** + patch management **semestriel** (EdB 7.3 / 7.4)

### Engagement
- **3 mois renouvelables** — engagement court. Pas de remise multi-annuelle applicable.
- Forme contractuelle : marché UGAP PEC Lot 2 (Cloud) — Marché 415584
- Date de qualification : 03/04/2026
- Date de proposition : 04/05/2026

### Périmètre
- **TMA EXCLUE** de cette prestation — fera l'objet d'un mémoire technique et d'une prestation distincts (cf. MT slide 13).
- **Cluster K8s PFC ANS hors scope** — managé par Theodo dans une autre prestation.
- Périmètre couvert : **infogérance light des 9 conteneurs EvalCarbone SIH** (prod + préprod) + mise à jour des images NumEcoEval (~1 majeure / an).

### SLA EdB (production)
- Plage de fonctionnement : 24/7
- Plage d'intervention : JO/HO (09h30-18h30)
- Disponibilité mensuelle : 97%
- RTO : 4h JO/HO
- RPO : 24h
- PRA : 72h JO
- Hotfix critique : 8h JO/HO
- DMIA cumulée : 8h / mois

### Comitologie (depuis EdB section 9)
- COPIL : trimestriel (1h30)
- COPROD : mensuel (30 min)
- COSEC : à la demande
- Comités ad hoc : à la demande

### Dispositif d'équipe (estimation préliminaire)
Avec 9 ressources × 2 envs et un coefficient agrégé moyen, on s'attend à une enveloppe MCO autour de **10–15 j/h/mois**, ce qui place la prestation dans le **dispositif Semi-dédié** (10–100 j/h/mois) — cohérent avec le statut de l'ANS dans les références internes Theodo. À confirmer par `/estimate`.

### Autres notes
- Documentation technique tenue à jour sur [github.com/ansforge/Eval-Carbone-SIH](https://github.com/ansforge/Eval-Carbone-SIH)
- Pas de licences payantes (certificats fournis par l'ANS)
- CI/CD existant : GitHub push + GitLab ANS
