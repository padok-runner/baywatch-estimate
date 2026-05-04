---
name: estimate
description: Estimation & Pricing for Infogérance Cloud — calculates man-days/month and computes the final price from qualification data
user_invocable: true
---

# /estimate — Estimation & Pricing

You are a Solutions Architect assistant computing the Infogérance Cloud estimate and price. You read the client's `qualification.md` and produce a complete `estimate.md`.

## Prerequisites

Before starting, locate and read the client's `qualification.md`. If it doesn't exist, tell the user to run `/qualify` first.

Also read the reference files:

<reference>
Read the file at `skills/shared/item-types.md` relative to the `.claude/skills` directory for item types and MCO base rates.
</reference>

<reference>
Read the file at `skills/shared/coefficients.md` relative to the `.claude/skills` directory for size/complexity coefficients.
</reference>

<reference>
Read the file at `skills/shared/service-levels.md` relative to the `.claude/skills` directory for plage horaire, SLA tables, immobilisation, and selection guides.
</reference>

<reference>
Read the file at `skills/shared/pricing-rules.md` relative to the `.claude/skills` directory for engagement modes, discounts, and business rules.
</reference>

<reference>
Read the file at `skills/shared/daily-rates.md` relative to the `.claude/skills` directory for role TJMs and team composition.
</reference>

<reference>
Read the file at `skills/shared/services.md` relative to the `.claude/skills` directory for the service catalogue (descriptions, activities, deliverables) used in the synthesis table.
</reference>

---

## Phase A — Quantity Estimation

### Step 0: Identify missing information and define working assumptions

Read the **"Informations manquantes"** section from `qualification.md`. For each missing item, define an explicit working assumption ("hypothèse de travail") that will be used throughout the estimate. These assumptions must be:

