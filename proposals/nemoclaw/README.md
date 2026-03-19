# Governance Patterns for NemoClaw

**Complementary architectural patterns for agent behavior governance inside the OpenShell sandbox.**

---

## The Gap

NemoClaw and OpenShell solve runtime containment — kernel-level isolation, network namespace enforcement, credential management, and process sandboxing. These are necessary and well-engineered infrastructure controls.

What remains unaddressed is **behavioral governance**: how does an agent earn the right to take progressively more autonomous action? How is context curated to minimize data exposure when the agent legitimately needs to reach cloud models? How do you maintain an immutable record of every decision the agent made and every decision the human made about the agent's output?

Industry analysts have identified this gap: runtime security is well-served by OpenShell, but observability, policy enforcement, rollback, and audit trails need to be embedded throughout the development lifecycle, not just at the runtime layer.

NemoClaw provides the sandbox. These patterns govern what happens inside it.

## Four Patterns

Each pattern was designed around a specific failure mode discovered through adversarial red-teaming, not a feature wishlist.

### 1. Workflow Autonomy Levels (WAL 0–3)

**Failure mode addressed:** Uncalibrated risk thresholds ship errors on day one. Default autonomy settings are either too permissive (agent acts without oversight) or too restrictive (agent is useless).

**Pattern:** Graduated trust earned through demonstrated reliability. All new capabilities start at WAL-0 (human reviews everything). Promotion requires measurable evidence — minimum transaction count, approval rate, zero sensitive data mishandling. Anomalies trigger automatic demotion. Model version changes reset the trust clock.

**NemoClaw integration point:** WAL can layer over OpenShell's existing policy YAML. Instead of static allow/deny rules, WAL provides a dynamic state machine that adjusts agent permissions based on operational history recorded in the audit log.

### 2. Context Packager with Memory Selection Policy

**Failure mode addressed:** Agent retrieves "relevant" context from memory and attaches it to a cloud API call. The retrieval inadvertently includes sensitive material. The Privacy Router sanitizes known PII patterns but cannot detect contextual sensitivity.

**Pattern:** A four-step sequential pipeline (receive → retrieve → redact → assemble) with a defined sensitivity taxonomy and memory selection policy. Memory objects carry confidence scores and shareability classifications. Only explicitly tagged "shareable" context is eligible for outbound payloads. Default is local-only.

**NemoClaw integration point:** The Context Packager can operate as a preprocessing layer before payloads reach the `inference.local` Privacy Router. The Packager handles semantic-level context curation; the Privacy Router handles transport-level credential injection and PII stripping. Complementary, not redundant.

### 3. Append-Only Audit Log (Two-Stream Design)

**Failure mode addressed:** Sycophantic convergence in multi-model rebuttal. When models see each other's responses, they converge toward agreement regardless of position strength. Without an immutable record of original positions, artificial consensus masks real conflicts.

**Pattern:** Cryptographically hashed, append-only journal with two-stream separation. Stream 1 (routing metadata) captures every decision without sensitive content — sufficient for accountability, WAL governance, and anomaly detection. Stream 2 (optional, encrypted) captures full payloads for debugging, with separate retention and access controls.

**NemoClaw integration point:** OpenShell already generates telemetry. The two-stream audit log provides the structured accountability layer that endpoint security and AI defense platforms can consume for behavioral analysis. It also provides the evidence base that WAL's state machine depends on.

### 4. Job-Based UX with Human Decision Loop

**Failure mode addressed:** Operator approval fatigue. The TUI surfaces blocked network requests; operators rubber-stamp approvals to unblock workflows. The agent exploits this pattern.

**Pattern:** Interactions are jobs, not chat sessions. Submit → Acknowledge → Process → Deliver. Background processing with asynchronous notification. Every job logged with full provenance. The human reviews results, not permission prompts — reducing the cognitive surface area for approval fatigue.

**NemoClaw integration point:** The job model can wrap OpenShell's TUI approval flow. Instead of surfacing individual network requests for real-time approval, the job model batches related requests into
