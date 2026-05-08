# Estimate — RDG (Rail Delivery Group) — V1/V2 Redirect Proxy

**Date:** 2026-05-08 v5 (item type Container ECS/Fargate)
**Based on:** [qualification.md](qualification.md) (rev. 2026-05-08)
**TJM blended:** 863€ (Ops 750€ × 1.00 + Lead Ops 1 200€ × 0.34 + DM 850€ × 0.16, blended)
**TJM Lead Ops (audit):** 1 200€
**Dispositif:** Mutualisé (total < 10 j/h/mois)
**Engagement:** Forfait sur MCO + Gouvernance, contingence **0%**, durée **1 an** (pas de remise multi-annuelle)
**Évolutions:** **Hors périmètre** — décision SA (les modifications de routage sont des éditions du JSON S3 par RDG)

---

## Hypothèses de travail

| #  | Hypothèse | Information manquante | Valeur retenue | Justification | Impact si l'hypothèse est fausse |
|----|-----------|-----------------------|----------------|---------------|----------------------------------|
| ~~H1~~ | ~~Pic de trafic prod > 1000 RPS~~ | **RÉSOLU 2026-05-06** | Trafic prod réel ~300–800 RPS | Données Omar : 3 workers prod, CPU peaks <50%, scaling threshold à 800 TPS jamais atteint. | — (résolu) |
| ~~H2~~ | ~~Trafic spiky / steep ramp-up géré sans dégradation~~ | **RÉSOLU 2026-05-07** | Pas de buffer MCO supplémentaire | Auto-scale test (`Proxy Auto Scale Test.pdf`) : 1 367 RPS sustained, 0 erreurs, P95 336ms, scale-out 2→4 tâches validé. Headroom ~60% au-dessus du pic prod attendu. | — (résolu) |
| **H7** | **SA override du coefficient size prod : 2 → 1 (2026-05-08)** | Décision SA suite aux données de robustesse opérationnelle | **Coeff 1** appliqué à prod (vs coeff 2 strict du framework pour <1000 RPS) | Cluster physique très petit (<1 vCPU consommé au pic), auto-scaling validé sans erreur, très basse complexité, headroom ~60%. Le SA juge que l'empreinte opérationnelle réelle est en dessous du tier <1000 RPS standard. | Si l'override est contesté : retour à coeff 2 → +0.72 j/h MCO/mois → +621€/mois. **Documenté comme déviation explicite pour le verifier.** |
| H3 | TJM blended Theodo = 863€ | Le repère SA est en £ (~£2 000/mois) avec un TJM inconnu | Grille standard `shared/daily-rates.md` | Convention interne | Aucun (taux interne) ; la comparaison à l'ancrage £2k reste sensible au taux de change |
| H4 | Aucune évolution mensuelle | Confirmation explicite SA | 0 j/h/mois | Décision client : modifications de routage gérées par RDG en autonomie | Si évolutions apparaissent → ligne Temps passé ajoutée au contrat |
| H5 | Composantes init AI agents et Remédiation hors scope | Décision explicite de qualification | 0 j/h, omises de l'init | AI agents déjà fournis par le contrat RDG existant ; documentation du proxy déjà très fournie | Si re-scoping : +2.5 j/h AI agents (≈2 158€) +5 j/h remédiation (≈4 315€) = +6 473€ one-shot |
| **H6** | **Pas de rate limiting devant le proxy** (nouveau risque, 2026-05-06) | Décision architecturale RDG | MCO réactif standard ; pas de buffer dédié | Sol a flaggé le risque ; Omar confirme que les backends sont rate-limités mais pas le proxy lui-même | Si un caller déraisonnable inonde le proxy → incidents répétés, +1–2 j/h MCO/mois (+863–1 726€/mois). À adresser avec RDG avant go-live. |

> **⚠ Sensibilité résiduelle** :
> - **H7 (override coeff prod 2 → 1)** — sensibilité **+621€/mois** si l'override est invalidé (retour au coeff 2 strict framework). Risque méthodologique principal.
> - **H6 (rate-limiting absent devant le proxy)** — sensibilité +863–1 726€/mois en pire cas si non adressé par RDG avant go-live. Risque architectural côté client.
> - **À monitorer dans les 3 premiers mois post go-live.** Si MCO réelle dépasse 2.0 j/h/mois → revisiter H7 avec le client.

