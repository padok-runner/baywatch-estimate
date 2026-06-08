# `estimate.md` Output Template

This is the exact structure to follow when generating `{client-name}/estimate.md`.

The file has four parts:

1. **Hypothèses de travail** — assumptions for missing info from the qualification
2. **Cross-check empirique** — sanity comparison vs FTE / ticket signals (no adjustment, just flag)
3. **Synthèse** — client-facing summary (init block + monthly grid + price table)
4. **Annexes** — calculation detail (A: monthly with bucket-by-bucket scaling and v3 modifiers, B: initialization)

**v3 (2026-06) note.** Annexe A includes a "v3 modifiers applied" subsection. For clients where all modifiers are at the default neutral value (ramp=none, specializations=[], stakeholder=low), it lists each as "default — no impact" but **must still be present** for auditability. Note: the v3 power-law core (`scale = N^0.8`) is **not** numerically equal to v2 — even at default modifiers the MCO differs from a v2 estimate. Multi-tenant fragmentation lives in the bucket `scale(N,T)` column, not in the modifiers subsection.

## Template

```markdown
# Estimate — {Client Name}

**Date:** {date}
**Based on:** qualification.md ({date of qualification})
**TJM:** {amount}€ (blended) — voir `shared/daily-rates.md`
**Dispositif:** {Mutualisé / Semi-dédié / Dédié}
**Précision :** j/h exprimés au dixième de jour (0.1)

---

## Hypothèses de travail

{MANDATORY. List every assumption due to missing or incomplete info. If none, write "Aucune — toutes les informations étaient disponibles dans la qualification."}

| # | Hypothèse | Information manquante | Valeur retenue | Justification | Impact si l'hypothèse est fausse |
|---|-----------|----------------------|----------------|---------------|----------------------------------|
| H1 | {e.g. "Taille du cluster K8s prod"} | {e.g. "Sizing exact non fourni"} | {e.g. "Coefficient 1 (medium)"} | {e.g. "Standard pour ce profil"} | {e.g. "±500€/mois si réel = 0.5 ou 2"} |

> **⚠ Sensibilité** : {If any high-impact assumption, summarize. E.g., "H1 représente ±15% du prix final. Confirmer la taille du cluster avant contractualisation."}

---

## Cross-check empirique

> **Note :** ce bloc est un **sanity check**, pas un ajustement. La méthode déductive (abaque + scaling sublinéaire de `shared/item-types.md`) produit le chiffre final tel quel. Les signaux empiriques permettent de **flagger des anomalies** mais ne modifient pas le total.

**Signaux empiriques observés (depuis qualification.md) :**

| Signal | Valeur |
|---|---|
| Tickets sur 12 mois | {N} → {N/12}/mois |
| Incidents | {N} ({type/cause}) |
| Problèmes récurrents | {N} |
| FTE empirique (si disponible) | {N} j/h/mois |
| Stabilité observée | {commentaire qualitatif} |

**Comparaison déductive vs empirique (si FTE disponible) :**

| | Déductive (calculée) | FTE × 20 | Delta % |
|---|---|---|---|
| Total j/h/mois | {X} | {Y} | {%} |

**Verdict :**
- Delta < ±20% : ✅ déductive confirmée par l'empirique. Pas de flag.
- Déductive > FTE par >20% : ⚠ client peut-être sous-staffé aujourd'hui ; ou inventaire inclut du non-MCO. À investiguer, pas à compenser.
- FTE > Déductive par >20% : ⚠ inventaire incomplet ou complexité cachée. À investiguer, pas à compenser.

> Le scaling de l'abaque (loi de puissance `scale(N, T) = T^(1−k) × N^k`, k≈0.8) intègre déjà le bénéfice automation/orchestration sur les ressources identiques **et** la pénalité de fragmentation multi-tenants. Aucun discount n'est appliqué après coup.

---

## Synthèse

### Phase d'initialisation (one-shot)

Plateforme construite par Theodo : **{Oui / Non}**

> Monitoring est **toujours** présent (palier Minimal = 1 j/h possible pour petits périmètres). Système d'agents IA est **optionnel** (omettre la ligne si sizing = None / 0 j/h). Audit et remédiation prioritaire **n'apparaissent que si** la plateforme n'a pas été construite par Theodo (sinon, omettre les deux lignes).

- {Si Non} **Audit** : {audit_jh} j/h Lead Ops — cartographie ressources, qualité, résilience, sécurité, observabilité.
- {Si Non} **Remédiation prioritaire** (cible ROSE/YAMAS) : {remediation_jh} j/h — docs, durcissement résilience, gaps qualité.
- **Mise en place du monitoring** : {monitoring_jh} j/h — métriques, alerting, dashboards, sondes, runbooks.
- {Si ai_agent_jh > 0} **Mise en place du système d'agents IA** : {ai_agent_jh} j/h — déploiement agent ChatOps, intégrations.

**Total initialisation : {total_init_jh} j/h — {total_init_price}€ HT (one-shot, payée une seule fois en début d'engagement)**

> Cette enveloppe est indépendante du prix mensuel récurrent ci-dessous.

---

### Tableau de synthèse mensuelle

|                         | {Env 1 category}                                                                                                                                                                                       | {Env 2 category}                                                     | {Env N category} | Transverse                                                                                                          |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------- | ---------------- | ------------------------------------------------------------------------------------------------------------------- |
| **Nom des envs**        | {env names}                                                                                                                                                                                            | {env names}                                                          | {env names}      | —                                                                                                                   |
| **Inventaire**          | **Servers :**{newline}{list servers with count, type, size}{newline}**Applications :**{newline}{list apps with count, type, complexity}                                                                | **Servers :**{newline}{...}{newline}**Applications :**{newline}{...} | {same format}    | {cross-cutting resources: organization, audit trail, IAM, CI/CD, etc.}                                              |
| **Services au Forfait socle** *(engagés mensuellement)* | {if plage != Standard: Capacité réservée 24/7 — astreinte (immobilisation)}{else: —}                                                                                                          | —                                                                    | —                | Gouvernance : COPIL/COPROD, audits ROSE/YAMAS/LEAF                                                                  |
| **Services au Temps passé** *(facturés à la consommation)* | MCO : Gestion des incidents, demandes, problèmes, changements de version, continuité{if plage != Standard: , surveillance, interventions d'astreinte}, patching, monitoring drift             | {same, adapted per env}                                              | {same}           | Évolutions : Gestion des Changements mineurs                                                                        |
| **Niveaux de services** | {Bronze / Silver / Gold / Platine}                                                                                                                                                                     | {level}                                                              | {level}          | —                                                                                                                   |
| **Plages de service**   | {Standard / Étendue / Complète}                                                                                                                                                                        | {plage}                                                              | {plage}          | —                                                                                                                   |
| **Dispositif**          | Ops : {X} j/mois, Lead Ops : {Y} j/mois, Delivery Manager : {Z} j/mois                                                                                                                                |                                                                      |                  |                                                                                                                     |

#### Prix mensuel €HT — Forfait socle + carnet temps passé

Configuration par défaut (engagement ≥2 ans). Voir `shared/pricing-rules.md` pour les cas particuliers (Forfait classique si <2 ans, Temps passé pur si PoC).

| Mode | Périmètre | j/h/mois | Montant €HT/mois |
|------|-----------|----------|-------------------|
| **Forfait socle** | Gouvernance ({y}) + Audits + Immobilisation | {y} | {amount_socle}€ |
| **Temps passé MCO** | MCO (toutes catégories : incidents, demandes, problèmes, changements, patching, monitoring). Facturé à la consommation réelle. | sur conso | Sur consommation |
| **Temps passé Évolutions** | À la demande | — | Sur consommation |
| | **Mensuel socle seul** (0 j/h MCO consommé) | **{y}** | **{amount_socle}€** |
| | **Mensuel espéré** ({avg conso} j/h MCO) | **{y + avg}** | **~{expected}€** |
| | **Référence enveloppe déductive (Forfait classique équivalent)** | **{x+y}** | **{cap total}€** |

> Engagement contractuel ≥2 ans (remise -3% / -8% applicable). L'immobilisation et la gouvernance forment le socle protégeant la capacité réservée et la cadence contractuelle. La consommation MCO est facturée à la réalité, sans plancher ni plafond ; la référence déductive sert de jalon de dimensionnement et de pédagogie client.

### Notes on the synthesis table

- **Init block placed above the synthesis grid.** Audit and remédiation lines are **omitted** if the platform was built by Theodo. The init price is **never** added to the monthly recurring price.
- **Inventaire**: human-readable. Group by "Servers" (K8s clusters, VMs, hypervisors, managed DBs, networking) and "Applications" (off-the-shelf and custom). Include count, name, size.
- **Services**: split en deux lignes pour expliciter le mode de facturation. **Forfait socle** : capacité réservée 24/7 (immobilisation, uniquement si plage ≠ Standard) + gouvernance/audits dans Transverse. **Temps passé** : tout le MCO opérationnel par env + évolutions dans Transverse. Cette distinction est cruciale — le client doit voir d'un coup d'œil ce qu'il paie chaque mois (socle) vs ce qu'il consomme (carnet).
- **Transverse column**: cross-cutting resources (org, IAM, audit trail) in inventory; governance dans la ligne Forfait socle, évolutions dans la ligne Temps passé. No SLA/plage.
- **Prix mensuel**: separate table below the grid. Three rows in the default model — Forfait socle (engagé) / Temps passé MCO (consommation) / Temps passé Évolutions (à la demande) — to make committed vs consumed clear.

---

## Annexe A — Detailed Calculation

### MCO Breakdown (deductive)

#### Environment: {env_name}

**SLA:** {level} (coeff {x})
**Plage:** {plage}

| Resource     | Item Type | Base Rate | Coeff   | MCO (j/h/mois) |
| ------------ | --------- | --------- | ------- | -------------- |
| {name}       | {type}    | {rate}    | {coeff} | {result}       |
| **Subtotal** |           |           |         | **{sum}**      |

{Repeat for each environment.}

#### MCO bucket-by-bucket (loi de puissance + tenancy)

> **Colonne T (tenants_spanned).** Si T=1 (défaut), `scale(N, T) = N^0.8`. Si T>1, `scale = T^0.2 × N^0.8` (ou la somme exacte `Σ N_t^0.8` si les tenants sont inégaux). Deux buckets distincts peuvent exister pour un même (item, coeff) — un consolidé (T=1) et un fragmenté (T>1).

| Bucket (item type, coeff, T) | N | base | T | scale(N,T) | coeff | MCO base bucket | Distribution par env (prorata count) | MCO après SLA |
|---|---|---|---|---|---|---|---|---|
| {ex. Public managed VM, 0.8, T=1} | {N} | {base} | {T} | {scale} | {coeff} | {base × scale × coeff} | {prod: x, recette: y, shared: z} | {x×SLA_prod + y×SLA_rec + z×SLA_sh} |

#### MCO (somme des buckets après SLA)

| | j/h/mois |
|---|---|
| Sum buckets après SLA par env | {MCO} |

#### Application des modificateurs v3

| Étape | Valeur | Impact j/h |
|---|---|---|
| MCO (après SLA) | — | {MCO} |
| year_1_ramp ({none / light / migration / heavy}) | {×1.00 / ×1.20 / ×1.30 / ×1.50} | {delta} |
| MCO_after_ramp | — | {MCO_after_ramp} |
| Specialization premium ({roles}) | {Σ j/h} | +{spec_jh} |
| **MCO_final_jh** | | **{MCO_final_jh}** |

> Si aucun modificateur n'est déclaré (ramp=none, specializations=[]), les lignes ramp/spec montrent delta 0 et `MCO_final_jh = MCO`. La fragmentation multi-tenants n'apparaît **pas** ici — elle est déjà dans la colonne `scale(N,T)` du tableau bucket-by-bucket ci-dessus.

### Gouvernance

| Activité | Fréquence | Effort/session | j/h/mois |
|----------|-----------|----------------|----------|
| COPROD   | {freq}    | 0.5 j/h        | {calc}   |
| COPIL    | {freq}    | 0.5 j/h        | {calc}   |
| ROSE     | semestriel | 0.5 j/h       | {calc}   |
| YAMAS    | semestriel | 0.5 j/h       | {calc}   |
| LEAF     | semestriel | 0.5 j/h       | {calc}   |
| **Gouvernance_base** | | | **{y_base} j/h/mois** |

| Modificateur v3 | Valeur | j/h |
|---|---|---|
| stakeholder_complexity_multiplier | {×1.0 / ×1.5 / ×2.0} | {y_base × mult} |
| **Gouvernance_final** | | **{y_final} j/h/mois** |

### Évolutions

- Estimées : **{x} j/h/mois**
- Base : {explanation}

### Total quantité

| Catégorie         | j/h/mois    |
| ----------------- | ----------- |
| MCO core (after_ramp) | {x_core}  |
| Specialization premium ({roles}, j/h-équiv) | {spec_jh} |
| MCO_final_jh (core + spec) | {x_core + spec_jh} |
| Gouvernance_final | {y}         |
| Évolutions        | {z}         |
| **Total**         | **{total}** |

### Prix

| Ligne | j/h/mois | TJM | Montant |
| --- | --- | --- | --- |
| MCO core (after_ramp) | {x_core} | 863€ blended | {x_core × 863} € |
{For each declared specialization role:} | {role} | {role.jh} | {role.tjm}€ | {role.jh × role.tjm} € |
| **Specialization subtotal** | {spec_jh} | (mix) | **{spec_amount} €** |
| Gouvernance_final | {y} | 863€ blended | {y × 863} € |
| Évolutions | {z} | 863€ blended | {z × 863} € |
| **Sous-total services** | | | **{services_amount} €** |
| Immobilisation | {plage} × {dispositif} | | {immo_amount} € |
{If forfait:} | Contingence | {level} (+{x}%) sur Gouv | | +{cont_amount} € |
{If multi-year:} | Remise multi-annuelle | {x} ans | | −{discount_amount} € |
| **Total mensuel** | | | **{total} €** |
| **Total annuel** | | | **{total × 12} €** |

> **v3 — specialist TJMs.** Les rôles spécialisés (SecOps, FinOps, K8s Spec, HDS Officer) sont facturés à leur **propre TJM** (cf. `shared/daily-rates.md`), pas au blended. Si `specializations[]` est vide, omettre les lignes correspondantes et la ligne "Specialization subtotal".

---

## Annexe B — Initialisation (one-shot)

| Composante | Sizing | j/h | TJM | Montant €HT |
|------------|--------|-----|-----|-------------|
| {Si Non} Audit (Lead Ops) | {Small / Medium / Large} | {audit_jh} | {tjm_lead_ops}€ | {amount}€ |
| {Si Non} Remédiation prioritaire | {Light / Medium / Heavy} | {remediation_jh} | {blended_tjm}€ | {amount}€ |
| Mise en place du monitoring | {Minimal / Simple / Medium / Complex} | {monitoring_jh} | {blended_tjm}€ | {amount}€ |
| {Si ai_agent_jh > 0} Mise en place système d'agents IA | {Simple / Medium / Complex} | {ai_agent_jh} | {blended_tjm}€ | {amount}€ |
| **Total initialisation** | | **{total_init_jh}** | | **{total_init_price}€** |

**Notes :**
- Audit et remédiation **omis** (lignes non affichées) si la plateforme a été construite par Theodo.
- Système d'agents IA **omis** (ligne non affichée) si sizing = None (0 j/h).
- Audit facturé au **TJM Lead Ops** (cf. `shared/daily-rates.md`).
- Remédiation, monitoring et système d'agents IA facturés au **TJM blended (Ops + Lead Ops + DM)**.
- Cette enveloppe est **payée une seule fois** en début d'engagement et n'entre pas dans le prix mensuel récurrent.

---

## Annexe C — Cross-check FTE (si applicable)

{Inclure cette annexe SEULEMENT si la qualification contient un FTE breakdown. Sinon, l'omettre entièrement.}

| Catégorie  | Déductive (j/h/mois) | FTE-derived (j/h/mois) | Delta     | Delta % |
| ---------- | -------------------- | ---------------------- | --------- | ------- |
| MCO        | {x}                  | {a}                    | {x-a}     | {%}     |
| Gouvernance | {y}                 | {b}                    | {y-b}     | {%}     |
| Évolutions | {z}                  | {c}                    | {z-c}     | {%}     |
| **Total**  | **{X}**              | **{A}**                | **{X-A}** | **{%}** |

**Analyse :** {Explanation. Flag if deductive > FTE by >20% (calibration justifies discount per Step 3 table) ; flag if FTE > deductive by >20% (inventory gap or hidden complexity — investigate).}

---

## Notes

- {HDS applicability}
- {Nearshore flag if relevant}
- {Engagement period and discounts}
- {Other considerations}
```
