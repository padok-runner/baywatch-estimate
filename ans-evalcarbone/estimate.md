# Estimate — ANS / EvalCarbone SIH

**Date :** 2026-05-04
**Based on :** [qualification.md](qualification.md) (2026-05-04)
**TJM blended :** 863€ HT (Ops 750€ + Lead Ops 1 200€ + DM 850€, ratio 1 : 0.34 : 0.16)
**TJM Lead Ops (audit) :** 1 200€ HT
**Dispositif :** **Semi-dédié** (équipe dédiée ANS — mutualisée sur les prestations PFC + EvalCarbone SIH + futures)
**Mode d'engagement :** Forfait MCO + Gouvernance (sans contingence — scope bien connu) + Évolutions au temps passé
**Durée :** 3 mois renouvelable

---

## Hypothèses de travail

| # | Hypothèse | Information manquante (qualification) | Valeur retenue | Justification | Impact si fausse |
|---|-----------|----------------------------------------|----------------|---------------|------------------|
| H1 | Accès au repo Helm + ArgoCD disponible dès la signature | Accès direct au repo de déploiement non confirmé | Audit Small (2.5 j/h) tient | L'utilisateur a confirmé une structure simple (Helm + Application ArgoCD) | Faible (~0.5–1 j/h en plus si l'accès est tardif → +500–1 000€ sur l'init) |
| H2 | Volumétrie post-bascule reste comparable au pré-bascule | Historique post-bascule limité (2 mois mars→mai 2026) | 1–2 incidents / 6 mois post-bascule | La bascule a été un événement ponctuel, pas un changement de pattern d'usage | Moyen (±1 j/h/mois MCO si la PFC introduit de nouveaux modes de défaillance) |
| H3 | Coefficient Kafka "very high" (×5) sur-estime la charge réelle | Sizing exact Kafka non fourni | Calibration empirique retenue plutôt que déductive pure | Kafka ici tourne en config par défaut, faible volumétrie ; le palier ×5 du tableau cible les déploiements production-grade haute charge | Élevé sur la sensibilité prix (cf. ⚠ ci-dessous) |
| H4 | Aucune évolution applicative cliente sur 3 mois | Backlog "Aucune", dev en sommeil | 0 j/h/mois Évolutions au temps passé | Cohérent avec qualification (0 dev actif côté ANS) | Faible (chaque évolution serait facturée hors forfait au temps passé) |
| H5 | Mises à jour NumEcoEval (1 majeure/an + ~3 mineures sélectives) intégrées au MCO | Cadence exacte d'intégration des mineures non figée | ~2 montées de version intégrées sur 12 mois → ~0.3 j/h/mois | Cadence empirique observée par le MTE sur le projet open source | Faible |
| H6 | Remédiation post-audit à 0 j/h (opt-out client) | Magnitude inconnue | 0 j/h, à reproposer en amendement si l'audit Small remonte des gaps significatifs | Choix client documenté (engagement 3 mois, scope volontairement allégé) | Élevé si l'audit révèle des risques (saturation disque non monitorée, pas de backup automatisé) → ré-évaluation au renouvellement |
| H7 | Cadence gouvernance EdB (COPROD mensuel, COPIL trimestriel) maintenue, sessions allégées | Abaque Semi-dédié prévoit COPROD mensuel, pas de COPIL | Cadence client honorée mais sessions courtes — total **0.55 j/h/mois** (réduit de 50% vs estimation initiale 1.1) | SA confirme "peu de travail réel" sur la gouvernance vu la stabilité de la plateforme et le scope restreint | Faible — si les sessions demandent plus de prep, ajustement possible au temps passé |
| H8 | Pas de contingence forfait | — (choix SA) | +0% sur (MCO + Gouv) | SA confirme connaissance fine du scope (repo public, opérations connues, bascule PFC déjà absorbée par l'autre prestation Theodo) | **Élevé** — pas de matelas si surprise opérationnelle (saturation disque récidivante, défaillance Kafka inhabituelle, montée de version NumEcoEval problématique) |
| H9 | Pas d'immobilisation | Dispositif assimilé Semi-dédié | 0€/mois (Semi-dédié × Standard, conforme `service-levels.md`) | EvalCarbone SIH s'inscrit dans l'équipe dédiée ANS (déjà en place via la prestation PFC). Pas de coût d'immobilisation supplémentaire | Faible — basculer vers Mutualisé pur réintroduirait +500€/mois |

> **⚠ Sensibilité — H3 + H8 + H9 cumulés** : la combinaison "calibrée + pas de contingence + pas d'immobilisation" produit le prix le plus tendu possible (3 064€/mois HT vs ~10 200€ déductif strict avec immobilisation et +10%). **Aucun matelas** dans la structure de prix. Si la réalité opérationnelle s'approche du déductif (Kafka instable, incidents fréquents), le forfait sera structurellement insuffisant. **Recommandation** : revue formelle des indicateurs en fin de période 1 (mois 3), ajustement à la hausse au renouvellement si nécessaire (réintégration possible d'une contingence +10–20% ou réévaluation de la calibration).