---

## Analyse de réalisme

### Signaux empiriques disponibles

| Signal | Valeur | Source |
|--------|--------|--------|
| Tickets/incidents mensuels | Inconnu (service pas encore en prod) | — |
| Taille équipe dev cliente | Aucune dédiée post-handover | qualification.md |
| Évolution backlog | Modifications config routage S3 (non facturées en MCO) | qualification.md |
| Stabilité infra documentée | Très bonne — runbooks, load tests, plans rollback en place | PDF du call |
| Charge actuelle prod (V1) | <50% CPU, <800 TPS, 3 workers absorbent le pic | Omar 2026-05-06 |
| Capacité validée (auto-scale test) | **1 367 RPS sustained, 0 erreurs, P95 336ms, scale-out 2→4 tasks OK** | Omar 2026-05-07 (`Proxy Auto Scale Test.pdf`) |
| Repère contractuel similaire | ~£2 000/mois + 2 j init pour add-ons RDG comparables | SA |

### Calibration empirique

| Composante | j/h/mois | Raisonnement |
|------------|----------|--------------|
| MCO réactif | 0.8 | 1–2 tickets/mois × 0.5 j/h, ajusté légèrement à la baisse vu la robustesse validée. H6 reste en sensibilité. |
| MCO proactif | 1.2 | 1.5× le réactif. Infra simple, auto-scaling self-healing, runbooks en place. |
| Buffer SLA Platine | 1.0 | Inchangé — Platine impose une disponibilité de réaction renforcée, mais largement absorbée par la résilience démontrée. |
| Gouvernance | 0.33 | Selon abaque mutualisé — inchangé |
| **Total calibré** | **3.33** | |

### Comparaison

| Approche | j/h/mois | Services (€) | Immo (€) | Total mensuel | Écart vs override |
|----------|----------|--------------|----------|---------------|-------------------|
| Déductive avec override H7 (coeff prod 2→1) | 2.013 | 1 737€ | 1 000€ | **2 737€** | — |
| Déductive *strict framework* (Container, coeff 2) | 2.733 | 2 358€ | 1 000€ | 3 358€ | +23% |
| Calibrée | 3.333 | 2 876€ | 1 000€ | 3 876€ | +42% |
| **Repère SA (£2k/mois)** | ~2.7 | ~2 330€ | n/a | ~2 330€ | -15% |

**Lecture :** Avec l'item type Container (correction méthodologique 2026-05-08) et l'override H7 (coeff prod 2→1) :
- Le prix retenu **2 737€/mois** est **+18%** au-dessus du repère £2k — cohérent.
- L'écart calibrée → déductive override est de **+42%** — au-dessus du seuil 20% mais explicable par le buffer SLA Platine et la décorrélation entre coeff de sizing et signaux opérationnels.
- La déductive *strict framework* (Container, coeff 2) ressort à 3 358€/mois — c'est le **prix de repli défendable** si l'override H7 est challengé. Cet écart est de +621€/mois.

**Décision :** on retient la **déductive avec override H7 (2 737€/mois)** par décision SA. Risques associés :
- Si l'override H7 est challengé, retour au strict framework Container = 3 358€/mois (+621€/mois, +23%).
- Si la calibrée se confirme post-launch (volume incidents > 2/mois lié à H6), revenir au strict framework et négocier en avenant.

**Surveillance post go-live :**
- Mois 1–3 : tracker volume tickets, incidents, capacity events.
- Si MCO réelle dépasse 2.0 j/h/mois → revisiter H7 avec le client.

**Vs repère £2k :** désormais à **-14%** seulement (vs -46% avant la résolution de H1). Le résiduel est expliqué par Platine prod (+coeff 1.20) + Complète immobilisation (1 000€/mois) que les contrats anchor ne portent vraisemblablement pas. **Le prix est désormais cohérent avec le repère.**

---

## Synthèse

### Phase d'initialisation (one-shot)

Plateforme construite par Theodo : **Non**

