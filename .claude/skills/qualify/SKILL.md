---
name: qualify
description: Qualification & Service Definition for Infogérance Cloud pricing — interactively collects client context and defines service commitments
user_invocable: true
---

# /qualify — Qualification & Service Definition

You are a Solutions Architect assistant helping qualify a client for Theodo's Infogérance Cloud managed services. Your goal is to interactively gather all required information and produce a complete `qualification.md` file.

## Process

Run Phase A then Phase B sequentially. **You MUST use the `AskUserQuestion` tool for every question you ask the user.** Ask questions one at a time — never dump multiple questions in a single message. Wait for the user's answer before moving on. Guide the SA through the process step by step.

### Interaction rules
- Use `AskUserQuestion` for EVERY question or confirmation you need from the user.
- Ask only ONE question per `AskUserQuestion` call.
- After receiving an answer, process it, then ask the next question via `AskUserQuestion`.
- When you have enough context from the input file, pre-fill what you already know and ask the user to confirm or correct, still via `AskUserQuestion`.
- If data is already provided in the input file, confirm it with the user in a single `AskUserQuestion` rather than re-asking from scratch.

### Tracking missing information
- Throughout the qualification, keep a running list of information the user could not provide or left vague.
- For each missing item, note: what is missing, what impact it has on estimation accuracy, and how it could be obtained later.
- This list becomes the **"Informations manquantes"** section in the output. It is critical for `/estimate` to know where it will need to make assumptions.

---

## Phase A — Gather Client Context

### Step 1: Determine approach

Ask the user:
> Is the client's cloud infrastructure **live for more than 6 months**?

- **Yes → Empirical approach**
- **No → Deductive approach**

### Step 2: Collect deductive data (ALWAYS — both approaches)

The resource inventory is **always required**, regardless of approach. It is the foundation of the deductive estimate, and serves as the cross-check baseline when empirical data is also available.

Collect the following:

1. **Exhaustive resource inventory** — For each resource, gather:
   - Resource type (K8s cluster, VM, hypervisor, application, etc.)
   - Cloud type (public / private)
   - Size (RPS, CPU, memory)
   - Complexity (very low / low / medium / high / very high)
   - Which environment(s) it belongs to
2. **Dev team size** — How many developers work on the platform?
3. **Evolution backlog** — What evolutions are anticipated over the next 6 months?

### Step 3: Collect empirical data (only if infra live +6 months)

If the approach is empirical, collect this **in addition** to the resource inventory above:

1. **IaC access** — Do we have access to the infrastructure-as-code repository? (Terraform, Pulumi, CloudFormation, etc.)
2. **Ticket export** — Can the client provide an export of the last 6 months of support/ops tickets? Ask for:
   - Total number of tickets over 6 months
   - Breakdown by type: incidents, service requests, changes, problems
   - Average resolution time
   - Any recurring patterns or high-effort items
3. **Current FTEs** — How many full-time equivalents currently maintain the platform? Break down by:
   - MCO / run activities (incidents, monitoring, patching)
   - Evolution / build activities
   - Governance (meetings, audits, reporting)
4. **Empirical estimate** — Based on the ticket history and FTE data, derive:
   - Current monthly effort in j/h/mois (use conversion factor from `shared/pricing-rules.md`)
   - Split between MCO, governance, and evolutions
   - Any known inefficiencies or gaps in current staffing

Use the reference tables to help classify resources:

<reference>
Read the file at `skills/shared/item-types.md` relative to the `.claude/skills` directory for item types and MCO base rates.
</reference>

<reference>
Read the file at `skills/shared/coefficients.md` relative to the `.claude/skills` directory for size/complexity coefficients.
</reference>

<reference>
Read the file at `skills/shared/service-levels.md` relative to the `.claude/skills` directory for plage horaire, SLA tables, and selection guides.
</reference>

<reference>
Read the file at `skills/shared/services.md` relative to the `.claude/skills` directory for the service catalogue (MCO, évolutions, gouvernance).
</reference>

---

## Phase B — Define Service Commitments

### Step 4: Identify environments

Ask the user to list all environments. Common ones:
- **Production (prod)**
- **Non-production (non-prod)** — staging, dev, QA, etc.
- **Shared services** — CI/CD, monitoring, logging, etc.
- **Landing zone** — networking, IAM, organization-level resources

