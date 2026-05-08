# Qualification — RDG (Rail Delivery Group) — V1/V2 Redirect Proxy

**Date:** 2026-05-06
**Approach:** Deductive
**Solutions Architect:** Emmanuel Lilette
**Scope:** Add-on to existing RDG managed services contract (single new service)

## Client Context

RDG (Rail Delivery Group), via The Bountiful Cow, is adding a new **V1→V2 redirect proxy service** on top of its existing managed services with Theodo (Transit Gateway, multiple API services). The proxy silently forwards legacy traffic from the V1 backend to the V2 backend because retailers have struggled to amend their systems to hit the new V2 domain directly.

The service was built by The Bountiful Cow's team (Omar) using AWS CDK. It is **not yet in production** — go-live planned within ~2 weeks of the qualification date. Architecture is described as "architecturally simple": two ECS Fargate tasks running NGINX extended with OpenResty (Lua) for response-header manipulation, with routing decisions driven by a JSON file in an S3 configuration bucket. A Lambda function handles config reloads when the S3 file changes (admin path only — not in the request path). Documentation is reportedly very in-depth (load tests, runbooks, go-live plans, rollback procedures).

### Deductive Data (always present)
- **Dev team size (post-handover):** No active dev team — evolutions are essentially routing-config changes (S3 JSON edits) by RDG ops; rare Lua/NGINX tweaks if needed.
- **Evolution backlog (next 6 months):** Mostly routing-config changes as retailers cut over from V1 to V2; occasional minor tweaks. No major refactor anticipated.

### Empirical Data
Not applicable — service not yet live. **Reference anchor (provided by SA):** similar add-on contracts for RDG have priced at ~£2 000/month with ~2 days of initialization. Deductive estimate should be sanity-checked against this anchor; significant divergence warrants discussion.

## Resource Inventory

> Per the SA's modeling decision, the redirect-proxy stack is captured as **two MCO items per environment**: (1) the Fargate cluster, modeled as **Public Cloud Managed Container** (item type added to `shared/item-types.md` 2026-05-08, base 0.1 j/h — distinguishes container-as-a-service offerings like ECS / Cloud Run / Fargate from full managed K8s clusters which carry a heavier operational burden: no control plane, no kubectl/Helm, no node-group lifecycle), and (2) the NGINX/OpenResty redirect application (off-the-shelf NGINX extended with a thin Lua layer). Ancillary resources (S3 config bucket, admin Lambda, NLB, CloudWatch dashboard, SNS topics, Route53 CNAMEs) are operationally bundled into the application sizing — they are not separately MCO-priced because they have no independent ops footprint.

### Environment: dev
**Plage horaire:** Standard
**SLA:** Bronze

| Resource | Type | Cloud | Size | Complexity |
|----------|------|-------|------|------------|
| Fargate cluster (NGINX/OpenResty hosting) | Public Cloud Managed Container | Public (AWS) | <10 RPS, small Fargate tasks | Very low |
| NGINX/OpenResty redirect app (+ S3 config, admin Lambda, NLB, CW dashboard, SNS, Route53) | Off-the-shelf application (NGINX extended) | Public (AWS) | <10 RPS feature-test traffic | Very low |

### Environment: preprod (acc)
**Plage horaire:** Standard
**SLA:** Bronze

| Resource | Type | Cloud | Size | Complexity |
|----------|------|-------|------|------------|
| Fargate cluster (NGINX/OpenResty hosting) | Public Cloud Managed Container | Public (AWS) | <10 RPS (perf-test branch) | Very low |
| NGINX/OpenResty redirect app (+ S3 config, admin Lambda, NLB, CW dashboard, SNS, Route53) | Off-the-shelf application (NGINX extended) | Public (AWS) | <10 RPS (perf-test branch) | Very low |

### Environment: prod
**Plage horaire:** Complète (7j/7, 24h/24)
**SLA:** Platine

