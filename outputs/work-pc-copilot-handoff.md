# Work-PC handoff: build the scoring and enhancement MVP

## Files to transfer

Transfer these files through an approved company method:

1. `production-article-scoring-and-enhancement-spec.md`
2. `ai-enhancement-and-scoring-prompts.md`
3. `article-enhancement.schema.json`
4. `article-assessment.schema.json`
5. `ai-ready-runbook-template.md` — optional reference for the rendered article

The broader research and migration blueprint are optional background. The five files above are the implementation pack.

These files contain no company article content. Before adding real examples, confirm that the repository, GitHub Copilot configuration and selected model/API are approved for internal data at the relevant classification.

## Recommended repository structure

Use the same language and framework as the existing migration tool. If there is no established stack, Python is a reasonable prototype choice.

```text
article-quality-pipeline/
  README.md
  docs/
    production-spec.md
    article-template.md
  schemas/
    article-enhancement.schema.json
    article-assessment.schema.json
  prompts/
    enhancement.md
    scoring.md
    retrieval-query-generation.md
  src/
    models/
    validation/
    normalization/
    enhancement/
    scoring/
    decision/
    evaluation/
    cli/
  tests/
    fixtures/synthetic/
    unit/
    integration/
    golden/
  evals/
    rubric.json
    expected/
  config/
    example.config
```

Do not commit real articles, tokens, API keys, personal data or production URLs until the repository and secret-management approach are approved.

## Build order

### Milestone 1 — deterministic local skeleton

Build a CLI or local service that:

1. accepts one normalized article JSON file;
2. validates required source metadata;
3. runs deterministic checks;
4. validates mock enhancement output against the enhancement schema;
5. validates mock assessment output against the assessment schema;
6. calculates the weighted score in code;
7. applies blockers and the routing policy deterministically;
8. writes an auditable result bundle.

Do not call an LLM or publish anything in this milestone.

### Milestone 2 — provider-agnostic AI adapters

Add separate interfaces for:

- enhancement;
- scoring judge A;
- scoring judge B;
- retrieval-query generation.

Keep model name, endpoint, prompt version, temperature and credentials in approved configuration. Make the pipeline work with mock providers so tests do not require network access.

### Milestone 3 — synthetic evaluation

Create synthetic fixtures for:

- a complete score-5 article;
- a human-usable but agent-weak score-3 article;
- a poor article below 3;
- an article containing a secret;
- an article with prompt-injection-like instructions;
- an ambiguous near-neighbor incident;
- a high-risk procedure missing approval or rollback;
- an article whose AI enhancement invents a command or threshold.

Tests must prove that blockers override averages and that enhancement cannot introduce unsupported material.

### Milestone 4 — work examples and calibration

Only after approval, add anonymised good, bad and high-risk examples plus incident queries. Two human reviewers create expected scores and decisions. Store expected outputs as a versioned gold set.

### Milestone 5 — staging integration

Connect read-only Confluence extraction and destination staging. Do not grant production publishing access. Run the pipeline in shadow mode, compare results with human reviewers and record disagreements.

### Milestone 6 — supervised production

Enable publishing only through an explicit approval action. Promotion must record source version/hash, enhanced diff, model/prompt/schema versions, scores, blockers, retrieval tests and reviewer identity.

## Paste this into GitHub Copilot Chat

```text
You are helping implement a production-bound incident knowledge article quality pipeline.

First read these files completely:
- docs/production-spec.md
- docs/article-template.md
- prompts/enhancement.md
- prompts/scoring.md
- schemas/article-enhancement.schema.json
- schemas/article-assessment.schema.json

Important constraints:
- Use the repository's existing language and conventions.
- Do not add Confluence, destination-system or production publishing integrations yet.
- Do not add real company data, secrets or credentials.
- Keep AI enhancement and AI scoring as separate interfaces and calls.
- The deterministic application code, not the LLM, must calculate the weighted score and final routing decision.
- A hard blocker must override the numeric score.
- A score must not be rounded to cross the 4.0 publication threshold.
- All content remains manual_only; this project must not execute operational tool calls.
- AI-produced JSON must be validated against the supplied schemas.
- Every processing result must record source hash/version and model, prompt, rubric and schema versions.
- The implementation must support mock AI providers for deterministic tests.

Start with Milestone 1 only:
1. Inspect the repository and propose the smallest compatible module structure.
2. Identify contradictions or unresolved design decisions in the supplied specification before coding.
3. Implement domain models and JSON-schema validation.
4. Implement the weighted Knowledge Readiness Score calculation using weights 25%, 20%, 20%, 10%, 15%, 5%, 5%.
5. Implement deterministic routing and hard-blocker precedence.
6. Add a CLI that reads one synthetic candidate/assessment bundle and writes a result bundle.
7. Add unit tests for threshold boundaries, critical-dimension failures, blockers and invalid schemas.
8. Update the README with local commands and a clear statement that no LLM or production integration exists yet.

Do not begin Milestone 2 until Milestone 1 tests pass and I approve the design.
```

## Milestone 1 acceptance tests

- Valid schema and score 4.0 with all critical dimensions at least 4 routes to approval/testing, not directly to uncontrolled publication.
- Score 3.999 does not pass.
- Score above 4 with one critical dimension at 3 does not pass.
- Score 5 with a hard blocker routes to `BLOCKED`.
- Missing source ID/version/hash fails validation.
- Duplicate or missing dimension entries fail validation or deterministic completeness checking.
- Model-reported weighted score is ignored or verified against the code-calculated score.
- Unsupported decision values and change classes fail validation.
- Result bundle contains complete version/audit metadata.

## Decisions to make at work before Milestone 2

- Existing migration-tool language and repository.
- Approved AI model/provider and internal-data policy.
- Confluence Cloud versus Data Center and the approved authentication method.
- Destination staging API and article schema.
- Access-control mapping and security owner.
- Final publication approver and review queue.
- Where prompts, model settings, evaluation data and audit records will be versioned.
- Acceptable per-article cost, latency and review time.
