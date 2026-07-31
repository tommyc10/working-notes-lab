# Production specification: AI article scoring and enhancement

## Scope

This specification covers new Confluence articles moving into the internal incident knowledge system.

It delivers:

- AI-assisted restructuring and metadata enrichment;
- a consistent 1–5 Knowledge Readiness Score;
- hard safety and governance blockers;
- human-review routing;
- retrieval testing before publication;
- production monitoring tied to incident outcomes;
- fields that preserve a path to future tool execution without enabling it now.

Out of scope for this version:

- re-auditing the 1,300 articles already migrated;
- production tool execution or autonomous remediation;
- treating an AI score as proof of technical truth without provenance or human accountability.

## Product outcome

The system should answer four questions for every candidate article:

1. Is it grounded in an accountable source?
2. Can the agent identify the incidents to which it does—and does not—apply?
3. Does it contain enough information to give a correct, useful resolution?
4. Can it be retrieved reliably and published without creating a security or access problem?

The final decision is not based on score alone:

```text
PUBLISH when:
  Knowledge Readiness Score >= 4.0
  AND every critical dimension >= 4
  AND no hard blocker exists
  AND deterministic checks pass
  AND positive and negative retrieval tests pass
  AND the required human approval is recorded
```

`Above 3` therefore means **4.0 or higher**, without rounding.

## Processing architecture

```text
1. Extract source
   -> 2. Normalize
   -> 3. Deterministic pre-checks
   -> 4. Classify
   -> 5. AI enhancement
   -> 6. Deterministic post-checks
   -> 7. Independent AI scoring
   -> 8. Human-review routing
   -> 9. Staging retrieval tests
   -> 10. Publish and monitor
```

### 1. Extract source

Preserve an immutable source snapshot containing:

- Confluence page ID, URL, version, title, body, owner, author, created/updated dates, space/team, parent path and labels;
- attachments, referenced pages, macros, links and code blocks;
- page and inherited access restrictions;
- extraction timestamp, source hash and pipeline version.

### 2. Normalize

- Remove navigation, repeated banners, comments and unrelated page chrome.
- Preserve headings, ordered steps, code whitespace, table relationships and link targets.
- Resolve or record include/excerpt dependencies.
- Extract meaningful text from supported attachments and images, while retaining the source link.
- Produce both `raw_source` and `normalized_source`; never overwrite the evidence copy.

### 3. Deterministic pre-checks

Check before calling an LLM:

- required source identifiers and owner/team;
- readable body and supported encoding;
- source version and hash;
- access-control data;
- secrets, tokens, credentials and obvious personal data;
- broken or inaccessible critical links/attachments;
- empty pages, placeholder pages and obvious duplicates;
- malformed code blocks or lost table cells.

Failed security/access checks go directly to `BLOCKED`. Other failures go to `REWORK` with machine-readable reasons.

### 4. Classify

Assign:

```yaml
document_type: knowledge_article | diagnostic_playbook | remediation_runbook
content_risk: low | medium | high | critical
source_trust: governed | owned | uncertain | conflicting
change_class_allowed: editorial_only | source_backed_semantic | substantive_sme_required
automation_readiness: not_applicable | manual_only
agent_mode_allowed: retrieval_only | recommendation
```

No item can become executable in this version.

### 5. AI enhancement

The enhancement call receives the normalized source as untrusted data and returns:

- a structured article using the canonical template;
- a precise title, short summary, symptoms, environment, positive and negative applicability, resolution and verification;
- user-language phrases, exact error/alert strings, keywords and questions answered;
- source-backed restructuring of procedure steps;
- missing-information questions for the owner/SME;
- duplicate/conflict candidates;
- an evidence map linking each material field to its source location;
- a change log that classifies every edit.

The model must use `MISSING` or `UNVERIFIED` rather than inventing a fact.

#### Enhancement change classes

| Class | Examples | Publication treatment |
|---|---|---|
| A — editorial | Headings, spacing, grammar, formatting, deterministic field extraction | Can be streamlined after calibration |
| B — source-backed semantic | Paraphrase, summary, synonyms, questions answered, reordered source-backed steps | Review by evidence comparison; sampling may be possible later |
| C — substantive | New step, command, cause, threshold, permission, scope, expected result, rollback or escalation rule | AI may only flag the gap; an SME must supply and approve the content |

### 6. Deterministic post-checks

Validate the enhanced output against the schema and confirm:

