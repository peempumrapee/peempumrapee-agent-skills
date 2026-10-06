---
name: software-engineering
description: Build or change production functionality with test-first development where appropriate and evidence-based validation. Use debugging for failures and proof-of-concept for bounded feasibility experiments; disposable design prototypes use prototype.
---

# Software Engineering

## Scope and authorization

Read project instructions, the conversation history, any assigned tickets/designs, relevant code, and existing tests. To prevent "agent drift," the worker must read `intent.md` (typically in `docs/intent/`) $\rightarrow$ Design Artifacts (e.g., `docs/designs/<name>/`) $\rightarrow$ `todo.md` before starting any implementation block. This ensures the current task remains aligned with the high-level mission and technical plan. Define expected behavior and meaningful validation. Scope and approval may be supplied via an authorized ticket handoff, a formal implementation request, or a direct request within the conversation. Do not repeat task-list or plan confirmation if the scope is already clear. Ask only about unresolved material decisions or scope expansion. Authorized files may include configuration, documentation, migrations, and paths outside src/tests.

Use one primary workflow with supporting skills as needed. Apply debugging for failures; use proof-of-concept for a bounded feasibility question, carrying existing authorization. Experiments that expand scope, incur new external effects, or require consequential design decisions return to the parent/user. Return to production implementation after interpreting the evidence; do not promote disposable experiment code without production validation.

## Implement and validate

Follow a strict Test-First Development (TDD) cycle for all production behavior:
1. **Write a meaningful failing test**: Define the expected behavior via a test and verify that it fails.
2. **Implement the smallest change**: Find the most minimal solution by climbing the Ponytail Ladder (Does this need to exist? $\rightarrow$ Reuse existing $\rightarrow$ Stdlib $\rightarrow$ Native platform $\rightarrow$ Installed dependency $\rightarrow$ One-liner $\rightarrow$ Minimal custom code). Write just enough code to make the test pass.
3. **Refactor**: Clean up the code while keeping the test green.

Do not write implementation code before a failing test is verified, except for low-impact documentation or configuration changes. If the code is not easily testable, the worker must first propose a design change to make it testable before proceeding.

Use existing test conventions. When no framework exists, choose a proportionate validation method; ask before a substantial new framework or other scope expansion. Run targeted checks after meaningful changes and required project checks before completion. Broaden testing when failures, integration risks, or repository requirements justify it; do not repeat a full suite after every edit without reason.

Correct tests/fixtures when evidence demonstrates incorrect expectations or an authorized behavior change. Explain the correction and preserve coverage; never weaken assertions solely to hide defects. If tests fail and a simple correction is not supported, use debugging to reproduce, isolate, and verify the cause instead of guessing repeatedly.

## Handoff

Report files changed, evidence for each acceptance criterion, actual checks and results, unverified behavior, and blockers. Distinguish unrelated pre-existing failures from regressions. Keep changes aligned with the provided scope (ticket, design, or conversation). Send material design departures to the parent/user; do not silently redesign. During an orchestrated repair, honor the assigned findings and parent's remaining repair budget.