---

## Analyse de réalisme

### Signaux empiriques (depuis qualification.md)

| Signal | Valeur 6 mois | Annualisé |
|--------|--------------|-----------|
| Incidents (toute criticité) | 1 | 2 |
| Hotfix | 1 | 2 |
| MEP / changements majeurs | 1 (bascule PFC) | 2 |
| Service requests | 0 | 0 |
| Changements standards | 0 | 0 |
| Incidents sécurité | 0 | 0 |
| FTE actuel infogérance | 0 (best effort) | — |

### Calibration empirique

| Composante | Détail | j/h/mois |
|------------|--------|----------|
| MCO réactif | 8 j/an = (2 incidents × 1j) + (2 hotfix × 1j) + (2 MEP × 2j) | 0.67 |
| MCO proactif — Versions NumEcoEval | ~2 montées intégrées/an × 2j | 0.33 |
| MCO proactif — Patch management | 2/an × 1j | 0.17 |
| MCO proactif — Scan sécurité | 2/an × 0.5j | 0.08 |
| MCO proactif — Surveillance & runbook | 1j/mois | 1.00 |
| MCO proactif — Reporting mensuel | 0.5j/mois | 0.50 |
| Buffer SLA Bronze + plage Standard | Pas d'astreinte, pas de garantie HNO | 0 |
| **Sous-total MCO calibré** |  | **2.75 → arrondi 3.0** |
| Gouvernance (COPIL trim. + COPROD mens. + ROSE/LEAF semestriels + COSEC ad hoc), **sessions allégées −50%** | Cadence EdB, charge réelle faible | 0.55 |
| **Total calibré** |  | **3.55** |

### Comparaison déductif vs calibré

| Approche | j/h/mois | Prix mensuel HT (sans contingence ni immo) | Écart |
|----------|----------|---------------------------------------------|-------|
| Déductive pure (avec gouv 1.1) | 14.10 | ~12 168€ | référence |
| **Calibrée retenue (gouv 0.55, sans contingence, sans immo)** | **3.55** | **3 064€** | **−75%** |

