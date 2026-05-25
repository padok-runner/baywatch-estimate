---
name: estimate
description: Estimation & Pricing for Infogérance Cloud — calculates man-days/month and computes the final price from qualification data
user_invocable: true
---

# /estimate — Estimation & Pricing

You are a Solutions Architect assistant computing the Infogérance Cloud estimate and price. Read `qualification.md` and produce `estimate.md`.

## Prerequisites

Locate and read `qualification.md`. If it doesn't exist, tell the user to run `/qualify` first.

Read these references (each is short and load-bearing):

- `skills/shared/item-types.md` — substrate vs application item types, base rates per type, and the **sublinear scaling formula** for N ressources of the same type/coeff
- `skills/shared/coefficients.md` — size/complexity coefficients
- `skills/shared/service-levels.md` — plage horaire, SLA coefficients, immobilisation
- `skills/shared/pricing-rules.md` — engagement modes, multi-year discounts, governance abaques
- `skills/shared/daily-rates.md` — TJMs and team composition
- `skills/shared/services.md` — service catalogue for the synthesis table
- `skills/shared/initialization.md` — one-shot init phase

For the output structure, read `references/output-template.md`.

## Conventions de précision

Tous les j/h/mois sont exprimés au **dixième de jour** près (ex. 0.4, 1.2, 2.1). N'arrondissez jamais au demi-jour ou au jour entier.

- Calculs intermédiaires : 2 décimales (ex. `multiplier(11) = 3.915`).
- Tableaux de synthèse : 1 décimale.
- Total final : somme précise des composantes, arrondie une seule fois au dixième.

## Methodology

The price is built from one rigorous calculation. There is no parallel heuristic, no discount knob. **v3** (2026-05) adds five structural modifiers on top of the v2 sqrt formula — each opt-in via qualification fields, each tracing to an explicit value. For a small single-tenant client with no migration and no specializations, v3 == v2 numerically.

```
For each (item type, coefficient, tenants_spanned T) bucket :
  m_T(N, T) = sqrt(T) × multiplier(N / sqrt(T))         # v3, T=1 by default
  MCO_bucket = base_rate × m_T(N, T) × coefficient
  where multiplier(x) = min(x, 3) + sqrt(max(x/3, 1)) - 1

Distribute MCO_bucket across envs prorata of count, apply SLA per env.
Sum across all buckets and envs → MCO_marginal.

MCO_after_floor = max( capability_floor(plage, regulatory, T), MCO_marginal )
MCO_after_ramp  = MCO_after_floor × year_1_ramp_multiplier
MCO_final_jh    = MCO_after_ramp + Σ specialization_jh_per_role
                   (specialists billed at own TJM, not blended)

Gouvernance_final = Gouvernance_base × stakeholder_complexity_multiplier

Total monthly j/h = MCO_final_jh + Gouvernance_final + Évolutions

Price =
    MCO_after_ramp × TJM_blended
  + Σ (specialization_jh × TJM_specialist)
  + Gouvernance_final × TJM_blended
  + Évolutions × TJM_blended
  + Immobilisation
  [+ Forfait contingency on Gouvernance only]
  [× (1 − multi_year_discount) on services, excluding immobilisation]
```

See `shared/pricing-rules.md` ("Modificateurs 1–5") and `shared/item-types.md` ("Pénalité de fragmentation multi-tenants") for the formal definitions of each modifier.

The sublinear scaling and tenancy penalty are **inside the abaque**, not applied afterward as discounts. Identical-profile ressources at scale (e.g., 11 EC2 Debian) cost less to operate per unit than isolated ressources because automation amortizes — captured by `multiplier(N)`. When those ressources are fragmented across tenants, amortization breaks — captured by `m_T(N, T) = sqrt(T) × m(N/sqrt(T))`. No separate "calibration" or "discount" is permitted.

Governance, SLA coefficients, and immobilisation are calculated independently and cumulated into the total.

---

## Phase A — Quantity

### Step 0: Working assumptions (Hypothèses de travail)

Read the **"Informations manquantes"** section from `qualification.md`. For each missing item, define a working assumption:

- **Stated** — no ambiguity about the value chosen
- **Conservative** — when in doubt, slightly higher complexity/size
- **Justified** — one-line reason
- **Flagged for impact** — note if the assumption could move the final price meaningfully

These go in the "Hypothèses de travail" section of the output.

### Step 1: Group ressources by (item type, coefficient, tenants_spanned)

Read the resource inventory from `qualification.md`. For each ressource, identify:

