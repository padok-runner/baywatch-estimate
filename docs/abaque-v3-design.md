# Abaque v3 — Structural Upgrade

**Status:** draft, not yet wired into `shared/` skills
**Author:** —
**Date:** 2026-05-25
**Supersedes:** sublinear-sqrt abaque (commit `e629d81`, 2026-05-11) for large/regulated/multi-tenant clients. Small single-tenant clients keep the same numbers.

---

## 1. Why v3 exists

The v2 abaque (`min(N,3) + sqrt(max(N/3,1)) - 1` per `(item, coeff)` bucket) models **marginal** per-resource effort against a zero baseline. On small single-tenant clients (Carenity, ANS, RDG, CISAC) it lands in the right zone. On large regulated multi-tenant clients (BIOGROUP) it under-prices by **~3×**:

| | v2 deductive | Realistic team need |
|---|---|---|
| Biogroup (1100+ VMs, 30 SELAS, HDS, 24/7) | 36.5 j/h/mois | 100–120 j/h/mois (1 SDM + 1 Lead Ops + 1 SecOps + 2 Ops + 1 Senior Ops) |

Five structural omissions account for the gap. v3 patches each as an **opt-in qualification field** so small clients are unaffected.

| Omission | v2 behaviour | v3 patch |
|---|---|---|
| Capability floor (minimum viable team) | scales from 0 | `capability_floor(plage, regulatory, tenancy_count)` |
| Specialization premium (SecOps, FinOps, K8s spec, HDS officer) | absent | declared `specializations[]` with own TJMs |
| Tenancy fragmentation (amortization breaks across tenants) | same multiplier whether N VMs sit in 1 tenant or 30 | bucket-level `tenants_spanned` → `sqrt(T) × m(N/sqrt(T))` |
| Year-1 stabilization ramp (migration-from-on-prem) | absent | declared `year_1_ramp` with end-date |
| Governance scaled by stakeholder count | base abaque only | declared `stakeholder_complexity` multiplier |

---

## 2. Formula evolution

### v2 (current)

```
For each (item_type, coeff) bucket:
  m(N) = min(N, 3) + sqrt(max(N/3, 1)) - 1
  MCO_bucket = base × m(N) × coeff
Total MCO = Σ buckets ( MCO_bucket × SLA-per-env distribution )
Total = MCO + Gouvernance + Évolutions
Price = Total × TJM + Immobilisation [+ Forfait contingency]
```

### v3

```
For each (item_type, coeff, tenants_spanned) bucket:
  T = tenants_spanned (default 1)
  m_T(N, T) = sqrt(T) × m(N / sqrt(T))    # tenancy-aware multiplier
  MCO_bucket = base × m_T(N, T) × coeff

MCO_marginal = Σ buckets ( MCO_bucket × SLA-per-env distribution )

MCO_after_floor = max( capability_floor(plage, regulatory, tenancy_count),
                       MCO_marginal )

MCO_after_ramp  = MCO_after_floor × year_1_ramp_multiplier

MCO_final = MCO_after_ramp + specialization_jh_total

Gouvernance_final = Gouvernance_base × stakeholder_complexity_multiplier

Total j/h = MCO_final + Gouvernance_final + Évolutions

Price = MCO_after_ramp × blended_TJM
      + Σ (spec_jh × spec_TJM)            # specialization at own TJM
      + Gouvernance_final × blended_TJM
      + Évolutions × blended_TJM
      + Immobilisation
      [+ Forfait contingency on Gouv only]
```

**Properties preserved.** Deterministic. Auditable. Every modifier traces to a qualification field. When `tenants_spanned = 1`, `year_1_ramp = none`, `specializations = []`, `stakeholder_complexity = low` (i.e., a small single-tenant client), v3 == v2.

**Properties added.** Capacity floor (you cannot deliver HDS + 24/7 + multi-tenant with a 0.5-FTE team), tenancy fragmentation, time-bounded ramp, specialist roles, ceremony scaling.