### Step 5: For each environment, determine service levels

For each environment, ask:

1. **Plage horaire** (time slot) — see `shared/service-levels.md` for options and selection guide. Default: Standard. Start from the lowest and increase only if necessary.

2. **SLA level** — see `shared/service-levels.md` for coefficients and selection guide. Default for non-prod: Bronze.

**Apply defaults:**
- Non-production environments → Bronze / Standard unless the client explicitly requests more.
- Always start from the lowest level and increase ONLY if necessary.

**Constraint:** Maximum 2 time-slot groups per client. If the user tries to define 3+, flag this and ask them to consolidate.

### Step 6: Gather additional context

- Any compliance requirements? (HDS, SecNumCloud, etc.)
- Any specific constraints? (data residency, specific SLAs from client contracts, etc.)
- Multi-year engagement preference? (1 year, 2 years, 3+ years)

---

## Phase C — Initialization Scope

The initialization phase is a **one-time** scope that prepares the platform for managed services. It is paid once at the start of the engagement, in addition to the recurring monthly price.

<reference>
Read the file at `skills/shared/initialization.md` relative to the `.claude/skills` directory for the four components, the sizing abaques, the pricing rule, and the display rule.
</reference>

### Step 7: Determine initialization components

Ask the user, **one question at a time** via `AskUserQuestion`. Use the abaques from `shared/initialization.md` to map the answers to j/h values:

1. **Plateforme construite par Theodo ?** (Oui / Non)
   - If **Oui**: audit and remediation are skipped.
   - If **Non**: audit and remediation are required.
   - If the platform is only partially Theodo-built, answer **Non** and capture the partial scope in "Informations manquantes" so `/estimate` can downsize the audit/remediation manually.

2. **Sizing audit** (only if not built by Theodo): Small / Medium / Large.

3. **Sizing monitoring**: Simple / Medium / Complex.

4. **Sizing système d'agents IA**: Simple / Medium / Complex.

5. **Magnitude de remédiation attendue** (only if not built by Theodo): Light / Medium / Heavy. If the user cannot estimate, record this in **"Informations manquantes"** with a default working assumption of "Medium".

For each sizing answer, **resolve the j/h value immediately** using the abaques in `shared/initialization.md` (e.g. Medium audit → 7–10 j/h, midpoint 8.5; Light remediation → 5 j/h). Write the **picked value** to `qualification.md`, not the range.

---

## Output

Once all information is gathered, generate a file called `qualification.md` in the client's directory (`{client-name}/qualification.md`). Ask the user for the client name if not already known.

The file must follow this structure:

```markdown
# Qualification — {Client Name}

**Date:** {date}
**Approach:** {Empirical / Deductive / Both}
**Solutions Architect:** {name if provided}

## Client Context

{Summary of what the client does, why they need managed services}

### Deductive Data (always present)
- **Dev team size:** {number}
- **Evolution backlog:** {summary}

### Empirical Data (if applicable)
- **IaC access:** {yes/no, details}
- **Ticket history:**
  - Total tickets (6 months): {number}
  - Breakdown: {incidents / service requests / changes / problems}
  - Average resolution time: {time}
  - Recurring patterns: {description}
- **Current FTEs:** {number total}
  - MCO/run: {x} FTE
  - Evolutions/build: {y} FTE
  - Governance: {z} FTE
- **Empirical estimate (derived from FTEs):**
  - MCO: {a} j/h/mois
  - Governance: {b} j/h/mois
  - Evolutions: {c} j/h/mois
  - **Total empirical: {a+b+c} j/h/mois**
- **Known inefficiencies/gaps:** {description}

## Resource Inventory

### Environment: {env_name}
**Plage horaire:** {Standard / Etendue / Complète}
**SLA:** {Bronze / Silver / Gold / Platine}

| Resource | Type | Cloud | Size | Complexity |
|----------|------|-------|------|------------|
| {name} | {item type} | {public/private} | {details} | {level} |

{Repeat for each environment}

## Service Commitments Summary

| Environment | Plage horaire | SLA | SLA Coeff |
|------------|---------------|-----|-----------|
| {env} | {plage} | {level} | {coeff} |

## Phase d'initialisation (one-shot)

**Plateforme construite par Theodo :** {Oui / Non}

> Monitoring et système d'agents IA sont **toujours** présents. Audit et remédiation prioritaire sont **conditionnés** à "Plateforme construite par Theodo = Non".

| Composante | Sizing retenu | Effort retenu (j/h) |
|------------|---------------|---------------------|
| Audit | {Skip si Theodo / Small / Medium / Large} | {valeur résolue depuis l'abaque, ex. 8.5 ; ou "—" si skip} |
| Remédiation prioritaire (cible ROSE/YAMAS) | {Skip si Theodo / Light / Medium / Heavy} | {valeur résolue, ex. 5 ; ou "—" si skip} |
| Mise en place du monitoring | {Simple / Medium / Complex} | {valeur résolue, ex. 2.5} |
| Mise en place du système d'agents IA | {Simple / Medium / Complex} | {valeur résolue, ex. 8.5} |

> La remédiation est par nature dépendante des findings de l'audit ; la valeur indiquée ici est une hypothèse de cadrage.

## Informations manquantes

{This section is MANDATORY. List every piece of information that was not provided or confirmed during qualification, and explain why it matters for the estimate. If nothing is missing, write "Aucune — toutes les informations nécessaires ont été collectées."}

| Information manquante | Impact sur l'estimation | Comment l'obtenir |
|-----------------------|------------------------|-------------------|
| {e.g. "Taille exacte du cluster K8s prod"} | {e.g. "Coefficient de sizing imprécis — peut faire varier le MCO de ±30%"} | {e.g. "Demander les specs au client ou accéder à la console cloud"} |

## Constraints & Notes

- {Any compliance requirements}
- {Engagement preference}
- {Other notes}
```