- **Stated clearly** — no ambiguity about what value was chosen
- **Conservative** — when in doubt, assume slightly higher complexity/size (it's better to over-estimate than under-estimate)
- **Justified** — explain briefly why this assumption is reasonable (e.g., "based on similar client profiles", "default for this resource type")
- **Flagged for impact** — note whether the assumption could significantly change the final price if wrong

These assumptions will be documented in the **"Hypothèses de travail"** section of the output, immediately after the header.

### Step 1: Deductive estimate (ALWAYS computed)

The deductive estimate is always the primary calculation, based on the resource inventory.

For each resource in each environment from `qualification.md`:

```
MCO per resource = item_base_rate × size_complexity_coefficient
```

Use the item base rates from `shared/item-types.md` and the size/complexity coefficients from `shared/coefficients.md`. When a resource has both a server size and an application complexity assessment, use the **higher** of the two coefficients.

Sum all MCO values per environment, then across all environments.

**Governance (deductive):**

Calculate governance days using the abaques in `shared/pricing-rules.md`. Sum the monthly equivalent for each activity based on the dispositif:
- **COPROD**: frequency depends on dispositif (weekly for dédié, monthly for semi-dédié, quarterly for mutualisé)
- **COPIL**: quarterly, dédié only
- **Audits ROSE, YAMAS, LEAF**: semestriel, 1 j/h each (YAMAS only if HDS)

Convert each to j/h/mois: `effort_per_session × sessions_per_year / 12`.

**Evolutions:**
Estimate evolution days like build work. Use the evolution backlog from `qualification.md` to determine a monthly average. If the user hasn't provided enough detail, ask them to estimate the evolution effort.

**Deductive total:**

```
Deductive total j/h/mois = MCO + Governance + Evolutions
```

### Step 2: Empirical estimate (only if empirical data available)

If `qualification.md` contains an "Empirical Data" section with FTE breakdown, extract the empirical estimate:

```
Empirical MCO      = MCO FTEs × 20 j/h/mois
Empirical Governance = Governance FTEs × 20 j/h/mois
Empirical Evolutions = Evolution FTEs × 20 j/h/mois
Empirical total    = sum of above
```

Also note any inefficiencies or gaps flagged in the qualification (e.g., "currently understaffed on monitoring" or "no dedicated governance time"). These inform whether the empirical number is a floor or a ceiling.

### Step 3: Cross-check (when both approaches available)

When both estimates exist, present them side by side:

```markdown
| Category   | Deductive (j/h/mois) | Empirical (j/h/mois) | Delta     | Delta % |
| ---------- | -------------------- | -------------------- | --------- | ------- |
| MCO        | {x}                  | {a}                  | {x-a}     | {%}     |
| Governance | {y}                  | {b}                  | {y-b}     | {%}     |
| Evolutions | {z}                  | {c}                  | {z-c}     | {%}     |
| **Total**  | **{X}**              | **{A}**              | **{X-A}** | **{%}** |
```

**Interpretation rules:**

- **Delta < 20%**: Estimates are coherent. Use the **deductive estimate** as the basis for pricing (it's more granular and auditable). Mention that empirical data confirms the estimate.
- **Deductive > Empirical by >20%**: The client may be understaffed today. Flag this: "Current team may be under-resourced — deductive analysis suggests {X} j/h/mois is needed vs {A} currently spent. Investigate whether quality/SLA is being met today."
- **Empirical > Deductive by >20%**: Either the resource inventory is incomplete, complexity is underestimated, or the current team has inefficiencies. Flag this and ask the SA to investigate:
  - Are there resources missing from the inventory?
  - Is the current team handling tasks outside the MCO scope?
  - Are there operational inefficiencies that Theodo would optimize?
- **Always use the deductive estimate for pricing** — it's the standard methodology. But document the empirical data as context and flag discrepancies clearly.

### Step 4: Reality check — Calibrate the estimate

**This step is mandatory.** Before finalizing, step back and critically evaluate whether the deductive number makes sense given the empirical signals available.

Even when a full empirical estimate (FTE breakdown) is not available, there are almost always qualitative signals: ticket volume, incident count, team size, infrastructure stability. Use these to calibrate.

#### 4a. Estimate effort from empirical signals

Build a bottom-up "calibrated" estimate from observable data:

| Composante | How to estimate |
|------------|----------------|
| **MCO réactif** | `(tickets per year × avg effort per ticket) / 12`. If avg effort unknown, assume 0.5-1 j/h per ticket. |
| **MCO proactif** | Monitoring, patching, security updates. Typically 3-5× the reactive effort for well-managed infra. Adjust for infra complexity: simple LAMP stack = 3×, complex K8s+microservices = 5×. |
| **Buffer SLA** | Add overhead proportional to SLA commitment: Bronze = 0, Silver = +0.5, Gold = +1.0, Platine = +2.0 j/h/mois. Higher plage (Étendue, Complète) adds astreinte overhead. |
| **Gouvernance** | Keep the abaque calculation (unchanged). |

#### 4b. Compare deductive vs calibrated

Present a comparison table:

```markdown
| Approche | j/h/mois | Prix/mois | Écart |
|----------|----------|-----------|-------|
| Déductive pure | {X} | {price_X}€ | — |
| Calibrée | {Y} | {price_Y}€ | {%} |
```

#### 4c. Choose the final estimate

- **If deductive and calibrated are within 20%**: use the deductive (more auditable).
- **If deductive > calibrated by >20%**: the deductive likely overestimates. **Use the calibrated estimate** and explain why. The deductive serves as a ceiling/reference.
- **If calibrated > deductive by >20%**: the inventory may be incomplete or the client has hidden complexity. Investigate before proceeding.

**Key principle:** the estimate should reflect what we will actually do, not what the model mechanically produces. A stable infra with 5 tickets/year does not need 7 j/h/mois of MCO just because the formula says so.

### Step 5: Final total & Dispositif

Use the estimate chosen in Step 4c:

```
Total jour-homme/mois = MCO (deductive or calibrated) + Governance + Evolutions
```

Determine the dispositif using the thresholds in `shared/pricing-rules.md`.

### Step 5b: Initialization (one-time) estimate

Read the **"Phase d'initialisation"** section from `qualification.md`. Convert each sizing answer to a j/h value using the abaques in `shared/initialization.md`, then compute the one-shot price using the pricing rule from the same file.

<reference>
Read the file at `skills/shared/initialization.md` relative to the `.claude/skills` directory for the components, sizing abaques, pricing rule, and display rule.
</reference>

Reminders (full detail in the shared file):
- Audit and remédiation are **omitted** if the platform was built by Theodo.
- Audit is priced at the **Lead Ops TJM**; the other three components use the **blended TJM** (see `shared/daily-rates.md`).
- The initialization price is **paid once** and is **separate from the recurring monthly price** — never fold it into the monthly total or into the contingency calculation.

---

## Phase B — Pricing

### Step 6: Base price

Use the blended TJM from `shared/daily-rates.md`. If the user specifies a different TJM, use theirs instead. Then:

```
Base MCO price per env = MCO_days_env × SLA_coefficient × TJM
```

Apply the SLA coefficients per environment from `shared/service-levels.md`.

```
Total MCO price = sum of (MCO_days_env × SLA_coeff_env × TJM) for all environments
Governance price = Governance_days × TJM
Evolution price = Evolution_days × TJM
```

### Step 7: Immobilisation

Add immobilisation fees based on dispositif and plage horaire — use the immobilisation table from `shared/service-levels.md`.

If the client has multiple plage horaires, apply the immobilisation for the highest plage.

For Etendue and Complète, there is also a prix horaire HNO (heures non ouvrées) — see the HNO column in `shared/service-levels.md`.

### Step 8: Engagement model

Ask the user which model:

**Temps passé (default):**

- Staffed envelope of man-days, reportable M+1
- Priority: incidents > problems > version changes > minor changes
- Price = Base price + Immobilisation

**Forfait:**

- Add contingency based on uncertainty:
  - No uncertainty: +0%
  - Low: +10%
  - Medium: +20%
  - High: +30-40%
- **Forfait applies to MCO + Governance ONLY — NOT evolutions**
- Price = (MCO price + Governance price) × (1 + contingency%) + Evolution price + Immobilisation

### Step 9: Multi-year discounts

Apply multi-year discounts from `shared/pricing-rules.md` if applicable.

### Step 10: Additional considerations

- **HDS**: Default in France. Flag if applicable.
- **Nearshore**: If the client wants to reduce costs, flag that nearshore should be discussed with Hugo/Lila/Manu. Do NOT calculate nearshore pricing — just flag it.

---

## Output

Generate `estimate.md` in the client's directory (`{client-name}/estimate.md`).

The file has three parts:

1. **Analyse de réalisme** — the reality check comparing deductive vs calibrated estimates
2. **Synthèse** — the client-facing summary, composed of a **one-shot initialization block** (above) and the **monthly synthesis table** (below). Primary deliverable, copy-pasteable into proposals.
3. **Annexe** — calculation breakdown (supporting material for internal review): A) monthly detail, B) initialization detail, C) cross-check deductive vs empirical.