**Décision :** retenir la **calibrée**. Le déductif sur-estime fortement à cause du coefficient Kafka ×5 (sur un déploiement faible volumétrie) et de la duplication intégrale du MCO sur la préprod (alors qu'en pratique la préprod en Bronze nécessite un effort moindre). Les 8 événements/an observés ne justifient pas 13 j/h/mois. Le déductif sert de **plafond** ; un écart soutenu vers le haut sur la période 1 déclencherait une revue.

---

## Synthèse

### Phase d'initialisation (one-shot)

Plateforme construite par Theodo : **Non**

> Audit et remédiation prioritaire ne sont incluses **que** parce que la plateforme n'a pas été construite par Theodo. Le client a explicitement écarté la remédiation prioritaire et le système d'agents IA pour cette première période de 3 mois (engagement court, scope volontairement allégé).

- **Audit** : 2.5 j/h Lead Ops — cartographie ressources EvalCarbone, qualité IaC (Helm + ArgoCD), résilience, sécurité, observabilité. Cible ROSE.
- **Remédiation prioritaire** (cible ROSE) : **0 j/h — opt-out client**. À reproposer en amendement si l'audit remonte des findings critiques.
- **Mise en place du monitoring** : 2.5 j/h — métriques, alerting (saturation disque en priorité), dashboards par service, sondes blackbox, runbooks.
- **Mise en place du système d'agents IA** : **0 j/h — opt-out client**. ChatOps non déployé sur cette période 3 mois.

**Total initialisation : 5 j/h — 5 158€ HT (one-shot, payé une seule fois en début d'engagement)**

> Cette enveloppe est indépendante du prix mensuel récurrent ci-dessous et n'entre pas dans la base de la contingence forfait.

---

|                         | Production                                                                                                                                                                                                                       | Préproduction / dev                                                                                                                          | Transverse                                                                                                                                |
| ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| **Nom des envs**        | prod                                                                                                                                                                                                                             | preprod                                                                                                                                      | gouvernance & évolutions                                                                                                                  |
| **Inventaire**          | **Servers :** N/A (cluster K8s PFC ANS hors scope, autre prestation Theodo)<br>**Applications :** 9 conteneurs Docker — 1 NextJS (custom, S), 1 NGINX (S), 1 PostgreSQL (M, 23 tables), 1 Kafka (M), 1 Zookeeper (S), 4 microservices Java NumEcoEval (S) | **Servers :** N/A<br>**Applications :** 9 conteneurs Docker (mêmes images que prod, dimensionnement allégé)                                  | RACI (interactions PFC ANS, ATIH/Plage), comitologie, suivi qualité ROSE, FinOps/Green IT (LEAF) |
| **Services**            | Maintien : Gestion des demandes de service, des incidents, des problèmes, des changements de version (mises à jour NumEcoEval), de la continuité, de la surveillance native PFC                                                  | Maintien : idem prod, en best-effort sur plage Standard sans astreinte                                                                       | Gouvernance : COPIL trimestriel (1h30), COPROD mensuel (30 min), audit ROSE semestriel, audit LEAF (FinOps/Green IT) semestriel, COSEC à la demande<br>Évolutions : Gestion des Changements mineurs (< 5 j ouvrés) au temps passé |
| **Niveaux de services** | Bronze                                                                                                                                                                                                                           | Bronze                                                                                                                                       | —                                                                                                                                         |
| **Plages de service**   | Standard (Lundi-Vendredi 09h30-18h30, conforme JO/HO EdB)                                                                                                                                                                        | Standard                                                                                                                                     | —                                                                                                                                         |
| **Dispositif**          | Ops : 2.4 j/mois, Lead Ops : 0.8 j/mois, Delivery Manager : 0.4 j/mois (équipe **dédiée ANS**, mutualisée sur les prestations Theodo pour ANS)                                                                                                       |                                                                                                                                              |                                                                                                                                           |

#### Prix mensuel HT

| Mode | Périmètre | j/h/mois | Montant €HT/mois |
|------|-----------|----------|------------------|
| **Forfait** | MCO (3.0 j/h Bronze × 2 envs) + Gouvernance (0.55 j/h, sessions allégées) — sans contingence, sans immobilisation | 3.55 | **3 064€** |
| **Temps passé** | Évolutions (changements mineurs <5j), facturé à la consommation | 0 prévu | 0€ |
| | **Total mensuel** | **3.55** | **3 064€** |

**Total annualisé indicatif :** 36 768€ HT/an
**Total période ferme 3 mois :** 5 158€ (init one-shot) + 9 192€ (récurrence) = **14 350€ HT**

---

## Annexe A — Détail du calcul mensuel

### Détail MCO

#### Environnement : Production
**SLA :** Bronze (coeff 1.00)
**Plage :** Standard

| Resource | Item Type | Base Rate | Coeff | MCO déductif (j/h/mois) |
|----------|-----------|-----------|-------|-------------------------|
| Frontend NextJS | Custom application | 0.25 | 0.8 (low) | 0.20 |
| NGINX reverse proxy | Off-the-shelf application | 0.5 | 0.5 (very low) | 0.25 |
| PostgreSQL | Off-the-shelf application | 0.5 | 2.0 (high — DB) | 1.00 |
| Kafka | Off-the-shelf application | 0.5 | 5.0 (very high — kafka) | 2.50 |
| Zookeeper | Off-the-shelf application | 0.5 | 0.8 (low) | 0.40 |
| api-rest-expositiondonneesentrees | Off-the-shelf application | 0.5 | 0.8 (low) | 0.40 |
| api-rest-referentiels | Off-the-shelf application | 0.5 | 0.8 (low) | 0.40 |
| api-event-donneesentrees | Off-the-shelf application | 0.5 | 0.8 (low) | 0.40 |
| api-event-calculs | Off-the-shelf application | 0.5 | 0.8 (low) | 0.40 |
| **Sous-total prod (déductif)** | | | | **5.95** |

#### Environnement : Préproduction / dev
**SLA :** Bronze (coeff 1.00)
**Plage :** Standard

Inventaire identique à la prod → sous-total déductif = **5.95 j/h/mois** (avant pondération réalisme).

#### MCO Summary (déductif)

| Environnement | MCO (j/h/mois) | Coeff SLA | MCO ajusté |
|---------------|----------------|-----------|------------|
| Production | 5.95 | 1.00 (Bronze) | 5.95 |
| Préprod | 5.95 | 1.00 (Bronze) | 5.95 |
| **Total MCO déductif** | | | **11.90** |
| **Total MCO retenu (calibré)** | | | **3.00** |

> Le total retenu (3.0 j/h/mois) est issu de la calibration empirique (cf. Analyse de réalisme). Le déductif (11.9) sert de plafond de référence.

### Gouvernance (sessions allégées −50% — H7)

| Activité | Fréquence | Effort/session | j/h/mois |
|----------|-----------|----------------|----------|
| COPROD | Mensuel (per EdB §9.2) | 0.25 j/h (allégé) | 0.25 |
| COPIL | Trimestriel (per EdB §9.1) | 0.25 j/h (allégé) | 0.08 |
| Audit ROSE (Qualité) | Semestriel | 0.5 j/h (allégé) | 0.08 |
| Audit YAMAS (HDS) | N/A — HDS non requis | — | 0.00 |
| Audit LEAF (FinOps / Green IT) | Semestriel | 0.5 j/h (allégé) | 0.08 |
| COSEC + comités ad hoc | À la demande (réserve) | — | 0.06 |
| **Total Gouvernance** | | | **0.55 j/h/mois** |

> Réduction 50% appliquée sur la base de l'estimation initiale (1.10 j/h/mois). Justification : SA confirme une charge réelle faible vu la stabilité de la plateforme et le scope restreint. La cadence client EdB (mensuel/trimestriel) est préservée.

### Évolutions

- Estimées : **0 j/h/mois** prévues
- Justification : dev en sommeil côté ANS, aucun backlog d'évolution sur les 6 prochains mois (qualification §Backlog 6 mois). Les changements mineurs (<5 j ouvrés) restent disponibles en temps passé à la consommation.

### Total quantité (retenu)

| Catégorie | j/h/mois |
|-----------|----------|
| MCO (calibré, ajusté SLA Bronze ×1.00) | 3.00 |
| Gouvernance (sessions allégées) | 0.55 |
| Évolutions | 0.00 |
| **Total** | **3.55** |

### Détail prix

| Ligne | j/h/mois | TJM | Montant HT |
|-------|----------|-----|------------|
| MCO (3.0 × 1.00 SLA × 863€) | 3.00 | 863€ | 2 589€ |
| Gouvernance (0.55 × 863€) | 0.55 | 863€ | 475€ |
| Évolutions (au temps passé, 0 prévu) | 0.00 | 863€ | 0€ |
| **Sous-total** | 3.55 | | **3 064€** |
| Contingence forfait | Retirée — H8 | | 0€ |
| Immobilisation | Retirée — H9 (Semi-dédié × Standard = 0€) | | 0€ |
| **Total mensuel** | **3.55** | | **3 064€** |
| **Total annualisé indicatif** | | | **36 768€** |

> Aucune remise multi-annuelle : engagement 3 mois renouvelable, sous le seuil 2 ans (-3%).
> Aucune contingence forfait : SA confirme connaissance fine du scope (H8). Aucun matelas — voir Sensibilité dans Hypothèses de travail.
> Aucune immobilisation : prestation rattachée à l'équipe dédiée ANS (H9).

---

## Annexe B — Initialisation (one-shot)

| Composante | Sizing | j/h | TJM | Montant HT |
|------------|--------|-----|-----|-------------|
| Audit (Lead Ops) | Small | 2.5 | 1 200€ | 3 000€ |
| Remédiation prioritaire | Opt-out client (0) | 0 | 863€ | 0€ |
| Mise en place du monitoring | Simple | 2.5 | 863€ | 2 158€ |
| Mise en place système d'agents IA | Opt-out client (0) | 0 | 863€ | 0€ |
| **Total initialisation** | | **5.0** | | **5 158€** |

**Notes :**
- Audit facturé au **TJM Lead Ops** (1 200€) ; monitoring au **TJM blended** (863€).
- Remédiation et système d'agents IA sont en **opt-out client** (0 j/h) — décision tracée dans la qualification (§Phase d'initialisation et §Informations manquantes).
- Si l'audit remonte des findings critiques (ex. pas de backup automatisé, saturation disque non monitorée correctement), une remédiation pourra être proposée en amendement ou au renouvellement.
- Cette enveloppe est **payée une seule fois** en début d'engagement et n'entre pas dans le prix mensuel récurrent ni dans la base de la contingence forfait.

---

## Annexe C — Cross-check Déductif vs Empirique

| Catégorie | Déductive (j/h/mois) | Empirique (j/h/mois) | Calibrée retenue (j/h/mois) | Delta Déd. vs Calibré |
|-----------|----------------------|----------------------|------------------------------|------------------------|
| MCO | 11.90 | ~0 (FTE = 0) | 3.00 | −75% |
| Gouvernance | 1.10 (cadence pleine) | ~0 | 0.55 (sessions allégées) | −50% |
| Évolutions | 0 | 0 | 0 | — |
| **Total** | **13.00** | **~0** | **3.55** | **−73%** |

**Analyse :**
- L'écart **Déductif vs Empirique** est extrême (>1000%). L'empirique reflète l'absence d'infogérance formelle (best-effort, 0 FTE) ; il n'est pas exploitable comme baseline cible. La saturation disque non monitorée sur les 6 derniers mois confirme que le statu quo n'est pas tenable.
- L'écart **Déductif vs Calibré** est de −73% : le déductif sur-estime principalement à cause du coefficient Kafka ×5 (cible déploiements production-grade haute charge) et de la duplication 2× du MCO sur la préprod. La gouvernance est elle-même réduite de 50% (sessions allégées) sur la base de la connaissance terrain du SA.
- La calibrée est construite bottom-up à partir des signaux empiriques (8 tickets/an + activités proactives nécessaires) et représente l'effort réellement attendu sur cette plateforme stable et faiblement chargée.

**Décision :** facturation basée sur l'estimation **calibrée** (3.55 j/h/mois). Le déductif sert de plafond de référence à surveiller pendant la période 1.

---

## Notes

- **HDS** : non applicable (sécurité FAIBLE, EdB §1.5 et §7.1). Pas de surcoût HDS sur cette prestation.
- **Nearshore** : non envisagé sur cette prestation (engagement court 3 mois, scope critique mais réduit). Si réduction de coût recherchée au renouvellement, à discuter avec Hugo / Lila / Manu.
- **TMA** : exclue de cette prestation (cf. MT slide 13). Mémoire technique et prestation distincts.
- **Cluster K8s PFC ANS** : géré par Theodo dans une prestation distincte (référence interne Semi-dédié).
- **Renouvellement** : à 3 mois, prévoir une revue formelle avec indicateurs réels (volumétrie tickets post-bascule, findings d'audit, gap remédiation) pour ajuster la période 2.