---

## 3. The five modifiers in detail

### 3.1 Capability floor

A baseline j/h/mois that captures the structural minimum to fulfill the contract — regardless of how few resources the marginal sum produces. Triggered **only** when the engagement is genuinely multi-tenant (`T ≥ 5`); single-tenant clients (Mutualisé pool of small clients, single-tenant Semi-dédié) are unaffected.

```
capability_floor(plage P, regulatory R, tenancy_count T):
  if T < 5:
    return 0    # marginal sum is enough

  base       = 10  if 5  ≤ T ≤ 9
              25  if 10 ≤ T ≤ 19
              50  if T ≥ 20

  hds_bonus  = 10  if HDS in R else 0
  secnum     = 5   if SecNumCloud in R else 0
  plage_bonus= 5   if P = Étendue
              10  if P = Complète
              0   else

  return base + hds_bonus + secnum + plage_bonus
```

| Client | T | P | R | Floor |
|---|---|---|---|---|
| Carenity | 1 | Complète | HDS | **0** |
| ANS | 1 | Standard | none | **0** |
| CISAC | 1 | Étendue | none | **0** |
| RDG | 1 | Complète | none | **0** |
| Biogroup | 30 | Complète | HDS | **70** (50 + 10 + 10) |

Floor only ever **raises** the MCO baseline; it does not reduce it. A client whose marginal sum is already 120 j/h is not capped — `max(floor=70, sum=120) = 120`.

### 3.2 Tenancy penalty (multi-tenant amortization break)

Per-bucket modifier. For each bucket, qualification declares `tenants_spanned` (default 1) — the number of distinct tenant boundaries the bucket's resources sit across. Within a tenant, amortization holds (same change window, same script, same comms). Across tenants, amortization breaks (T separate change tickets, T separate communications, T separate validations).

Mathematical formulation:
```
m_T(N, T) = sqrt(T) × m(N / sqrt(T))
where m(x) = min(x, 3) + sqrt(max(x/3, 1)) - 1
```

Interpretation: imagine the bucket split into `sqrt(T)` parallel sub-pools of size `N/sqrt(T)` each. Each sub-pool amortizes internally; coordination across sub-pools follows the same sqrt-saving (the team learns once, applies many times).

Properties:
- T=1 → `m_T(N,1) = m(N)` — unchanged for single-tenant clients
- T grows → effective multiplier grows; never collapses to linear because sqrt is applied twice (once on T, once on N/sqrt(T)).

Worked example — Biogroup "App VMs very small" bucket (N=615, coeff 0.5, T=30):

| | m(N) | bucket j/h |
|---|---|---|
| v2 | m(615) = 16.32 | 0.1 × 16.32 × 0.5 = **0.82** |
| v3 | sqrt(30) × m(615/sqrt(30)) = 5.48 × m(112.3) = 5.48 × (2 + sqrt(37.4)) = 5.48 × 8.12 = **44.5** | 0.1 × 44.5 × 0.5 = **2.22** |

Ratio v3 : v2 = 2.7× on this bucket.

For buckets that are inherently single-tenant (shared services, landing-zone, GKE control plane), `tenants_spanned = 1` is declared and the bucket behaves as v2.

### 3.3 Year-1 stabilization ramp

Migration-from-on-prem clients carry extra MCO during the first 12 months: building automation, training the team, writing runbooks from scratch, absorbing the long tail of legacy quirks. This is real but **time-bounded**.

Qualification declares `year_1_ramp` (with an explicit end-date or month-count):

| Value | Multiplier | When to use |
|---|---|---|
| `none` (default) | 1.00 | Greenfield or steady-state |
| `light_migration` | 1.20 | Lift-and-shift with mature IaC and minor reskilling |
| `migration` | 1.30 | Typical migration-from-on-prem |
| `heavy_migration` | 1.50 | Greenfield platform + workload migration + ramp (e.g., Biogroup) |