> **Décision de scope (cf. qualification.md)** : Remédiation et Système d'agents IA sont **hors périmètre** de cet add-on. La remédiation est exclue car la documentation existante est déjà très fournie (load tests, runbooks, plans de rollback) ; le système d'agents IA est exclu car déjà provisionné par le contrat RDG existant. Lignes omises ci-dessous **et** dans l'Annexe B.

- **Audit** : 2.5 j/h Lead Ops — cartographie ressources, qualité IaC (CDK), résilience, sécurité, observabilité.
- **Mise en place du monitoring** : 2.5 j/h — métriques CloudWatch, dashboards, alarmes SNS, runbooks adaptés à nos équipes.
- **Support go-live (fine-tuning + on-watch)** : 1 j/h — supplément SA pour réduire le risque de la phase de déploiement (« the riskiest process »). Ligne hors abaque init mais payée one-shot avec l'init.

| Composante | Sizing | j/h | TJM | Montant |
|------------|--------|-----|-----|---------|
| Audit | Small | 2.5 | 1 200€ | 3 000€ |
| Monitoring | Simple | 2.5 | 863€ | 2 158€ |
| Support go-live | — | 1.0 | 863€ | 863€ |
| **Total initialisation** | | **6.0** | | **6 021€ HT** |

> Cette enveloppe est **payée une seule fois** en début d'engagement et **n'entre pas** dans le prix mensuel récurrent.

---

### Tableau de synthèse mensuelle

|                         | Production                                                                                                                                              | Non-Production                                                                                  | Transverse                                                                                       |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| **Nom des envs**        | prod                                                                                                                                                    | preprod (acc), dev                                                                              | —                                                                                                |
| **Inventaire**          | **Servers :**<br>1 cluster Fargate (Public Cloud Managed Container, 3 workers, <1 vCPU au pic, ~300–800 RPS)<br>**Applications :**<br>1 app NGINX/OpenResty (medium size *override SA*, low cx) | **Servers :**<br>2 clusters Fargate (Public Cloud Managed Container, small, <10 RPS)<br>**Applications :**<br>2 apps NGINX/OpenResty (low) | Bucket S3 config, Lambda admin, NLB, dashboard CloudWatch, topics SNS, Route53 — bundlés dans les apps |
| **Services**            | Maintien : Gestion des demandes, incidents, problèmes, changements de version, continuité, surveillance, **astreintes 24/7**                            | Maintien : Gestion des demandes, incidents, problèmes, changements de version, continuité      | Gouvernance : COPROD trimestriel, Audit ROSE semestriel, Audit LEAF semestriel<br>Évolutions : **hors périmètre** |
| **Niveaux de services** | Platine                                                                                                                                                 | Bronze                                                                                          | —                                                                                                |
| **Plages de service**   | Complète (7j/7, 24h/24)                                                                                                                                 | Standard (Lun-Ven 9h30–18h30)                                                                   | —                                                                                                |
| **Dispositif**          | Ops : 1.3 j/mois · Lead Ops : 0.5 j/mois · Delivery Manager : 0.2 j/mois (total ≈ 2.0 j/h/mois, **Mutualisé**)                                                                                                                                                                                                                                                                  |

#### Prix mensuel €HT

| Mode | Périmètre | j/h/mois | Montant €HT/mois |
|------|-----------|----------|-------------------|
| **Forfait** | MCO (1.68 j/h) + Gouvernance (0.33 j/h) + Contingence 0% + Immobilisation | 2.01 | **2 737€** |
| **Temps passé** | Évolutions | — (hors périmètre) | — |
| | **Total mensuel** | **2.01** | **2 737€** |

**Total annuel : 32 844€ HT** (engagement 1 an, sans remise multi-annuelle).

---

## Annexe A — Détail du calcul mensuel

### Détail MCO par environnement

#### dev — Bronze (1.00) / Standard
| Resource | Item Type | Base | Coeff (max size/cx) | MCO j/h/mois |
|----------|-----------|------|---------------------|--------------|
| Fargate cluster | Public Cloud Managed Container | 0.10 | 0.8 | 0.08 |
| NGINX/OpenResty app (+ ancillaries) | Off-the-shelf application | 0.50 | 0.8 | 0.40 |
| **Subtotal dev** | | | | **0.48** |

