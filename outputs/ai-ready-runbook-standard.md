# AI-ready incident knowledge standard

## Executive recommendation

Use hard gates plus two separate readiness scorecards, not one subjective score:

1. **Hard gates** catch content that must never reach the production agent: unverified facts, ambiguous or unsafe actions, stale ownership, access-control failures, secrets, conflicts, and prompt-injection-like content.
2. **Knowledge Readiness Score (KRS)** determines whether content is safe and useful for retrieval, citation, and recommendations. It applies to every article now.
3. **Automation Readiness Score (ARS)** determines whether a particular runbook is safe enough to drive tool calls. It applies only when tool execution is enabled later.

The current retrieval-only publication rule should be:

```text
Publish to the production agent only when:
  all hard gates pass
  AND Knowledge Readiness Score >= 4.0
  AND each critical knowledge dimension >= 4
  AND retrieval tests pass
```

When tool calls are enabled, an executable runbook must also have an Automation Readiness Score of at least 4.0, every critical automation dimension at least 4, explicit SME/change-authority approval, and successful dry-run or sandbox evidence. A document can therefore be `knowledge_ready` while remaining `manual_only`.

This avoids two failures: blocking useful knowledge today because tools do not yet exist, and later mistaking well-written prose for safe executable authority. It also avoids the main failure of averages: excellent formatting cannot offset unsafe remediation steps.

Keep **readiness** separate from **business value**. A poor but frequently needed article should go to the front of the enhancement queue, not be discarded. A polished article for a rare issue may be publishable but low priority.

## Why the standard needs to be different for an AI agent

A human can resolve vague phrases such as “restart if needed,” infer which cluster is meant, notice a dangerous command, or ask a colleague. An agent needs explicit applicability, inputs, permissions, state transitions, success criteria, stop conditions, failure branches, and escalation behavior.

The design combines four bodies of practice:

- Knowledge-Centered Service (KCS): simple, consistent structure; issue, environment, resolution, cause; correct metadata; demand-led improvement; and “reuse is review.”
- Incident operations: prerequisites, detection, analysis, containment, eradication, recovery, expected outcomes, escalation, and tested rollback.
- Retrieval-augmented generation: structured chunks, inherited metadata, representative queries, retrieval evaluation, groundedness, completeness, relevance, and correctness.
- Agent safety: atomic actions, least privilege, policy-enforced permissions, risk-tiered human approval, auditability, input/output validation, and prompt-injection resistance.

## Content model: one core template, two extensions

Do not force every item into the same body. Use one core standard and classify each item as one of three document types:

1. **Knowledge article** — explains a symptom, cause, concept, or known issue; it can support diagnosis but is not itself an executable procedure.
2. **Diagnostic playbook** — branches through observations to identify scope or root cause; primarily read-only.
3. **Remediation runbook** — performs actions that change system state; requires the strongest safety and validation controls.

The core fields stay the same. Diagnostic and remediation content adds structured step blocks. This follows the KCS finding that most of a content standard can be shared while a smaller part is tailored to the domain.

## Canonical structure

### Required metadata

| Field | Purpose |
|---|---|
| `article_id` | Stable destination identifier; never recycle it |
| `source_system`, `source_id`, `source_url` | Provenance and traceability to the original |
| `source_updated_at`, `content_hash` | Change detection and migration lineage |
| `title` | `[Service/component] – [observable symptom or intent] – [environment if distinguishing]` |
| `document_type` | `knowledge_article`, `diagnostic_playbook`, or `remediation_runbook` |
| `service`, `component`, `environment`, `versions` | Retrieval filters and applicability |
| `trigger_signals` | Exact alert names, event IDs, error codes, log phrases, or symptoms |
| `user_phrases` | Non-technical terms people use to describe the issue |
| `questions_answered` | Representative natural-language queries this content should answer |
| `exclusions` | Similar situations in which the article must not be used |
| `risk_tier` | `read_only`, `notify`, `approval_required`, or `prohibited` |
| `approved_agent_actions` | Allowlisted action or tool names; never free-form authority |
| `required_permissions` | Minimum role/scope; do not include credentials |
| `owner`, `technical_approver` | Named accountable role or durable team alias |
| `status`, `version` | Lifecycle and version control |
| `last_validated_at`, `review_by` | Freshness and revalidation |
| `access_classification` | ACL/security inheritance at document and chunk level |
| `related_articles` | Dependencies, alternatives, superseded items, and known conflicts |

### Required body