The multiplier applies to `MCO_after_floor`, not to gouvernance or évolutions. The estimate must record the **end date** of the ramp — the engagement contract should re-price at end-of-ramp.

| Client | Ramp |
|---|---|
| Carenity (live infra, no migration) | `none` |
| ANS (best-effort → first formal contract) | `none` (small enough, captured in init not ramp) |
| CISAC (greenfield post-Castelis, but stable platform) | `none` (greenfield, not migration-from-on-prem) |
| RDG (single new service add-on) | `none` |
| Biogroup (On-prem VMware → GCP souverain, 18-24 mo program) | **`heavy_migration` (1.50)** |

### 3.4 Specialization premium

Standard team composition (`daily-rates.md`) is **Ops : Lead Ops : DM = 1.00 : 0.34 : 0.16, blended TJM = 863€**. This is the right composition for the *core run*, but it has no place for SecOps, FinOps, K8s specialists, or HDS officers — roles that some engagements genuinely need.

v3 adds a `specializations[]` field to qualification. Each declared specialization adds a j/h/mois baseline AT ITS OWN TJM:

| Role | Default j/h/mois | TJM | When to declare |
|---|---|---|---|
| SecOps Lead | 5.0 | 1 400€ | HDS scope, critical-data engagements, regulated multi-tenant |
| FinOps Lead | 2.5 | 1 200€ | Multi-cloud, >20 VMs, explicit FinOps cadence (LEAF heavy) |
| K8s Specialist | 5.0 | 1 100€ | Managed K8s in scope with ≥10 nodes or multi-cluster |
| HDS Officer / Compliance Lead | 2.5 | 1 400€ | HDS scope with audit cadence > semestriel or multi-SELAS HDS |

