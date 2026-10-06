# SDLC State Machine Reference

Loaded on demand by the `sdlc-guide` skill. Each section describes one state:
purpose, allowed actions, expected outputs, exit criteria, next state, and legal
back-transitions.

Flow overview:

```
intent → design → planning → implementation → review → release → retrospective
            ↑          ↑             |           |         |
            |          |             |           |         └── bugs found → implementation
            |          |             |           └── rejected → planning (re-ticket) or design
            |          └── redesign needed ── design
            └── requirements change ── intent
```

---

## State: `intent`

**Purpose.** Capture what is being built and why, before any technical shaping.

**What you can do.**
- Interview the user to extract goals, constraints, and success criteria — `to-intent`, supported by `grill-me` / `grilling`.
- Study the existing codebase for context.
- Ask clarifying questions; surface contradictions early.

**Outputs.**
- `docs/intent/intent.md` with problem statement, goals, non-goals, constraints.
- `docs/sdlc/state.json` recording the state and artifact paths.

**Exit criteria.**
- `intent.md` exists with goals, non-goals, and at least one success measure.
- No unresolved "must-have" requirement questions pending from the user.

**Next state:** `design`.

**Back-transition:** none (entry state). From any later state, new/changed
requirements return here.

---

## State: `design`

**Purpose.** Turn requirements into a durable, testable design another session can
execute.

**What you can do.**
- Produce the design spec — `to-design`.
- Model the domain if concepts are complex — `domain-modeling`.
- Run gap analysis: hidden dependencies, breaking changes, edge cases, failure
  modes, security constraints.

**Outputs.**
- `docs/designs/<feature-name>/` containing the design doc, use cases,
  architecture/sequence diagrams, and API contracts.

**Exit criteria.**
- Design covers all non-goal-excluded requirements in `intent.md`.
- All critical ambiguities resolved (no placeholder "TODO: decide" in the spec).

**Next state:** `planning`.

**Back-transitions.**
- Requirements changed or a contradiction surfaces → `intent` (update `intent.md` first).

---

## State: `planning`

**Purpose.** Decompose the design into bounded, independently verifiable tickets
with dependencies and acceptance criteria.

**What you can do.**
- Convert the design into tickets — `to-ticket`.
- Order tickets, mark dependencies and blockers, assign worker handoffs.

**Outputs.**
- Ticket set under the project's tickets location (e.g. `docs/tickets/<feature>/`),
  each with objective, scope, links to the design, dependencies, and acceptance
  criteria including a test-first requirement.

**Exit criteria.**
- Every design requirement maps to at least one ticket (traceability complete).
- No ticket is blocked by an unresolved decision; blockers carry explicit
  questions and affected behavior.
- User has approved the ticket set.

**Next state:** `implementation`.

**Back-transitions.**
- Decomposition reveals a design gap → `design`, update the design doc to reflect
  resolved decisions, then re-plan affected tickets only.

---

## State: `implementation`

**Purpose.** Execute tickets, producing working, verified code.

**What you can do.**
- Implement one ticket at a time — `software-engineering` (primary skill for
  production changes per the ticket set).
- Run a bounded feasibility experiment when a ticket demands it — `proof-of-concept`
  (POC evidence never substitutes for production delivery).
- Diagnose failing tests or regressions — `debugging`.
- Scaffold throwaway exploration — `prototype` (must be reconciled back into
  production tickets if adopted).

**Outputs.**
- Code changes plus a passing test that existed *before* the fix/feature (evidence
  of the failing test first).
- Updated ticket status (completed / blocked with reason).
- Notes for review: deviations from the design and why.

**Exit criteria.**
- All tickets completed or explicitly descoped with a recorded decision.
- Relevant tests and validation commands from the tickets pass.

**Next state:** `review`.

**Back-transitions.**
- A bug is found mid-implementation → stay in `implementation`, use `debugging`.
- The implementation proves the design wrong → `design`, update the design doc.

---

## State: `review`

**Purpose.** Critically examine the delivered work against the design and intent.

**What you can do.**
- Request a code/design review — `review` (read-only; do not fix during review).
- Stress-test the delivered docs and claims — `grill-with-docs`.

**Outputs.**
- Review findings: verdict (approve / reject), concrete findings, and required
  changes as an annotated list.

**Exit criteria.**
- A verdict exists and, if approved, no critical finding is unresolved.

**Next state:** `release` when approved; when rejected:
- ticket-level fixes only → `planning` (add fix tickets) → `implementation`
- systemic issue → `design` (rework the design)

**Back-transitions:** as above; review findings are inputs, not a state to skip.

---

## State: `release`

**Purpose.** Ship the verified work through the project's own release convention.

**What you can do.**
- Update changelog, bump versions, tag, and run the project's release checklist —
  no dedicated skill; follow the repo's documented release process or a user-provided
  checklist.

**Outputs.**
- Version bump / changelog entry / tag or published artifact, per project convention.
- Release notes derived from the ticket set.

**Exit criteria.**
- The project's release checklist is complete and the artifact is published or
  tagged.

**Next state:** `retrospective`.

**Back-transitions.**
- A bug is found before the artifact is final → `implementation` with a fix ticket.

---

## State: `retrospective`

**Purpose.** Close the loop: capture lessons, propagate learnings, and queue
follow-up work.

**What you can do.**
- Summarize what deviated from intent/design and why.
- Update process docs (this skill's guardrails, ticket templates, checklists).
- File follow-up tickets for deferred items.

**Outputs.**
- Retro notes (e.g. `docs/sdlc/<feature>/retro.md`).
- Follow-up tickets for deferred work.
- Any updates to skills/templates that review exposed.

**Exit criteria.**
- Follow-ups filed; learnings recorded where they will be seen next time.

**Next state:** terminal for this feature — a new feature starts at `intent` with a
fresh state branch (`features.<name>` in `state.json`).

---

## State file format

`docs/sdlc/state.json`:

```json
{
  "current": "implementation",
  "waivers": [],
  "artifacts": {
    "intent": ["docs/intent/intent.md"],
    "design": ["docs/designs/checkout-v2/"],
    "tickets": ["docs/tickets/checkout-v2/"]
  },
  "history": [
    { "from": "design", "to": "planning", "at": "2026-10-01T09:00:00Z", "reason": "design approved" }
  ],
  "features": {
    "checkout-v2": { "state": "implementation" }
  }
}
```

Rules: single-feature projects may keep `current` at top level; multi-feature work
moves it under `features.<name>.state`. Every transition appends a `history` entry.
Waivers (skipped states) are recorded in `waivers` with who waived and why.