1. **Objective** — one observable outcome.
2. **Issue/symptoms** — what the requester or monitoring system observes, including exact strings.
3. **Environment** — product, component, version, region, tenant, deployment type, and relevant recent changes.
4. **Use when / do not use when** — positive and negative applicability criteria.
5. **Impact and urgency** — what is affected and whether evidence must be preserved.
6. **Cause** — confirmed cause, or explicitly “unknown/not required.” Do not let AI infer one.
7. **Prerequisites and inputs** — tools, access, data, maintenance window, backups, dependencies, and required incident fields.
8. **Safety and authorization** — permitted actions, approval tier, blast-radius limit, stop conditions, and prohibited actions.
9. **Diagnosis/decision logic** — explicit conditions and branches.
10. **Procedure** — atomic step blocks for playbooks and runbooks.
11. **Verification** — observable end state, monitoring window, and evidence to attach to the incident.
12. **Rollback/recovery** — compensating action and confirmation of restored state.
13. **Escalation/handoff** — when, where, and what context to provide; use durable team routes rather than personal details where possible.
14. **Known limitations and related content**.
15. **Validation record and change history**.

### Atomic step contract

Every executable step should state:

```yaml
step_id: S1
objective: What this one step proves or changes
run_if: Machine-checkable condition
inputs:
  - name, type, allowed format, source
action:
  tool: Allowlisted tool/API/script
  operation: Exact operation
  parameters: Explicit values or constrained placeholders
permissions: Minimum role and resource scope
risk_tier: read_only | notify | approval_required | prohibited
approval: Required approver or none
expected_result: Exact output, status, metric, or state
success_condition: Boolean or measurable condition
on_success: Next step or finish
on_failure: Next diagnostic step, retry limit, or escalation
timeout: Maximum execution time
side_effects: Known effects and blast radius
rollback: Compensating action and its verification
evidence_to_record: Command result, query ID, timestamp, metric, or ticket field
```

Avoid “check the server,” “restart if necessary,” “repeat until fixed,” “use the usual account,” or “contact John.” These hide decisions that the agent cannot make safely or consistently.

## Readiness rubrics

Score each dimension from 1 to 5, using evidence in the article and authoritative linked sources. Missing evidence is a gap, not permission for the scoring model to infer an answer.

### Knowledge Readiness Score — applies now

| Dimension | Weight | What is evaluated | Critical? |
|---|---:|---|:---:|
| Technical correctness and provenance | 25% | Claims, versions, authoritative source, conflict handling, source trust, validation | Yes |
| Applicability and diagnosis | 20% | Trigger signals, environment, positive/negative scope, distinguishing conditions, branches | Yes |
| Resolution completeness | 20% | Complete answer or manual procedure, verification, failure handling, evidence capture | Yes |
| Retrieval and machine structure | 20% | Precise title, user vocabulary, exact signals, questions answered, chunkable headings, metadata, deduplication | Yes |
| Ownership and freshness | 10% | Owner, version, validation date, review trigger, change history | No, but key fields are hard gates |
| Security and access | 5% | No secrets/PII; source and destination ACLs correctly mapped | Yes |

```text
KRS = sum(knowledge dimension rating × dimension weight) / 100
```

### Automation Readiness Score — applies to future tool calls

| Dimension | Weight | What is evaluated | Critical? |
|---|---:|---|:---:|
| Atomic executability | 25% | Explicit tool/operation, bounded parameters, single-purpose steps, deterministic transitions | Yes |
| Inputs, outputs, and branching | 20% | Validated inputs, expected results, success/failure branches, retry and timeout limits | Yes |
| Safety, permissions, and approval | 25% | Least privilege, allowlisted tools, risk tier, approval, blast radius, stop/prohibited conditions | Yes |
| Rollback, recovery, and escalation | 15% | Tested compensation/rollback, restored-state verification, durable escalation route | Yes |
| Testing, evidence, and observability | 15% | Sandbox/dry-run or successful incident evidence, tool logging, audit fields, version tested | Yes |

```text
ARS = sum(automation dimension rating × dimension weight) / 100
```

### Rating anchors

| Rating | Meaning |
|---:|---|
| 1 | Missing, incorrect, contradictory, or actively unsafe |
| 2 | Major gaps; a specialist would need to reconstruct the answer |
| 3 | Usable by a knowledgeable human with interpretation; not reliable enough for an agent |
| 4 | Explicit, grounded, reviewable, and agent-ready; minor non-critical improvement remains |
| 5 | Production-proven: successfully tested/reused, measurable outcomes, complete evidence and safety controls |

Use 2 and 4 for content between the anchors. Require a reason and evidence for every dimension, not just a number.

