# `estimate.md` Output Template

This is the exact structure to follow when generating `{client-name}/estimate.md`.

The file has four parts:

1. **Hypothèses de travail** — assumptions for missing info from the qualification
2. **Calibration empirique** — discount justification (Step 3)
3. **Synthèse** — client-facing summary (init block + monthly grid + price table)
4. **Annexes** — calculation detail (A: monthly, B: initialization, C: cross-check if FTE data)

## Template

```markdown
# Estimate — {Client Name}

**Date:** {date}
**Based on:** qualification.md ({date of qualification})
**TJM:** {amount}€ (blended) — voir `shared/daily-rates.md`
**Dispositif:** {Mutualisé / Semi-dédié / Dédié}
**Précision :** j/h exprimés au dixième de jour (0.1)

---

## Hypothèses de travail

{MANDATORY. List every assumption due to missing or incomplete info. If none, write "Aucune — toutes les informations étaient disponibles dans la qualification."}

| # | Hypothèse | Information manquante | Valeur retenue | Justification | Impact si l'hypothèse est fausse |
|---|-----------|----------------------|----------------|---------------|----------------------------------|
| H1 | {e.g. "Taille du cluster K8s prod"} | {e.g. "Sizing exact non fourni"} | {e.g. "Coefficient 1 (medium)"} | {e.g. "Standard pour ce profil"} | {e.g. "±500€/mois si réel = 0.5 ou 2"} |

> **⚠ Sensibilité** : {If any high-impact assumption, summarize. E.g., "H1 représente ±15% du prix final. Confirmer la taille du cluster avant contractualisation."}

---

## Calibration empirique

**Signaux empiriques observés (depuis qualification.md) :**

| Signal | Valeur |
|---|---|
| Tickets sur 12 mois | {N} → {N/12}/mois |
| Incidents | {N} ({type/cause}) |
| Problèmes récurrents | {N} |
| FTE empirique (si disponible) | {N} j/h/mois |
| Stabilité observée | {commentaire qualitatif} |

**Discount appliqué : {−X%} sur la MCO déductive**

**Justification :** {Cite la ligne de la table de discounts du Step 3 du skill qui correspond. Ex: "Pas de FTE communiqué ; tickets < 1/mois (5/an = 0.42/mois) sur 12 mois ; aucun problème récurrent → ligne 'infra ultra-stable' → fourchette −30% à −50%, retenu −X% (mid-range / borne basse / borne haute)."}

**Ajustement MCO :**

| | Déductive | Discount | Final |
|---|---|---|---|
| MCO (toutes envs, SLA appliqué) | {X} | −{X%} | {X × (1−d)} |
| Gouvernance | {Y} | (jamais discountée) | {Y} |
| **Total mensuel j/h** | **{X+Y}** | | **{(X(1−d)+Y)}** |

> **Garde-fous :**
> - Discount plafonné à −50% (au-delà : justification SA + revue stakeholder requise)
> - Gouvernance jamais discountée (audits ROSE/YAMAS/LEAF + COPROD = engagement contractuel)
> - SLA et immobilisation jamais discountés (déjà appliqués au niveau ressource ou plateforme)

---

## Synthèse

### Phase d'initialisation (one-shot)

Plateforme construite par Theodo : **{Oui / Non}**

> Monitoring et système d'agents IA sont **toujours** présents. Audit et remédiation prioritaire **n'apparaissent que si** la plateforme n'a pas été construite par Theodo (sinon, omettre les deux lignes).

- {Si Non} **Audit** : {audit_jh} j/h Lead Ops — cartographie ressources, qualité, résilience, sécurité, observabilité.
- {Si Non} **Remédiation prioritaire** (cible ROSE/YAMAS) : {remediation_jh} j/h — docs, durcissement résilience, gaps qualité.
- **Mise en place du monitoring** : {monitoring_jh} j/h — métriques, alerting, dashboards, sondes, runbooks.
- **Mise en place du système d'agents IA** : {ai_agent_jh} j/h — déploiement agent ChatOps, intégrations.

**Total initialisation : {total_init_jh} j/h — {total_init_price}€ HT (one-shot, payée une seule fois en début d'engagement)**

> Cette enveloppe est indépendante du prix mensuel récurrent ci-dessous.

---

### Tableau de synthèse mensuelle

|                         | {Env 1 category}                                                                                                                                                                                       | {Env 2 category}                                                     | {Env N category} | Transverse                                                                                                          |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------- | ---------------- | ------------------------------------------------------------------------------------------------------------------- |
| **Nom des envs**        | {env names}                                                                                                                                                                                            | {env names}                                                          | {env names}      | —                                                                                                                   |
| **Inventaire**          | **Servers :**{newline}{list servers with count, type, size}{newline}**Applications :**{newline}{list apps with count, type, complexity}                                                                | **Servers :**{newline}{...}{newline}**Applications :**{newline}{...} | {same format}    | {cross-cutting resources: organization, audit trail, IAM, CI/CD, etc.}                                              |
| **Services**            | Maintien : Gestion des demandes de service, des incidents, des problèmes, des changements de version, de la continuité{if plage != Standard: , de la surveillance et astreintes}                       | {same, adapted per env}                                              | {same}           | Gouvernance : COPIL/COPROD, audits ROSE/YAMAS/LEAF{newline}Évolutions : Gestion des Changements mineurs             |
| **Niveaux de services** | {Bronze / Silver / Gold / Platine}                                                                                                                                                                     | {level}                                                              | {level}          | —                                                                                                                   |
| **Plages de service**   | {Standard / Étendue / Complète}                                                                                                                                                                        | {plage}                                                              | {plage}          | —                                                                                                                   |
| **Dispositif**          | Ops : {X} j/mois, Lead Ops : {Y} j/mois, Delivery Manager : {Z} j/mois                                                                                                                                |                                                                      |                  |                                                                                                                     |

#### Prix mensuel €HT

| Mode | Périmètre | j/h/mois | Montant €HT/mois |
|------|-----------|----------|-------------------|
| **Forfait** | MCO ({x} j/h) + Gouvernance ({y} j/h) + Contingence {z}% + Immobilisation | {x+y} | {amount}€ |
| **Temps passé** | Évolutions | {e} | {amount}€ |
| | **Total** | **{total}** | **{total amount}€** |

### Notes on the synthesis table

- **Init block placed above the synthesis grid.** Audit and remédiation lines are **omitted** if the platform was built by Theodo. The init price is **never** added to the monthly recurring price.
- **Inventaire**: human-readable. Group by "Servers" (K8s clusters, VMs, hypervisors, managed DBs, networking) and "Applications" (off-the-shelf and custom). Include count, name, size.
- **Services**: MCO per env column; governance + evolutions in "Transverse". Don't put the engagement model here — it's in the price table.
- **Transverse column**: cross-cutting resources (org, IAM, audit trail) in inventory; governance + evolutions in services. No SLA/plage.
- **Prix mensuel**: separate table below the grid. Always show two rows (forfait vs temps passé) to make committed vs consumed clear.

---

## Annexe A — Detailed Calculation

### MCO Breakdown (deductive)

#### Environment: {env_name}

**SLA:** {level} (coeff {x})
**Plage:** {plage}

| Resource     | Item Type | Base Rate | Coeff   | MCO (j/h/mois) |
| ------------ | --------- | --------- | ------- | -------------- |
| {name}       | {type}    | {rate}    | {coeff} | {result}       |
| **Subtotal** |           |           |         | **{sum}**      |

{Repeat for each environment.}

#### MCO Summary

| Environment   | MCO base (j/h/mois) | SLA Coeff | MCO ajusté SLA |
| ------------- | ------------------- | --------- | -------------- |
| {env}         | {days}              | {coeff}   | {adjusted}     |
| **Total MCO déductive** |             |           | **{total}**    |

### Discount empirique

| Étape | j/h/mois |
|---|---|
| MCO déductive | {X} |
| × (1 − discount {d%}) | {X × (1−d)} |
| **MCO finale** | **{final}** |

### Gouvernance

| Activité | Fréquence | Effort/session | j/h/mois |
|----------|-----------|----------------|----------|
| COPROD   | {freq}    | 0.5 j/h        | {calc}   |
| COPIL    | {freq}    | 0.5 j/h        | {calc}   |
| ROSE     | semestriel | 0.5 j/h       | {calc}   |
| YAMAS    | semestriel | 0.5 j/h       | {calc}   |
| LEAF     | semestriel | 0.5 j/h       | {calc}   |
| **Total Gouvernance** | | | **{y} j/h/mois** |

### Évolutions

- Estimées : **{x} j/h/mois**
- Base : {explanation}

### Total quantité

| Catégorie         | j/h/mois    |
| ----------------- | ----------- |
| MCO finale        | {x}         |
| Gouvernance       | {y}         |
| Évolutions        | {z}         |
| **Total**         | **{total}** |

### Prix

| Ligne             | j/h/mois | TJM    | Montant       |
| ----------------- | -------- | ------ | ------------- |
| MCO finale        | {x}      | {tjm}€ | {amount}€     |
| Gouvernance       | {y}      | {tjm}€ | {amount}€     |
| Évolutions        | {z}      | {tjm}€ | {amount}€     |
| **Sous-total**    |          |        | **{amount}€** |
| Immobilisation    | {plage} × {dispositif} | | {amount}€ |
{If forfait:} | Contingence | {level} (+{x}%) sur MCO+Gouv | | +{amount}€ |
{If multi-year:} | Remise multi-annuelle | {x} ans | | −{amount}€ |
| **Total mensuel** |          |        | **{total}€**  |
| **Total annuel**  |          |        | **{total × 12}€** |

---

## Annexe B — Initialisation (one-shot)

| Composante | Sizing | j/h | TJM | Montant €HT |
|------------|--------|-----|-----|-------------|
| {Si Non} Audit (Lead Ops) | {Small / Medium / Large} | {audit_jh} | {tjm_lead_ops}€ | {amount}€ |
| {Si Non} Remédiation prioritaire | {Light / Medium / Heavy} | {remediation_jh} | {blended_tjm}€ | {amount}€ |
| Mise en place du monitoring | {Simple / Medium / Complex} | {monitoring_jh} | {blended_tjm}€ | {amount}€ |
| Mise en place système d'agents IA | {Simple / Medium / Complex} | {ai_agent_jh} | {blended_tjm}€ | {amount}€ |
| **Total initialisation** | | **{total_init_jh}** | | **{total_init_price}€** |

**Notes :**
- Audit et remédiation **omis** (lignes non affichées) si la plateforme a été construite par Theodo.
- Audit facturé au **TJM Lead Ops** (cf. `shared/daily-rates.md`).
- Remédiation, monitoring et système d'agents IA facturés au **TJM blended (Ops + Lead Ops + DM)**.
- Cette enveloppe est **payée une seule fois** en début d'engagement et n'entre pas dans le prix mensuel récurrent.

---

## Annexe C — Cross-check FTE (si applicable)

{Inclure cette annexe SEULEMENT si la qualification contient un FTE breakdown. Sinon, l'omettre entièrement.}

| Catégorie  | Déductive (j/h/mois) | FTE-derived (j/h/mois) | Delta     | Delta % |
| ---------- | -------------------- | ---------------------- | --------- | ------- |
| MCO        | {x}                  | {a}                    | {x-a}     | {%}     |
| Gouvernance | {y}                 | {b}                    | {y-b}     | {%}     |
| Évolutions | {z}                  | {c}                    | {z-c}     | {%}     |
| **Total**  | **{X}**              | **{A}**                | **{X-A}** | **{%}** |

**Analyse :** {Explanation. Flag if deductive > FTE by >20% (calibration justifies discount per Step 3 table) ; flag if FTE > deductive by >20% (inventory gap or hidden complexity — investigate).}

---

## Notes

- {HDS applicability}
- {Nearshore flag if relevant}
- {Engagement period and discounts}
- {Other considerations}
```
