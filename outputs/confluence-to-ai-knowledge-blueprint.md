# Confluence-to-AI incident knowledge blueprint

## Known context and working assumptions

- Source: Confluence.
- Destination: an internal incident knowledge system used to ground an AI incident agent.
- Current agent capability: retrieval, citation, and recommendations; no tool execution yet.
- Intended future capability: bounded tool calls and eventual incident remediation.
- Scope: a broad mixture of incident types, services, and teams.
- Scale: about 1,300 articles already migrated and at least another 1,000 expected.
- Available source metadata includes owner, version, estimated reading time, Confluence space/team, and last-updated date.
- Good, bad, and high-risk examples will be supplied later from the work environment.

The exact Confluence edition (Cloud or Data Center), destination API/schema, permissions model, and whether migration is one-off or continuously synchronized still need confirmation.

## Key design decision

Maintain two independent readiness states:

```text
Knowledge ready
  Safe and useful for retrieval, citation, and recommendations today.

Automation ready
  Safe enough to be registered as a future executable workflow.
```

An article may be `knowledge_ready` and `manual_only`. Tool execution must never be inferred from publication in the knowledge base.

Recommended fields:

```yaml
knowledge_readiness: draft | staged | knowledge_ready | blocked
automation_readiness: not_applicable | manual_only | tool_specified | approval_tested | autonomous_eligible
agent_mode_allowed: retrieval_only | recommendation | tool_preview | approval_execution | autonomous_execution
```

## Target pipeline

```text
Confluence
  -> immutable source snapshot + migration manifest
  -> normalize bodies, macros, tables, code, links, and attachments
  -> preserve effective permissions
  -> classify document type, service, risk, and source-trust tier
  -> deduplicate and detect conflicts
  -> deterministic validation
  -> AI extraction/enhancement with evidence mapping
  -> Knowledge Readiness Score
  -> human review queue based on risk and confidence
  -> staging index
  -> positive and negative retrieval tests
  -> production knowledge index
  -> monitoring, source-drift detection, feedback, and revalidation

Future executable subset only:
  -> tool contract conversion
  -> Automation Readiness Score
  -> SME/change-authority approval
  -> sandbox/dry-run tests
  -> separately controlled tool registry
```

## Confluence extraction contract

Capture at least the following for every page:

- Confluence page ID, title, status, URL, version number, version author/date/message, created date, owner/author, space ID/key/name, parent ID and full ancestor path.
- Labels, content properties used by the organization, source body representation, outgoing links, and referenced page IDs.
- Attachments with IDs, filenames, media types, versions, checksums, source links, and extracted text where appropriate.
- Page-level restrictions plus effective restrictions inherited from parents and the space.
- Macro inventory, especially include/excerpt, page properties, code, expand, table, panel, Jira/ticket, diagram, and attachment macros.
- A source content hash, extraction timestamp, migration pipeline version, and destination article ID.

Do not treat estimated reading time as a quality signal. It is useful for user experience and workload estimates, but neither a short nor long article is inherently better.

### Confluence-specific failure modes

- A page may look complete only because it includes another page through a macro.
- Tables may be flattened incorrectly, separating conditions from actions.
- Screenshots may contain the only command, error, or button sequence.
- Attachments may be versioned independently from the page.
- links may be relative, restricted, obsolete, or point to an old page version.
- code blocks must retain whitespace and characters exactly.
- a page may inherit a restriction from any ancestor; copying only direct restrictions can expose content.
- duplicate articles often exist across spaces with different owners and dates.
- personal spaces, archived spaces, and pages owned by inactive staff need special treatment.

The normalizer should output both an unchanged raw snapshot and a cleaned, structured representation. Never discard the source version used to generate the destination article.

## Source-trust tiers

AI cannot independently prove that a Confluence statement is technically true. Use governance metadata to decide how much human validation is required.

| Tier | Example evidence | Treatment |
|---|---|---|
| A — governed | Approved operational space, active owner, current supported version, recent successful reuse or formal approval | Eligible for streamlined structural review and sampling |
| B — owned | Active team/owner and reasonably current, but no validation/reuse evidence | Owner/SME review before score 4 publication for consequential content |
| C — uncertain | Old, personal space, inactive owner, missing version, sparse evidence | Stage and require SME validation |
| D — conflicting/unsafe | Contradicts active content, contains secrets, unsafe actions, or inaccessible dependencies | Block/quarantine |