- **Item type** — from the substrate or application taxonomy in `shared/item-types.md`. Distinguish carefully: managed K8s vs self-hosted K8s, managed off-the-shelf (RDS, ElastiCache) vs self-hosted off-the-shelf (MySQL on VM, Postgres on VM), etc.
- **Coefficient** — from `shared/coefficients.md`. Pick the higher of (server size, application complexity).
- **Tenants spanned (v3)** — from the qualification's "Tenants spanned" column. Default 1. Different values within the same `(item, coeff)` produce **distinct buckets** (e.g., 480 prod VMs across 30 SELAS = one bucket with T=30; 121 consolidated non-prod VMs of the same type/coeff = a separate bucket with T=1).

Group ressources globally (across all envs) by `(item type, coefficient, tenants_spanned)`.

For each group, compute the **tenancy-aware multiplier** and the MCO base:

```
m_T(N, T) = sqrt(T) × multiplier(N / sqrt(T))
MCO_bucket = base_rate × m_T(N, T) × coefficient
```

This is the **MCO base** for that bucket, before SLA. For single-tenant buckets (`T=1`), `m_T = multiplier(N)` — identical to v2.

### Step 2: Distribute per env and apply SLA → MCO_marginal

For each bucket, distribute the MCO base across environments **prorata of ressource count**. Apply the SLA coefficient per environment (Bronze 1.00, Silver 1.05, Gold 1.10, Platine 1.20).

```
MCO_marginal = Σ buckets ( Σ envs ( MCO_bucket × (count_env / count_total_bucket) × SLA_coeff_env ) )
```

The total `MCO_marginal` is the sum of these env-and-SLA-adjusted contributions across all buckets.

### Step 2b: Apply capability floor (v3)

Read `tenancy_count`, `regulatory_profile`, and the max `plage horaire` from qualification's "v3 Modificateurs structurels" section. Compute the capability floor per `shared/pricing-rules.md` ("Modificateur 1") :

```
floor = capability_floor(plage, regulatory, tenancy_count)
MCO_after_floor = max(floor, MCO_marginal)
```

For single-tenant (`T<5`), floor = 0 and `MCO_after_floor = MCO_marginal` (no effect).

If the floor activates (i.e., `floor > MCO_marginal`), the estimate must call this out explicitly in the breakdown — show both values and the resulting `MCO_after_floor`.

### Step 2c: Apply year-1 ramp (v3)

Read `year_1_ramp` from qualification.

```
MCO_after_ramp = MCO_after_floor × year_1_ramp_multiplier
```

| Value | Multiplier |
|---|---|
| `none` | 1.00 |
| `light_migration` | 1.20 |
| `migration` | 1.30 |
| `heavy_migration` | 1.50 |

Record the **end date** of the ramp in the estimate notes. The contract should re-price at end-of-ramp.

### Step 2d: Add specialization premium (v3)

Read `specializations[]` from qualification. For each declared role, compute the j/h premium:

```
specialization_jh_total = Σ over declared roles (role.jh_per_month)
MCO_final_jh = MCO_after_ramp + specialization_jh_total
```

Specialist j/h are billed at **their own TJM** (see `shared/daily-rates.md` → Specialization roles), **not** at the blended TJM. They are added to `MCO_after_ramp`; they do **not** receive the tenancy penalty nor the year-1 ramp multiplier (already sized for the engagement profile).

Empty `specializations[]` → no premium added.

### Step 3: Governance and Evolutions

**Governance base:** compute from the abaques in `shared/pricing-rules.md` — COPROD per dispositif + COPIL if dédié + audits ROSE/YAMAS/LEAF. Convert each to j/h/mois: `effort_per_session × sessions_per_year / 12`. The result is `Gouvernance_base`.

**Stakeholder multiplier (v3):** read `stakeholder_complexity` from qualification.

```
Gouvernance_final = Gouvernance_base × stakeholder_complexity_multiplier
```

| Value | Multiplier |
|---|---|
| `low` (default) | 1.0 |
| `medium` | 1.5 |
| `high` | 2.0 |

The multiplier scales the entire `Gouvernance_base` (including COPROD frequency from dispositif and audit cadence). It does **not** stack on top of dispositif's COPROD frequency (which is already an axis of differentiation — Dédié already has weekly COPROD).

**Évolutions:** estimate from the evolution backlog in `qualification.md`. If unclear, ask the user. Évolutions are **not** affected by v3 modifiers.

### Step 4: Empirical cross-check (sanity, no adjustment)

If `qualification.md` provides FTE breakdown, compute :
```
Empirical estimate = (MCO_FTE + governance_FTE + evolutions_FTE) × 20 j/h/mois
```

Compare with the deductive total. **No adjustment** — just a sanity check :

- Within ±20% : deductive holds, FTE confirms. Document the alignment in `estimate.md`.
- Deductive > FTE by >20% : flag in the report. Either client is currently understaffed, or the inventory has hidden non-MCO scope. Investigate — don't silently discount.
- FTE > Deductive by >20% : flag in the report. Either inventory is incomplete, or the client has hidden complexity. Investigate.

If qualification has no FTE data, also compare deductive against simple stability signals (ticket volume, incident count). Flag — don't adjust the number.

