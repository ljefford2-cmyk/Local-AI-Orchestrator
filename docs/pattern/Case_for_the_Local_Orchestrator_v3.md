# The Case for the Local Orchestrator

**A Reference Architecture for Local-First AI Agent Orchestration**

v3.0 — Reconciled Edition — March 2026

---

## 1. The Problem

Current AI agent frameworks share default assumptions that produce predictable failure modes. Most route every request to a cloud model with full context, creating privacy exposure as a feature rather than a bug. Most offer binary trust: fully autonomous or fully manual, with no mechanism for earning autonomy through demonstrated reliability. Most treat memory as an append-only dump with no provenance, no lifecycle governance, and no distinction between high-confidence facts and low-confidence inferences.

The local orchestrator pattern addresses these defaults by separating the **acting** concern (deterministic, verifiable, local) from the **reasoning** concern (quality-sensitive, cloud-escalated). A local model handles classification and dispatch. Cloud models handle complex work. The user retains decision authority.

## 2. Core Principle: Route, Don't Reason

A local model in the 7–13B parameter range can reliably perform a bounded set of tasks: parsing user intent into structured commands, selecting which cloud model to route a request to, assembling curated context packages, validating structured responses against schemas, and dispatching deterministic operations. It cannot reliably generate substantive analysis, evaluate competing arguments, or make judgment calls with real-world consequences. The orchestrator draws a hard line between these categories.

### Operational Boundary Definition

*New in v3.0 — formalizes what "route, don't reason" means in practice.*

**Permitted (routing):** Intent classification, request/response formatting, memory retrieval by tag/category, rule-based sensitive data stripping, job state management, API dispatch, deterministic template fills, schema validation of cloud responses.

**Prohibited (reasoning):** Generating substantive answers, semantic evaluation of content sensitivity, evaluating competing model outputs, making judgment calls about ambiguous requests, performing any task where the local model's output becomes the user-facing answer.

**The gray zone:** Some orchestration tasks approach the boundary: context selection, output summarization for constrained displays, edge-case classification. The architecture handles these with deterministic rules and templates wherever possible. Where model inference is required, the decision is logged with a gray-zone flag for future evaluation.

## 3. The Context Packager

Context curation is the orchestrator's highest-value function and primary privacy mechanism. Cloud models never see the complete memory layer. Sensitive data is protected through a layered, sequential pipeline.

### The Pipeline

The Context Packager is a distinct architectural component, separated from the routing model. It executes four operations in sequence, each producing an audit log entry:

1. **Receive** a structured task specification from the router (task type, required context classes, target model).
2. **Retrieve** relevant context from the memory store using the task specification's context class tags, applying the memory selection policy.
3. **Redact:** scan structured fields and unstructured content for sensitive data. Strip or mask per the configured sensitivity taxonomy.
4. **Assemble** the outbound payload within the target model's token budget. Re-scan the assembled payload for inadvertent sensitive content inclusion before transmission.

### Sensitivity Taxonomy Method

The specific categories of sensitive data vary by deployment context. Healthcare deployments define categories from HIPAA's 18 Safe Harbor identifiers. Legal deployments define categories from privilege and confidentiality obligations. Federal deployments define categories from CUI/SSI classification. Personal deployments define categories from the user's own privacy preferences.

The method is consistent across all contexts: define categories, assign detection rules (regex, keyword lists, tag matching), define the default action per category (always strip, strip unless task requires, never strip), and define the escalation path when the Packager must make a judgment call (surface to user at WAL-0).

### Memory Selection Policy

Not all stored context is eligible to leave the local perimeter. The Packager applies a memory selection policy:

**Confidence scoring:** Each memory object carries a confidence score based on source type. User-stated facts score highest. Inferences from documents score medium. Summaries of summaries score lowest. The Packager preferentially includes high-confidence objects in outbound payloads. This also provides defense against memory poisoning — information injected through external sources inherits lower trust.

**Shareability classification:** Memory objects are classified as either eligible for cloud transmission or local-only. The classification method depends on deployment context: in regulated environments, classification follows the applicable data governance policy. In personal deployments, the user explicitly tags what can leave the perimeter. Default is local-only.

### The Tokenization Question

When the Packager redacts sensitive data, it can either strip it entirely (simpler, more private, reduces payload utility) or replace it with reference tokens that preserve payload structure while masking identity. Token substitution requires a secure local mapping vault and careful lifecycle management. Early deployments should start with simple stripping and add tokenization as a maturity enhancement.

## 4. Workflow Autonomy Levels

*New in v3.0 — WAL governance framework made explicit.*

Every new capability starts at WAL-0: human reviews every output before any action. Trust is earned through demonstrated reliability, not granted by default. Anomalies trigger automatic demotion.

| Level | Name | Behavior |
|-------|------|----------|
| WAL-0 | Recommend Only | Agent suggests. Human reviews and decides every time. All new capabilities start here. |
| WAL-1 | Draft Artifacts | Agent drafts. Human reviews and approves before execution. |
| WAL-2 | Execute with Pre-Approval | Pre-approved routine action classes only. Scope is narrow and bounded. |
| WAL-3 | Limited Autonomous | Earned through sustained zero-incident reliability. Automatic demotion on any anomaly. |

### Promotion Criteria

Promotion to a higher WAL level requires measurable thresholds: minimum transaction count, minimum approval rate, zero sensitive data mishandling, and a documented workflow definition. Specific thresholds are set per deployment context. Multi-factor validation is required — consecutive successes alone do not justify promotion, to prevent conditioning attacks.

### Demotion Triggers

Any output causing real-world harm: immediate demotion to WAL-0. Error rate exceeding threshold: demote one level. Sensitive data mishandling: immediate WAL-0. Underlying model changes (version, provider, configuration): demote to WAL-0 and restart the promotion clock. Demotion is not failure — it is the governance system working correctly.

## 5. Audit Log Design

*New in v3.0 — resolves the replay-vs-privacy tension.*

Every routing decision, packaging decision, model request/response, and user decision is logged in an append-only, cryptographically chained journal (`chattr +a`). The agent cannot modify or delete its own logs.

The log uses two separate streams to resolve the tension between replayability and privacy:

**Routing metadata stream:** Event type, timestamp, job ID, routing decision, target model, context classes requested, redaction actions taken, packager rule version. Enables replay of routing decisions without containing sensitive content. Sufficient for WAL governance, anomaly detection, and accountability.

**Content stream (optional, encrypted):** Pre-redaction and post-redaction payloads for debugging. Encrypted at rest. Separate retention and access controls. Enabled only when the operator opts in.

## 6. Validation Methodology

This architecture has been stress-tested through multi-model adversarial review across four implementation domains: federal regulatory enforcement, clinical AI governance, a small-to-medium business framework, and a personal-scale gateway. Each domain applied the same architectural pattern to its own regulatory and operational context.

Adversarial reviews were conducted independently by multiple frontier AI models. Findings from each review were incorporated into successive versions, with a cross-domain reconciliation pass tracing every structural concern back across all implementations to distinguish documentation debt from architectural debt.

This process is itself a differentiator: multi-model adversarial review as a development methodology, not just a validation step. The architecture is stronger because it has been attacked from multiple directions by models with different training biases and failure modes.

## 7. What This Architecture Is Not

It is not a product. It is a reference pattern that can be implemented with different technology stacks for different contexts. It is not a replacement for domain-specific compliance work — the architecture provides a foundation that makes compliance more tractable, not automatic. It is not an argument against cloud AI — the entire architecture depends on cloud models for reasoning. It is an argument for local control over what context reaches those models and what actions their outputs can trigger.
