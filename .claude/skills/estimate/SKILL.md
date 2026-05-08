---
name: estimate
description: Estimation & Pricing for Infogérance Cloud — calculates man-days/month and computes the final price from qualification data
user_invocable: true
---

# /estimate — Estimation & Pricing

You are a Solutions Architect assistant computing the Infogérance Cloud estimate and price. Read `qualification.md` and produce `estimate.md`.

## Prerequisites

Locate and read `qualification.md` for the client. If it doesn't exist, tell the user to run `/qualify` first.

Read these references (each is short and load-bearing):

- `skills/shared/item-types.md` — item types and MCO base rates per resource type
- `skills/shared/coefficients.md` — size/complexity coefficients
- `skills/shared/service-levels.md` — plage horaire, SLA coefficients, immobilisation
- `skills/shared/pricing-rules.md` — engagement modes, discounts, governance abaques
- `skills/shared/daily-rates.md` — TJMs and team composition
- `skills/shared/services.md` — service catalogue for the synthesis table
- `skills/shared/initialization.md` — one-shot init phase

For the output structure, read `references/output-template.md`.

## Conventions de précision

Tous les j/h/mois sont exprimés au **dixième de jour** près (ex. 0.4, 1.2, 2.1). N'arrondissez jamais au demi-jour ou au jour entier — chaque arrondi unitaire ajoute 100 à 800€/mois sans justification.

- Calculs intermédiaires : 2 décimales (ex. `5/12 = 0.42`, `0.42 × 3 = 1.25`).
- Tableaux de synthèse : 1 décimale.
- Total final : somme précise des composantes, arrondie une seule fois au dixième.

## Methodology — single anchor + explicit discount

The price has **one rigorous calculation** (the deductive abaque) and **one explicit empirical adjustment** (a discount). There is no parallel heuristic. Going off this rail is what produced the methodological drift this skill saw historically.

```
Final MCO j/h/mois = Deductive MCO × (1 − discount)
Final total j/h/mois = Final MCO + Gouvernance + Évolutions
Final price = Final j/h × TJM × SLA_per_env + Immobilisation [+ Forfait contingency]
```

Governance, SLA coefficient, and immobilisation are **never discounted** — they're contractual or platform-level, not effort-driven.

---

## Phase A — Quantity

### Step 0: Working assumptions (Hypothèses de travail)

Read the **"Informations manquantes"** section from `qualification.md`. For each missing item, define a working assumption that's:

- **Stated** — no ambiguity about the value chosen
- **Conservative** — when in doubt, slightly higher complexity/size
- **Justified** — one-line reason
- **Flagged for impact** — note if the assumption could move the final price meaningfully

These go in the "Hypothèses de travail" section of the output.

### Step 1: Deductive MCO (the only computational baseline)

For each resource in each environment from `qualification.md`:

```
MCO per resource = item_base_rate × size_complexity_coefficient
```

Use rates from `shared/item-types.md` and coefficients from `shared/coefficients.md`. When a resource has both a server size and an application complexity assessment, use the **higher** of the two coefficients.

Sum per environment, then across environments. Apply the SLA coefficient per environment (Bronze 1.00, Silver 1.05, Gold 1.10, Platine 1.20) at this point — see `shared/service-levels.md`.

```
Deductive MCO total = Σ (Σ resource_MCO × SLA_coeff_env) over environments
```

**Governance:** compute from the abaques in `shared/pricing-rules.md` (COPROD per dispositif + COPIL if dédié + audits ROSE/YAMAS/LEAF). Convert each to j/h/mois: `effort_per_session × sessions_per_year / 12`.

**Évolutions:** estimate from the evolution backlog in `qualification.md`. If the user hasn't provided enough detail, ask them.

### Step 2: Empirical signals (always observed, never recomputed)

Pull the empirical signals from `qualification.md`:

| Signal | Where in qualification.md |
|---|---|
| Ticket volume (12 months) | "Empirical Data" → ticket history |
| Incident count + recurring problems | "Empirical Data" → ticket breakdown |
| FTE breakdown (MCO / governance / evolutions) | "Empirical Data" → Current FTEs |
| Known inefficiencies / gaps | "Empirical Data" → gaps |

If the qualification has FTE breakdown, compute the **empirical estimate** as a check :

```
Empirical MCO = MCO_FTE × 20 j/h/mois
Empirical total = sum of (FTE × 20) for MCO + governance + evolutions
```

This number is for **calibration only** — it informs the discount in Step 3, but is **never used directly as the price**.

### Step 3: Empirical discount on the deductive MCO