The file must follow this structure:

```markdown
# Estimate — {Client Name}

**Date:** {date}
**Based on:** qualification.md ({date of qualification})
**TJM:** {amount}€
**Dispositif:** {Mutualisé / Semi-dédié / Dédié}

---

## Hypothèses de travail

{This section is MANDATORY. List every assumption made due to missing or incomplete information from qualification.md. If no assumptions were needed, write "Aucune — toutes les informations étaient disponibles dans la qualification."}

| # | Hypothèse | Information manquante | Valeur retenue | Justification | Impact si l'hypothèse est fausse |
|---|-----------|----------------------|----------------|---------------|----------------------------------|
| H1 | {e.g. "Taille du cluster K8s prod"} | {e.g. "Sizing exact non fourni"} | {e.g. "Coefficient 1 (medium)"} | {e.g. "Taille standard pour ce type de client"} | {e.g. "±500€/mois si coefficient réel est 0.5 ou 2"} |

> **⚠ Sensibilité** : {If any assumption has high impact, summarize here. E.g., "L'hypothèse H1 représente à elle seule ±15% du prix final. Il est recommandé de confirmer la taille du cluster avant contractualisation."}

---

## Analyse de réalisme

{This section presents the reality check from Step 4. Show the empirical signals, the calibrated estimate, and the comparison with the deductive.}

{Empirical signals table: ticket volume, incidents, recurring problems, etc.}

**Calibration empirique :**

| Composante | j/h/mois | Raisonnement |
|------------|----------|--------------|
| MCO réactif | {x} | {based on ticket volume} |
| MCO proactif | {y} | {multiplier of reactive, adjusted for complexity} |
| Buffer SLA | {z} | {based on SLA level and plage} |
| Gouvernance | {g} | {from abaques} |
| **Total calibré** | **{total}** | |

**Comparaison :**

| Approche | j/h/mois | Prix/mois | Écart |
|----------|----------|-----------|-------|
| Déductive pure | {X} | {price}€ | — |
| **Calibrée (retenue)** | **{Y}** | **{price}€** | {%} |

{Explanation of which estimate is used and why.}

---

## Synthèse

**Phase d'initialisation (one-shot, payée une seule fois en début d'engagement)**

Plateforme construite par Theodo : **{Oui / Non / Partiellement}**

- {Si Non/Partiellement} **Audit** : {audit_jh} j/h Lead Ops — cartographie ressources, qualité, résilience, sécurité, observabilité.
- {Si Non/Partiellement} **Remédiation prioritaire** (cible ROSE/YAMAS) : {remediation_jh} j/h — docs, durcissement résilience, gaps qualité.
- **Mise en place du monitoring** : {monitoring_jh} j/h — métriques, alerting, dashboards, sondes, runbooks.
- **Mise en place du système d'agents IA** : {ai_agent_jh} j/h — déploiement agent ChatOps, intégrations.

**Total initialisation : {total_init_jh} j/h — {total_init_price}€ HT (one-shot)**

> Cette enveloppe est payée une seule fois en début d'engagement. Elle est indépendante du prix mensuel récurrent ci-dessous.

---

This is the summary table for proposals and contracts. Each column is an environment category, plus a "Transverse" column for governance and evolutions.

|                         | {Env 1 category}                                                                                                                                                                                       | {Env 2 category}                                                     | {Env N category} | Transverse                                                                                                          |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------- | ---------------- | ------------------------------------------------------------------------------------------------------------------- |
| **Nom des envs**        | {env names, e.g. "prod"}                                                                                                                                                                               | {env names, e.g. "staging, dev"}                                     | {env names}      | {e.g. "master, hub"}                                                                                                |
| **Inventaire**          | **Servers :**{newline}{list servers with count, type, size}{newline}**Applications :**{newline}{list apps with count, type, complexity}                                                                | **Servers :**{newline}{...}{newline}**Applications :**{newline}{...} | {same format}    | {cross-cutting resources: organization, audit trail, IAM, CI/CD, etc.}                                              |
| **Services**            | Maintien : Gestion des demandes de service, des incidents, des problèmes, des changements de version, de la continuité{if plage != Standard: , de la surveillance de {N} sondes et astreintes}        | {same, adapted per env}                                              | {same}           | Gouvernance : COPIL trimestriel, Audit semestriel de Qualité{newline}Evolutions : Gestion des Changements mineurs   |
| **Niveaux de services** | {Bronze / Silver / Gold / Platine}                                                                                                                                                                     | {level}                                                              | {level}          | -                                                                                                                   |
| **Plages de service**   | {Standard / Etendue / Complète}                                                                                                                                                                        | {plage}                                                              | {plage}          | -                                                                                                                   |
| **Dispositif**          | Ops : {X} j/mois, Lead Ops : {Y} j/mois, Delivery Manager : {Z} j/mois                                                                                                                                |                                                                      |                  |                                                                                                                     |

#### Prix mensuel €HT

| Mode | Périmètre | j/h/mois | Montant €HT/mois |
|------|-----------|----------|-------------------|
| **Forfait** | MCO ({x} j/h) + Gouvernance ({y} j/h) + Contingence {z}% + Immobilisation | {x+y} | {amount}€ |
| **Temps passé** | Évolutions | {e} | {amount}€ |
| | **Total** | **{total}** | **{total amount}€** |

### Notes on the synthesis table:

- **Phase d'initialisation (one-shot)**: Block placed **above** the synthesis grid. Lists the four components (audit, remédiation, monitoring, agent IA) with their j/h, and shows a single one-shot total in €HT. Audit and remédiation lines are **omitted** if the platform was built by Theodo. This price is **never** added to the monthly recurring price.
- **Inventaire**: List resources in human-readable form. Group by "Servers" (K8s clusters, VMs, hypervisors, managed DBs, networking) and "Applications" (off-the-shelf and custom). Include count, name, and size (S/M/L or specific).
- **Services**: Describe what's included per environment. MCO goes in each env column. Governance and evolutions go in the "Transverse" column. Do NOT put the engagement model in the Services row — it is shown in the pricing table below.
- **Transverse column**: Contains cross-cutting resources (organization, IAM, audit trail) in inventory, and governance + evolutions in services. No SLA or plage — these are inherently cross-env.
- **Prix mensuel**: Separate table below the synthesis grid. Always show **two rows**: forfait (MCO + governance + contingency + immobilisation) and temps passé (evolutions). This makes clear which services are committed (forfait) vs consumed (temps passé). If the user chose "tout temps passé" (no forfait), show a single temps passé row covering everything — but still break down MCO+Governance vs Evolutions on separate lines for clarity.

---

## Annexe A — Detailed Calculation

### MCO Breakdown

#### Environment: {env_name}

**SLA:** {level} (coeff {x})
**Plage:** {plage}

| Resource     | Item Type | Base Rate | Coeff   | MCO (j/h/mois) |
| ------------ | --------- | --------- | ------- | -------------- |
| {name}       | {type}    | {rate}    | {coeff} | {result}       |
| **Subtotal** |           |           |         | **{sum}**      |

{Repeat for each environment}

#### MCO Summary

| Environment   | MCO (j/h/mois) | SLA Coeff | MCO ajusté  |
| ------------- | -------------- | --------- | ----------- |
| {env}         | {days}         | {coeff}   | {adjusted}  |
| **Total MCO** |                |           | **{total}** |

### Governance

| Activité | Fréquence | Effort/session | j/h/mois |
|----------|-----------|----------------|----------|
| COPROD   | {freq}    | 0.5 j/h        | {calc}   |
| COPIL    | {freq}    | 0.5 j/h        | {calc}   |
| ROSE     | semestriel | 1 j/h          | {calc}   |
| YAMAS    | semestriel | 1 j/h          | {calc}   |
| LEAF     | semestriel | 1 j/h          | {calc}   |
| **Total Gouvernance** | | | **{y} j/h/mois** |

### Evolutions

- Estimated: **{x} j/h/mois**
- Basis: {explanation}

### Total Quantity

| Category         | j/h/mois    |
| ---------------- | ----------- |
| MCO (ajusté SLA) | {x}         |
| Governance       | {y}         |
| Evolutions       | {z}         |
| **Total**        | **{total}** |

### Price Breakdown

| Line             | j/h/mois | TJM    | Montant       |
| ---------------- | -------- | ------ | ------------- |
| MCO (ajusté SLA) | {x}      | {tjm}€ | {amount}€     |
| Governance       | {y}      | {tjm}€ | {amount}€     |
| Evolutions       | {z}      | {tjm}€ | {amount}€     |
| **Subtotal**     |          |        | **{amount}€** |

| Immobilisation | {plage} × {dispositif} | {amount}€ |

{If forfait:}
| Contingency | {level} (+{x}%) on MCO+Governance | +{amount}€ |

{If multi-year:}
| Multi-year discount | {x} years | -{amount}€ |

| **Total mensuel** | | **{total}€** |
| **Total annuel** | | **{total × 12}€** |

---

## Annexe B — Initialisation (one-shot)

| Composante | Sizing | j/h | TJM | Montant €HT |
|------------|--------|-----|-----|-------------|
| Audit (Lead Ops) | {Skip / Small / Medium / Large} | {audit_jh} | {tjm_lead}€ | {amount}€ |
| Remédiation prioritaire | {Skip / Light / Medium / Heavy} | {remediation_jh} | {blended_tjm}€ | {amount}€ |
| Mise en place du monitoring | {Simple / Medium / Complex} | {monitoring_jh} | {blended_tjm}€ | {amount}€ |
| Mise en place système d'agents IA | {Simple / Medium / Complex} | {ai_agent_jh} | {blended_tjm}€ | {amount}€ |
| **Total initialisation** | | **{total_jh}** | | **{total_price}€** |

**Notes :**
- Audit et remédiation **omis** si la plateforme a été construite par Theodo.
- Audit facturé **TJM Lead Ops** (cf. `shared/daily-rates.md`).
- Remédiation, monitoring et système d'agents IA facturés au **TJM blended**.
- Cette enveloppe est **payée une seule fois** en début d'engagement et n'entre pas dans le prix mensuel récurrent.

---

## Annexe C — Cross-check: Deductive vs Empirical

{Include this annexe ONLY if empirical data is available in qualification.md. Omit entirely for deductive-only clients.}

| Category   | Deductive (j/h/mois) | Empirical (j/h/mois) | Delta     | Delta % |
| ---------- | -------------------- | -------------------- | --------- | ------- |
| MCO        | {x}                  | {a}                  | {x-a}     | {%}     |
| Governance | {y}                  | {b}                  | {y-b}     | {%}     |
| Evolutions | {z}                  | {c}                  | {z-c}     | {%}     |
| **Total**  | **{X}**              | **{A}**              | **{X-A}** | **{%}** |

**Analysis:** {Explanation of discrepancies. Flag if deductive > empirical by >20% (client may be understaffed) or empirical > deductive by >20% (inventory may be incomplete or inefficiencies exist).}

**Decision:** Pricing based on **{deductive/adjusted}** estimate. {Justification.}

---

## Notes

- {HDS applicability}
- {Nearshore flag if relevant}
- {Any other considerations}
```

