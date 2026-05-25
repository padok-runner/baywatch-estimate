# Pricing Rules

## Dispositif d'équipe

| Dispositif | Quantité par client    | Description                                                             | Exemples                  |
| ---------- | ---------------------- | ----------------------------------------------------------------------- | ------------------------- |
| Mutualisé  | < 10 jour-homme/mois   | Equipe ~7 personnes pour ~20 clients                                    | Ontex, Relevanc, Beneteau |
| Semi-dédié | 10-100 jour-homme/mois | Equipe ~7 personnes pour <5 clients. Le client a un tech dédié au moins | GTT, BNPP, ANS            |
| Dédié      | > 100 jour-homme/mois  | Equipe ~7 personnes pour 1 client                                       | BPI, LCL, ANS (futur)     |

## Modes d'engagement

Deux modes de facturation, à combiner selon la configuration de périmètre choisie (voir section suivante).

### 1. Temps passé

- Enveloppe de jours-homme staffée et reportable sur le mois suivant uniquement (M+1).
- **Facturation à la consommation réelle**, plafonnée par l'enveloppe.
- Priorités dans l'enveloppe : incidents > problèmes > versions > changements mineurs.
- S'adapte aux besoins du client, au budget et aux aléas.

### 2. Forfait

- **Facturation engagée** : le client paie l'enveloppe en totalité, qu'il la consomme ou non.
- Enveloppe + contingence pour risque :
  - Pas d'incertitude : +0%
  - Incertitude faible : +10%
  - Incertitude moyenne : +20%
  - Incertitude haute : +30 à 40%
- **Le forfait n'est jamais appliqué aux évolutions** (par construction, hors-périmètre contingence).

## Configuration par défaut : Forfait socle + carnet temps passé

- **Forfait socle** : Gouvernance + Audits + Immobilisation. Couvre la cadence contractuelle (ceremonies, audits HDS/ROSE/LEAF) et la capacité réservée (24/7 si plage Étendue/Complète).
- **Temps passé** : **MCO (toutes catégories : incidents, demandes, problèmes, changements, patching, monitoring)** + Évolutions. Le client paie ce qu'il consomme.
- **Profil client visé** : tout client avec engagement ≥2 ans. Particulièrement avantageux pour les infras stables ou compétitives sur le prix d'entrée.
- **Réservée aux clients engagés ≥2 ans** (voir règles de protection ci-dessous).

### Règles de protection

Le modèle repose sur la consommation pure côté MCO. La protection principale vient du **forfait socle** (immobilisation + gouvernance) et de l'**engagement multi-annuel**.

1. **Engagement contractuel ≥2 ans** : indispensable. La remise multi-annuelle (-3% pour 2 ans, -8% pour 3+ ans) s'applique normalement.

2. **Immobilisation comme protection capacitaire** : l'immobilisation mensuelle couvre la capacité réservée (24/7 si plage Étendue/Complète, slot équipe en mutualisé). C'est elle qui protège le revenu sur les mois calmes — pas un plancher artificiel sur le MCO.

3. **TJM facturé à la consommation** : MCO temps passé facturé au TJM blended (cf. `daily-rates.md`) ou au TJM par rôle pour les évolutions, selon ce qui est convenu contractuellement.

> Pas de plancher mensuel ni de plafond contractuel sur la consommation MCO. Si le client consomme zéro un mois, il paie le socle. Si la consommation explose, elle est facturée intégralement — ce qui doit déclencher une revue avec le client (peut-être faut-il passer en avenant Forfait, ou élargir l'enveloppe d'évolutions).

## Cas particulier : Forfait classique (engagement <2 ans)

Si le client refuse l'engagement ≥2 ans ou exige une prédictibilité absolue du budget mensuel, repli sur un **Forfait classique** :

- **Forfait** : MCO + Gouvernance + Immobilisation (= enveloppe déductive complète calculée par `/estimate`)
- **Temps passé** : Évolutions uniquement
- À traiter comme une exception ; la valeur compétitive du modèle se trouve dans la Configuration par défaut.

## Cas particulier : Temps passé pur (rare)

Pour des contextes très spécifiques (PoC, audit-only) où même la gouvernance ne peut être engagée :

- **Forfait** : Immobilisation uniquement
- **Temps passé** : tout le reste (MCO + Gouvernance + Évolutions)
- Rare en pratique — la gouvernance et les audits HDS ont une cadence contractuelle qui s'accommode mal d'une facturation pure consommation.

## Remises multi-annuelles

