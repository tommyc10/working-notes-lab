---
article_id: ""
source_system: ""
source_id: ""
source_url: ""
source_updated_at: "YYYY-MM-DDThh:mm:ssZ"
content_hash: ""
title: "[Service/component] – [observable symptom or intent] – [environment if needed]"
document_type: "knowledge_article | diagnostic_playbook | remediation_runbook"
service: ""
component: ""
environment: []
versions: []
trigger_signals: []
user_phrases: []
questions_answered: []
exclusions: []
risk_tier: "read_only | notify | approval_required | prohibited"
approved_agent_actions: []
required_permissions: []
owner: ""
technical_approver: ""
status: "imported | ai_enhanced | sme_validated | sandbox_tested | published | flagged | archived"
knowledge_readiness: "draft | staged | knowledge_ready | blocked"
automation_readiness: "not_applicable | manual_only | tool_specified | approval_tested | autonomous_eligible"
agent_mode_allowed: "retrieval_only | recommendation | tool_preview | approval_execution | autonomous_execution"
version: "1.0"
last_validated_at: "YYYY-MM-DD"
review_by: "YYYY-MM-DD"
access_classification: ""
related_articles: []
---

# Objective

State one observable outcome.

# Issue and symptoms

- User-visible symptom:
- Alert/event name:
- Exact error or log text:
- Impact:
- Severity indicators:

# Environment

- Product/service:
- Component:
- Version/build:
- Deployment/region/tenant scope:
- Relevant recent changes:

# Applicability

## Use this when

- Observable condition:
- Required evidence:

## Do not use this when

- Similar symptom with different cause:
- Excluded environment/version:
- Condition requiring a different runbook or escalation:

# Cause

Confirmed cause, or `Unknown/not required`. Include the source of confirmation.

# Prerequisites and inputs

- Required incident fields:
- Required evidence/logs:
- Required tools:
- Minimum permissions and resource scope:
- Dependencies:
- Backup/snapshot/maintenance-window requirement:

# Safety and authorization

- Risk tier:
- Permitted actions:
- Prohibited actions:
- Human approval required from:
- Maximum blast radius:
- Stop conditions:
- Evidence-preservation requirement:

# Diagnosis and decision logic

1. If `<machine-checkable condition>`, go to `<step/runbook>`.
2. If `<machine-checkable condition>`, go to `<step/runbook>`.
3. If evidence is missing, conflicting, or outside scope, stop and escalate with the evidence collected.

# Procedure

Delete this section for a purely informational knowledge article.

## Step S1 — [single objective]

- **Run if:**
- **Inputs and allowed formats:**
- **Tool/API/script:**
- **Exact operation and parameters:**
- **Minimum permission and resource scope:**
- **Risk tier:**
- **Approval:**
- **Expected result:**
- **Success condition:**
- **On success:**
- **On failure:**
- **Retry limit:**
- **Timeout:**
- **Side effects/blast radius:**
- **Rollback/compensating action:**
- **Evidence to record:**

## Step S2 — [single objective]

- **Run if:**
- **Inputs and allowed formats:**
- **Tool/API/script:**
- **Exact operation and parameters:**
- **Minimum permission and resource scope:**
- **Risk tier:**
- **Approval:**
- **Expected result:**
- **Success condition:**
- **On success:**
- **On failure:**
- **Retry limit:**
- **Timeout:**
- **Side effects/blast radius:**
- **Rollback/compensating action:**
- **Evidence to record:**

# Verification

- Final service/system state:
- Metrics/logs/status to verify:
- Observation window:
- User or monitoring confirmation:
- Evidence to attach to incident:

# Rollback and recovery

- Rollback trigger:
- Exact rollback action:
- Approval required:
- Expected restored state:
- Verification:
- If rollback fails:

# Escalation and handoff

- Escalate when:
- Durable team/queue/on-call route:
- Severity and response target:
- Context to provide: incident ID, service/environment, article ID/version, steps attempted, outputs, timestamps, approvals, and rollback state.

# Known limitations

-

# Related content

- Alternative/near-neighbor runbook:
- Dependency/reference:
- Supersedes/is superseded by:

# Validation record

- Validated by:
- Validation method: incident reuse | sandbox | dry run | peer review
- Test environment:
- Test date:
- Result/evidence link:
- Known deviations:

# Change history

| Version | Date | Change | Author/approver | Reason/evidence |
|---|---|---|---|---|
| 1.0 | YYYY-MM-DD | Initial migration |  |  |

# Knowledge publication assessment

| Dimension | Weight | Score (1–5) | Evidence | Gaps/remediation |
|---|---:|---:|---|---|
| Technical correctness and provenance | 25% |  |  |  |
| Applicability and diagnosis | 20% |  |  |  |
| Resolution completeness | 20% |  |  |  |
| Retrieval and machine structure | 20% |  |  |  |
| Ownership and freshness | 10% |  |  |  |
| Security and access | 5% |  |  |  |

- **Knowledge Readiness Score:**
- **Critical knowledge dimensions all at least 4:** Yes / No
- **Hard blockers:**
- **Positive retrieval tests passed:**
- **Negative/near-neighbor retrieval tests passed:**
- **Knowledge publication decision:** Publish / Stage / Quarantine / Block

# Automation assessment

Complete only for content intended to drive future tool calls.

| Dimension | Weight | Score (1–5) | Evidence | Gaps/remediation |
|---|---:|---:|---|---|
| Atomic executability | 25% |  |  |  |
| Inputs, outputs, and branching | 20% |  |  |  |
| Safety, permissions, and approval | 25% |  |  |  |
| Rollback, recovery, and escalation | 15% |  |  |  |
| Testing, evidence, and observability | 15% |  |  |  |

- **Automation Readiness Score:**
- **All automation dimensions at least 4:** Yes / No
- **Sandbox/dry-run passed:** Yes / No / N/A
- **SME/change-authority approval:**
- **Automation decision:** Manual only / Tool preview / Approval execution / Autonomous eligible