### Hard blockers

Any one of these sets the decision to `BLOCK`, regardless of the weighted score:

- No authoritative source, owner, or technical validation for a material claim.
- Contradiction with another active source and no declared precedence.
- Unscoped destructive, security-sensitive, data-changing, customer-impacting, or externally communicating action.
- Missing approval, stop condition, expected result, or rollback for a consequential action.
- Secrets, tokens, passwords, personal data, or credentials embedded in content.
- Agent action exceeds its allowlisted tools, identity, resource scope, or the acting user's authority.
- Instructions that attempt to alter agent policy, reveal prompts/data, bypass approval, or invoke unapproved tools.
- Applicability cannot distinguish this incident from a plausible near-neighbor with a different remedy.
- Unsupported product/version, dead critical dependency, stale content with no accountable owner, or inaccessible source.
- Executable procedure has not been dry-run, sandbox-tested, or otherwise validated against a real successful incident.
- Required ACL/security metadata cannot be preserved in the destination and inherited by every chunk.

### Publication and execution decisions

| Result | Destination treatment |
|---|---|
| KRS 4.0–5.0, no blockers, critical knowledge dimensions ≥4 | Publish for retrieval/citation/recommendation |
| KRS 3.0–3.99 or one remediable critical gap | Stage; AI-enhance and send to owner/SME |
| KRS 1.0–2.99 | Quarantine; rewrite, merge, or retire |
| Any knowledge blocker | Block even if average is high |
| ARS below 4.0 or not assessed | Keep `manual_only`; never expose as an executable workflow |
| ARS 4.0–5.0, all automation dimensions ≥4, approved and sandbox-tested | Eligible for the separately controlled tool-execution registry |
| High-impact or irreversible action | Human approval remains mandatory regardless of ARS |

Do not round 3.95 to 4.0. If the policy is “above 3,” make 4.0 the explicit threshold for both scorecards.

## AI-assisted enhancement workflow

### 1. Preserve and classify

- Copy the original unchanged into a restricted evidence store.
- Record source ID, URL, timestamps, hash, ACL, author/owner, and attachments.
- Remove navigation, banners, repeated footers, and irrelevant UI text from the working copy.
- Classify document type, service/domain, risk tier, sensitivity, and likely duplicates.
- Scan attachments, hidden text, comments, metadata, and images for secrets and indirect prompt injection.

### 2. Extract, do not invent

Use AI to map source facts into the canonical fields. Require an evidence map from every extracted fact to the source location. The model must write `MISSING` or `UNVERIFIED` when the source does not provide the value.

The AI may safely propose:

- clearer titles and summaries;
- user-language synonyms and exact keywords;
- questions the article can answer;
- consistent product/version names;
- extraction of prerequisites, branches, commands, expected outputs, contacts, and links already present;
- restructuring into atomic steps;
- duplicate/conflict candidates;
- gap lists and questions for the SME.

The AI must not invent:

- commands, parameters, thresholds, account names, permissions, approvals, rollback actions, contacts, causes, success criteria, or supported versions;
- a fact merely because it is common practice or appears in another unapproved article.

### 3. Apply a bounded enhancement level

Enhancement level is a discrete policy choice, not a creativity setting:

| Level | Value | Permitted behavior |
|---:|---|---|
| 0 | `assess_only` | Score and identify gaps without changing article content |
| 1 | `clean_up` | Editorial cleanup and direct metadata extraction |
| 2 | `restructure` | Default: source-backed rewriting, sections, summaries, synonyms, applicability and reordered steps |
| 3 | `expert_assisted` | Deepest source-backed restructuring plus explicit SME placeholders/questions for missing technical facts |

The application calculates the maximum level from source trust, content risk and review policy. The model must record the requested, maximum and applied levels and may never exceed the maximum. No level permits invention. Future `automation_preparation` is a separate, locked workflow governed by the Automation Readiness Score.

### 4. Enhance with a source-backed diff

Generate a proposed revised article plus a change report using these classes:

- `editorial`
- `source_backed_semantic`
- `substantive_sme_required`

Every substantive addition needs a source citation. Keep the scorer and enhancer separate: a scoring model should not silently repair the article it is judging.

### 5. Run deterministic checks first

Before an LLM score, validate:

- required fields and allowed enum values;
- stable IDs, timestamps, version, owner, and review date;
- links and referenced attachments;
- duplicate IDs/content hashes and semantic duplicate candidates;
- code blocks, placeholder syntax, parameter schemas, and branch targets;
- secret/PII patterns and prohibited commands;
- ACL mapping and source access;
- each step has expected result and failure path;
- each consequential action has risk tier, approval, and rollback.

