# Abaque v3 — Migration notes per client

**Date:** 2026-05-25
**Companion to:** [`abaque-v3-design.md`](abaque-v3-design.md), [`abaque-v3-backtest.md`](abaque-v3-backtest.md)

This document records, per existing client, **which v3 modifier(s) moved the deductive price and why**, and what recompute pass (if any) is owed.

> **Important.** v3 does **not** retroactively rewrite existing `estimate.md` files. The migration table below states what `/qualify` + `/estimate` would produce **today** for each client if re-run. Existing contracts are unchanged until anniversary repricing.

---

## Carenity — no v3 impact

**Current estimate value:** 4.9 j/h/mois total (5 229€/mois Forfait classique equivalent).

**v3 modifiers triggered:** none.

**Why unchanged:**
- `tenancy_count = 1` (single-tenant infogérance LAMP).
- `year_1_ramp = none` (live infrastructure, not migration-from-on-prem).
- `specializations = []` — small Mutualisé client; SecOps capacity is mutualised at the pool level (not declared per client per the v3 Mutualisé rule).
- `stakeholder_complexity = low` (2 devs + 1 PO + 1 CTO).
- `regulatory_profile = HDS` — but with T=1 the floor remains 0.

**Recompute pass owed:** No. v3 produces the same number as v2-as-shipped.

**Action:** none.

---

## RDG-redirect — no v3 impact

**Current estimate value:** 2.0 j/h/mois total (2 737€/mois Forfait).

**v3 modifiers triggered:** none.

**Why unchanged:**
- `tenancy_count = 1` (single new add-on service on top of existing RDG contract).
- `year_1_ramp = none` (greenfield new service, no migration).
- `specializations = []` — small Mutualisé add-on; specialists provisioned at the existing-contract level.
- `stakeholder_complexity = low` (Bountiful Cow + RDG ops).
- `regulatory_profile = none`.

**Recompute pass owed:** No.

**Action:** none.

> Note: the v2-clean recompute revealed a minor +14% drift (1.96 j/h MCO vs 1.68 shipped) due to the NGINX/OpenResty classification choice (self-hosted vs deprecated `base 0.5`). This is within ±15% and is a pre-existing methodology choice, **not** a v3 effect. No action required unless the client renegotiates.

---

## CISAC — no v3 impact (but separate v1→v2 recompute owed)

**Current estimate value:** 19.17 j/h/mois total calibrated (16 895€/mois Étendue) — methodology uses pre-sqrt calibrated MCO + 3× multiplier + Buffer SLA (deprecated).

**v3 modifiers triggered:** none.

**Why unchanged by v3:**
- `tenancy_count = 1` (single Azure tenant).
- `year_1_ramp = none` (greenfield post-Castelis transition; the platform itself is new but it's not a migration-from-on-prem to the new managed contract — it's a fresh prod cutover).
- `specializations = []` — no managed K8s, no HDS, no multi-cloud cadence.
- `stakeholder_complexity = low`.
- `regulatory_profile = none`.

**Recompute pass owed:** **Yes — independent v1 → v2 recompute** owed. The current estimate uses the deprecated calibrated/3×/Buffer methodology; a clean v2 recompute (per current `item-types.md` with 0.3 / 0.6 self-hosted split and sqrt scaling) lands at ~13.4 j/h/mois total — **a −30% drop vs the calibrated price**, and a **−52% drop vs the deductive baseline shown in the current estimate**. This drift is separate from v3 and should be handled in a dedicated pass coordinated with the client (CISAC contract is engagement 1 year initial).

**Action:** flag for v1→v2 recompute at next CISAC contract review.

---

## ANS / EvalCarbone SIH — no v3 impact (but separate v1→v2 recompute owed)

**Current estimate value:** 3.55 j/h/mois total calibrated (3 064€/mois Forfait sans contingence ni immo) — also uses pre-sqrt calibrated + 3× methodology.

**v3 modifiers triggered:** none.