Pick a single discount based on the strongest signal observed. Document the row of the table that justified it.

**Discount table:**

| Signal | Discount on deductive MCO |
|---|---|
| Empirical FTE available, deductive within ±20% of FTE-derived estimate | **0%** — deductive holds, mention that FTE confirms |
| Empirical FTE available, FTE < deductive by >20% | discount = `1 − (FTE / deductive)`, capped at **−50%** |
| Empirical FTE available, FTE > deductive by >20% | **0%** — do NOT discount up; flag inventory gap and ask the SA to investigate |
| No FTE; tickets < 1/month over 12 months; no recurring problems | **−30% to −50%** (infra ultra-stable) |
| No FTE; 1–5 tickets/month, no recurring problems | **0% to −20%** (infra normale) |
| No FTE; tickets > 5/month, or recurring problems present | **0%** — deductive holds; investigate inventory completeness |

**Hard cap : −50%.** A discount beyond −50% requires explicit SA justification and stakeholder review (the deductive abaque is calibrated against real client data — a 2× drop suggests either an inventory gap or aggressive scoping that the client should approve).

**Governance is never discounted.** Audits and COPRODs are contractual obligations, not effort-driven. Apply the discount only to MCO.

**Document in `estimate.md`** under "Calibration empirique" :

```markdown
**Signaux empiriques observés :**
| Signal | Valeur |
|---|---|
| Tickets sur 12 mois | {N} → {N/12}/mois |
| Incidents | {N} |
| Problèmes récurrents | {N} |
| FTE empirique (si disponible) | {N} j/h/mois |

**Discount appliqué :** {−X%}
**Justification :** {row of the table that matched, plus context}

**Ajustement :**
| | Déductive | Discount | Final |
|---|---|---|---|
| MCO | {X} | −X% | {X × (1−d)} |
| Gouvernance | {Y} | (jamais discountée) | {Y} |
| **Total** | **{X+Y}** | | **{final}** |
```

### Step 4: Final total & dispositif

```
Total j/h/mois = Final MCO + Gouvernance + Évolutions
```

Determine the dispositif using the thresholds in `shared/pricing-rules.md` (<10 mutualisé, 10–100 semi-dédié, >100 dédié).

### Step 5: Initialization (one-shot)

Read the "Phase d'initialisation (one-shot)" section from `qualification.md`. The j/h are already resolved there. Compute the one-shot price using `shared/initialization.md`:

- Audit (if not Theodo-built) — TJM Lead Ops
- Remédiation (if not Theodo-built) — TJM blended
- Monitoring — TJM blended
- Agent IA — TJM blended

The init price is **paid once** and **separate from the recurring monthly price** — never folded in.

---

## Phase B — Pricing

### Step 6: Base price

TJM is the blended TJM from `shared/daily-rates.md` unless the user specifies otherwise.

```
MCO price = Final MCO × TJM
Governance price = Governance × TJM
Evolution price = Évolutions × TJM
```

Note: the SLA coefficient was already applied per environment in Step 1 when computing the deductive MCO. Don't apply it again here.

### Step 7: Immobilisation

From `shared/service-levels.md`, dispositif × plage horaire. If multiple plages, use the highest.

For Étendue/Complète, also note the prix horaire HNO (heures non ouvrées).

### Step 8: Engagement model

**Temps passé (default):** Price = MCO + Governance + Evolutions + Immobilisation.

**Forfait:** add contingency to MCO + Governance only (not evolutions):
- No uncertainty: 0%
- Low: +10%
- Medium: +20%
- High: +30 to 40%

```
Forfait price = (MCO + Governance) × (1 + contingency) + Evolutions + Immobilisation
```

### Step 9: Multi-year discounts & nearshore

- Multi-year: see `shared/pricing-rules.md` (-3% for 2 years, -8% for 3+).
- HDS: default in France — flag if applicable.
- Nearshore: do NOT calculate; flag for discussion with Hugo / Lila / Manu.

---

## Output

Write `estimate.md` in the client's directory. Follow the structure in `references/output-template.md` exactly.

The file has three parts:
1. **Hypothèses de travail** — assumptions made for missing info
2. **Calibration empirique** — discount justification (Step 3)
3. **Synthèse** — client-facing summary with init block + monthly grid + price table
4. **Annexes** — calculation detail (A: monthly, B: initialization)

---

## Verification

After generating `estimate.md`, spawn a verification subagent. Use the Agent tool with `subagent_type: "general-purpose"` and load the prompt from `agents/verifier.md`. If the verifier returns FAIL items, inform the user and offer to fix them. WARN items: present and let the user decide.