| Durée d'engagement | Remise |
| ------------------ | ------ |
| 2 ans              | -3%    |
| 3 ans ou plus      | -8%    |

- Possibilité de rompre le contrat à partir de 6 mois de collaboration avec préavis de 3 mois.

## Immobilisation

Les frais d'immobilisation dépendent du dispositif et de la plage horaire. Voir `service-levels.md` pour le tableau complet.

## HDS & Nearshore

- **HDS** : proposé par défaut en France (équipes Cloud plus développées en France).
- **Nearshore** : si le client souhaite réduire les coûts → en discuter avec Hugo/Lila/Manu.

## Abaques gouvernance & audits

### COPROD

| Dispositif | Fréquence     | Effort estimé       |
| ---------- | ------------- | ------------------- |
| Mutualisé  | 1 / trimestre | 0.5 j/h par session |
| Semi-dédié | 1 / mois      | 0.5 j/h par session |
| Dédié      | 1 / semaine   | 0.5 j/h par session |

### COPIL

| Dispositif       | Fréquence     | Effort estimé       |
| ---------------- | ------------- | ------------------- |
| Dédié uniquement | 1 / trimestre | 0.5 j/h par session |

### Audits

| Audit                    | Fréquence    | Effort estimé     |
| ------------------------ | ------------ | ----------------- |
| ROSE (Qualité)           | 1 / semestre | 0.5 j/h par audit |
| YAMAS (HDS / ISO 27001)  | 1 / semestre | 0.5 j/h par audit |
| LEAF (FinOps / Green IT) | 1 / semestre | 0.5 j/h par audit |

> YAMAS applicable uniquement si périmètre HDS.

## Facteur de conversion FTE

- 1 FTE = 20 j/h/mois (base : ~20 jours ouvrés par mois)

## Formule de prix finale (v3)

L'abaque v3 (2026-05) ajoute cinq modificateurs structurels au formule v2 (sublinéaire sqrt) pour les engagements à grande échelle, régulés ou multi-tenants. **Pour les petits clients single-tenant sans migration et sans spécialisations déclarées, v3 == v2 numériquement** (tous les modificateurs sont au défaut neutre).

```
Pour chaque bucket (item_type, coefficient, tenants_spanned T) :
  m_T(N, T) = sqrt(T) × m(N / sqrt(T))               # cf. item-types.md
  MCO_bucket = base × m_T(N, T) × coefficient

MCO_marginal = Σ buckets ( MCO_bucket × distribution_SLA_par_env )

MCO_after_floor = max( capability_floor(plage, régulation, T), MCO_marginal )

MCO_after_ramp  = MCO_after_floor × year_1_ramp_multiplier

MCO_final_jh    = MCO_after_ramp + Σ specialization_jh_per_role
                   (les j/h spécialistes sont facturés à leur propre TJM, voir
                    `daily-rates.md` — pas au TJM blended)

Gouvernance_final = Gouvernance_base × stakeholder_complexity_multiplier

Total j/h = MCO_final_jh + Gouvernance_final + Évolutions

Prix mensuel =
    MCO_after_ramp × TJM_blended
  + Σ (specialization_jh × TJM_spécialiste)
  + Gouvernance_final × TJM_blended
  + Évolutions × TJM_blended
  + Immobilisation
  [+ contingence forfait sur Gouvernance uniquement, jamais sur Évolutions]
  [× (1 − remise_multi_annuelle) sur services, hors immobilisation]
```

Chaque modificateur trace à un **champ déclaré** dans la qualification — pas de fudge factor, pas de détection magique sur l'inventaire.

### Modificateur 1 — capability_floor (plancher capacitaire)

Plancher en j/h/mois reflétant l'équipe minimum viable pour honorer la prestation. Ne se déclenche **que** sur les engagements genuinement multi-tenants (`T ≥ 5`). Les clients single-tenant (Mutualisé pool, single-tenant Semi-dédié) ne sont pas affectés (floor = 0).

```
capability_floor(plage P, régulation R, tenancy_count T) :
  if T < 5 :
    return 0

  base = 10  si  5 ≤ T ≤ 9
        25  si 10 ≤ T ≤ 19
        50  si T ≥ 20

  hds_bonus   = 10 si HDS dans R
  secnum      =  5 si SecNumCloud dans R
  plage_bonus =  5 si P = Étendue
              = 10 si P = Complète
              =  0 sinon

  return base + hds_bonus + secnum + plage_bonus
```

Le floor n'agit que comme **majorant** : `MCO_after_floor = max(floor, MCO_marginal)`. Si la somme marginale dépasse déjà le floor, le floor est inactif.