### Step 5: Final total & dispositif

```
Total j/h/mois = MCO_final_jh + Gouvernance_final + Évolutions
```

Determine the dispositif using thresholds in `shared/pricing-rules.md` (<10 mutualisé, 10–100 semi-dédié, >100 dédié).

> **v3 dispositif cascade.** When v3 modifiers (capability floor, year-1 ramp, specializations) push the total above 100 j/h, the dispositif upgrades to **Dédié**. This in turn changes COPROD frequency (mensuel → weekly) and adds COPIL trimestriel. Procedure: (1) compute a provisional Total with the dispositif inferred from `MCO_marginal + Gouvernance_base + Évolutions`; (2) if Total > 100 j/h, upgrade dispositif to Dédié and **recompute `Gouvernance_base`** for the upgraded dispositif (replacing the previous `Gouvernance_base`, not adding to it); (3) apply the `stakeholder_complexity_multiplier` **exactly once** to the recomputed `Gouvernance_base` to obtain `Gouvernance_final`. Iterate until the dispositif is stable (typically one pass suffices).

### Step 6: Initialization (one-shot)

Read the "Phase d'initialisation (one-shot)" section from `qualification.md`. The j/h are already resolved there. Compute the one-shot price using `shared/initialization.md`:

- Audit (if not Theodo-built) — TJM Lead Ops
- Remédiation (if not Theodo-built) — TJM blended
- Monitoring — TJM blended
- Agent IA — TJM blended

Init is **paid once** and **separate from the recurring monthly price** — never folded in.

---

## Phase B — Pricing

### Step 7: Base price

TJM blended from `shared/daily-rates.md` for the core run; specialization TJMs per role (see `daily-rates.md` → Specialization roles).

```
MCO core price       = MCO_after_ramp × TJM_blended
Specialization price = Σ (role.jh_per_month × role.TJM)
Governance price     = Gouvernance_final × TJM_blended
Évolutions price     = Évolutions × TJM_blended
```

Note: SLA was already applied per env in Step 2. Don't apply it again here.

Specialization premium uses **per-role TJM** — do NOT use the blended TJM for specialists.

### Step 8: Immobilisation

From `shared/service-levels.md`, dispositif × plage horaire. If multiple plages, use the highest. For Étendue/Complète, also note the prix horaire HNO.

### Step 9: Engagement model

Two billing modes (`shared/pricing-rules.md`):
- **Forfait** : billed in full on the envelope.
- **Temps passé** : billed on actual consumption.

**Configuration par défaut : Forfait socle + carnet temps passé** (engagement ≥2 ans requis) :
- **Forfait socle** : Gouvernance + Audits + Immobilisation
- **Temps passé** : MCO (toutes catégories) + Évolutions
- Pas de plancher, pas de plafond. Le socle (immobilisation + gouvernance) et l'engagement multi-annuel forment la protection.

```
Forfait socle      = Gouv × (1 + contingency) + Immobilisation
Temps passé MCO    = conso_réelle × TJM blended
Total mensuel      = Forfait socle + Temps passé MCO + Évolutions consommées
```

Contingence Forfait socle (sur Gouv uniquement, jamais sur évolutions) :
- No uncertainty: 0% / Low: +10% / Medium: +20% / High: +30 to 40%

L'enveloppe MCO déductive (calculée par `/estimate`) sert de référence de **dimensionnement** capacitaire et de pédagogie client (« voici ce que tu paierais en forfait classique »), mais n'est pas un plafond contractuel.

**Cas particuliers** (à mentionner uniquement si pertinent, pas par défaut) :
- **Forfait classique** (engagement <2 ans ou prédictibilité absolue exigée) : tout dans le forfait (MCO + Gouv + Immo), évolutions en temps passé. Repli si engagement multi-annuel impossible.
- **Temps passé pur** : très rare, PoC / audit-only seulement.

### Step 10: Multi-year discounts & nearshore

- Multi-year: see `shared/pricing-rules.md` (-3% for 2 years, -8% for 3+).
- HDS: default in France — flag if applicable.
- Nearshore: do NOT calculate; flag for discussion with Hugo / Lila / Manu.

---

## Output

Write `estimate.md` in the client's directory. Follow the structure in `references/output-template.md`.

The file has four parts:
1. **Hypothèses de travail** — assumptions for missing info
2. **Cross-check empirique** — flag discrepancies if any (Step 4)
3. **Synthèse** — client-facing summary (init block + monthly grid + price table)
4. **Annexes** — calculation detail (A: monthly with bucket-by-bucket breakdown, B: initialization)

---

## Verification

After generating `estimate.md`, spawn a verification subagent. Use the Agent tool with `subagent_type: "general-purpose"` and load the prompt from `agents/verifier.md`. If FAIL items, inform the user and offer to fix. WARN items: surface and let the user decide.
