# Estimate — RDG (Rail Delivery Group) — V1/V2 Redirect Proxy

**Date:** 2026-05-07 (rev. with Omar's auto-scale test report)
**Based on:** [qualification.md](qualification.md) (rev. 2026-05-07)
**TJM blended:** 863€ (Ops 750€ × 1.00 + Lead Ops 1 200€ × 0.34 + DM 850€ × 0.16, blended)
**TJM Lead Ops (audit):** 1 200€
**Dispositif:** Mutualisé (total < 10 j/h/mois)
**Engagement:** Forfait sur MCO + Gouvernance, contingence **0%**, durée **1 an** (pas de remise multi-annuelle)
**Évolutions:** **Hors périmètre** — décision SA (les modifications de routage sont des éditions du JSON S3 par RDG)

---

## Hypothèses de travail

| #  | Hypothèse | Information manquante | Valeur retenue | Justification | Impact si l'hypothèse est fausse |
|----|-----------|-----------------------|----------------|---------------|----------------------------------|
| ~~H1~~ | ~~Pic de trafic prod > 1000 RPS~~ | **RÉSOLU 2026-05-06** | Coefficient size **2** (high, <1000 RPS) | Données Omar : 3 workers prod, CPU peaks <50%, scaling threshold à 800 TPS jamais atteint. | — (résolu) |
| ~~H2~~ | ~~Trafic spiky / steep ramp-up géré sans dégradation~~ | **RÉSOLU 2026-05-07** | Pas de buffer MCO supplémentaire | Auto-scale test (`Proxy Auto Scale Test.pdf`) : 1 367 RPS sustained, 0 erreurs, P95 336ms, scale-out 2→4 tâches validé. Headroom ~60% au-dessus du pic prod attendu. | — (résolu) |
| H3 | TJM blended Theodo = 863€ | Le repère SA est en £ (~£2 000/mois) avec un TJM inconnu | Grille standard `shared/daily-rates.md` | Convention interne | Aucun (taux interne) ; la comparaison à l'ancrage £2k reste sensible au taux de change |
| H4 | Aucune évolution mensuelle | Confirmation explicite SA | 0 j/h/mois | Décision client : modifications de routage gérées par RDG en autonomie | Si évolutions apparaissent → ligne Temps passé ajoutée au contrat |
| H5 | Composantes init AI agents et Remédiation hors scope | Décision explicite de qualification | 0 j/h, omises de l'init | AI agents déjà fournis par le contrat RDG existant ; documentation du proxy déjà très fournie | Si re-scoping : +2.5 j/h AI agents (≈2 158€) +5 j/h remédiation (≈4 315€) = +6 473€ one-shot |
| **H6** | **Pas de rate limiting devant le proxy** (nouveau risque, 2026-05-06) | Décision architecturale RDG | MCO réactif standard ; pas de buffer dédié | Sol a flaggé le risque ; Omar confirme que les backends sont rate-limités mais pas le proxy lui-même | Si un caller déraisonnable inonde le proxy → incidents répétés, +1–2 j/h MCO/mois (+863–1 726€/mois). À adresser avec RDG avant go-live. |

> **⚠ Sensibilité résiduelle** : depuis la résolution de H1 et H2, **seul H6 (rate-limiting absent devant le proxy) reste un risque opérationnel ouvert** — sensibilité +863–1 726€/mois en pire cas si non adressé par RDG avant go-live. Le risque est désormais essentiellement architectural (côté client) et non lié à la robustesse du proxy lui-même. **À monitorer dans les 3 premiers mois post go-live.**

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
| MCO réactif | 1.0 | Estimation 1–2 tickets/mois × 0.5 j/h. Un peu plus de tickets possibles à cause de H6 (pas de rate limit proxy) — capté en sensibilité. |
| MCO proactif | 1.5 | 1.5× le réactif. Infra simple, charge stable bien en deçà des limites, runbooks et auto-scaling validés. |
| Buffer SLA Platine | 1.0 | Réduit vs estimation précédente (1.5 → 1.0) suite à la résolution de H2 : auto-scaling validé et headroom de capacité ~60% au-dessus du pic attendu réduisent le risque opérationnel. |
| Gouvernance | 0.33 | Selon abaque mutualisé — inchangé |
| **Total calibré** | **3.83** | |

### Comparaison

| Approche | j/h/mois | Prix mensuel (hors immo.) | Prix mensuel (avec immo. 1 000€) | Écart vs déductive |
|----------|----------|---------------------------|----------------------------------|--------------------|
| Déductive pure | 3.33 | 2 876€ | **3 876€** | — |
| Calibrée | 3.83 | 3 305€ | 4 305€ | +15% |
| **Repère SA (£2k/mois)** | ~2.7 (à 863€) | ~2 330€ (≈£2 000) | ~3 330€ | -14% |

**Lecture :** la calibrée dépasse la déductive de **15%** (auparavant 22%). Le delta s'est réduit suite à la validation de l'auto-scaling (résolution de H2). Les deux méthodes sont désormais **clairement cohérentes** (delta < 20%).

**Décision :** on retient la **déductive (3 876€/mois)** — méthode standard, auditable. Le delta de 15% avec la calibrée est dans la zone normale (sous le seuil de 20%) et confirme la robustesse de l'estimation. **À surveiller post go-live** : si le volume d'incidents lié à H6 (rate-limit absent devant le proxy) dépasse 2/mois, re-négocier en avenant.

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
| **Inventaire**          | **Servers :**<br>1 cluster Fargate (Public Cloud K8s, 3 workers, <1000 RPS, <50% CPU)<br>**Applications :**<br>1 app NGINX/OpenResty (high size, low cx) | **Servers :**<br>2 clusters Fargate (small, <10 RPS)<br>**Applications :**<br>2 apps NGINX/OpenResty (low) | Bucket S3 config, Lambda admin, NLB, dashboard CloudWatch, topics SNS, Route53 — bundlés dans les apps |
| **Services**            | Maintien : Gestion des demandes, incidents, problèmes, changements de version, continuité, surveillance, **astreintes 24/7**                            | Maintien : Gestion des demandes, incidents, problèmes, changements de version, continuité      | Gouvernance : COPROD trimestriel, Audit ROSE semestriel, Audit LEAF semestriel<br>Évolutions : **hors périmètre** |
| **Niveaux de services** | Platine                                                                                                                                                 | Bronze                                                                                          | —                                                                                                |
| **Plages de service**   | Complète (7j/7, 24h/24)                                                                                                                                 | Standard (Lun-Ven 9h30–18h30)                                                                   | —                                                                                                |
| **Dispositif**          | Ops : 2.2 j/mois · Lead Ops : 0.8 j/mois · Delivery Manager : 0.4 j/mois (total ≈ 3.3 j/h/mois, **Mutualisé**)                                                                                                                                                                                                                                                                  |

#### Prix mensuel €HT

| Mode | Périmètre | j/h/mois | Montant €HT/mois |
|------|-----------|----------|-------------------|
| **Forfait** | MCO (3.00 j/h) + Gouvernance (0.33 j/h) + Contingence 0% + Immobilisation | 3.33 | **3 876€** |
| **Temps passé** | Évolutions | — (hors périmètre) | — |
| | **Total mensuel** | **3.33** | **3 876€** |

**Total annuel : 46 512€ HT** (engagement 1 an, sans remise multi-annuelle).

---

## Annexe A — Détail du calcul mensuel

### Détail MCO par environnement

#### dev — Bronze (1.00) / Standard
| Resource | Item Type | Base | Coeff (max size/cx) | MCO j/h/mois |
|----------|-----------|------|---------------------|--------------|
| Fargate cluster | Public Cloud Managed K8s | 0.25 | 0.8 | 0.20 |
| NGINX/OpenResty app (+ ancillaries) | Off-the-shelf application | 0.50 | 0.8 | 0.40 |
| **Subtotal dev** | | | | **0.60** |

#### preprod (acc) — Bronze (1.00) / Standard
| Resource | Item Type | Base | Coeff | MCO j/h/mois |
|----------|-----------|------|-------|--------------|
| Fargate cluster | Public Cloud Managed K8s | 0.25 | 0.8 | 0.20 |
| NGINX/OpenResty app (+ ancillaries) | Off-the-shelf application | 0.50 | 0.8 | 0.40 |
| **Subtotal preprod** | | | | **0.60** |

#### prod — Platine (1.20) / Complète
| Resource | Item Type | Base | Coeff | MCO j/h/mois |
|----------|-----------|------|-------|--------------|
| Fargate cluster | Public Cloud Managed K8s | 0.25 | 2 | 0.50 |
| NGINX/OpenResty app (+ ancillaries) | Off-the-shelf application | 0.50 | 2 | 1.00 |
| **Subtotal prod (avant SLA)** | | | | **1.50** |

#### Synthèse MCO

| Env | MCO base (j/h/mois) | Coeff SLA | MCO ajusté (j/h/mois) |
|-----|--------------------|-----------|----------------------|
| dev | 0.60 | 1.00 | 0.60 |
| preprod (acc) | 0.60 | 1.00 | 0.60 |
| prod | 1.50 | 1.20 | 1.80 |
| **Total MCO** | | | **3.00** |

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
| MCO (ajusté SLA) | 3.000 |
| Gouvernance | 0.333 |
| Évolutions | 0.000 |
| **Total** | **3.333** |

→ Dispositif : **Mutualisé** (< 10 j/h/mois) ✓

### Détail prix

| Ligne | j/h/mois | TJM | Montant |
|-------|----------|-----|---------|
| MCO (ajusté SLA) | 3.000 | 863€ | 2 589€ |
| Gouvernance | 0.333 | 863€ | 287€ |
| Évolutions | 0.000 | 863€ | 0€ |
| **Sous-total services** | 3.333 | | **2 876€** |
| Immobilisation (Mutualisé × Complète, plus haute plage) | | | 1 000€ |
| Contingence forfait (0%) | | | 0€ |
| Remise multi-annuelle (1 an) | | | 0€ |
| **Total mensuel** | | | **3 876€** |
| **Total annuel** | | | **46 512€** |

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
  2. **Lock-in Platine sur prod** — valider avec le client que la SLA Platine reflète bien la criticité business. Drop à Gold = -376€/mois.
  3. **Sub-second spike** non testé (le test du 2026-05-07 valide une rampe de 5 min, pas un saut sub-seconde). Risque considéré faible étant donné le headroom de capacité validé, mais à mentionner si RDG demande une SLA de réactivité spike-instant.
- **Référence externe** : compatible avec [qualification.md](qualification.md) section « Constraints & Notes ».
- **Historique de versions** :
  - v1 (2026-05-06 matin) — initial estimate à 6 206€/mois avec coeff 5 (hypothèse SA).
  - v2 (2026-05-06 PM) — révisé à 3 876€/mois suite aux données initiales d'Omar (coeff 2 confirmé). Gain de 2 330€/mois.
  - **v3 (2026-05-07) — résolution de H2 suite au rapport `Proxy Auto Scale Test.pdf` : auto-scaling validé, capacité 1 367 RPS sustained sans erreur. Prix mensuel inchangé (3 876€/mois) ; calibration empirique resserrée (delta calibré-déductif passe de +22% à +15%). Confiance accrue dans le pricing.**
