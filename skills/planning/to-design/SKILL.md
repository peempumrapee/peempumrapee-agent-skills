---
name: to-design
description: Discuss requirements with the user and develop a design specification with use cases, architecture and sequence diagrams, and relevant API contracts. Use when turning requirements into a design or revising a design before implementation planning.
---

# To Design

Work with the user in the main conversation to turn requirements into a durable design that another session can understand. The primary source of truth for requirements is the `intent.md` file (typically found in `docs/intent/`). If `intent.md` does not exist, the agent should recommend running the `to-intent` skill first.

## Ground and discuss

Inspect applicable project instructions, relevant code, existing designs, and domain documentation before asking questions. Distinguish existing behavior from proposed changes. When no repository is available, use the supplied context and label unverified technical assumptions.

**Proactive Clarification Phase:**
Do not move to the "Produce the spec" phase until all critical ambiguities are resolved. Act as a critical architect and perform a **Gap Analysis**:
- Compare the `intent.md` against the current codebase.
- Identify "hidden" dependencies or potential breaking changes.
- Surface contradictions between the requested feature and existing architecture.
- Ask focused, high-impact questions that force the user to define edge cases, failure modes, and security constraints.

Carry forward settled decisions and existing authorization without repeating approval questions. Keep a useful draft as the discussion progresses. Record unresolved questions explicitly rather than inventing answers.

## Clarification inputs

Collect relevant grill-me or grill-with-docs conclusions, glossary entries, ADRs, and resolved Wayfinder decisions. Link sources and preserve settled choices; reopen them only when new evidence exposes a conflict, explaining that conflict to the user. Carry unresolved questions into the spec and distinguish decision tickets from implementation work.

## Produce the spec

Instead of a single monolithic document, decompose the design into a set of atomic artifacts stored in a dedicated directory. This ensures clarity and easier maintenance. Produce the following files as needed:

- **`overview.md`**: Problem statement, goals, scope, exclusions, and high-level success criteria.
- **`architecture.md`**: Proposed architecture, component responsibilities, chosen tech stack, and a Mermaid architecture or ER diagram.
- **`interactions.md`**: Mermaid sequence diagrams, activity diagrams, and key behavioral flows (including failure paths).
- **`deployment.md`**: (Produce only if deployment architecture is not yet documented in the project or if this feature introduces material changes to the infrastructure). Include deployment architecture, infrastructure requirements, environment configuration (dev/staging/prod), network flow, and a Mermaid deployment diagram.
- **`api.md`**: Detailed API contracts, endpoints, inputs/outputs, auth, and error codes.
- **`validation.md`**: Detailed success criteria, DoD (Definition of Done), and validation scenarios.

Scale detail to the requirement. For a genuinely inapplicable diagram or section, state why in the `overview.md` rather than fabricating components. Keep diagrams, API examples, and prose consistent across files. Use stable identifiers so tickets can cite specific files and sections. Distinguish accepted choices from proposed alternatives.

## Save and hand off

Honor a user-specified destination or existing project design convention; otherwise create a directory `docs/designs/[YYYYMMDD-HHMM]_[name]/` in the target project and save the atomic artifacts within it. Reuse the same directory when revising a design, preserving unrelated content. Do not write project artifacts into the skill installation directory. If the target project is ambiguous, establish it before saving.

In Plan Mode, keep the draft in the conversation and report its intended destination; write files only when the active mode permits. If asked for an in-chat output only, honor that choice.

Before handing off, check requirement coverage, diagram consistency, and unresolved decisions. Return the spec link and any blockers. The user can request `$to-ticket` to turn the design into work. This skill creates design artifacts; it does not start implementation, dispatch workers, or publish to a tracker. Existing domain-modeling, research, and prototype workflows may support a concrete need, but are not prerequisites.