Sizing is a default — `/qualify` may override with documented justification. Specializations are **added** to MCO_after_ramp; they are **not** affected by `tenancy_penalty` or `year_1_ramp` (they're already sized for the engagement profile).

| Client | Specializations |
|---|---|
| Carenity | none (HDS but small Mutualisé, shared SecOps capacity covered by the pool — declared on the Mutualisé team, not per client) |
| ANS | none |
| CISAC | none |
| RDG | none |
| Biogroup | SecOps Lead (5.0) + FinOps Lead (2.5) + K8s Spec (5.0) + HDS Officer (2.5) = **15.0 j/h/mois @ specialist TJMs** |

> Note for Mutualisé clients: specializations declared at the **team** level (not the client level) are absorbed in the blended TJM and not double-charged. v3 specializations are for engagements where the client's profile justifies dedicated specialist capacity.

### 3.5 Governance scaled by stakeholder complexity

The base governance abaque (`pricing-rules.md`) counts ceremony attendance only (COPROD 0.5 j/h × frequency, audits 0.5 j/h × 2/yr). For major accounts, real run-governance includes prep, follow-up, CAB, ITSM triage, postmortems, HDS comité, monthly reporting — none of which the v2 abaque captures.

Qualification declares `stakeholder_complexity`:

| Value | Multiplier | When to declare |
|---|---|---|
| `low` (default) | 1.0 | 1–5 stakeholders (typical single-app, single-team client) |
| `medium` | 1.5 | 6–15 stakeholders (multi-product, multi-team, or multi-app) |
| `high` | 2.0 | 16+ stakeholders (multi-SELAS, multi-BU, federated entity) |

Multiplier applies to **Gouvernance_base** (the abaque sum, including COPROD, COPIL, audits, allégement factors). It does NOT compound on top of dispositif-driven COPROD frequency (which is already an axis of differentiation).

| Client | Stakeholder count | Multiplier |
|---|---|---|
| Carenity | small dev team + PO + CTO | **low (1.0)** |
| ANS | ANS internal team | **low (1.0)** |
| CISAC | CISAC central + Castelis transition | **low (1.0)** |
| RDG | Bountiful Cow + RDG ops | **low (1.0)** |
| Biogroup | 30 SELAS + central IT + multiple comités | **high (2.0)** |

---

## 4. Worked examples — three reference profiles

### Profile A: Small single-tenant Mutualisé (Carenity-style)

**Qualification:**
- T = 1, HDS, Complète prod / Standard non-prod
- ramp = none
- specializations = []
- stakeholder_complexity = low

**v3 evaluation:**
- capability_floor = 0 (T < 5)
- m_T(N, T) = m(N) for every bucket (T=1)
- ramp = 1.0
- specialization_jh = 0
- gouvernance × 1.0

**Result:** v3 == v2 numerically. Carenity remains at 4.5 j/h MCO + 0.4 j/h gov.

### Profile B: Medium single-tenant Semi-dédié (CISAC-style)

**Qualification:**
- T = 1, no HDS, Étendue prod
- ramp = none (greenfield, not migration)
- specializations = [] (could declare K8s Spec if ≥10 nodes; CISAC has 0 K8s)
- stakeholder_complexity = low

**v3 evaluation:**
- capability_floor = 0 (T < 5)
- m_T = m (T=1 on every bucket)
- ramp = 1.0
- spec = 0
- gov × 1.0

**Result:** v3 == v2 numerically. CISAC re-baselined to its v2 sqrt-formula value (any "calibration discount" in the current estimate.md is pre-sqrt methodology and is the subject of a separate recompute pass, not of v3).

### Profile C: Large regulated multi-tenant Dédié (Biogroup-style)

**Qualification:**
- T = 30 (30 SELAS), HDS, Complète prod + shared / Standard non-prod
- ramp = heavy_migration (1.50, ends 2027-09-30 — 14 months from contract start 2026-07)
- specializations = [SecOps 5.0, FinOps 2.5, K8s 5.0, HDS Officer 2.5] = 15 j/h
- stakeholder_complexity = high (2.0)

**v3 step-by-step:**

```
Bucket-level tenancy penalty (only on per-SELAS buckets):
  - App VMs L&S (615+196 VMs, mostly per-SELAS): T=30
  - KaliSil PREMI3NS VMs (105): T=30 (1 per major SELAS group)
  - KaliSil BDD CRYPT3NS (7 substrate + 7 self-hosted): T=7 (consolidated)
  - KaliSil middlewares (30 custom): T=30 (1 per SELAS)
  - Shared services (Bastion, AD, GCBDR, etc.): T=1
  - GKE cluster (1): T=1

Recomputed MCO_marginal ≈ 60 j/h (vs 27.7 v2 — see Section 5 backtest for exact)

capability_floor(Complète, HDS, T=30) = 50 + 10 + 10 = 70

MCO_after_floor = max(70, 60) = 70

MCO_after_ramp = 70 × 1.50 = 105

MCO_final (incl spec) = 105 + 15 = 120 j/h equivalent
  (but specs are priced at own TJM — see Section 5 for €)

Gouvernance_base ≈ 0.75 j/h (semi-dédié abaque, no COPIL)
Gouvernance_final = 0.75 × 2.0 = 1.5 j/h

Évolutions = 8.0 (unchanged — declared backlog)

Total = 105 (MCO core) + 15 (spec) + 1.5 (gov) + 8.0 (evol) = 129.5 j/h
```

→ Pushes Biogroup from Semi-dédié to **Dédié** (>100 j/h/mois threshold). Realistic. Matches the team-size sanity check (1 SDM + 1 Lead Ops + 1 SecOps + 2 Ops + 1 Senior Ops ≈ 6 FTE = 120 j/h).

---

## 5. Backtest — five existing clients

See `docs/abaque-v3-backtest.md` for the full numerical recompute. Summary:

| Client | v2 (sqrt) total j/h/mois | v3 total j/h/mois | Δ% | Comment |
|---|---|---|---|---|
| Carenity | 4.9 | 4.9 | 0% | T=1, no migration, low stakeholders → v3 == v2 |
| ANS | ~11.9 (v2 recomputed) | ~11.9 | 0% | T=1, no migration, low stakeholders → v3 == v2 |
| CISAC | ~13.4 (v2 recomputed) | ~13.4 | 0% | T=1, no migration, no K8s → v3 == v2 |
| RDG | 2.0 | 2.0 | 0% | T=1, no migration, low stakeholders → v3 == v2 |
| Biogroup | 36.5 | **~133** | **+265%** | T=30 + HDS + Complète + heavy_migration + 4 specs + high stakeholders |

The four small clients are **identically priced** by v3 and v2 (constraint satisfied: small clients move <±15%; here they move 0%). Biogroup is corrected to its realistic team size.

Note: the comparison column "v2 (sqrt)" refers to v2 sqrt formula clean-recompute. For CISAC and ANS, current `estimate.md` uses pre-sqrt methodology (calibrated MCO + 3× multiplier) — that's an independent v1→v2 recompute pass. See backtest doc for both columns.

---

## 6. Qualification fields added

| Field | Type | Default | Used by |
|---|---|---|---|
| `tenancy_count` | int | 1 | capability_floor, per-bucket tenancy penalty |
| `year_1_ramp` | enum {none, light_migration, migration, heavy_migration} + end_date | none | ramp multiplier |
| `specializations` | list of {role, jh_per_month, tjm} | [] | specialization premium |
| `stakeholder_complexity` | enum {low, medium, high} | low | governance multiplier |
| `regulatory_profile` | enum {none, HDS, SecNumCloud, HDS+SecNumCloud} | (already implicit via HDS scope; v3 makes explicit) | capability_floor |
| Per-bucket `tenants_spanned` | int | 1 | bucket-level tenancy penalty (only when ≠ 1) |

All fields are **opt-in**. A qualification that doesn't declare them produces v2 behaviour.

---

## 7. Migration & ops impact

- **No retroactive repricing.** Existing client `estimate.md` files are not rewritten; clients renegotiate at contract anniversary.
- **Biogroup is the trigger case** but its `estimate.md` is being handled separately for the 2026-05-27 RFP deadline (manual override). After v3 ships, Biogroup can be recomputed cleanly.
- **Audit trail.** Every v3 modifier traces to a declared qualification field. The estimate must include a "v3 modifiers applied" section showing each value and its source.
- **Verifier checks.** New verifier sections enforce that (a) `tenancy_count` is declared, (b) `year_1_ramp` is consistent with migration context in qualification, (c) declared specializations match the regulatory/scale profile, (d) gov multiplier is consistent with stakeholder count.

---

## 8. Open questions (for review)

1. **Specialization defaults vs custom sizing.** Should SecOps default to 5.0 j/h or be sized to engagement? Proposed: 5.0 default, justified deviation in `/qualify`.
2. **Tenancy penalty on Mutualisé pools.** A Mutualisé team serves ~20 clients. Within that pool, each "client" is itself a tenant for the team. Does this mean Mutualisé tickets should carry a tenancy penalty? Proposed answer: no — the Mutualisé pool is structurally a multi-client model and its blended TJM already reflects this. The v3 tenancy penalty applies to **per-client multi-tenancy** (a single client with many internal tenants, e.g., 30 SELAS within Biogroup).
3. **Year-1 ramp expiry.** Should the ramp decay (linearly over 12 mo) or step (1.5× for 12 mo, then 1.0×)? Proposed: step, simpler to enforce and document. Decay can be approximated by a second ramp at lower multiplier in year 2 if needed.
4. **Stakeholder count vs SELAS count.** For Biogroup these align. For other federated entities they may not. The qualification asks for stakeholder_complexity bucket; the estimate must document the count that drove the bucket selection.