| Resource | Type | Cloud | Size | Complexity |
|----------|------|-------|------|------------|
| Fargate cluster (NGINX/OpenResty hosting) | Public Cloud Managed Container | Public (AWS) | ~300–800 RPS at peak (current V1 traffic) ; cluster physique très petit (<1 vCPU au pic) | Very low |
| NGINX/OpenResty redirect app (+ S3 config, admin Lambda, NLB, CW dashboard, SNS, Route53) | Off-the-shelf application (NGINX extended) | Public (AWS) | ~300–800 RPS at peak | Very low |

> **Coefficient rule for `/estimate`:** Non-prod → coeff 0.8 (driven by size <10 RPS ; complexity 0.5 dominée). Prod → **coeff 1 — SA judgment override (2026-05-08)**. La table stricte du framework mappe <1000 RPS → coeff 2, mais le SA applique un coeff réduit à 1 car : (1) auto-scaling validé sans erreur (`Proxy Auto Scale Test.pdf`), (2) cluster physique sous-utilisé (<1 vCPU consommé au pic réel), (3) très basse complexité (NGINX+Lua minimal), (4) headroom de capacité ~60% au-dessus du pic prod. **Cet override est explicite et documenté pour `/estimate`.**

## Service Commitments Summary

| Environment   | Plage horaire | SLA    | SLA Coeff |
|---------------|---------------|--------|-----------|
| dev           | Standard      | Bronze | 1.00      |
| preprod (acc) | Standard      | Bronze | 1.00      |
| prod          | Complète      | Platine| 1.20      |

Time-slot groups: **2** (Standard for non-prod, Complète for prod) — within the 2-group cap.

## Phase d'initialisation (one-shot)

**Plateforme construite par Theodo :** Non (built by The Bountiful Cow / Omar using AWS CDK)

> **Scope decision by the SA:** this engagement is an **add-on to an existing RDG managed services contract**. The SA has explicitly excluded **Remediation** and **AI agent system setup** from the contract scope:
> - **Remediation** is excluded because the existing documentation is already in-depth (load tests, runbooks, go-live plans, rollback procedures) and no significant ROSE/YAMAS gap is anticipated up-front. Any remediation found necessary post-audit will be handled as `changes mineurs` rather than a one-shot init line.
> - **AI agent system** is excluded because the existing RDG contract already provisions the agent system at the platform level — no incremental setup is needed for this add-on.
>
> This is a deliberate scoping decision and overrides the framework's default (which keeps AI agents always-on). `/estimate` must honor this decision: do not auto-restore those line items.

| Composante                                    | Sizing retenu                       | Effort retenu (j/h) |
|-----------------------------------------------|-------------------------------------|---------------------|
| Audit                                         | Small (downsized SA, < palier 2.5)  | 1                   |
| Remédiation prioritaire (cible ROSE/YAMAS)    | Hors périmètre (scope decision)     | —                   |
| Mise en place du monitoring                   | Simple (downsized SA, < palier 2.5) | 2                   |
| Mise en place du système d'agents IA          | Hors périmètre (scope decision)     | —                   |

**Total init abaque : 3 j/h** (Audit 1 + Monitoring 2 ; Remédiation et AI agents hors périmètre). With the +1 j/h go-live support documented in Constraints, **grand total init = 4 j/h**.

**SA downsizing decision (2026-05-08):** the abaques in `shared/initialization.md` set the lowest paliers at 2.5 j/h each, but the SA judges that for this add-on the documentation is so complete (runbooks, load tests, rollback plans, validated auto-scaling) and the scope so narrow that audit can be done in 1 day and monitoring setup in 2 days. This aligns with the SA's call commitment ("2–3 days for doc review, onboarding, workshop") and the £2k anchor's "2-day init" reference.

## Informations manquantes

