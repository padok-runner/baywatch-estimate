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

- Calculs intermédiaires : 2 décimales (ex. `scale(11) = 11^0.8 = 6.81`).
- Tableaux de synthèse : 1 décimale.
- Total final : somme précise des composantes, arrondie une seule fois au dixième.

## Methodology

The price is built from one rigorous calculation. There is no parallel heuristic, no discount knob. **v3** (2026-06) replaces the v2 piecewise-sqrt core with a single **power-law** curve `scale(N, T) = T^(1−k) × N^k` (k ≈ 0.8), summed per tenant so fleet economy-of-scale and multi-tenant fragmentation come from the same exponent. Three opt-in modifiers (year-1 ramp, specializations, stakeholder governance) apply on top, each tracing to a declared qualification field. This re-prices every fleet size vs v2 (deliberately — v2 under-priced large fleets); it is **not** v2-compatible.

```
For each (item type, coefficient, tenants_spanned T) bucket :
  scale(N, T) = Σ_tenant N_t^k = T^(1−k) × N^k         # k ≈ 0.8, T=1 by default
  MCO_bucket = base_rate × scale(N, T) × coefficient

Distribute MCO_bucket across envs prorata of count, apply SLA per env.
Sum across all buckets and envs → MCO.

MCO_after_ramp = MCO × year_1_ramp_multiplier
MCO_final_jh   = MCO_after_ramp + Σ specialization_jh_per_role
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

See `shared/pricing-rules.md` ("Modificateurs 1–3") and `shared/item-types.md` ("Scaling sublinéaire (loi de puissance)") for the formal definitions.

The power-law scaling and the per-tenant fragmentation are **inside the abaque**, not applied afterward as discounts. Identical-profile ressources at scale (e.g., 11 EC2 Debian) cost less to operate per unit than isolated ressources because automation amortizes — captured by `N^k`. When those ressources are fragmented across tenants, amortization partially breaks — captured by summing `N_t^k` per tenant (a `× T^(1−k)` factor). No separate "calibration" or "discount" is permitted.

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

For each group, compute the **power-law scale** and the MCO base:

```
scale(N, T) = T^(1−k) × N^k          # k ≈ 0.8 ; exact form Σ_tenant N_t^k if tenants are uneven
MCO_bucket = base_rate × scale(N, T) × coefficient
```

This is the **MCO base** for that bucket, before SLA. For single-tenant buckets (`T=1`), `scale = N^k`.

### Step 2: Distribute per env and apply SLA → MCO

For each bucket, distribute the MCO base across environments **prorata of ressource count**. Apply the SLA coefficient per environment (Bronze 1.00, Silver 1.05, Gold 1.10, Platine 1.20).

```
MCO = Σ buckets ( Σ envs ( MCO_bucket × (count_env / count_total_bucket) × SLA_coeff_env ) )
```

The total `MCO` is the sum of these env-and-SLA-adjusted contributions across all buckets.

### Step 2b: Apply year-1 ramp (v3)

Read `year_1_ramp` from qualification.

```
MCO_after_ramp = MCO × year_1_ramp_multiplier
```

| Value | Multiplier |
|---|---|
| `none` | 1.00 |
| `light_migration` | 1.20 |
| `migration` | 1.30 |
| `heavy_migration` | 1.50 |

Record the **end date** of the ramp in the estimate notes. The contract should re-price at end-of-ramp.

### Step 2c: Add specialization premium (v3)

Read `specializations[]` from qualification. For each declared role, compute the j/h premium:

```
specialization_jh_total = Σ over declared roles (role.jh_per_month)
MCO_final_jh = MCO_after_ramp + specialization_jh_total
```

Specialist j/h are billed at **their own TJM** (see `shared/daily-rates.md` → Specialization roles), **not** at the blended TJM. They are added to `MCO_after_ramp`; they do **not** receive the core scaling nor the year-1 ramp multiplier (already sized for the engagement profile).

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

> **v3 dispositif cascade.** The dispositif threshold is tested against the **post-modifier** Total `MCO_final_jh + Gouvernance_final + Évolutions` — i.e. after ramp and specializations, not on the raw bucket sum. When these push the Total above 100 j/h, the dispositif upgrades to **Dédié**, which changes COPROD frequency (mensuel → weekly) and adds COPIL trimestriel. Because `Gouvernance_base` itself depends on the dispositif, resolve the circularity by iterating: (1) compute the Total with the dispositif inferred from the current `Gouvernance_base`; (2) if Total > 100 j/h, set dispositif = Dédié and **recompute `Gouvernance_base`** for Dédié (replacing it, not adding); (3) re-apply `stakeholder_complexity_multiplier` **exactly once** to obtain `Gouvernance_final`, and recompute the Total. Iterate until the dispositif is stable (one pass usually suffices).

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
