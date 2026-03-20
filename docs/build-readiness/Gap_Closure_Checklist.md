# DRNT Gap-Closure Checklist

**Consolidated Build-Readiness Assessment**
March 2026 · ljefford2-cmyk / Local-AI-Orchestrator

---

## About This Document

This checklist consolidates findings from two independent adversarial reviews of the Local-AI-Orchestrator repository into a single prioritized build-readiness assessment. It defines what must be formalized before V1 build begins and what should be completed during hardening.

### How These Reviews Were Produced

Both reviews were generated using frontier AI models prompted to adopt specific expert perspectives and stress-test the architecture. **No external organizations participated in, endorsed, or were consulted for these reviews.** This is the same multi-model adversarial review methodology documented throughout this repository: independent AI-generated analyses from scoped expert personas, evaluated for convergence and divergence, with findings incorporated into architectural revisions.

The two review perspectives were:

- **Architectural Completeness Review** — Assessed specification tightness, structural weaknesses, internal contradictions, unstated assumptions, and what must be formalized for implementation readiness. Identified four priority formalization targets: the WAL state machine, Memory Object and Context Package schemas, Audit Event schema, and the job review contract.

- **Runtime Security & Threat Model Review** — Assessed the architecture from a runtime containment and sandbox security perspective, assuming hostile actors inside the trust boundary. Identified five active attack vectors: routing manipulation via prompt injection, Context Packager bypass through semantic encoding, WAL privilege escalation via sleeper strategy, audit log poisoning and storage exhaustion, and job state deserialization attacks.

### What This Document Is

- A prioritized gap-closure checklist with explicit acceptance criteria
- An honest accounting of what the architecture specifies well and what it does not yet specify tightly enough
- A dependency-ordered build sequence for converting a reference architecture into a buildable specification

### What This Document Is Not

- A claim that any external organization reviewed this repository
- A certification or endorsement of any kind
- A substitute for implementation testing and real-world validation

### The Shared Verdict

Both reviews reached the same conclusion independently:

> *The architecture is likely right. The specifications are not yet tight enough to prevent a weaker implementation from claiming compliance.*

This checklist converts that gap into actionable work items.

---

## Tier 1: Resolve Before V1 Build

These items are build-blocking. Without them, the architecture's core safety claims — graduated trust, structural privacy, immutable accountability, and human decision authority — cannot be enforced at runtime. Items are ordered by dependency: G-01 governs what actions are permitted, G-02 enforces routing decisions, G-03 governs what data leaves, G-04 records everything, G-05 protects state integrity, and G-06 keeps human review meaningful.

---

### G-01 · WAL State Machine · CRITICAL

**Sources:** Architectural Review §2 (WAL underspecified); Threat Model Finding 3 (sleeper attacks / privilege escalation)

**Finding**

WAL is conceptually strong but operationally underspecified. There is no formal state model, event schema, capability namespace, or promotion evidence ledger. Two teams could claim to implement WAL while building materially different safety postures. The threat model review identified a specific attack: a compromised agent processes hundreds of benign jobs flawlessly to accumulate positive evidence, earns WAL-3 autonomy, and then executes a malicious payload once human oversight is removed. Trust decay — where stale trust becomes a governance gap — remains an open problem.

**Required Deliverable**

Formal WAL state machine specification defining: states (WAL-0 through WAL-3, plus DEMOTED and SUSPENDED), transition events, guard conditions, timer-based trust decay, demotion precedence rules, model-change reset semantics, and evidence thresholds. A capability namespace binding WAL levels to bounded action classes — not to the agent "in general." Random mandatory human audit at WAL-3, so that no WAL level means zero visibility. Anti-gaming controls requiring multi-factor promotion criteria where consecutive successes alone are insufficient to justify promotion.

**Acceptance Criteria**

The state machine is machine-checkable. Two independent implementers produce equivalent safety behavior from the spec alone.

---

### G-02 · Deterministic Post-Classification Validator · CRITICAL

