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

- `skills/shared/item-types.md` — substrate vs application item types, base rates per type, and the **linear scaling** for N ressources of the same type/coeff
- `skills/shared/coefficients.md` — size/complexity coefficients
- `skills/shared/service-levels.md` — plage horaire, SLA coefficients, immobilisation
- `skills/shared/pricing-rules.md` — engagement modes, multi-year discounts, governance abaques
- `skills/shared/daily-rates.md` — TJMs and team composition
- `skills/shared/services.md` — service catalogue for the synthesis table
- `skills/shared/initialization.md` — one-shot init phase

For the output structure, read `references/output-template.md`.

## Conventions de précision

Tous les j/h/mois sont exprimés au **dixième de jour** près (ex. 0.4, 1.2, 2.1). N'arrondissez jamais au demi-jour ou au jour entier.

- Calculs intermédiaires : 2 décimales.
- Tableaux de synthèse : 1 décimale.
- Total final : somme précise des composantes, arrondie une seule fois au dixième.

## Methodology

The price is built from one rigorous calculation. There is no parallel heuristic, no discount knob.

```
For each (item type, coefficient) bucket :
  MCO_bucket = base_rate × N × coefficient

Distribute MCO_bucket across envs prorata of count, apply SLA per env.
Sum across all buckets and envs.

Total monthly j/h = MCO + Governance + Evolutions
Price = Total × TJM + Immobilisation [+ Forfait contingency]
```

Scaling is linéaire — l'amortissement orchestration est déjà capturé dans le `base_rate` calibré par type (cf. `item-types.md`). Pas de discount sublinéaire appliqué après coup.

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

### Step 1: Group ressources by (item type, coefficient)

Read the resource inventory from `qualification.md`. For each ressource, identify:

- **Item type** — from the substrate or application taxonomy in `shared/item-types.md`. Distinguish carefully: managed K8s vs self-hosted K8s, managed off-the-shelf (RDS, ElastiCache) vs self-hosted off-the-shelf (MySQL on VM, Postgres on VM), etc.
- **Coefficient** — from `shared/coefficients.md`. Pick the higher of (server size, application complexity).

Group ressources globally (across all envs) by `(item type, coefficient)`.

For each group, compute :

```
MCO_bucket = base_rate × N × coefficient
```

This is the **MCO base** for that bucket, before SLA.

### Step 2: Distribute per env and apply SLA

For each bucket, distribute the MCO base across environments **prorata of ressource count**. Apply the SLA coefficient per environment (Bronze 1.00, Silver 1.05, Gold 1.10, Platine 1.20).

```
MCO_total = Σ buckets ( Σ envs ( MCO_bucket × (count_env / count_total_bucket) × SLA_coeff_env ) )
```

The total MCO j/h/mois is the sum of these env-and-SLA-adjusted contributions across all buckets.

### Step 3: Governance and Evolutions

**Governance:** compute from the abaques in `shared/pricing-rules.md` — COPROD per dispositif + COPIL if dédié + audits ROSE/YAMAS/LEAF. Convert each to j/h/mois: `effort_per_session × sessions_per_year / 12`.

**Évolutions:** estimate from the evolution backlog in `qualification.md`. If unclear, ask the user.

### Step 4: Cross-checks (confirmation, no adjustment)

These are **confirmations**, never adjustments. The deductive total stands as-is; cross-checks only flag anomalies to investigate. Run both when the data is available.

**4a — Empirical (FTE) cross-check.** If `qualification.md` provides FTE breakdown, compute :
```
Empirical estimate = (MCO_FTE + governance_FTE + evolutions_FTE) × 20 j/h/mois
```

Compare with the deductive total. **No adjustment** — just a sanity check :

- Within ±20% : deductive holds, FTE confirms. Document the alignment in `estimate.md`.
- Deductive > FTE by >20% : flag in the report. Either client is currently understaffed, or the inventory has hidden non-MCO scope. Investigate — don't silently discount.
- FTE > Deductive by >20% : flag in the report. Either inventory is incomplete, or the client has hidden complexity. Investigate.

If qualification has no FTE data, also compare deductive against simple stability signals (ticket volume, incident count). Flag — don't adjust the number.

**4b — Market calibration via cloud spend.** If `qualification.md` provides the cloud bill, compute the deductive RUN cost as a share of annual cloud spend :
```
indicative RUN annuel = (Total j/h/mois × TJM × 12) + (Immobilisation × 12)
ratio = indicative RUN annuel / facture cloud annuelle HT
```
Compare `ratio` against the **Calibration marché** band in `shared/pricing-rules.md`. **No adjustment** — confirmation only :

- In-band → the deductive is confirmed by the market. Document it (cite the sources from `pricing-rules.md` — never invent a band).
- Out-of-band → flag and interpret per the table in `pricing-rules.md` (above-band = exigent perimeter or over-scoped inventory ; below-band = strong platform leverage or under-scoped). Investigate, don't remise mechanically.

> This ratio is meaningful only at scale. Below ~50 ressources the fixed-effort floor dominates and inflates the %, so treat it as a **weak** signal and say so rather than over-reading it. If no cloud bill was provided, skip 4b and note in the report that the market cross-check wasn't possible.

### Step 5: Final total & dispositif

```
Total j/h/mois = MCO + Governance + Evolutions
```

Determine the dispositif using thresholds in `shared/pricing-rules.md` (<10 mutualisé, 10–100 semi-dédié, >100 dédié).

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

TJM is the blended TJM from `shared/daily-rates.md` unless the user specifies otherwise.

```
MCO price = MCO j/h × TJM
Governance price = Governance × TJM
Evolution price = Évolutions × TJM
```

Note: SLA was already applied per env in Step 2. Don't apply it again here.

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
2. **Cross-check empirique** — flag discrepancies if any: FTE (4a) + calibration marché ratio cloud spend (4b)
3. **Synthèse** — client-facing summary (init block + monthly grid + price table)
4. **Annexes** — calculation detail (A: monthly with bucket-by-bucket breakdown, B: initialization)

---

## Verification

After generating `estimate.md`, spawn a verification subagent. Use the Agent tool with `subagent_type: "general-purpose"` and load the prompt from `agents/verifier.md`. If FAIL items, inform the user and offer to fix. WARN items: surface and let the user decide.