Deterministic failures are cheaper, reproducible, and should not be delegated to a probabilistic judge.

### 6. Use evidence-bound AI scoring

Run two independent scoring passes with the same locked rubric and low variability. Each pass returns structured JSON containing:

```json
{
  "dimension_scores": [
    {
      "dimension": "technical_correctness_and_provenance",
      "score": 1,
      "evidence": ["source location or exact field"],
      "gaps": ["specific missing or conflicting item"],
      "remediation": ["specific SME question or edit"]
    }
  ],
  "hard_blockers": [],
  "weighted_score": 1.0,
  "decision": "PUBLISH | STAGE | QUARANTINE | BLOCK",
  "confidence": 0.0
}
```

Adjudicate when the judges differ by more than one point on any dimension, disagree on a blocker, or have low confidence. Calibrate AI results against a human-scored “gold” set before allowing automation.

### 7. Test retrieval and execution separately

For each article create positive and negative test queries:

- exact alert/error wording;
- non-technical user wording;
- acronym and product-name variations;
- partial context;
- a closely related incident that should retrieve a different article;
- an out-of-scope version/environment;
- a malicious or irrelevant instruction embedded in retrieved content.

Measure retrieval before response generation. Then measure the end-to-end agent separately.

Suggested pilot targets—not universal standards—are:

- relevant article retrieved in top 5 for at least 90% of positive test queries;
- incorrect runbook not retrieved or selected for at least 95% of negative/near-neighbor queries;
- zero unauthorized or unapproved actions in the safety test set;
- at least 90% grounded/correct final responses;
- at least 85% successful completion in sandbox for eligible executable runbooks;
- at least 95% correct abstention or escalation when evidence is missing or conflicting.

Tune these after measuring incident risk, baseline performance, and the cost of false action versus false abstention.

### 8. Publish with continuous review

Use lifecycle states such as:

```text
Imported -> AI Enhanced -> SME Validated -> Sandbox Tested -> Published
                                                    |-> Restricted/Approval-only
Published -> Flagged/Stale -> Revalidated or Archived
```

At runtime record article ID/version, retrieved chunks, source citations, chosen actions, approvals, tool inputs/outputs, verification results, rollback, escalation, and human override. Feed real use back into the article. Archive rather than delete content that has been linked from incident records.

## Retrieval and destination-site design

- Keep the original text and a normalized/indexing representation; never lose provenance.
- Chunk by semantic headings and atomic procedure sections, not arbitrary character counts. Do not split a condition from its action, expected result, or rollback.
- Inherit article ID, title, service, environment, version, ACL, risk tier, owner, status, and source URL into every chunk.
- Index both exact identifiers and semantic representations. Error codes, alert names, commands, host patterns, and version numbers need keyword/exact-match support; narrative symptoms benefit from vector search.
- Include title, summary, keywords, tags, questions answered, source, and language as searchable/returnable metadata where the platform supports it.
- Do not adopt a universal chunk size without testing. If the platform needs a starting point, trial roughly 400–800 tokens with modest overlap, then tune against the gold query set.
- Exclude drafts, blocked articles, superseded versions, inaccessible content, and expired high-risk procedures from the production index.
- Apply permissions at retrieval and again at tool execution. A document cannot grant the agent authority.

## Governance model

| Role | Accountability |
|---|---|
| Knowledge owner/SME | Technical truth, applicability, successful validation, updates |
| Knowledge manager | Template, taxonomy, rubric, deduplication, coaching, lifecycle |
| AI/platform owner | Ingestion, chunking, retrieval, evaluation, observability, rollback of index/config |
| Security/risk owner | Action tiers, least privilege, approvals, injection testing, ACL enforcement |
| Incident responders | Flag/fix through use, capture missing branches and actual outcomes |

Use random monthly samples and targeted reviews of high-use, high-risk, changed, failed, or overridden content. A blanket annual review is not enough for volatile systems, while reviewing every unused article wastes effort. Revalidate on service/version/tool/permission changes, after failed use, and after related incidents.

## Rollout plan

### Phase 1 — Define and calibrate

- Agree document types, metadata enums, action risk tiers, and hard blockers.
- Select 30–50 representative source articles: good, bad, old, high-risk, duplicate, and contradictory.
- Have two SMEs score them independently and resolve disagreements into a gold set.
- Tune AI prompts/rules until false publication of unsafe content is zero on the gold set.

### Phase 2 — Pilot

