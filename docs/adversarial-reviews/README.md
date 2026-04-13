# Adversarial Review Methodology and Findings

> **Historical note:** This document reflects the architecture at v5.1.1 — the version at the time these reviews were performed. The current architecture version is v7.0, maintained in the [local-first-ai-orchestration](https://github.com/ljefford2-cmyk/local-first-ai-orchestration) repository.

## Approach

Every version of this architecture has been subjected to multi-model adversarial review — not to confirm assumptions, but to find breaking conditions. The methodology uses frontier models as independent red teams, each with different training biases and failure modes.

## Process

1. All architecture documents are provided to each model simultaneously
2. A structured adversarial prompt defines the review scope, requires document-level citations, and prohibits generic concerns — every finding must name a specific gap in a specific document
3. Reviews are conducted independently (models do not see each other's output)
4. Findings are compared for convergence — concerns raised by multiple models independently carry higher weight
5. Converged findings drive architectural revisions

## Key Insight: Prompt Discipline Matters

Early adversarial review attempts using loosely scoped prompts produced off-target results — models generated impressive-sounding analyses of adjacent systems rather than the architecture under review.

Tightly scoped prompts that name the target system, list documents explicitly, require citations, and structure the output into defined sections consistently produce on-target, actionable reviews. This is itself evidence for the architecture's core principle: the quality of output is determined by the quality of the routing instruction — the packaging — not the raw capability of the model.

## Domains Reviewed

| Domain | Version Reviewed | Key Findings |
|--------|-----------------|--------------|
| Predecessor concept | v1 | Scope overreach, ambient surveillance requirements, undefined failure behavior |
| Discrepancy Engine | v1 spec | Normalization drift, uncalibrated thresholds, escalation overload |
| Personal gateway | Post-reconciliation | 9 structural findings — see cross-domain validation |
| Federal, clinical, SMB, K-12 | Domain-specific | Domain adaptations validated; pattern-level specs confirmed transferable |

## What Changed After Review

Major architectural revisions driven by adversarial findings:

- **v1 → v2:** Abandoned continuous ambient monitoring. Adopted job-based UX. Separated L1 routing from L2 reasoning.
- **v2 → v3:** Added operational boundary definition for route-don't-reason. Specified Context Packager pipeline with memory selection policy. Added two-stream audit log design. Formalized WAL promotion/demotion criteria. Resolved nine cross-domain findings through reconciliation pass.
- **v3 → v5.0:** All four independent reviewers (Claude, ChatGPT, Gemini, Copilot) converged on the same structural criticism: the "route, don't reason" framing created a tension with what the architecture actually does — the local model obviously reasons about retrieval and packaging. This convergent finding drove the reframe to the current principle: **"Know what you have. Package what you need."** The boundary moved from a binary (reasoning vs. not-reasoning) to a spectrum based on error recoverability.
- **v5.0 → v5.1.1:** Added evaluation scaling principle and corrected the characterization of the Lightweight Evaluation Loop relative to the formal evaluation harness.

The v3 → v5.0 revision is the strongest evidence that the adversarial methodology works: four models with different training data, different architectures, and different biases independently identified the same structural problem. That convergence is more credible than any single review.

## Canonical Architecture

The current version of the architecture (v5.1.1) is maintained in the [local-first-ai-orchestration](https://github.com/ljefford2-cmyk/local-first-ai-orchestration) repository. This companion repository retains the v3 specification and adversarial findings as historical record of the review process.