- all material claims have source evidence or are marked missing/unverified;
- no source-backed value changed meaning;
- no new command, number, threshold, version or permission appeared without evidence;
- required headings and metadata exist;
- links remain traceable;
- procedure branches point to valid steps;
- no secrets, prompt-policy overrides or unapproved execution instructions were introduced;
- destination ACL fields match the source policy.

### 7. Independent AI scoring

The scorer must run in a separate model call from enhancement. It receives:

- raw and normalized source;
- enhanced candidate;
- source-trust classification;
- deterministic check results;
- the locked scoring rubric.

It returns only structured assessment JSON. It must not repair the candidate while scoring it.

For the pilot, run two independent scoring passes. Human review is required if:

- any dimension differs by more than 1 point;
- the judges disagree about a blocker or decision;
- either confidence is below the agreed threshold;
- the final score is close to the publication boundary;
- the content is high/critical risk.

## Knowledge Readiness Score

| Dimension | Weight | Critical? |
|---|---:|:---:|
| Technical correctness and provenance | 25% | Yes |
| Incident matchability and applicability | 20% | Yes |
| Resolution completeness and usefulness | 20% | Yes |
| Operational clarity | 10% | No |
| AI retrieval readiness | 15% | Yes |
| Ownership and freshness | 5% | No; required fields are blockers |
| Security and access | 5% | Yes |

```text
KRS = sum(dimension score × weight) / 100
```

### Rating anchors

#### Technical correctness and provenance — 25%

- **1:** unsupported, contradictory, wrong version, or no accountable source.
- **3:** plausible and partly sourced, but material claims or validation are uncertain.
- **4:** all material claims map to the source; active owner/team and applicable version are clear; no unresolved conflict.
- **5:** score 4 plus successful incident reuse, formal validation or equivalent outcome evidence.

#### Incident matchability and applicability — 20%

- **1:** generic topic with no usable symptom or environment.
- **3:** basic issue and environment exist, but similar incidents could select the wrong article.
- **4:** exact signals, user symptoms, environment, `use when`, `do not use when` and distinguishing conditions are explicit.
- **5:** score 4 plus validated positive and near-neighbor incident tests.

#### Resolution completeness and usefulness — 20%

- **1:** no usable resolution or materially dangerous/incomplete advice.
- **3:** knowledgeable human could use it, but steps, failure handling or verification require interpretation.
- **4:** complete path from diagnosis to resolution with expected outcome, verification and escalation/failure handling.
- **5:** score 4 plus successful use evidence and complete outcome capture.

#### Operational clarity — 10%

- **1:** vague, contradictory or disordered.
- **3:** understandable but contains implicit decisions or ambiguous references.
- **4:** concise, ordered, explicit terminology and atomic manual steps where applicable.
- **5:** score 4 plus consistently structured branches, evidence capture and reusable step contracts.

#### AI retrieval readiness — 15%

- **1:** generic title, missing metadata and unchunkable body.
- **3:** reasonable title and tags, but weak user vocabulary, identifiers or section context.
- **4:** precise title, service/environment, exact signals, synonyms, questions answered, chunkable headings and inherited metadata.
- **5:** score 4 plus passed positive, negative and near-neighbor retrieval tests.

#### Ownership and freshness — 5%

- **1:** no accountable owner, version or useful date.
- **3:** owner and updated date exist but review state or supported version is unclear.
- **4:** active owner/team, source version, validation date and review trigger are recorded.
- **5:** score 4 plus automatic source-drift/change detection and a proven update history.

#### Security and access — 5%

- **1:** secret/PII exposure, unsafe content or incorrect audience.
- **3:** no known exposure but classification or ACL mapping is incomplete.
- **4:** security scan passes and effective source access is preserved in retrieval metadata.
- **5:** score 4 plus successful unauthorized-user and adversarial-content tests.

Ratings 2 and 4 represent performance between the adjacent anchors. Every score needs cited evidence and a specific gap/remediation statement.

## Hard blockers

- Missing stable source ID, source version, owner/team or source snapshot.
- Material claim conflicts with an active source and precedence is unresolved.
- Secret, credential, token, prohibited personal data or access-control failure.
- Content contains instructions attempting to alter agent policy, bypass controls, reveal data/prompts or invoke tools.
- Destination cannot enforce the source audience or restrictions.
- Unsupported version or inaccessible dependency changes the resolution materially.
- AI introduced a substantive operational fact not grounded in the source.
- A dangerous or irreversible recommendation lacks an explicit human decision point, warning, scope and escalation.
- The incident cannot be distinguished from a plausible near-neighbor with a materially different resolution.