- Migrate 25–50 high-value, relatively low-risk articles.
- Create positive and negative retrieval tests and sandbox scenarios.
- Run the agent in recommendation-only or shadow mode.
- Compare AI score, human score, retrieval, task success, unsafe-action rate, abstention, and time saved.

### Phase 3 — Controlled production

- Publish score 4+ content; keep mutating actions approval-gated.
- Monitor citations, tool actions, failures, overrides, incident outcomes, and rollback use.
- Re-score changed articles and run regression tests before index promotion.

### Phase 4 — Scale by risk and value

- Prioritize high incident volume × severity × potential time saved.
- Expand autonomous action only for bounded, reversible, repeatedly successful runbooks.
- Keep high-impact or irreversible actions human-approved regardless of article score.

## Metrics that matter

Track three levels separately:

**Content health**

- percentage passing hard gates and score 4+;
- gap categories, duplicates, conflicts, stale ownership, invalid links;
- time from import to validated publication.

**Retrieval and answer quality**

- Recall@K or top-K success on labeled queries;
- near-neighbor false-selection rate;
- groundedness, completeness, relevance, and correctness;
- correct abstention/escalation rate.

**Operational outcome and safety**

- incident task success, time to mitigation/recovery, reopens, human overrides;
- unauthorized/unapproved action rate;
- rollback and escalation rate;
- failures by article ID/version and by procedure step.

Do not use article page views or a thumbs-up count as the sole measure of value. Link usage to incident resolution where possible.

## Decisions needed before implementation

1. What are the source and destination platforms, and what fields, APIs, ACLs, versioning, and chunking controls do they support?
2. Will the agent only recommend/cite, or can it execute tools and change systems?
3. Which incident domains are in scope: application operations, infrastructure, end-user support, security response, or all of them?
4. What action/approval model already exists in change and incident management?
5. How many articles, in which formats/languages, and what is their current metadata quality?
6. Do you have linked incidents, usage counts, successful resolutions, postmortems, and SME ownership data for prioritization and test cases?
7. Which errors are worse in your context: failure to act, wrong article selection, or an incorrect action?
8. Who has final publication authority, and who can validate commands and rollback procedures?

## Research basis

- [KCS article structure](https://library.serviceinnovation.org/KCS/KCS_v6/KCS_v6_Practices_Guide/030/040/010/020)
- [KCS content standard](https://library.serviceinnovation.org/KCS/KCS_v6/KCS_v6_Practices_Guide/030/040/010/040)
- [KCS content health indicators and Content Standard Checklist](https://library.serviceinnovation.org/KCS/KCS_v6/KCS_v6_Practices_Guide/030/040/010/065)
- [KCS article state and lifecycle](https://library.serviceinnovation.org/KCS/KCS_v6/KCS_v6_Practices_Guide/030/040/010/030)
- [AWS incident response playbook guidance](https://docs.aws.amazon.com/wellarchitected/2025-02-25/framework/sec_incident_response_playbooks.html)
- [AWS operational playbook guidance](https://docs.aws.amazon.com/wellarchitected/latest/framework/ops_ready_to_support_use_playbooks.html)
- [Microsoft incident response strategy and playbooks](https://learn.microsoft.com/en-us/security/zero-trust/security-adoption-discipline-security-operations-incidents)
- [Google SRE emergency response](https://sre.google/sre-book/emergency-response/)
- [Microsoft RAG chunk enrichment](https://learn.microsoft.com/en-us/azure/architecture/ai-ml/guide/rag/rag-enrichment-phase)
- [Microsoft RAG evaluation](https://learn.microsoft.com/en-us/azure/architecture/ai-ml/guide/rag/rag-llm-evaluation-phase)
- [AWS Agentic AI predictable task execution](https://docs.aws.amazon.com/wellarchitected/latest/agentic-ai-lens/agentrel02.html)
- [AWS Agentic AI human oversight](https://docs.aws.amazon.com/wellarchitected/latest/agentic-ai-lens/agentsec04-bp02.html)
- [AWS Agentic AI testing](https://docs.aws.amazon.com/wellarchitected/latest/agentic-ai-lens/agentops06-bp01.html)
- [OWASP prompt injection prevention](https://cheatsheetseries.owasp.org/cheatsheets/LLM_Prompt_Injection_Prevention_Cheat_Sheet.html)
- [NIST AI Risk Management Framework](https://www.nist.gov/itl/ai-risk-management-framework)
- [NIST SP 800-61 Rev. 3 incident response recommendations](https://csrc.nist.gov/pubs/sp/800/61/r3/final)
