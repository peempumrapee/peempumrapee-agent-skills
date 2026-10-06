---
name: review
description: Evidence-based read-only review of code changes, scoped codebases, designs, and implementation tickets, with optional specialist review focuses. Shared methodology for reviewer agents.
---

# Review

Review the assigned scope against its intended behavior and project constraints. The primary benchmark for validation is the `intent.md` file (typically found in `docs/intent/`). Read applicable instructions, `intent.md`, design/tickets, relevant code, and tests, and supplied validation evidence. Review a diff when supplied; a repository or scoped-code review does not require a diff. Inspect surrounding code as needed to substantiate findings.

## Method

State a concise understanding of intent and important assumptions. Continue without routine confirmation; send a material ambiguity to the parent while reviewing independent areas. Do not narrate every file and function or assume a design is wrong merely because another approach is possible.

Check correctness, regressions, failure handling, maintainability, design conformance, and meaningful testing gaps. Specifically, verify that the implementation fulfills all goals in `intent.md` and has avoided all listed Anti-Goals. Challenge unnecessary complexity with concrete evidence of impact. Optional simplifications are suggestions, not automatically defects. Compare behavior against domain invariants in the glossary, ADRs, and design.

When assigned a specialist focus, read only the relevant guide:

- [Data engineering](references/data-engineering.md): pipelines, transformations, storage contracts, and data quality.
- [Machine learning engineering](references/machine-learning-engineering.md): training, evaluation, reproducibility, and serving.
- [AI engineering](references/ai-engineering.md): prompts, retrieval, tool orchestration, and AI evaluations.

Use task-specific focus instructions for other domains; identify limits where evidence or expertise is missing. These focused reviews do not claim to be exhaustive security scans.

## Evidence and output

Use the parent's stable target identity. If the target changes, report it; do not combine observations from different revisions. Distinguish code inspection from tests actually run by the worker. Do not execute tests, linters, formatters, experiments, or commands that modify files. Return findings in a message; the parent persists reports and dispatches fixes.

Return:

1. Scope, target identity, focus, and concise intent/assumptions.
2. Findings ordered by severity, each with a local ID, severity, file/line or artifact location, affected behavior and trigger, supporting evidence, suggested correction, and confidence or uncertainty. Cite the design criterion when relevant.
3. Optional improvements separately from defects.
4. Validation limitations, unresolved questions, and areas not inspected.

Severity: critical = severe immediate impact; high = major correctness or operational failure; medium = material failure in a narrower scenario; low = minor defect. Calibrate to demonstrated conditions, not hypothetical worst cases. An unsupported suspicion is an open question, not a confirmed defect. Return no findings when none are supported; never invent issues to fill a quota.

For re-review, verify the original finding's cause is addressed, inspect affected regression paths, and report each finding as resolved, remaining, or unverified with evidence. New defects remain subject to the parent's repair limit.