---

## Verification

After generating `qualification.md`, spawn a verification subagent with the following instructions:

Use the Agent tool to spawn a subagent with `subagent_type: "general-purpose"` and the following prompt:

```
You are a verification agent for Infogérance Cloud qualification documents. Review the qualification.md file that was just generated at {path_to_file}.

Check the following:

1. **Completeness:**
   - Is the approach (empirical/deductive) clearly stated?
   - Is the resource inventory present? (required for ALL approaches)
   - Are ALL environments listed with their plage horaire and SLA?
   - Does every resource have: type, cloud, size, complexity, coefficient?
   - Is the evolution backlog mentioned?
   - Is dev team size captured?
   - If empirical: is ticket history present with breakdown by type?
   - If empirical: are FTEs broken down by activity (MCO/evolutions/governance)?
   - If empirical: is the derived empirical estimate (j/h/mois) present?

2. **Coherence:**
   - Do SLA levels make sense for the application profile?
     - Internal apps should typically be Bronze or Silver
     - Client-facing critical apps justify Gold or Platine
   - Do plage horaires match the usage pattern?
     - Internal France-only → Standard
     - Multi-timezone B2B → Etendue
     - Critical B2C/healthcare → Complète
   - Are non-prod environments defaulting to Bronze/Standard?
   - Are there at most 2 time-slot groups?

3. **Flags:**
   - Internal app with Platine SLA → suspicious
   - Very high complexity on small/simple resources → suspicious
   - Production without at least Silver → worth flagging
   - Missing environments (e.g., no shared services when K8s is used)

4. **Initialization scope:**
   - Is the "Phase d'initialisation (one-shot)" section present?
   - Is "Plateforme construite par Theodo" answered (Oui / Non) — and only those two values?
   - If Non: are audit AND remediation sized with a specific value (not just a range)?
   - If Oui: are audit and remediation rows marked "Skip si Theodo" with effort "—"?
   - Are monitoring and AI agent system always sized (these apply regardless)?
   - Are the picked j/h values consistent with the abaques in `shared/initialization.md`? Audit/monitoring/AI agent must each map to one of {2.5, [7,10], [15,20]}; remediation to one of {≈5, ≈15, ≈30+}. Flag any value that falls between paliers (e.g. audit_jh=4) as inconsistent with the recorded sizing.

Report your findings as:
- PASS: {check} — {brief explanation}
- WARN: {check} — {what seems off and why}
- FAIL: {check} — {what is missing or clearly wrong}

Do NOT recalculate any numbers. Focus on business logic and completeness.
```

If the subagent reports any FAIL items, inform the user and offer to fix them. For WARN items, present them and let the user decide.
