# The Case for the Local Orchestrator

**A reference architecture for local-first AI agent orchestration.**

Route, don't reason. The local model classifies and dispatches. Cloud models do the thinking. The human decides.

---

## What This Is

A set of architectural patterns for governing autonomous AI agents — designed around failure modes, not feature wishlists. Each pattern exists because a specific adversarial scenario demanded it:

- **Workflow Autonomy Levels (WAL 0–3)** — Graduated trust earned through demonstrated reliability. All capabilities start at WAL-0 (human reviews everything). Promotion requires evidence. Anomalies trigger automatic demotion.
- **Context Packager** — Structural privacy gate with a defined sensitivity taxonomy, memory selection policy, and sequential audit pipeline. Cloud models never see your complete memory layer.
- **Append-Only Audit Log** — Cryptographically hashed, immutable journal with two-stream separation (routing metadata + optional encrypted content). The agent cannot modify or delete its own logs.
- **Job-Based UX** — Interactions are jobs, not chat sessions. Submit → Acknowledge → Process → Deliver. Jobs queue during intermittent connectivity and resume without losing state.

## Validation

This architecture has been stress-tested through multi-model adversarial review across four implementation domains:

| Domain | Context | Regulatory Alignment |
|--------|---------|---------------------|
| Federal | Departmental orchestrator service | FedRAMP, CUI/SSI |
| Clinical | Clinical AI orchestration framework | HIPAA, PHI |
| SMB | Six-document business adaptation | GLBA, privilege |
| Personal | Single-user gateway | Privacy-by-architecture |

Adversarial reviews conducted independently by multiple frontier AI models. Cross-domain reconciliation traced nine structural findings across all implementations to separate documentation debt from architectural debt. Four resolved, four partially addressed, one genuinely new.

### Build Readiness

Two independent adversarial reviews — an architectural completeness 
assessment and a runtime security threat model — have been consolidated 
into a [Gap-Closure Checklist](docs/build-readiness/Gap_Closure_Checklist.md) 
that defines what must be formalized before build begins. Both reviews 
were AI-generated using the same multi-model adversarial methodology 
documented in this repository. No external organizations participated 
in or endorsed these reviews.

## Repository Structure

```
├── README.md                          ← You are here
├── docs/
│   ├── pattern/
│   │   └── Case_for_the_Local_Orchestrator_v3.md    ← Full pattern specification
│   ├── adversarial-reviews/
│   │   └── README.md                  ← Summary of adversarial review methodology and findings
│   └── cross-domain-validation/
│       └── README.md                  ← Gap map summary and reconciliation results
├── proposals/
│   └── nemoclaw/
│       ├── README.md                  ← NemoClaw governance proposal overview
│       └── NemoClaw_Governance_Proposals.pptx
└── presentations/
    └── DRNT_Architecture_Deck.pptx    ← Full architecture deck (post-reconciliation)
```

## For NemoClaw / OpenShell Developers

These patterns are designed to complement NemoClaw's sandbox architecture, not compete with it. NemoClaw solves runtime containment (kernel-level isolation, network namespaces, credential management). These patterns address what governs the agent's behavior *inside* that sandbox — graduated trust, privacy-aware context packaging, immutable audit trails, and human decision authority.

**→ See [`proposals/nemoclaw/`](proposals/nemoclaw/) for the specific governance proposal.**

## License

MIT — use these patterns however they serve you.

## Contact

Lawrence Jeffords · Nahunta, Georgia
