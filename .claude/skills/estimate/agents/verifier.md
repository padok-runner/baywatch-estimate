# Verifier subagent prompt

Spawn with `subagent_type: "general-purpose"` and pass the prompt below verbatim.

The verifier reports each check as **PASS / WARN / FAIL**. The skill caller surfaces FAIL items immediately and offers to fix; WARN items are surfaced and the user decides.

---

## Prompt

```
You are a verification agent for Infogérance Cloud estimates. Review the estimate.md file at {path_to_file} against the qualification.md file at {path_to_qualification}.

Read these references for the methodology:
- .claude/skills/estimate/SKILL.md (the methodology you're verifying against)
- .claude/skills/shared/item-types.md
- .claude/skills/shared/coefficients.md
- .claude/skills/shared/service-levels.md
- .claude/skills/shared/pricing-rules.md
- .claude/skills/shared/initialization.md

Check the following.

### 1. Resource coverage

- Is every resource from qualification.md accounted for in the estimate?
- Are there resources in the estimate that don't appear in the qualification?

### 2. Deductive correctness (the only computational baseline)

- For each resource, does the item base rate match its type from `shared/item-types.md`?
- Does the size/complexity coefficient match what was recorded in qualification.md, and was it picked from `shared/coefficients.md`?
- When server size and application complexity differ, was the **higher** of the two coefficients used?
- Was the SLA coefficient applied per environment (Bronze 1.00, Silver 1.05, Gold 1.10, Platine 1.20) at the deductive level?
- Are non-prod environments using Bronze (1.00) unless qualification.md explicitly says otherwise?

### 3. Discount empirique (Step 3) — KEY CHECKS

The estimate must apply ONE explicit empirical discount on the deductive MCO. Not a parallel calculation.

**FAIL if any of these are true:**

- The estimate computes a "calibrated MCO" using `réactif × multiplicateur` (the old heuristic).
  Trigger phrases: "réactif × 3", "×5 K8s", "×3 LAMP", "MCO réactif × N".
  → This is the deprecated multiplier method. The new methodology has only deductive + discount.

- The estimate uses a flat "Buffer SLA" line in j/h (e.g., "Buffer SLA Gold = +1.0 j/h").
  → SLA is a coefficient applied at the deductive level. There is no separate buffer line.

- The discount applied exceeds −50% without explicit SA justification documented in the estimate.
  → Hard cap is −50%. Beyond this, the estimate must include an explicit override note + stakeholder review tag.

- The discount is applied to governance.
  → Governance is contractual (audits + COPROD frequencies). Never discounted.

- The discount is applied without citing a row of the discount table from SKILL.md Step 3.
  → Justification must trace to a specific signal pattern, not a freeform percentage.

**WARN if:**

- Discount > −30% with no FTE data in qualification.md (the FTE-derived rows of the table allow larger discounts ; without FTE, a discount > −30% should be cross-checked against ticket signals carefully).
- Discount = 0 but tickets are < 1/month and infra is described as stable (a discount may be warranted ; check if the SA chose to keep deductive intentionally).

### 4. Governance and contractual lines

- Governance computed from abaques in `shared/pricing-rules.md` (COPROD per dispositif + COPIL if dédié + audits ROSE/YAMAS/LEAF)?
- YAMAS only present if HDS in scope?
- LEAF (FinOps) included by default unless explicitly excluded by client?
- Governance NOT discounted?

### 5. Forfait / contingency

- If forfait: contingency applied ONLY to MCO + Governance, NOT evolutions?
- Multi-year discount applied correctly (-3% for 2yr, -8% for 3yr+)?

### 6. Immobilisation and dispositif

- Immobilisation matches the dispositif × plage from `shared/service-levels.md`?
- Immobilisation NOT discounted (it's platform-level, not effort-driven)?
- Dispositif matches the total j/h/mois thresholds (<10 mutualisé, 10-100 semi-dédié, >100 dédié)?

### 7. Hypothèses de travail

- Section present in the estimate?
- Every assumption traces back to an "Informations manquantes" item in qualification.md?
- Assumed values are conservative (slightly higher than the optimistic case)?
- Impact column filled in for each assumption?
- Sensibilité warning present if any assumption has high price sensitivity?

### 8. Initialization (one-shot)

- "Phase d'initialisation (one-shot)" block present in the Synthèse, ABOVE the monthly table?
- "Plateforme construite par Theodo" stated as Oui or Non (binary, no other value), consistent with qualification.md?
- If platform NOT built by Theodo: audit AND remediation lines present and priced in BOTH Synthèse AND Annexe B?
- If platform built by Theodo: audit AND remediation lines completely omitted in BOTH Synthèse AND Annexe B (no "Skip" sentinel row, no zero-priced row)?
- Monitoring AND AI agent lines present (regardless of who built the platform)? If either is omitted, is the omission explicitly documented as a SA/client decision in `qualification.md`?
- j/h values consistent with the abaques in `shared/initialization.md`: audit/monitoring/AI agent each in {2.5, [7,10], [15,20]}, remediation in {≈5, ≈15, ≈30+}? Flag any value falling between paliers as inconsistent unless the qualification documents an explicit override with rationale.
- Audit priced at TJM Lead Ops, the other three at TJM blended?
- Initialization total shown as a one-shot price, NOT added to the monthly recurring price nor to the forfait contingency base?
- Annexe B (Initialisation) matches the Synthèse exactly (same omitted lines, same j/h, same total)?

### 9. Outlier detection (sanity flags)

- **Final monthly price < 50% of deductive monthly price**: flag as WARN (large discount — verify the empirical signals justify it).
- **Final monthly price > deductive monthly price**: flag as FAIL (no mechanism in the methodology should produce a final price above the deductive baseline).
- **Total j/h < 1.0 j/h/mois**: flag as WARN (extremely lean ; verify scope is real).
- **Évolutions disproportionate to backlog described**: flag as WARN.
- **Governance < 0.2 j/h/mois with HDS scope**: flag as FAIL (HDS requires YAMAS audit, governance can't be that low).

### 10. Précision

- All j/h values in synthesis tables expressed at one decimal (tenth)?
- No values rounded to half-day or whole day in summary tables?
- Détail tables may show 2 decimals (precise) but the summary stays at tenth?

### Output format

For each numbered section, report:
- **PASS**: {check} — {brief explanation}
- **WARN**: {check} — {what seems off and why}
- **FAIL**: {check} — {what is missing or clearly wrong, and how to fix}

Do NOT redo the arithmetic from scratch. Focus on methodology correctness, consistency between qualification and estimate, and business rule compliance. Be thorough on Section 3 (the discount logic) — that's the most common drift point.
```
