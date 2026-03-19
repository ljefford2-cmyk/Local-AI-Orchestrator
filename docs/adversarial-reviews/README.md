# Adversarial Review Methodology and Findings

## Approach

Every version of this architecture has been subjected to multi-model adversarial review — not to confirm assumptions, but to find breaking conditions. The methodology uses frontier models as independent red teams, each with different training biases and failure modes.

### Process

1. All architecture documents are provided to each model simultaneously
2. A structured adversarial prompt defines the review scope, requires document-level citations, and prohibits generic concerns — every finding must name a specific gap in a specific document
3. Reviews are conducted independently (models do not see each other's output)
4. Findings are compared for convergence — concerns raised by multiple models independently carry higher weight
5. Converged findings drive architectural revisions

### Key Insight: Prompt Discipline Matters

Early adversarial review attempts using loosely scoped prompts produced off-target results — models generated impressive-sounding analyses of adjacent systems rather than the architecture under review. Tightly scoped prompts that name the target system, list documents explicitly, require citations, and structure the output into defined sections consistently produce on-target, actionable reviews.

This is itself evidence for the "route, don't reason" principle: the quality of output is determined by the quality of the routing instruction, not the raw capability of the model.

## Domains Reviewed

| Domain | Version Reviewed | Key Findings |
|--------|-----------------|--------------| 
| Predecessor concept | v1 | Scope overreach, ambient surveillance requirements, undefined failure behavior |
| Discrepancy Engine | v1 spec | Normalization drift, uncalibrated thresholds, escalation overload |
| Personal gateway | Post-reconciliation | 9 structural findings — see cross-domain validation |
| Federal, clinical, SMB | Domain-specific | Domain adaptations validated; pattern-level specs confirmed transferable |

## What Changed After Review

Major architectural revisions driven by adversarial findings:

- **v1 → v2:** Abandoned continuous ambient monitoring. Adopted job-based UX. Separated L1 routing from L2 reasoning.
- **v2 → v3:** Added operational boundary definition for route-don't-reason. Specified Context Packager pipeline with memory selection policy. Added two-stream audit log design. Formalized WAL promotion/demotion criteria. Resolved nine cross-domain findings through reconciliation pass.