Source trust informs provenance; it does not replace content scoring or retrieval testing.

## Treatment of the 1,300 already migrated

Do not remigrate everything blindly. Build a reconciliation audit:

1. Match every destination article to a Confluence page ID and version.
2. Compare source and destination hashes or normalized content fingerprints.
3. Report pages changed in Confluence after migration.
4. Check missing or altered metadata, parent paths, labels, attachments, links, macros, and permissions.
5. Identify destination articles with no source, multiple destinations for one source, or multiple sources merged into one destination.
6. Classify document type, source-trust tier, and risk.
7. Run deterministic checks and KRS scoring.
8. Run retrieval tests against the existing production/staging index.
9. Route only failures, risky items, and low-confidence cases to humans.

Recommended audit outputs:

- `unchanged_and_passed`
- `source_changed_since_migration`
- `destination_transformation_loss`
- `missing_acl_or_metadata`
- `duplicate_or_conflict`
- `needs_owner_or_sme`
- `blocked_security_or_safety`
- `retrieval_failure`

This makes the 1,300 pages a usable baseline and potential training/evaluation corpus instead of assuming they are either perfect or unusable.

## Treatment of the remaining 1,000+

Process incrementally by space/team and risk, not as a single bulk release.

Suggested batch sequence:

1. High-volume, well-owned operational spaces with mostly informational or read-only procedures.
2. Common remediation runbooks, published as `manual_only` until automation review exists.
3. Sparse, older, or poorly owned spaces.
4. Security-sensitive, destructive, customer-communication, financial, identity, data, and broad production actions.

Start with batches of 50–100 while calibrating. Promote larger batches only after source mapping, transformation, score, permission, and retrieval checks are consistently passing.

## Human-review strategy at this scale

A human line-by-line review of every page is expensive, while fully automated technical approval is not defensible. Use risk-based routing:

- Review 100% of conflicts, security/ACL failures, secrets, unknown owners, low-confidence scores, and consequential procedures.
- Review 100% of anything proposed for future tool execution.
- Review all KRS results close to the boundary, for example 3.5–4.2, during calibration.
- Sample trusted, low-risk, clearly structured articles by space, team, document type, age, and score band.
- Increase sampling for teams or spaces with high defect rates; reduce it only after sustained evidence.
- Feed reviewer corrections back into the template, validation rules, taxonomy, and scoring examples.

The reviewer should approve technical truth; AI should do extraction, structural rewriting, gap detection, evidence mapping, deduplication suggestions, and first-pass scoring.

## Retrieval evaluation set

The existing incident records are the most valuable source of realistic queries. Build a labeled dataset containing:

- incident title and initial description;
- alerts, error codes, log phrases, service/component, environment, and version;
- article actually used, if known;
- successful resolution and verification evidence;
- articles considered but rejected;
- whether escalation was appropriate;
- near-neighbor incidents with similar symptoms but different causes.

For each knowledge article, create positive queries and negative/near-neighbor queries. Test retrieval separately from final-answer quality. Report by service, team, article type, risk, source-trust tier, and content age so an overall average cannot hide a weak domain.

## Capability roadmap

### Stage 0 — current: retrieval and recommendation

- Agent retrieves only `knowledge_ready` content the user is authorized to see.
- Response cites article ID, version, source, and relevant section.
- Agent distinguishes facts from suggestions and escalates on missing/conflicting evidence.
- No article can cause a tool call.

### Stage 1 — structured action preview

- Selected runbooks are converted to atomic tool contracts.
- Agent produces a proposed plan, parameters, expected effects, approval requirement, and rollback without executing.
- Human compares the preview with the runbook and actual incident context.

### Stage 2 — read-only tools

- Allowlisted diagnostic calls run under a dedicated, least-privilege agent identity.
- Tool input/output schemas are validated outside the model.
- Rate, timeout, resource, and data-scope limits are enforced.
- Results are logged and compared with expected outcomes.

### Stage 3 — approval-gated changes

