---
name: to-ticket
description: Convert a design into local implementation tickets with dependencies, acceptance criteria, and worker handoffs, or revise an existing ticket set. Use when the user wants actionable work from a design; not for unresolved-decision maps or automatic execution.
---

# To Ticket

Work with the user in the main conversation to turn a design into bounded implementation tasks. The primary inputs are the design directory (e.g., `docs/designs/<name>/`) and the `intent.md` (typically in `docs/intent/`). If these are missing or outdated, the agent should suggest updating them before proceeding.

## Establish the source

Read the selected design directory, `intent.md`, relevant project instructions and code, and existing tickets. If several designs are plausible, ask which one to use. If the source exists only in the conversation, preserve it as a set of atomic design documents in a directory when writing is permitted so another session can follow the ticket links. Preserve its decisions without silently redesigning it.

The request to convert a design authorizes ticket preparation. Ask only about unresolved choices that materially block implementation. Continue drafting independent work while those questions are pending. Mark affected tickets blocked with the question and affected behavior; never hide missing product or architecture decisions behind an implementation checklist.

## Decompose the work

Create one independently verifiable outcome per ticket, preferring useful vertical slices where feasible. A small design may need only one ticket. Capture shared contracts or prerequisites explicitly when they must precede other work. Avoid arbitrary ticket counts, token budgets, or file-by-file fragmentation.

Use [the ticket template](assets/ticket.md). Every ticket must carry enough context for a worker without the chat history:

- Objective, expected behavior, scope, and exclusions.
- Source design link and relevant section references; summarize the essential decisions locally.
- Implementation guidance, affected areas verified in the repository, and required interfaces or contracts. Label proposed paths as proposed.
- Dependencies and blockers, with links to prerequisite tickets.
- Observable acceptance criteria and appropriate validation scenarios or commands. Every implementation ticket must explicitly include a "Test-First" requirement: the worker must first provide evidence of a failing test before implementing the fix. Do not invent existing test commands.
- Primary skill required: `software-engineering` for production changes, `debugging` for an existing failure, or `proof-of-concept` for a feasibility experiment covered by the request. The primary skill may use supporting debugging or bounded POC within ticket scope. Experiments report evidence and do not stand in for production delivery.

Allow routine implementation choices within the design. If decomposition reveals a material design gap or contradiction, surface it to the user and update the source design only to reflect resolved decisions.

## Persist and revise

Honor a user-specified location or existing project ticket convention; otherwise use `docs/tickets/[YYYYMMDD-HHMM]_[design-name]/` in the target project. Use stable numbered filenames such as `001-<outcome>.md` and an `index.md` based on [the index template](assets/ticket-index.md). Link the source design, list dependencies and execution order, and map in-scope requirements to tickets.

Use relative links that resolve from each artifact. Reuse an existing ticket set for the same design. Preserve IDs, unrelated edits, completion evidence, and recorded progress; allocate new IDs after the highest existing number. When a design revision invalidates work, document the impact, mark affected tickets for revalidation, and record removed scope as superseded rather than silently deleting history. Update index and ticket state together.

Use these states unless the project defines its own: `ready` (actionable with prerequisites met), `blocked` (unresolved decision or unmet dependency), `in-progress`, `done`, `needs-revalidation`, and `superseded`. Ticket state is a record of readiness/progress, not authorization to execute. An unmet prerequisite can have fully specified work while remaining blocked.

In Plan Mode, draft in the conversation with intended paths; defer artifact writes until permitted. Establish the target project before saving if it is ambiguous. Publication to an external tracker requires a separate request.

## Check and hand off

Verify that every in-scope requirement is covered, ticket scopes do not conflict, links resolve, dependencies have no cycles, and acceptance criteria are testable. Include relevant negative paths, compatibility, migrations, or operational validation from the design. State any coverage gaps explicitly.

Return the ticket index, recommended first ready ticket, and remaining blockers. Preparing tickets does not dispatch workers or start implementation. When the user requests execution, the parent can give a worker the ticket path and design context; the worker reports acceptance-criterion evidence and any material departure from the design.

## Execution and review handoff

Orchestrator can consume this ticket/index directly; no multi-session plan is required. Workers execute sequentially. Include relevant review focuses when evident from the design, but do not request or dispatch reviews merely by listing them. Track implementation status separately from review state (not-requested, pending, passed, findings-open). The orchestrator records progress and any requested review's repair count; preserve these on revision. Supported defects or design changes affecting completed work require revalidation without erasing previous evidence.
