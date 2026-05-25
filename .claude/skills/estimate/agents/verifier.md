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

### 4. Sublinear scaling — KEY CHECK (v3 tenancy-aware)

The methodology applies `m_T(N, T) = sqrt(T) × multiplier(N / sqrt(T))` per `(item type, coefficient, tenants_spanned)` bucket. For `T=1` (single-tenant), this reduces to `multiplier(N) = min(N, 3) + sqrt(max(N/3, 1)) - 1` (v2 behaviour). For `T>1` (multi-tenant fragmentation), the multiplier grows.

**FAIL if:**
- The estimate computes MCO as `N × base × coeff` (linear) for buckets where N > 3.
- The estimate applies a "discount" or "calibration" factor on top of the deductive total (this was the deprecated approach — scaling is now in the abaque).
- The estimate uses the deprecated `réactif × multiplicateur` heuristic (3× LAMP / 5× K8s).
- The estimate uses a flat `Buffer SLA` line in j/h (deprecated).
- A bucket has `T > 1` declared in qualification but the estimate ignores it and uses `multiplier(N)` instead of `m_T(N, T)`.

**WARN if:**
- A bucket has N > 5 with no explicit scaling shown in the breakdown.
- Two ressources of the same type and coeff appear as separate lines instead of grouped (loses the scaling benefit).
- The MCO bucket-by-bucket table omits the `T` column (introduced in v3).

Verify the scaling values against the table:

| N | m(N) (v2 / T=1) | m_T(N, T=10) | m_T(N, T=30) |
|---|---|---|---|
| 1 | 1.00 | 1.00 | 1.00 |
| 3 | 3.00 | 3.00 | 3.00 |
| 5 | 3.29 | 5.00 | 5.00 |
| 10 | 3.83 | 9.57 | 10.00 |
| 30 | 5.16 | 11.94 | 18.37 |
| 100 | 7.77 | 16.59 | 24.49 |
| 480 | 14.65 | 28.79 | 40.55 |
| 1000 | 20.26 | 38.77 | 53.70 |

Lecture : à `N < 3T` (peu de ressources par tenant), le facteur tenant prédomine et `m_T ≈ T × (N/T) = N`. À `N >> 3T`, l'amortissement intra-tenant kick in et `m_T → sqrt(T) × sqrt(N/(3T)) = sqrt(N/3)` ×  √T — plus grand que `sqrt(N/3)` du v2 d'un facteur sqrt(T).

### 5. SLA application

- Is the SLA coefficient applied **per environment** (after the bucket-level scaling), not at the resource level before scaling?
- Is the prorata distribution count-based (`count_env / count_total_bucket`) when a bucket spans multiple envs?

### 6. Governance

- Computed from abaques in `shared/pricing-rules.md` (COPROD per dispositif + COPIL if dédié + audits ROSE/YAMAS/LEAF)?
- YAMAS only present if HDS in scope?
- LEAF (FinOps) included by default unless explicitly excluded?
- Governance NOT scaled or discounted (it's contractual)?

### 7. Forfait / contingency / multi-year

- If forfait: contingency applied ONLY to MCO + Governance, NOT evolutions?
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

The estimate **must** show explicit application of the v3 modifiers (or explicit "default, no impact" if defaults).

**FAIL if:**
- The estimate is missing the "Application des modificateurs v3" subsection in Annexe A.
- `tenancy_count ≥ 5` is declared in qualification but no `T` column appears in the MCO bucket table; OR the multiplier values match `multiplier(N)` (v2) instead of `m_T(N, T)`.
- `year_1_ramp ≠ none` declared but `MCO_after_ramp = MCO_after_floor` (multiplier ignored).
- `specializations[]` declared but no specialization lines appear in the price table, OR specialists priced at the blended TJM 863€ instead of their own TJM.
- `stakeholder_complexity ≠ low` declared but `Gouvernance_final = Gouvernance_base` (multiplier ignored).
- `capability_floor` activates (T≥5 with HDS/Complète) but the estimate does not show `max(floor, MCO_marginal)` explicitly.

**WARN if:**
- v3 modifiers are at defaults but the qualification context strongly suggests one should be triggered (e.g., 30 SELAS mentioned in client context but `tenancy_count = 1` declared).
- A specialization is declared with a custom j/h sizing without justification in the estimate's "Hypothèses de travail" or "Notes".

Verify the modifier flow matches the qualification's "v3 Modificateurs structurels" section:

| Modifier | Qualification value | Estimate must show |
|---|---|---|
| `tenancy_count` | 1 → "default, no floor" message ; ≥5 → floor computed | `capability_floor(P, R, T) = …`, `MCO_after_floor = max(…, …)` |
| `year_1_ramp` | none → ×1.00 ; else → multiplier and end_date | `MCO_after_ramp = MCO_after_floor × {1.00 / 1.20 / 1.30 / 1.50}` |
| `specializations[]` | empty → omit ; non-empty → one line per role | per-role j/h × per-role TJM, then subtotal "Specialization premium" |
| `stakeholder_complexity` | low → ×1.0 ; else multiplier | `Gouvernance_final = Gouvernance_base × {1.0 / 1.5 / 2.0}` |
| `regulatory_profile` | feeds into capability_floor | mentioned in the floor computation |

### 15. v3 dispositif cascade

If v3 modifiers push the total j/h above the 100 j/h threshold (Dédié), verify:

- Dispositif is recorded as **Dédié** in the estimate (not Semi-dédié, even if the v2-only total would have been below 100).
- `Gouvernance_base` is recomputed with Dédié-level COPROD (weekly) + COPIL (trimestriel), THEN the stakeholder multiplier is applied.
- The estimate's "Notes" or "Dispositif" line explains the upgrade with the v3 modifier breakdown.

FAIL if the dispositif is left at Semi-dédié while v3 total > 100 j/h.

### Output format

For each numbered section, report:
- **PASS**: {check} — {brief explanation}
- **WARN**: {check} — {what seems off and why}
- **FAIL**: {check} — {what is missing or clearly wrong, and how to fix}

Do NOT redo the arithmetic from scratch. Focus on methodology correctness — especially Section 2 (item type) and Section 4 (scaling), which are the new key checks.
```