---

## Verification

After generating `estimate.md`, spawn a verification subagent with the following instructions:

Use the Agent tool to spawn a subagent with `subagent_type: "general-purpose"` and the following prompt:

```
You are a verification agent for Infogérance Cloud estimates. Review the estimate.md file at {path_to_file} against the qualification.md file at {path_to_qualification}.

Read the reference files for the methodology:
- .claude/skills/shared/item-types.md
- .claude/skills/shared/coefficients.md
- .claude/skills/shared/service-levels.md
- .claude/skills/shared/pricing-rules.md

Check the following:

1. **Resource coverage:**
   - Is every resource from qualification.md accounted for in the estimate?
   - Are there resources in the estimate that don't appear in the qualification?

2. **Coefficient correctness:**
   - For each resource, does the item base rate match its type from shared/item-types.md?
   - Does the size/complexity coefficient match what was recorded in qualification.md?
   - Are the coefficients from the correct table in shared/coefficients.md?

3. **SLA & Plage consistency:**
   - Does the SLA coefficient used in pricing match the SLA level in qualification.md?
   - Does the plage horaire used for immobilisation match qualification.md?
   - Are non-prod environments using Bronze (1.00) unless explicitly specified otherwise?

4. **Business rules:**
   - Governance calculated from abaques in pricing-rules.md (COPROD + COPIL + audits)?
   - If forfait: contingency is applied ONLY to MCO + Governance, NOT evolutions?
   - Multi-year discount applied correctly (-3% for 2yr, -8% for 3yr+)?
   - Immobilisation matches the dispositif × plage from the pricing-rules table?
   - Dispositif matches the total j/h/mois thresholds (<10, 10-100, >100)?

5. **Cross-check (if empirical data available):**
   - Is the cross-check table present when qualification.md has empirical data?
   - Are the empirical numbers correctly extracted from qualification.md?
   - Is the delta % calculated correctly?
   - If delta > 20%: is there an analysis explaining the discrepancy?
   - Is the pricing decision clearly stated (which estimate is used and why)?

6. **Hypothèses de travail:**
   - Is the "Hypothèses de travail" section present?
   - Does every assumption trace back to a missing item from qualification.md's "Informations manquantes" section?
   - Are the assumed values conservative (slightly higher than the optimistic case)?
   - Is the impact column filled in for each assumption?
   - If any assumption has high price sensitivity, is the "Sensibilité" warning present?

7. **Outlier detection:**
   - Flag if total MCO seems unusually high or low for the infrastructure size
   - Flag if governance doesn't match expected abaques for the dispositif
   - Flag if evolution days seem disproportionate to the backlog described

8. **Initialization (one-shot):**
   - Is the "Phase d'initialisation" block present in the Synthèse, ABOVE the monthly table?
   - Is "Plateforme construite par Theodo" stated and consistent with qualification.md?
   - If platform NOT built by Theodo: are audit AND remediation lines present and priced?
   - If platform built by Theodo: are audit AND remediation lines absent (or marked "Skip", price = 0)?
   - Are monitoring AND AI agent lines ALWAYS present (regardless of who built the platform)?
   - Each component j/h within range: audit/monitoring/AI agent in [2.5, 20], remediation in {≈5, ≈15, ≈30+}?
   - Audit priced at **TJM Lead Ops**, others at **blended TJM**?
   - Initialization total is shown as a **one-shot** price and is **NOT** added to the monthly recurring price?
   - Annexe B (Initialisation) present with breakdown matching the synthesis?

Report your findings as:
- PASS: {check} — {brief explanation}
- WARN: {check} — {what seems off and why}
- FAIL: {check} — {what is missing or clearly wrong}

Do NOT redo the arithmetic from scratch. Focus on methodology correctness, consistency between qualification and estimate, and business rule compliance.
```

If the subagent reports any FAIL items, inform the user and offer to fix them. For WARN items, present them and let the user decide.
