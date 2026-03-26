# The Case for the Local Orchestrator

**A reference architecture for local-first AI agent orchestration**

**Companion repository** — focused on how these governance patterns may apply to agent sandbox architectures like [NemoClaw/OpenShell](https://github.com/NVIDIA/NemoClaw).

The canonical architecture is maintained at **[local-first-ai-orchestration](https://github.com/ljefford2-cmyk/local-first-ai-orchestration)**.

---

## Core Principle

**Know what you have. Package what you need.**

The local model reasons about two things: *Do I already have this?* And if not, *what does the cloud need to see?* The boundary is not between reasoning and not-reasoning — it is between reasoning where errors are recoverable and reasoning where errors are consequential.

This principle, and the governance patterns that follow from it, were developed and stress-tested independently of any specific runtime platform. They are offered here as a possible path for closing governance gaps in agent orchestration systems — not as a prescription.

## Governance Patterns

Each pattern exists because a specific adversarial scenario demanded it:

**Workflow Autonomy Levels (WAL 0–3)** — Graduated trust earned through demonstrated reliability. All capabilities start at WAL-0 (human reviews everything). Promotion requires evidence. Anomalies trigger automatic demotion. v5.2 adds temporal dynamics (trust scores decay over inactivity, recent performance is weighted more heavily than historical volume) and failure-type weighting (routing errors carry less trust impact than policy violations or external damage). The result is a governance model that responds proportionally to deviation — not just binary allow/deny.

**Context Packager** — Structural privacy gate with a defined sensitivity taxonomy, memory selection policy, and sequential audit pipeline. Cloud models never see your complete memory layer. Sensitive data is stripped by rule, not by judgment. v5.2 adds adversarial input handling: hard separation between instructions and data in every outbound payload, defending against prompt injection, malicious connector data, and memory poisoning.

**Execution Safety Layer** — Governs what happens when a decision becomes an action. Network isolation at the container topology level (not firewall rules), tool sandboxing with constrained permissions, connector trust levels (read-only / write-guarded / privileged), and WAL-gated execution that inherits the trust level of the capability that triggered it. Includes a formal threat model covering malicious connectors, compromised API endpoints, tool supply-chain attacks, and sandbox escape. Added in v5.2.

**Append-Only Audit Log** — Cryptographically hashed, immutable journal with two-stream separation (routing metadata + optional encrypted content). The agent cannot modify or delete its own logs. Source for all implementation metrics: local resolution rate, context reduction ratio, redaction success rate, human override frequency, retry rate, and trust demotion frequency.

**Job-Based UX** — Interactions are jobs, not chat sessions. Submit → Acknowledge → Process → Deliver. Jobs queue during intermittent connectivity and resume without losing state.

## Validation

The architecture has been stress-tested through multi-model adversarial review (Claude, ChatGPT, Gemini, Copilot) across five implementation domains:

| Domain | Context | Regulatory Alignment |
|--------|---------|---------------------|
| Federal Regulatory | Departmental orchestrator for a large federal agency | FedRAMP, FISMA, CUI/SSI |
| Clinical | AI orchestration in a clinical environment | HIPAA, FDA CDS, PHI |
| K-12 Education | District-level sovereign AI deployment | FERPA, COPPA, IDEA |
| SMB | Six-document business adaptation framework | GLBA, attorney-client privilege |
| Personal (DRNT) | Single-user gateway with self-imposed governance | Privacy-by-architecture — no external mandate |

Cross-domain reconciliation traced nine structural findings across all implementations, separating documentation debt from architectural debt. The most significant finding: all four reviewers independently identified that the original core principle ("route, don't reason") created a structural tension with what the architecture actually does — driving the v5.0 reframe to the current principle. v5.2 was driven by a structured redline review that identified the absence of execution governance as the most significant remaining gap.

Full validation details: [Adversarial Review Methodology](https://github.com/ljefford2-cmyk/local-first-ai-orchestration/blob/main/validation/adversarial-review-methodology.md)

## For NemoClaw / OpenShell Developers

These patterns are designed to complement NemoClaw's sandbox architecture, not compete with it.

NemoClaw solves runtime containment — kernel-level isolation, network namespaces, credential management. These patterns address a different question: **what governs the agent's behavior inside that sandbox?**

Graduated trust, privacy-aware context packaging, immutable audit trails, and human decision authority are control-plane concerns that exist regardless of how the execution boundary is enforced. If they are useful to the NemoClaw community, that advances the solutions we all need.

→ See `proposals/nemoclaw/` for the specific governance proposal.

## Repository Structure

```
├── README.md                          ← You are here
├── docs/
│   ├── pattern/
│   │   └── Case_for_the_Local_Orchestrator_v3.md  ← Pattern specification (v3)
│   ├── adversarial-reviews/
│   │   └── README.md                  ← Adversarial review methodology and findings
│   ├── build-readiness/
│   │   └── Gap_Closure_Checklist.md   ← Consolidated build-readiness assessment
│   └── cross-domain-validation/
│       └── README.md                  ← Gap map summary and reconciliation results
└── proposals/
    └── nemoclaw/
        ├── README.md                  ← NemoClaw governance proposal overview
        └── NemoClaw_Governance_Proposals.pptx
```

**Note:** The pattern specification in this repo is v3. The current version (v5.2) is maintained in the [canonical repository](https://github.com/ljefford2-cmyk/local-first-ai-orchestration/blob/main/docs/the-case-for-the-local-orchestrator.md), which includes the post-adversarial reframe, Execution Safety Layer, implementation metrics, and all domain applications.

## License

MIT — use these patterns however they serve you.

## Author

Lawrence Jeffords · Nahunta, Georgia