## Routing policy

| Condition | Route |
|---|---|
| Blocker present | `BLOCKED`; security/owner queue |
| KRS below 3.0 | `REWORK_OR_REJECT`; owner decides rewrite, merge or retire |
| KRS 3.0–3.99 | `ENHANCE_AND_REVIEW`; return gaps to owner/SME and re-score |
| KRS 4.0–4.49 | `HUMAN_APPROVAL`; publish only after review and retrieval tests |
| KRS 4.5–5.0, high/critical risk or Class C change | `HUMAN_APPROVAL` |
| KRS 4.5–5.0, low/medium risk, governed source, only Class A/B edits | `STREAMLINED_REVIEW_ELIGIBLE` after pilot calibration |

During the initial production pilot, require human approval for every publication. Selective streamlined publication should be enabled only after measured agreement with reviewers and zero critical false passes in the calibration set.

## Retrieval test gate

Generate at least:

- three positive queries using different language;
- one exact alert/error query when available;
- two near-neighbor queries that should select a different article;
- one out-of-scope environment/version query;
- one missing-evidence query for which the correct behavior is abstention/escalation.

Suggested initial gates:

- correct article in top 5 for at least 90% of positive tests;
- wrong article not selected for at least 95% of near-neighbor/out-of-scope tests;
- 100% permission enforcement in authorized/unauthorized tests;
- at least 90% grounded and correct generated answers;
- at least 95% correct abstention/escalation when required evidence is absent.

These are pilot targets and should be tuned against actual incident risk and baseline performance.

## Production value measurement

The score is a content-control metric, not the business outcome. Prove value at three levels.

### Pipeline value

- processing cost and time per article;
- human review minutes per article;
- percentage requiring SME input;
- first-pass and eventual publication rate;
- most common source gaps by team/space.

### Agent value

- top-K retrieval success on labeled incidents;
- groundedness, correctness, completeness and citation accuracy;
- wrong-article selection and correct abstention rates;
- agent/user feedback tied to article ID and version.

### Incident value

- percentage of incidents where a retrieved article contributed to resolution;
- resolution success without additional knowledge search;
- change in time to diagnosis and time to resolution;
- reopen, escalation and incorrect-recommendation rate;
- repeated failure by article, team, service and version.

Use a staged comparison or A/B design where operationally acceptable. Compare incidents with the enhanced corpus against a baseline, controlling at least for service, severity and incident type.

## Rollout

### Phase 1 — gold-set calibration

- Select 30–50 good, bad, borderline and high-risk articles.
- Have two reviewers score independently, then resolve disagreements.
- Create positive and negative incident queries.
- Tune prompts/rules until score agreement is acceptable and critical false passes are zero.

### Phase 2 — shadow processing

- Process 50–100 new articles without publishing automatically.
- Compare AI enhancement, scores, blockers and decisions with human review.
- Measure defects by dimension and source space/team.

### Phase 3 — supervised production

- Publish only human-approved score-4+ content.
- Record source, enhanced diff, score evidence, reviewer and retrieval results.
- Monitor incident outcomes and roll back article/index versions when necessary.

### Phase 4 — selective streamlining

- Streamline only low/medium-risk, governed-source, editorial/source-backed changes.
- Keep Class C changes and high-risk content human-approved.
- Re-run regression tests on model, prompt, rubric, schema, chunking or index changes.

## Minimum production components

- Confluence extractor and immutable evidence store.
- Normalizer for bodies, tables, code, links, macros and attachments.
- Deterministic validation and security checks.
- Enhancement service returning schema-valid JSON and a source-backed diff.
- Independent scoring service returning evidence-bound JSON.
- Human review queue showing source, candidate, diff, blockers, scores and retrieval tests.
- Staging and production indexes with versioned promotion/rollback.
- Evaluation dataset and regression runner.
- Audit store and dashboard tied to article ID/version and incident outcomes.

## Production acceptance criteria

- Every destination article is traceable to an immutable Confluence page ID/version/hash.
- Every material enhancement is source-backed or clearly marked missing/unverified.
- Every score contains evidence and actionable remediation.
- Hard blockers deterministically override numeric score.
- Published articles meet KRS and retrieval gates.
- Effective access controls are enforced at retrieval.
- Reviewer decisions, prompt/model/schema versions and publication events are auditable.
- The gold set shows zero critical false publications.
- Production monitoring connects article versions to retrieval and incident outcomes.
- All content remains `manual_only`; no tool execution is enabled by this release.