**Why unchanged by v3:**
- `tenancy_count = 1` (single ANS tenant on PFC cluster).
- `year_1_ramp = none` (operating an existing live application; the bascule PFC is itself a non-migration-from-on-prem context).
- `specializations = []` — no HDS, the K8s PFC cluster is **out of scope** (separate prestation).
- `stakeholder_complexity = low`.
- `regulatory_profile = none` (sécurité FAIBLE per EdB).

**Recompute pass owed:** **Yes — same as CISAC**, independent v1→v2 recompute. Under v2 strict, deductive lands at ~11.9 j/h/mois (vs 13.0 in the old deductive table, vs 3.55 calibrated retained). The Kafka coeff 5 dominates and the calibrated value reflects the SA's judgement that the deployed Kafka is not production-grade-high-charge — that calibration discussion is independent of v3 and should be revisited at engagement renewal (3-month renewable contract).

**Action:** flag for v1→v2 recompute at next ANS contract renewal review (period 1 review at T+3 months post-bascule).

---

## Biogroup Move to Cloud (BMC) — **major v3 impact** (handled separately)

**Current estimate value:** 36.5 j/h/mois total (~29 414€/mois Forfait classique equivalent, ~353k€/an).

**v3 modifiers that move the price:**

| Modifier | Declared value | Effect on MCO |
|---|---|---|
| `tenancy_count` | 30 SELAS (T=20+ band) | Per-bucket m_T penalty on per-SELAS buckets (App L&S, KaliSil PREMI3NS, middlewares). MCO_marginal moves 27.7 → ~58.4 j/h. |
| `capability_floor` | Complète + HDS + T=30 → 70 j/h | `max(70, 58.4) = 70`. Floor activates and raises MCO baseline to 70. |
| `year_1_ramp` | `heavy_migration` (×1.50) | On-prem VMware → GCP souverain, RFP §7 transformation. 70 → 105 j/h for the first 12 months. |
| `specializations` | SecOps 5.0 + FinOps 2.5 + K8s Spec 5.0 + HDS Officer 2.5 = 15 j/h-equiv | Added on top, billed at specialist TJMs (1 100€ to 1 400€). |
| `stakeholder_complexity` | `high` (×2.0) | Gouvernance scales up; dispositif cascade pushes total to Dédié → COPROD weekly. |
| **Cascade: dispositif** | **Semi-dédié → Dédié** (>100 j/h) | Gouvernance_base recomputes with weekly COPROD + trimestriel COPIL. |

**v3 result:** ~133 j/h/mois total, ~120 910€/mois HT (after 3+yr discount), ~1.45M€/an HT. **4.1× the current Biogroup monthly price.**

**Recompute pass owed:** **YES** — Biogroup is the trigger case for v3. However, per task instructions, `biogroup-m2c/estimate.md` is **being handled separately for the 2026-05-27 RFP deadline** with a manual override. **Do not modify it as part of this v3 work.**

After v3 ships:
1. Re-run `/qualify` on Biogroup with the new fields populated (tenancy_count=30, year_1_ramp=heavy_migration with end date contract_start + 12 mo, specializations=[SecOps, FinOps, K8s Spec, HDS Officer], stakeholder_complexity=high, regulatory_profile=HDS).
2. Re-run `/estimate`; expect ~133 j/h/mois total, ~1.45M€/an HT.
3. Coordinate with the team handling the manual RFP override to reconcile the two numbers before contract signature.

**Action:** post-v3-merge, schedule a coordinated Biogroup re-pricing session before final RFP submission. Cross-check the v3 number against the manual RFP override; identify any deltas and adjust.

---

## Summary

| Client | v3 impact | Recompute pass owed? | Action |
|---|---|---|---|
| Carenity | none (0%) | No | none |
| RDG-redirect | none (0%) | No | none |
| CISAC | none v3 (0%) | **Yes, v1→v2** (separate from v3) | flag at contract review |
| ANS | none v3 (0%) | **Yes, v1→v2** (separate from v3) | flag at T+3 mo review |
| Biogroup | **+265%** (36.5 → 133 j/h) | **Yes, v3** (separately handled) | post-v3-merge coordination with RFP team |

**Constraint check:** all small clients move 0% under v3 → satisfies the ±15% guardrail. Biogroup converges to ~133 j/h ≈ realistic team size (100–120 j/h target).