### Modificateur 2 — tenancy_penalty (fragmentation multi-tenants)

Capturée **par bucket** via `tenants_spanned T`. Voir `item-types.md` pour la formule m_T(N, T) = sqrt(T) × m(N / sqrt(T)).

Buckets dont les ressources sont consolidées dans un seul tenant (landing zone, services partagés, plateforme unique) gardent T=1 et m_T = m (comportement v2).

### Modificateur 3 — year_1_ramp (rampe stabilisation année 1)

Les clients en migration depuis on-prem portent un surplus structurel de MCO pendant les 12 premiers mois (automation à construire, training, runbooks à écrire from scratch, queue legacy à absorber). Time-bounded.

| Valeur déclarée | Multiplicateur | Quand utiliser |
|---|---|---|
| `none` (défaut) | 1.00 | Greenfield ou régime établi |
| `light_migration` | 1.20 | Lift-and-shift avec IaC mature et reskilling mineur |
| `migration` | 1.30 | Migration-from-on-prem standard |
| `heavy_migration` | 1.50 | Plateforme greenfield + migration workloads + ramp simultanés (ex. Biogroup Move to Cloud) |

Le multiplicateur s'applique à `MCO_after_floor`, **pas** à gouvernance ni évolutions. La qualification doit déclarer la **date de fin** du ramp ; le contrat reprice à end-of-ramp.

### Modificateur 4 — specialization_premium (rôles spécialistes)

L'équipe standard (cf. `daily-rates.md`) est Ops : Lead Ops : DM = 1.00 : 0.34 : 0.16. Elle couvre le run de base mais n'inclut pas SecOps, FinOps, K8s spécialistes, HDS officer — des rôles que certains engagements exigent structurellement.

La qualification déclare une liste `specializations[]`. Chaque rôle ajoute une baseline en j/h/mois facturée **à son propre TJM** (voir `daily-rates.md`) :

| Rôle | j/h/mois défaut | Quand déclarer |
|---|---|---|
| SecOps Lead | 5.0 | Périmètre HDS, données critiques, multi-tenant régulé |
| FinOps Lead | 2.5 | Multi-cloud, >20 VMs, cadence LEAF lourde |
| K8s Specialist | 5.0 | K8s managé en scope avec ≥10 nodes ou multi-cluster |
| HDS Officer / Compliance Lead | 2.5 | HDS + cadence audit > semestriel ou multi-SELAS HDS |

Les spécialistes sont **ajoutés** à `MCO_after_ramp` ; ils ne sont **pas** affectés par tenancy_penalty ni year_1_ramp (déjà dimensionnés pour le profil de l'engagement). Le sizing par défaut peut être surchargé en qualification avec justification.

> **Important — clients Mutualisé.** Les spécialisations qui sont mutualisées au niveau de l'équipe (SecOps partagé entre 20 clients) **ne sont pas déclarées** par client — leur coût est déjà absorbé dans le TJM blended. v3 ne déclare des spécialisations que pour les engagements où le profil client justifie une **capacité dédiée**.

### Modificateur 5 — stakeholder_complexity (gouvernance étendue)

L'abaque de gouvernance compte la cérémonie (COPROD, COPIL, audits) mais pas la prep, follow-up, CAB, ITSM triage, postmortems, HDS comité, reporting mensuel. Pour les grandes structures, ces activités multiplient la charge gouvernance réelle.

| Valeur déclarée | Multiplicateur | Quand déclarer |
|---|---|---|
| `low` (défaut) | 1.0 | 1–5 interlocuteurs (client mono-application, mono-équipe) |
| `medium` | 1.5 | 6–15 interlocuteurs (multi-produits, multi-équipes, multi-apps) |
| `high` | 2.0 | 16+ interlocuteurs (multi-SELAS, multi-BU, entité fédérée) |

Le multiplicateur s'applique à **Gouvernance_base** (somme abaque incluant COPROD, COPIL si dédié, audits, allégements). Il ne compose pas avec la cadence COPROD dispositif (qui est déjà un axe de différenciation distinct).

---

## Formule de prix (legacy v2, conservée pour référence single-tenant)

Pour rappel, la formule v2 (sans modificateurs v3) reste identique au cas particulier `T=1, ramp=none, specializations=[], stakeholder_complexity=low` :

```
Prix mensuel = (Total j/h MCO × coeff SLA × TJM) + Gouvernance + Evolutions + Immobilisation
```

avec les conventions de calcul de l'item-types.md.
