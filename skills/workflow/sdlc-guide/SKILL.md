---
name: sdlc-guide
description: Guide the user through their SDLC workflow as a state machine. Detects the current state (intent, design, planning, implementation, review, release, retrospective), explains what the user can do in it, what artifacts it produces, and which state comes next. Use when the user asks "where am I", "what's next", "what should I do", or wants orientation in the development lifecycle.
---

# SDLC Guide

You are a navigator for the user's software development lifecycle. Your job is to
answer three questions at any moment:

1. **Where am I?** — the current state.
2. **What can I do here?** — allowed actions in this state.
3. **What does this state produce, and where do I go next?** — outputs, exit criteria, and the next state.

## 1. Determine the current state

Read the state file `docs/sdlc/state.json` in the current project (walk up to the
repo root if needed).

- **If it exists:** it contains `current`, `history`, and `artifacts`. Trust it, and
  report the state.
- **If it is missing or stale:** infer the state from artifacts, then confirm with
  the user before writing the state file. Inference rules:

| Artifact found | Inferred state |
| --- | --- |
| nothing yet / empty repo | `intent` (not started) |
| `docs/intent/` exists with a draft, design dirs absent | `intent` |
| `docs/designs/<name>/` exists for the active feature | `design` |
| tickets exist (e.g. from the `to-ticket` skill) with none completed | `planning` |
| tickets exist with some in progress | `implementation` |
| implementation done, review pending or in progress | `review` |
| reviewed, changelog/tag/release done | `release` |
| released, post-mortem or follow-ups defined | `retrospective` |

If the user mentions a specific feature while other features sit further along, ask
which feature is in scope and track states per feature under
`docs/sdlc/state.json` → `features.<name>.state`.

## 2. On any state decision

When the user does work in a state or explicitly advances (`"next"`, "I'm done",
"move on"), do the following in order:

1. Verify the **exit criteria** of the current state (see
   [the state machine reference](references/state-machine.md)). If unmet, list
   exactly what is missing and do not advance.
2. On confirmation, update `docs/sdlc/state.json`: append the old state to
   `history` with a timestamp, set `current` to the next state, and record
   produced artifact paths in `artifacts`.
3. Announce the transition in one short block:

```
State: planning → implementation
Exit criteria met: tickets approved, no open blockers
Next: implementation (skill: software-engineering)
```

## 3. Quick reference

Details, exit criteria, and back-transitions live in
[the state machine reference](references/state-machine.md). Read it whenever a
state's details are needed. The flow:

```
intent → design → planning → implementation → review → release → retrospective
                     ↑                        |
                     └──── review rejected ───┘
```

Each state maps to the user's existing skills — point to them, don't duplicate them:

| State | Primary skills |
| --- | --- |
| intent | `to-intent` (with `grill-me` / `grilling` for interrogation) |
| design | `to-design`, `domain-modeling` |
| planning | `to-ticket` |
| implementation | `software-engineering`, `proof-of-concept`, `prototype`, `debugging` |
| review | `review`, `grill-with-docs` |
| release | none — use the project's own release/checklist convention |
| retrospective | `debugging` findings, updated `intent.md` |

## 4. Output format when asked "where am I / what's next"

Always answer in this structure, kept short:

- **Current state:** `<name>` — one line on what the state is for.
- **What you can do:** 3–6 concrete actions, naming the skill to invoke for each.
- **Outputs of this state:** the expected artifacts and their locations.
- **Next state:** `<next>` — name the exit criteria that must be met first.

## 5. Guardrails

- Never skip a state silently. Advancing from `intent` straight to
  `implementation` requires the user to explicitly waive `design`/`planning`
  (suitable only for tiny changes); record the waiver in `history`.
- Never fabricate artifacts. If you cannot find the outputs a state should have
  produced, say so and offer to generate them with the right skill.
- The state file is bookkeeping, not truth: the repo, `docs/intent/`,
  `docs/designs/`, and tickets are the source of truth. If they disagree with the
  state file, surface the conflict and fix the state file.