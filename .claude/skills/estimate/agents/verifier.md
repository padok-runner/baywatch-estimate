# Verifier subagent prompt

Spawn with `subagent_type: "general-purpose"` and pass the prompt below verbatim.

The verifier reports each check as **PASS / WARN / FAIL**. The skill caller surfaces FAIL items immediately and offers to fix; WARN items are surfaced for the user to decide.

---

## Prompt

```
You are a verification agent for Infogérance Cloud estimates. Review the estimate.md file at {path_to_file} against the qualification.md file at {path_to_qualification}.

Read these references for the methodology:
- .claude/skills/estimate/SKILL.md
- .claude/skills/shared/item-types.md
- .claude/skills/shared/coefficients.md
- .claude/skills/shared/service-levels.md
- .claude/skills/shared/pricing-rules.md
- .claude/skills/shared/initialization.md

Check the following.

### 1. Resource coverage

- Is every resource from qualification.md accounted for in the estimate?
- Are there resources in the estimate that don't appear in the qualification?

### 2. Item type assignment — KEY CHECK

For each ressource, verify the item type from `shared/item-types.md`:

- **Substrate** : Public/Private managed VM, Managed/Self-hosted K8s cluster, Public managed container service, Hypervisor.
- **Application** : Managed off-the-shelf, Self-hosted off-the-shelf, Custom application.

**FAIL if:**
- A self-hosted DB (e.g., MySQL on EC2, Postgres on VM) is classified as "Managed off-the-shelf" instead of "Self-hosted off-the-shelf".
- A managed DB (e.g., RDS, ElastiCache) is classified as "Self-hosted off-the-shelf".
- A K8s cluster is missing the substrate line entirely (the cluster itself, separate from the apps it hosts).
- A self-hosted DB has only the application line and not the underlying VM substrate line (or vice-versa for VM-based self-hosting).

### 3. Coefficient correctness

- For each ressource, does the coefficient match `shared/coefficients.md`?
- When server size and application complexity differ, is the **higher** of the two used?
- Are non-prod environments using Bronze (1.00) unless qualification.md explicitly says otherwise?

### 4. Power-law scaling — KEY CHECK

The methodology applies `scale(N, T) = T^(1−k) × N^k` (k ≈ 0.8) per `(item type, coefficient, tenants_spanned)` bucket. For `T=1` (single-tenant) this is `N^0.8`; for `T>1` (multi-tenant fragmentation) it is multiplied by `T^0.2`. There is **no** piecewise-sqrt `multiplier(N)` anymore (that was v2, deprecated).

**FAIL if:**
- The estimate computes MCO as `N × base × coeff` (linear) for buckets where N > 3.
- The MCO core uses the deprecated v2 `multiplier(N) = min(N,3) + sqrt(max(N/3,1)) - 1` instead of `N^0.8`.
- The estimate applies a "discount" or "calibration" factor on top of the deductive total (scaling is in the abaque).
- The estimate uses the deprecated `réactif × multiplicateur` heuristic (3× LAMP / 5× K8s) or a flat `Buffer SLA` line in j/h.
- A bucket has `T > 1` declared in qualification but the estimate ignores it (uses `N^0.8` instead of `T^0.2 × N^0.8`).

**WARN if:**
- A bucket has N > 3 with no explicit scaling shown in the breakdown.
- Two ressources of the same type and coeff appear as separate lines instead of grouped (loses the scaling benefit).
- The MCO bucket-by-bucket table omits the `T` column.

Verify single-tenant scaling against the table (k = 0.8):

| N | N^0.8 |
|---|---|
| 1 | 1.00 |
| 3 | 2.41 |
| 5 | 3.62 |
| 10 | 6.31 |
| 30 | 15.20 |
| 100 | 39.81 |
| 480 | 139.6 |
| 1000 | 251.2 |

For multi-tenant buckets, multiply by `T^0.2` (1.38 at T=5, 1.58 at T=10, 1.82 at T=20, 1.97 at T=30, 2.51 at T=100), assuming instances spread ~evenly across tenants. **Edge case:** if `N ≤ T` (≤1 instance per tenant), `scale = N` exactly (no amortization) — verify the estimate did not over-credit amortization there.

### 5. SLA application

- Is the SLA coefficient applied **per environment** (after the bucket-level scaling), not at the resource level before scaling?
- Is the prorata distribution count-based (`count_env / count_total_bucket`) when a bucket spans multiple envs?

### 6. Governance

- Computed from abaques in `shared/pricing-rules.md` (COPROD per dispositif + COPIL if dédié + audits ROSE/YAMAS/LEAF)?
- YAMAS only present if HDS in scope?
- LEAF (FinOps) included by default unless explicitly excluded?
- Governance NOT scaled or discounted (it's contractual)?

### 7. Forfait / contingency / multi-year

- If forfait socle (default model): contingency applied ONLY to Governance, never MCO or evolutions? If forfait classique (fallback): contingency on MCO + Governance, never evolutions?
- Multi-year discount applied correctly (-3% for 2yr, -8% for 3yr+)?

### 8. Immobilisation and dispositif

- Immobilisation matches the dispositif × plage from `shared/service-levels.md`?
- Dispositif matches the total j/h/mois thresholds (<10 mutualisé, 10-100 semi-dédié, >100 dédié)?

### 9. Hypothèses de travail

- Section present?
- Every assumption traces back to "Informations manquantes" in qualification.md?
- Conservative values?
- Impact column filled?

### 10. Empirical cross-check

- If FTE breakdown available in qualification.md, is the cross-check shown in the estimate?
- If deductive vs FTE diverges by >20%, is the discrepancy flagged (not silently adjusted)?
- Make sure NO "discount" or "calibration factor" is applied — the new methodology disallows it.

### 11. Initialization (one-shot)

- "Phase d'initialisation (one-shot)" block present in Synthèse, ABOVE the monthly table?
- "Plateforme construite par Theodo" stated as Oui/Non, consistent with qualification.md?
- If platform NOT built by Theodo: audit AND remediation lines present and priced in BOTH Synthèse AND Annexe B?
- If platform built by Theodo: audit AND remediation lines completely omitted in BOTH?
- Monitoring line always present (even at Minimal = 1 j/h)?
- AI agent line present if sized Simple/Medium/Complex; **omitted in BOTH Synthèse AND Annexe B** if sizing = None (0 j/h) — same convention as Theodo-skipped audit/remediation?
- j/h values consistent with `shared/initialization.md` paliers (or documented override)? Monitoring ∈ {1, 2.5, [7,10], [15,20]}; AI agent ∈ {0, 2.5, [7,10], [15,20]}.
- Audit at TJM Lead Ops, others at TJM blended?
- Init total NOT added to monthly recurring price?

### 12. Sanity flags

- **Substrate VM line for self-hosted DB present** : check that a self-hosted DB has both the VM line (substrate) and the self-hosted off-the-shelf line (application). Sum them rather than picking one.
- **Total monthly j/h < 1.0** : flag as WARN.
- **Évolutions disproportionate to backlog** : flag as WARN.
- **Governance < 0.2 j/h with HDS scope** : flag as FAIL.

### 13. Précision

- All j/h values in synthesis tables at one decimal (tenth)?
- Détail tables may show 2 decimals; summary stays at tenth?
- No round-up on small fractions (5/12 = 0.42 → 0.4, not 0.5)?

### 14. v3 modificateurs structurels — KEY CHECK

**Backward compatibility.** If the qualification.md has **no** "v3 Modificateurs structurels" section (a pre-v3 artifact), treat the estimate as v2-mode: do **not** FAIL on missing v3 subsections. Report PASS with the note "pre-v3 artifact — v2 mode." Apply the checks below only when the qualification declares the v3 section.

When the v3 section is present, the estimate **must** show explicit application of the modifiers (or "default — no impact" rows when at defaults), and the core MCO must use the power-law `scale(N, T)`.

**FAIL if:**
- The qualification declares the v3 section but the estimate is missing the "Application des modificateurs v3" subsection in Annexe A.
- `tenancy_count ≥ 2` with multi-tenant buckets is declared but no `T` column appears in the MCO bucket table, OR a bucket's `scale` uses `N^0.8` (T=1) while `T>1` was declared.
- The MCO core uses the deprecated v2 `multiplier(N)` instead of the power-law `scale(N, T)`.
- `year_1_ramp ≠ none` declared but `MCO_after_ramp = MCO` (multiplier ignored).
- `specializations[]` declared but no specialization lines appear in the price table, OR specialists priced at the blended TJM (863€) instead of their own TJM.
- `stakeholder_complexity ≠ low` declared but `Gouvernance_final = Gouvernance_base` (multiplier ignored).

**WARN if:**
- v3 modifiers are at defaults but the qualification context strongly suggests one should be triggered (e.g., 30 SELAS in client context but `tenancy_count = 1`).
- A specialization is declared with a custom j/h sizing without justification in the estimate's "Hypothèses de travail" or "Notes".

Verify the modifier flow matches the qualification's "v3 Modificateurs structurels" section:

| Modifier | Qualification value | Estimate must show |
|---|---|---|
| `tenants_spanned` (per bucket) | 1 → `scale = N^0.8` ; >1 → `scale = T^0.2 × N^0.8` | `T` column in the bucket table; the `scale(N,T)` value used |
| `year_1_ramp` | none → ×1.00 ; else → multiplier and end_date | `MCO_after_ramp = MCO × {1.00 / 1.20 / 1.30 / 1.50}` |
| `specializations[]` | empty → omit ; non-empty → one line per role | per-role j/h × per-role TJM, then subtotal "Specialization premium" |
| `stakeholder_complexity` | low → ×1.0 ; else multiplier | `Gouvernance_final = Gouvernance_base × {1.0 / 1.5 / 2.0}` |
| `regulatory_profile` | gates specialization eligibility (HDS → SecOps / HDS Officer) | consistency note only (no longer feeds a floor) |

### 15. v3 dispositif cascade

The dispositif threshold is tested against the **post-modifier** Total (`MCO_final_jh + Gouvernance_final + Évolutions`). If that pushes the total above 100 j/h (Dédié), verify:

- Dispositif is recorded as **Dédié** in the estimate (not Semi-dédié, even if the pre-modifier total would have been below 100).
- `Gouvernance_base` is recomputed with Dédié-level COPROD (weekly) + COPIL (trimestriel), THEN the stakeholder multiplier is applied (exactly once).
- The estimate's "Notes" or "Dispositif" line explains the upgrade.

FAIL if the dispositif is left at Semi-dédié while the post-modifier total > 100 j/h.

### Output format

For each numbered section, report:
- **PASS**: {check} — {brief explanation}
- **WARN**: {check} — {what seems off and why}
- **FAIL**: {check} — {what is missing or clearly wrong, and how to fix}

Do NOT redo the arithmetic from scratch. Focus on methodology correctness — especially Section 2 (item type) and Section 4 (scaling), which are the new key checks.
```
