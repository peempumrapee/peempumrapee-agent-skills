---
name: to-design
description: Turn intent into a concise, reviewable technical design, selecting relevant domain guidance and supporting contracts or diagrams only when needed. Use when creating or revising a design before implementation planning.
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

## Select domain guidance

Route by changed behavior and boundaries, not the project's label. Read only relevant references; combine them when needed or use none for a change without domain-specific concerns.

| Reference | Read when the change affects |
| --- | --- |
| [Web Application](references/web-application.md) | User-facing behavior, web APIs, authentication, or frontend/backend contracts |
| [Data Platform & Pipeline](references/data-platform-and-pipeline.md) | Data ingestion, transformation, storage ownership, orchestration, or data consumers |
| [LLM Engineering](references/llm-engineering.md) | Model, prompt, retrieval, tool-use, or evaluation behavior |
| [Infrastructure](references/infrastructure.md) | Environments, networking, identity, capacity, deployment, or recovery |

Guides surface consequential questions and failure modes; they are not mandatory section lists or technology prescriptions. Prefer project-specific architecture and contracts. Select output files independently of references: do not create one document per domain. Before drafting, announce the chosen references and files in one sentence with the reason; this adds no approval gate. Expand the selection only when discovery reveals additional impact.

## Produce the spec

Use **`overview.md`** as the compact human-review entry point, following [the overview template](assets/design-spec.md): link the intent, then provide **Change Summary**, **Validation**, **Risks and Mitigations**, and **Review Checklist**. Change Summary is a short opening paragraph about the technical delta and affected boundaries. Validation links source criteria and distinguishes planned checks from checks actually run. The checklist focuses on consequential technical decisions and unresolved questions, not generic approval boilerplate. A reader should understand what they are approving without opening every supporting file. For a small change, this may be the only design file; put necessary technical detail in short sections within it.

Intent owns Goal/Why (objective/rationale), constraints, anti-goals, and Definition of Done. **Do not repeat or paraphrase** Goal or Why in design; link the authoritative intent instead. Design owns technical decisions, their trade-offs, and contracts; tickets own bounded work and task-specific acceptance checks. Link exact source sections rather than copying shared requirements.

Add atomic supporting files only when implementation or review needs the detail:

- **`architecture.md`**: Changed component boundaries, responsibilities, dependencies, data ownership, or consequential technology decisions. Use an architecture or ER diagram only when useful; link unchanged project architecture.
- **`interactions.md`**: Consequential ordering, retries, async processing, state transitions, or cross-component flows, including failure paths. Interactions may be backend-only; omit unaffected frontend behavior. Use sequence or activity diagrams only when they clarify behavior.
- **`deployment.md`**: Changed infrastructure, environment configuration, rollout, migration, or recovery behavior. Link unchanged environment and network documentation.
- **`api.md`**: New or changed external contracts, including inputs/outputs, auth, and errors. Link unchanged contracts.
- **`validation.md`**: Intent-criterion mappings and concrete validation scenarios too substantial for the overview, including relevant negative paths. Do not restate the Definition of Done.

Every supporting file starts with a short technical summary: what changes, the selected approach, and its main risk or constraint. Follow with concise, change-focused sections, bullets, tables, or examples. Link shared risks and validation rather than repeating the overview checklist. Omit inapplicable files and sections without placeholder explanations. Use no hard whole-file word limit: preserve exact contracts, expected behavior, important failure/state transitions, compatibility/security/migration constraints, and testable validation. Resolve consequential unknowns or mark affected work blocked; brevity must not force an implementer to guess.

Keep diagrams, examples, and prose consistent. Use stable section references so tickets can cite the authoritative detail. Distinguish accepted choices from proposed alternatives.

## Save and hand off

Honor a user-specified destination or existing project design convention; otherwise create a directory `docs/designs/[YYYYMMDD-HHMM]_[name]/` in the target project and save the atomic artifacts within it. Reuse the same directory when revising a design, preserving unrelated content. Do not write project artifacts into the skill installation directory. If the target project is ambiguous, establish it before saving.

In Plan Mode, keep the draft in the conversation and report its intended destination; write files only when the active mode permits. If asked for an in-chat output only, honor that choice.

Before handing off, check requirement coverage, diagram consistency, links, and unresolved decisions. Verify that the overview and opening summaries make the technical changes and material risks understandable, and that the detailed sections support implementation without guessing. Return the overview link, relevant detailed section/contract links, and any blockers. Coding agents must read their task's relevant detail and linked contracts; summaries alone are not implementation instructions. The user can request `$to-ticket` to turn the design into work. This skill creates design artifacts; it does not start implementation, dispatch workers, or publish to a tracker. Existing domain-modeling, research, and prototype workflows may support a concrete need, but are not prerequisites.