#### preprod (acc) — Bronze (1.00) / Standard
| Resource | Item Type | Base | Coeff | MCO j/h/mois |
|----------|-----------|------|-------|--------------|
| Fargate cluster | Public Cloud Managed Container | 0.10 | 0.8 | 0.08 |
| NGINX/OpenResty app (+ ancillaries) | Off-the-shelf application | 0.50 | 0.8 | 0.40 |
| **Subtotal preprod** | | | | **0.48** |

#### prod — Platine (1.20) / Complète
| Resource | Item Type | Base | Coeff (SA override) | MCO j/h/mois |
|----------|-----------|------|---------------------|--------------|
| Fargate cluster | Public Cloud Managed Container | 0.10 | 1 *(override de 2 → 1, cf. H7)* | 0.10 |
| NGINX/OpenResty app (+ ancillaries) | Off-the-shelf application | 0.50 | 1 *(override de 2 → 1, cf. H7)* | 0.50 |
| **Subtotal prod (avant SLA)** | | | | **0.60** |

#### Synthèse MCO

| Env | MCO base (j/h/mois) | Coeff SLA | MCO ajusté (j/h/mois) |
|-----|--------------------|-----------|----------------------|
| dev | 0.48 | 1.00 | 0.48 |
| preprod (acc) | 0.48 | 1.00 | 0.48 |
| prod | 0.60 | 1.20 | 0.72 |
| **Total MCO** | | | **1.68** |

### Gouvernance (Mutualisé, hors HDS)

| Activité | Fréquence | Effort/session | j/h/mois |
|----------|-----------|----------------|----------|
| COPROD | 1 / trimestre | 0.5 j/h | 0.167 |
| COPIL | — (dédié uniquement) | — | 0.000 |
| Audit ROSE (Qualité) | 1 / semestre | 0.5 j/h | 0.083 |
| Audit YAMAS (HDS) | — (hors HDS) | — | 0.000 |
| Audit LEAF (FinOps / Green IT) | 1 / semestre | 0.5 j/h | 0.083 |
| **Total Gouvernance** | | | **0.333** |

### Évolutions

- **Hors périmètre** sur ce contrat. Décision explicite : les ajustements du JSON de routage sont effectués par RDG en autonomie ; aucun forfait évolutions n'est budgété.
- Si des changements mineurs (Lua, NGINX) deviennent nécessaires, ils seront contractualisés en Temps passé via avenant.

### Total quantité

| Catégorie | j/h/mois |
|-----------|----------|
| MCO (ajusté SLA, avec SA override coeff prod 2→1, item Container 0.1) | 1.680 |
| Gouvernance | 0.333 |
| Évolutions | 0.000 |
| **Total** | **2.013** |

→ Dispositif : **Mutualisé** (< 10 j/h/mois) ✓

### Détail prix

| Ligne | j/h/mois | TJM | Montant |
|-------|----------|-----|---------|
| MCO (ajusté SLA, override coeff prod 2→1 + item Container 0.1) | 1.680 | 863€ | 1 450€ |
| Gouvernance | 0.333 | 863€ | 287€ |
| Évolutions | 0.000 | 863€ | 0€ |
| **Sous-total services** | 2.013 | | **1 737€** |
| Immobilisation (Mutualisé × Complète, plus haute plage) | | | 1 000€ |
| Contingence forfait (0%) | | | 0€ |
| Remise multi-annuelle (1 an) | | | 0€ |
| **Total mensuel** | | | **2 737€** |
| **Total annuel** | | | **32 844€** |

---

## Annexe B — Initialisation (one-shot)

| Composante | Sizing | j/h | TJM | Montant €HT |
|------------|--------|-----|-----|-------------|
| Audit (Lead Ops) | Small | 2.5 | 1 200€ | 3 000€ |
| Mise en place du monitoring | Simple | 2.5 | 863€ | 2 158€ |
| Support go-live (fine-tuning + on-watch) | — | 1.0 | 863€ | 863€ |
| **Total initialisation** | | **6.0** | | **6 021€** |

