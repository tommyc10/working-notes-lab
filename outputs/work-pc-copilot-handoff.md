# Copilot handoff: plan integration into the existing project

## Purpose

Use the framework files as requirements and have GitHub Copilot inspect the real project before proposing how to add article enhancement, scoring, review routing and evaluation.

This is a **planning handoff**, not a request to generate a new standalone codebase or immediately implement features. The existing application's language, architecture, data flow, conventions, security controls and deployment process are authoritative.

## Framework files to add to the work repository

Place these in an appropriate documentation/design location without replacing existing project instructions:

1. `production-article-scoring-and-enhancement-spec.md`
2. `ai-enhancement-and-scoring-prompts.md`
3. `article-enhancement.schema.json`
4. `article-assessment.schema.json`
5. `ai-ready-runbook-template.md`

Optional background:

- `ai-ready-runbook-standard.md`
- `confluence-to-ai-knowledge-blueprint.md`

Before adding real articles, confirm that the repository, Copilot configuration and eventual AI provider are approved for the relevant internal-data classification.

## What Copilot should inspect

Copilot should read the framework files and then inspect enough of the existing project to understand:

- repository instructions such as `README`, `AGENTS.md`, contribution guidance and architecture decisions;
- package manifests, runtime versions and dependency conventions;
- current Confluence extraction and article transformation flow;
- destination-system client, article schema and publishing workflow;
- existing domain models, validation, configuration and error handling;
- authentication, authorization, secrets and access-control handling;
- logging, telemetry, audit and data-retention patterns;
- queues, jobs, APIs, CLI commands or UI surfaces involved in migration;
- test structure, fixtures, mocks, CI/CD and deployment environments;
- any existing AI/model abstractions, prompt storage or evaluation tooling.

It must cite concrete project files, symbols and flows in the plan instead of guessing.

## Required plan output

The plan should contain:

1. **Current-state architecture** — how an article moves through the project today.
2. **Framework mapping** — where normalization, enhancement, scoring, decisioning, review and retrieval evaluation fit.
3. **Reuse versus new work** — existing components to extend and genuinely new components required.
4. **File-level change map** — likely files/modules to modify or add and why.
5. **Data contracts** — mapping between current models and the enhancement/assessment schemas.
6. **Enhancement levels** — how `assess_only`, `clean_up`, `restructure` and `expert_assisted` should be represented in backend, configuration and any UI.
7. **Scoring and blockers** — deterministic calculation, critical-dimension rules and routing ownership.
8. **AI integration boundary** — separate enhancement and scoring calls, provider abstraction, configuration and failure behavior.
9. **Human review flow** — where diffs, evidence, scores, gaps and approval decisions appear.
10. **Evaluation strategy** — synthetic tests, gold examples, retrieval tests and production outcome metrics.
11. **Security and governance** — source provenance, ACLs, secrets, prompt injection, audit and retention.
12. **Rollout and rollback** — shadow mode, supervised release, feature flags, monitoring and safe disablement.
13. **Unknowns and decisions** — questions that must be answered before implementation.
14. **Phased implementation backlog** — small ordered slices, dependencies and definition of done.

The plan must distinguish confirmed facts from assumptions and recommendations.

## Paste this into GitHub Copilot Chat

```text
I want you to plan how to integrate a new AI-assisted knowledge-article enhancement and scoring framework into this existing project.

Do not implement or edit code yet.

First, read all repository-specific instructions and then read these framework files completely:
- {{path}}/production-article-scoring-and-enhancement-spec.md
- {{path}}/ai-enhancement-and-scoring-prompts.md
- {{path}}/article-enhancement.schema.json
- {{path}}/article-assessment.schema.json
- {{path}}/ai-ready-runbook-template.md

Optional background, if present:
- {{path}}/ai-ready-runbook-standard.md
- {{path}}/confluence-to-ai-knowledge-blueprint.md

Next, inspect the existing codebase to understand its actual architecture and current article flow. Examine the relevant README/AGENTS/contribution guidance, package manifests, source modules, domain models, Confluence extraction, transformations, destination publishing, configuration, security, logging, tests, CI/CD and any existing AI abstractions.

Planning rules:
- The existing codebase and repository instructions are authoritative.
- Reuse existing modules and conventions where sensible; do not propose a separate greenfield application unless the codebase proves that isolation is necessary.
- Cite specific project files and symbols as evidence for architectural statements.
- Mark every statement as confirmed, assumed or recommended when the distinction matters.
- Identify contradictions between the framework and the current implementation.
- Do not invent destination APIs, permissions, schemas or model-provider capabilities.
- Do not include real credentials or confidential article content in the plan.
- Keep enhancement and scoring as separate AI operations.
- Final score calculation, blocker precedence and routing must be deterministic application logic, not an LLM decision.
- All content remains manual_only; do not plan operational tool execution in the initial release.
- Enhancement level is a discrete policy enum, not model creativity: assess_only, clean_up, restructure, expert_assisted.
- The application must set the maximum allowed enhancement level from risk/source trust; the model may not exceed it.

Produce a detailed integration plan with these sections:
1. Current-state article flow and architecture
2. Proposed end-to-end flow
3. Existing components to reuse
4. New components required
5. File/module-level change map
6. Data-model and schema mapping
7. Enhancement-level design, including UI/configuration if relevant
8. Deterministic scoring, blockers and routing
9. AI provider/prompt integration boundary and failure handling
10. Human review and approval experience
11. Test and evaluation strategy
12. Security, privacy, permissions and audit controls
13. Observability and production-value metrics
14. Rollout, feature flags and rollback
15. Open questions and decisions
16. Phased implementation backlog with dependencies and definition of done

For each implementation phase include:
- objective;
- affected existing files/modules;
- new files/modules if required;
- dependencies and decisions;
- tests and acceptance criteria;
- risks and rollback considerations.

End with:
- the five highest-priority questions for the project owner;
- the smallest useful first implementation slice;
- an explicit list of anything you could not verify from the repository.

Return the plan for review. Do not start coding until I approve it.
```

Replace `{{path}}` with the directory containing the transferred framework files.

## Review the Copilot plan before implementation

Reject or revise the plan if it:

- proposes a second standalone application without evidence;
- skips the existing migration/publishing flow;
- combines enhancement and scoring into one model call;
- lets the LLM calculate or enforce the final publication decision;
- treats enhancement level as unrestricted creativity;
- allows the model to exceed the maximum enhancement level;
- omits evidence mapping, source version/hash, hard blockers or human review;
- enables tool execution in the initial release;
- lacks retrieval evaluation and incident-value measurement;
- fails to identify concrete codebase files and integration points;
- starts coding before the plan is approved.

## After the plan is approved

Ask Copilot to implement only the smallest approved phase. Require tests and a reviewable diff before moving to the next phase. Re-run the gold evaluation set whenever prompts, models, schemas, scoring rules, normalization, chunking or retrieval configuration changes.