- Reversible, bounded actions require explicit human approval with incident context, parameters, impact, and rollback.
- Approval and tool execution are recorded in the audit trail.
- Failed verification triggers rollback or escalation, not improvised retries.

### Stage 4 — bounded autonomy

- Only repeatedly successful, low-risk, reversible runbooks become `autonomous_eligible`.
- High-impact, irreversible, broad-scope, identity, data-deletion, financial, security-containment, and external-communication actions remain approval-gated.
- Runtime policy—not article prose—decides whether execution is permitted.

## Project workstreams and deliverables

### 1. Governance and taxonomy

- content types and required fields;
- Knowledge and Automation Readiness rubrics;
- source-trust tiers and action-risk tiers;
- lifecycle, ownership, review, archive, and conflict policy.

### 2. Data and migration

- source-to-destination field map;
- migration manifest and reconciliation report;
- macro/attachment/link/ACL handling rules;
- incremental sync and deletion/archive behavior.

### 3. AI enhancement and scoring

- evidence-bound extraction prompt;
- enhancement prompt producing a source-backed diff;
- scoring prompts with structured JSON;
- deterministic validators;
- calibrated gold set from supplied good, bad, and high-risk examples.

### 4. Retrieval and agent evaluation

- labeled incident-query dataset;
- positive, negative, version, access, and adversarial tests;
- retrieval, groundedness, correctness, abstention, and citation metrics;
- regression suite run before index releases.

### 5. Security and future execution

- permission/identity model;
- allowlisted tool registry and schemas;
- action tiers and approval workflow;
- sandbox, dry-run, rollback, audit, and shutdown controls.

### 6. Operations and reporting

- dashboards by space/team/risk/status;
- failure and override feedback loop;
- source-drift alerts;
- review queue and service-level targets;
- article/index version rollback.

## Suggested immediate next steps

1. Confirm whether Confluence is Cloud or Data Center and document the current extraction method.
2. Export the field/schema definition of the destination knowledge system.
3. Produce a manifest for the 1,300 migrated records with Confluence page ID/version and destination ID.
4. Select 30–50 representative articles, including supplied good, bad, and high-risk examples.
5. Add 30–50 historical incidents that should retrieve those articles, including near-neighbor failures.
6. Calibrate the KRS rubric with two human reviewers before batch scoring.
7. Audit a stratified sample of the 1,300 to estimate transformation, permission, duplicate, and quality defect rates.
8. Process the next 50–100 articles through the complete staging and retrieval-test pipeline.
9. Keep all remediation content `manual_only` until the separate automation programme reaches at least structured action preview.

## Definition of done for the planning phase

- Source and destination field/permission mapping is approved.
- Document types, KRS/ARS rubrics, hard blockers, source-trust tiers, and action-risk tiers are agreed.
- Existing 1,300-page reconciliation method is defined.
- Good/bad/high-risk gold examples and historical incident queries are labeled.
- Human-review routing and publication authority are assigned.
- Metrics and initial thresholds are agreed.
- Future tool-execution boundary is documented and separated from knowledge publication.
- Pilot batch, owners, test environment, and rollback path are selected.

## Open questions

1. Is the source Confluence Cloud or Data Center/Server?
2. Is this a one-off migration, scheduled refresh, or near-real-time synchronization?
3. How were the 1,300 existing pages extracted, transformed, and linked back to source IDs/versions?
4. Does the destination preserve per-user/group access controls at retrieval time?
5. What body format, metadata fields, API, maximum article size, and indexing/chunking behavior does the destination support?
6. Which Confluence macros and attachment types occur most often?
7. Are the incident records linked to the article that resolved them, or can that be inferred from comments/work notes?
8. Which teams own publication approval and technical validation across the different incident domains?
9. What is the acceptable review workload and publication throughput?
10. Are there internal information-classification, audit, change-management, or retention rules the destination must enforce?

## Relevant Confluence references

- [Confluence Cloud REST API v2 pages](https://developer.atlassian.com/cloud/confluence/rest/v2/api-group-page/)
- [Confluence permissions and restrictions](https://support.atlassian.com/confluence-cloud/docs/what-are-confluence-cloud-permissions-and-restrictions/)
- [Confluence page restrictions and inheritance](https://support.atlassian.com/confluence-cloud/docs/add-or-remove-page-restrictions/)