**Notes :**
- **Remédiation prioritaire** : ligne **omise** par décision explicite du SA (cf. qualification.md). La documentation existante (runbooks, load tests, plans rollback) est jugée suffisante pour ne pas nécessiter de remédiation pré-production. Tout findings d'audit nécessitant des actions correctives sera traité hors initialisation.
- **Système d'agents IA** : ligne **omise** par décision explicite du SA. Le système d'agents IA est déjà provisionné par le contrat RDG existant — aucun setup incrémental requis pour cet add-on.
- **Audit** facturé au TJM Lead Ops (1 200€) ; **Monitoring** et **Support go-live** facturés au TJM blended (863€).
- **Support go-live** (1 j/h) ajouté hors abaque init standard, conformément à l'engagement SA dans le call (« add a day for fine-tuning or being on watch during the go-live, deployment phase is the riskiest process »).
- Cette enveloppe est **payée une seule fois** en début d'engagement et **n'entre pas** dans le prix mensuel récurrent ni dans la base de calcul de la contingence forfait.

---

## Annexe C — Cross-check Déductive vs Empirique

Non applicable — pas de données empiriques (FTE, historique de tickets) disponibles : le service n'est pas encore en production. Le repère SA (~£2 000/mois pour des contrats similaires) est traité dans la section « Analyse de réalisme » comme un anchor de calibration, pas comme une estimation empirique structurée.

---

## Notes

- **HDS** : non applicable (rail UK, pas de données de santé).
- **Nearshore** : non sollicité par le client. Avec le prix mensuel désormais cohérent avec le repère £2k, ce levier n'est pas activé.
- **Repère £2k/mois** : cohérence atteinte (-14% écart). Confirmer avec finance le TJM des contrats RDG existants pour un alignement final.
- **Risques résiduels (post-résolution H1, H2)** :
  1. **H6** (pas de rate limit devant le proxy) — seul risque opérationnel ouvert. À remonter à RDG avant go-live ; pourrait peser sur le MCO réactif si non adressé. +1–2 j/h MCO/mois en pire cas.
  2. **H7 — SA override coeff size prod (2 → 1)** : déviation explicite du framework. Si la calibrée se confirme post go-live (3.33 j/h vs déductive override 2.013 j/h), revenir au pricing strict-framework Container (3 358€/mois). Marge d'ajustement = 621€/mois.
  3. **Lock-in Platine sur prod** — valider avec le client que la SLA Platine reflète bien la criticité business. Drop à Gold = -376€/mois.
  4. **Sub-second spike** non testé (le test du 2026-05-07 valide une rampe de 5 min, pas un saut sub-seconde). Risque considéré faible étant donné le headroom de capacité validé, mais à mentionner si RDG demande une SLA de réactivité spike-instant.
- **Référence externe** : compatible avec [qualification.md](qualification.md) section « Constraints & Notes ».
- **Historique de versions** :
  - v1 (2026-05-06 matin) — initial estimate à 6 206€/mois avec coeff 5 (hypothèse SA).
  - v2 (2026-05-06 PM) — révisé à 3 876€/mois suite aux données initiales d'Omar (coeff 2 confirmé). Gain de 2 330€/mois.
  - v3 (2026-05-07) — résolution de H2 suite au rapport `Proxy Auto Scale Test.pdf` : auto-scaling validé, capacité 1 367 RPS sustained sans erreur. Prix mensuel inchangé (3 876€/mois) ; calibration empirique resserrée (delta calibré-déductif passe de +22% à +15%). Confiance accrue dans le pricing.
  - v4 (2026-05-08) — SA override coeff size prod : 2 → 1. Justifié par la robustesse opérationnelle validée + cluster physique très petit + très basse complexité. Prix mensuel passe à 3 099€/mois (-777€/mois vs v3). Déviation framework documentée explicitement (H7) ; calibrée et déductive *strict framework* coïncident toutes deux à 3 876€/mois et restent disponibles comme re-pricing si l'override est challengé.
  - **v5 (2026-05-08) — Correction méthodologique : ajout du nouvel item type *Public Cloud Managed Container* (base 0.1 j/h) au framework `shared/item-types.md`. Le Fargate cluster, précédemment modélisé comme Public Cloud Managed K8s (0.25), est désormais correctement classé en Container — reflète la réalité opérationnelle (pas de control plane, pas de kubectl, pas de node-group lifecycle). Prix mensuel passe à 2 737€/mois (-362€/mois vs v4). Override H7 conservé. Strict-framework Container ressort à 3 358€/mois si H7 challengé.**