| Information manquante | Impact sur l'estimation | Comment l'obtenir |
|-----------------------|------------------------|-------------------|
| ~~Exact peak RPS in prod~~ — **RÉSOLU 2026-05-06** (Omar) | ~~Drives top-tier coefficient~~ → resolved: prod CPU peaks <50% with 3 workers; scaling threshold at 800 TPS not reached → **<1000 RPS confirmed → coeff 2**. | Resolved via Sol's follow-up; full Confluence/repo access pending (Omar provisioning). |
| ~~Spiky / steep ramp-up traffic profile~~ — **RÉSOLU 2026-05-07** (Omar — *Proxy Auto Scale Test* report) | ~~If scaling is fragile…~~ → resolved: 220 concurrent users, **1 367 RPS sustained, 0 errors, P95 336ms**. Auto-scaling triggered at 80% CPU, scaled out 2 → 4 tasks, CPU dropped to 45%. Pure sub-second spike not tested but 5-min ramp behaviour validated. | Resolved via load-test PDF in `rdg-redirect/Proxy Auto Scale Test.pdf`. |
| **No rate limiting in front of the proxy** (new risk flagged by Sol/Omar 2026-05-06) | Backends are rate-limited (return 429s), but the proxy itself is not. A misbehaving caller can still flood the proxy and create incident load on our team. Could increase reactive MCO post-launch. | Architectural decision for RDG to make. Worth raising before go-live; if not addressed, build a buffer or dedicated incident-response plan. |
| Existing RDG contract baseline pricing per service | Cross-check anchor — the SA mentioned ~£2 000/month for similar contracts; size now confirmed as <1000 RPS, so divergence reduces. | Pull existing RDG contract line items for comparison. |
| Repository access (CDK platform repo + Lua/Lambda repo) | Needed for any deeper audit or remediation; also informs whether the audit can stay Small. | Omar provisioning access (per Sol 2026-05-06); Confluence export PDF shared in the meantime. |
| TJM baseline used for the £2 000/month reference | Affects whether deductive output is consistent with the anchor. | Internal — confirm with finance / existing RDG contract. |

## Constraints & Notes

- **Compliance:** No new compliance scope for this add-on — inherits from the existing RDG managed services contract terms (UK-based national rail data, standard data-protection posture; no HDS/SecNumCloud).
- **Engagement:** Aligns with the existing RDG contract duration (multi-year, inherited).
- **Go-live risk:** SA flagged the deployment phase as "the riskiest process" and committed to budgeting **+1 day for fine-tuning / on-watch** during the go-live window. This is captured outside the init abaque and should be added by `/estimate` as a one-shot go-live support line.
- **Existing code-review contract:** Has remaining hours that may be used for further investigation/simplification of the proxy ahead of handover. Out of scope for managed-services pricing but worth noting in the broader engagement narrative.
- **Reference anchor (sanity check):** ~£2 000/month + 2 days init for similar RDG add-on contracts. After Omar's load-test confirmation (size coeff 2, not 5), the deductive output is much closer to this anchor; the residual gap is driven by Platine prod SLA + Complète plage immobilisation (which the £2k contracts likely don't carry).
- **Time-slot groups:** 2 groups (Standard / Complète) — at the cap. Adding any further differentiation would require consolidation.
- **Load-test data (initial, 2026-05-06):** Omar reported the redirect proxy was load-tested at the rate limits in place; workers stayed below 30% utilisation. With 3 workers in prod, scaling kicks in at 800 TPS and prod CPU peaks <50%.
- **Auto-scale test report (2026-05-07, ref. `Proxy Auto Scale Test.pdf`):** ECS 2×0.5 vCPU/1GB ARM64, auto-scale Min 2 / Max 20 / target 70% CPU, mTLS enabled. **Sustained 1 367 RPS (peak ~1 500), zero errors, P95 336ms, P99 470ms**. Auto-scaling triggered at 80% CPU, added 2 tasks, CPU dropped to 45% with load distributed across 4 tasks. Validates production-readiness with substantial headroom (test capacity is ~60% above any plausible prod peak per the 60–140-user comparison rows in the report). Sub-second spike test still untested, but 5-min ramp behaviour confirmed.
- **Architectural risk (Sol/Omar 2026-05-06):** the backend APIs are rate-limited but the proxy itself is not. A misbehaving or malicious caller can still flood the proxy at high rate even when the backends return 429s. Worth flagging to RDG before go-live.