**Sources:** Architectural Review §1 (route-don't-reason enforcement gap); Threat Model Finding 1 (routing manipulation via prompt injection)

**Finding**

The local 7–13B model is the single point of failure for the entire signal chain. Prompt injection or adversarial perturbation can cause misclassification, routing sensitive data to incorrect destinations or trapping the router in recursive loops. The architecture specifies "route, don't reason" as a principle but does not yet specify it as an enforcement mechanism. No formal boundary prevents the local orchestrator from drifting into quasi-reasoning at runtime. Gray-zone activities — context selection, summarization for constrained displays, edge-case classification — are acknowledged but unguarded.

**Required Deliverable**

A deterministic rules engine that executes after the LLM classifies but before dispatch occurs. The validator checks: (a) the classified action is within the capability namespace for the request's WAL level, (b) the destination matches the routing policy for this action class, (c) the context package passes the structural privacy gate. The LLM cannot override or bypass this layer. Hardcoded heuristic checks overlay the model's routing decisions. A bounded benchmark suite for routing correctness defines failure thresholds.

**Acceptance Criteria**

The validator blocks a misclassified dispatch in test. A prompt injection that fools the LLM is caught by the deterministic layer.

---

### G-03 · Memory Object & Context Package Schemas · CRITICAL

**Sources:** Architectural Review §3 (Context Packager fragility), Unstated Assumptions (memory lifecycle); Threat Model Finding 2 (semantic bypass of Context Packager)

**Finding**

Memory objects lack stable confidence and shareability labels. These labels are lifecycle properties, not static metadata — facts age, summaries drift, user preferences change, and derived inferences contaminate descendants. The Context Packager is the most important and most fragile component in the architecture. Regex and keyword filtering are insufficient against semantic leakage. The most realistic breach vector is not exotic steganography but the simpler problem of misclassified "shareable" memory objects that leak context through the structural privacy gate. The architecture acknowledges this risk but has not formalized the defense.

**Required Deliverable**

Canonical Memory Object schema with required fields: provenance source, confidence score with source methodology, shareability class (local-only default), expiry/refresh policy, derivation lineage, and redaction history. Context Package schema defining the four-step pipeline output: semantic retrieval result, field-level allowlist enforcement result, sensitivity scan result, and token budget enforcement result. Shareability classification method — user-tagged for personal deployment, policy-driven for regulated contexts. Initial implementation uses simple stripping; tokenization is a maturity enhancement.

**Acceptance Criteria**

Every memory object in the vector DB conforms to schema. Context Package output is auditable against the four-step pipeline.

---

### G-04 · Audit Event Schema & Chain Semantics · HIGH

**Sources:** Architectural Review §4 (audit log not operational); Threat Model Finding 4 (log poisoning / storage exhaustion)

**Finding**

The audit log is well-motivated but more architectural than operational. The repository does not yet define the event schema, hash-chain semantics, key management, retention classes, redaction lineage, or replay procedure. The two-stream design (metadata plus optional encrypted content) is sound, but the policy for when content capture becomes mandatory is unspecified. The threat model identified that "append-only" without rate limits is an invitation to denial-of-service: a compromised agent can flood the log with millions of micro-jobs, burying malicious actions in noise or exhausting storage.

**Required Deliverable**

Canonical Audit Event schema: event type, timestamp, job ID, action class, WAL level at time of action, model/provider used, context package hash, routing decision, human decision (if applicable), and result hash. Hash-chain semantics where each event includes the hash of the previous event. Two-stream design with metadata stream (always on) and optional encrypted content stream, with documented policy specifying when content capture becomes mandatory. Rate limiting on log appends. Storage quotas with alerting. Retention classes with minimum 90-day policy.

**Acceptance Criteria**

An investigator can reconstruct the complete decision chain for any job from Stream 1 alone. Rate-limit test confirms flood resistance.

---

### G-05 · Job State Integrity · HIGH

**Sources:** Threat Model Finding 5 (deserialization attacks on queued job state)

**Finding**

Job queuing and resume during intermittent connectivity requires serialization of job state. Unsigned serialized state is vulnerable to tampering. If an attacker gains access within the execution environment, they can modify queued job states while the system is offline. When connectivity resumes, deserialization of corrupted state could lead to unauthorized dispatch of manipulated payloads to cloud models. This attack requires prior compromise but targets a core feature (intermittent connectivity resilience) and has a disproportionate blast radius.

**Required Deliverable**

Cryptographic signing of all serialized job state. Jobs are signed at serialization time using a local key. At deserialization, signature verification occurs before any processing. Failed verification halts the job and emits an audit event. The SQLite-backed job queue (per existing spec) includes integrity verification on resume.

**Acceptance Criteria**

A tampered job state file is rejected on resume. The rejection is logged as a security event.

---

### G-06 · Job Review Contract · HIGH

**Sources:** Architectural Review §Contradictions (job UX vs. human authority tension)

**Finding**

The job-based UX abstracts sub-actions to reduce approval fatigue — a sound design goal. But this creates a risk: once permissions are abstracted into jobs, the operator may lose visibility into the actual risky sub-actions taken within the batch. A multi-dispatch job that consults three cloud models and returns a single result technically has human-in-the-loop but functionally may not, if the human cannot see what each model received and returned. The design solves prompt fatigue by moving the approval surface upward but has not specified how much sub-action provenance must remain visible to keep the review meaningful.

**Required Deliverable**

Formal job review contract specifying what the human must see before approving a result: intended outcome, actual sub-actions taken, systems touched, data classes exposed to each destination, model/provider used for each sub-action, any gray-zone decisions made by the router, and the WAL level under which the job executed. For multi-dispatch jobs, each cloud dispatch is individually visible.

**Acceptance Criteria**

A user reviewing a multi-dispatch job result can identify which models were consulted, what data each received, and what sub-actions occurred.

---

## Tier 2: Formalize During Hardening

These items are important for production robustness but are not build-blocking for V1. They represent the gap between a working governed system and a hardened production system. Work on these should begin during V1 development and be completed before any deployment beyond the owner's personal use.

| ID | Component | Finding Summary | Deliverable |
|----|-----------|----------------|-------------|
| H-01 | Control-Plane Data Model | The repo has the right nouns but not enough hard objects. Job, Capability, WAL State, Memory Object, Context Package, Audit Event, Approval Decision, Policy Version, Model Version, and Incident all need canonical schemas and lifecycle rules. | Entity-relationship model for all control-plane objects with lifecycle state diagrams. Schema versioning policy. |
| H-02 | Deterministic Policy Evaluation Order | No explicit policy evaluation order. Different implementations will invent different safety behavior without an ordered state transition model. | Canonical pipeline: request classification → capability mapping → WAL check → context eligibility check → model routing → response schema validation → execution eligibility → audit emission. |
| H-03 | Network Perimeter Closure | The one genuinely unresolved finding from cross-domain validation. Remote access, exposure, and boundary trust are not fully defined. | Network architecture document: Tailscale mesh topology, default-deny egress, allowlisted cloud endpoints, container network isolation, and trust boundaries for the Watch → iPhone → Desktop chain. |
| H-04 | Memory Lifecycle Model | Memory labels are treated as static metadata but are actually lifecycle properties. Facts age, summaries drift, user preferences change, derived inferences contaminate descendants. | Memory lifecycle model with inheritance rules, expiration policies, dispute handling, revocation, and derived-object contamination tracking. |
| H-05 | Router Capability Envelope Benchmark | The architecture assumes a 7–13B model can reliably stay inside the router role. This is a design premise, not a measured capability. | Golden task suite for routing correctness: classification accuracy targets, false-positive/negative thresholds, adversarial prompt injection test cases, and regression suite run before any model swap. |
| H-06 | Privacy-Debuggability Policy | Tension between privacy minimization and debuggability. The content stream is optional and operator-controlled. Some incidents may be unreconstructable if the wrong stream was disabled. | Policy defining when encrypted content capture becomes mandatory (e.g., WAL-2+ actions, security events, anomaly-triggered recording windows). |
| H-07 | WAL-Sandbox Synchronization | Dynamic WAL permission elevation on top of static sandbox containment introduces a synchronization problem. Which layer is authoritative if they diverge? | Conflict-resolution model: sandbox policy is the ceiling, WAL is the floor. WAL can restrict below sandbox policy but never exceed it. Race condition handling for stale WAL states. |

---

## Cross-Reference: Review Findings → Checklist Items

This matrix traces every finding from both reviews to its corresponding checklist item. Findings that were independently identified by both reviews are noted.

| Review Finding | Checklist Item(s) |
|---------------|-------------------|
| Architectural Review §1: Route-don't-reason enforcement gap | G-02, H-05 |
| Architectural Review §2: WAL operationally underspecified | G-01 |
| Architectural Review §3: Context Packager fragility | G-03 |
| Architectural Review §4: Audit log not operational | G-04 |
| Architectural Review: Job UX vs. human authority tension | G-06 |
| Architectural Review: Privacy vs. debuggability tension | H-06 |
| Architectural Review: WAL vs. sandbox divergence | H-07 |
| Architectural Review: Missing control-plane data model | H-01 |
| Architectural Review: Missing policy evaluation order | H-02 |
| Architectural Review: Network perimeter unresolved | H-03 |
| Architectural Review: Memory lifecycle assumptions | G-03, H-04 |
| Architectural Review: Router capability assumptions | G-02, H-05 |
| Threat Model Finding 1: Routing manipulation via prompt injection | G-02 |
| Threat Model Finding 2: Context Packager semantic bypass | G-03 |
| Threat Model Finding 3: WAL gaming / sleeper attacks | G-01 |
| Threat Model Finding 4: Audit log poisoning / storage exhaustion | G-04 |
| Threat Model Finding 5: Job state deserialization attacks | G-05 |

---

## Recommended Build Sequence

The following sequence reflects dependency order. Each item depends on the items above it being at least in draft.

| # | Item | Rationale |
|---|------|-----------|
| 1 | G-01: WAL State Machine | All runtime permissions depend on WAL. Build this first so every subsequent component can reference WAL states and transitions. |
| 2 | G-03: Memory Object & Context Package Schemas | The Context Packager's privacy enforcement depends on well-typed memory objects. The validator (G-02) needs these schemas to check context eligibility. |
| 3 | G-02: Deterministic Post-Classification Validator | Depends on G-01 (WAL states) and G-03 (context eligibility rules). This is the enforcement layer that makes routing trustworthy. |
| 4 | G-04: Audit Event Schema & Chain Semantics | Depends on G-01 through G-03 being defined, because audit events must capture WAL state, routing decisions, and context package hashes. |
| 5 | G-05: Job State Integrity | Depends on the job model being defined (implied by G-04). Cryptographic signing is a bounded addition to the job queue. |
| 6 | G-06: Job Review Contract | Depends on all of the above. The review contract specifies what humans see, which requires all underlying objects to be defined. |
| 7 | H-01 through H-07 | Hardening items. Begin in parallel with V1 development. Complete before deployment beyond personal use. |

---

## Closing

When all Tier 1 items have acceptance criteria met, the repository transitions from reference architecture to buildable specification. The architecture's safety claims become enforceable, not aspirational. Tier 2 items convert a buildable specification into a hardened production system.

---

*This document was produced as part of the Local-AI-Orchestrator project's multi-model adversarial review methodology. Both source reviews were AI-generated from scoped expert perspectives. No external organizations participated in or endorsed these reviews. The value is in the analytical rigor of the process and the convergence of independent findings, not in the identity of the reviewers.*

