# Peem's Agent Skills

Personal coding-agent skills for @peempumrapee's work, supporting both [Pi](https://github.com/earendil-works/pi-coding-agent) and [OpenAI Codex](https://openai.com/codex/). Skills use the `SKILL.md` format and are organized by workflow area under `skills/`.

## Structure

```text
skills/
├── development/   # debugging, proof-of-concept, software-engineering
├── planning/      # describe, to-design, to-intent, to-ticket, to-todo
├── reviewing/     # review, validator
└── workflow/      # codex-orchestrator, pi-orchestrator, sdlc-guide
```

Pi and Codex discover skill files recursively, so skills may be grouped in these category directories. The skills described below are the project and engineering workflow set; this README does not catalog every skill directory.

## Project and engineering workflow skills

### Planning — `skills/planning/`

- `describe` — concise project overviews for codebase orientation and documentation
- `to-intent` — interview-driven intent documents covering goals, constraints, and anti-goals
- `to-design` — design specifications, diagrams, and API contracts
- `to-ticket` — dependency-aware implementation tickets with acceptance criteria
- `to-todo` — actionable task tracking in `todo.md`

### Development — `skills/development/`

- `software-engineering` — production implementation with test-first development where appropriate
- `debugging` — reproduce failures, identify root causes, and verify minimal fixes
- `proof-of-concept` — bounded experiments for technical feasibility

### Reviewing — `skills/reviewing/`

- `validator` — acceptance validation against intent, design, and observed behavior
- `review` — evidence-based, read-only code and design reviews

### Workflow — `skills/workflow/`

- `sdlc-guide` — development lifecycle orientation, state tracking, and next-step guidance
- `pi-orchestrator` — supervised coordination of independent Pi sessions
- `codex-orchestrator` — supervised coordination of Codex sub-agents

The general workflow skills are intended for use with either agent; the orchestrator skills are agent-specific.

## SDLC workflow

Use `/skill:sdlc-guide` at any stage to identify the current state, available actions,
expected outputs, and next step. Invoke a stage's skill with `/skill:<name>`.

```text
intent → design → planning → implementation → review → release
```

| Stage | Skills to use | Main outputs | Ready to advance when |
| --- | --- | --- | --- |
| **Intent** — define what and why | `to-intent`; `describe` for codebase context | Intent document under `docs/intent/` with goals, constraints, non-goals, and success measures | Intent is documented and must-have requirement questions are resolved |
| **Design** — define how it should work | `to-design` | Design suite under `docs/designs/<feature>/`, including architecture, interactions, and relevant API contracts | In-scope requirements are covered and critical design ambiguities are resolved |
| **Planning** — create executable tasks | `to-ticket`, then `to-todo` | Dependency-ordered tickets under the project's ticket location, acceptance criteria, and `todo.md` | Requirements map to tickets, decision blockers are resolved, and the user approves the ticket set |
| **Implementation** — build and validate | `software-engineering`; `validator` for acceptance checks; `debugging` for failures; `proof-of-concept` for bounded feasibility tests; explicitly invoke an orchestrator for supervised execution | Code, validation evidence, updated tickets/checklist, and design deviation notes | Tickets are complete or explicitly descoped, and required validation passes |
| **Review** — verify against intent and design | `review` | Review verdict and evidence-backed findings | No unresolved critical findings remain |
| **Release** — ship verified work | Follow the project's release checklist | Changelog, version/tag or published artifact, and release notes | The release checklist is complete and the artifact is published or tagged |

POCs provide evidence, not a substitute for verified production implementation.

### Acceptance validation

Use `/skill:validator` with the relevant intent, design, acceptance criteria, and local
app/API targets. It checks artifact alignment and actual behavior using browser-control
or equivalent browser tools for frontend journeys, `curl`/`xh` for APIs, and appropriate
runtime tools for other outputs. Results include criterion-level evidence and
PASS, FAIL, BLOCKED, or UNVERIFIED statuses; validation does not modify implementation.

Testing is local-only, including indirect dependencies, unless you explicitly authorize
a remote target with a safe testing procedure. If login or other manual input is needed,
the validator identifies the browser tab and fields, pauses until you reply “done”, and
verifies the resulting state. Enter credentials in the browser, not chat.

Unlike read-only `review`, `validator` executes behavioral checks. Explicitly invoke
`pi-orchestrator` or `codex-orchestrator` to coordinate supervised work. See the
[validator skill](skills/reviewing/validator/SKILL.md).

### Transitions and tracking

- Rejected reviews return to **planning → implementation** for ticket-level fixes,
  or to **design** for systemic issues.
- Design gaps return to **design**; changed requirements return to **intent**.
- Bugs found before a release is final return to **implementation** with a fix ticket.
- `sdlc-guide` tracks state, history, and artifact paths in `docs/sdlc/state.json`.
  It checks exit criteria and obtains confirmation before advancing. Artifacts are the
  source of truth; skipping design/planning requires an explicit recorded waiver.

See the [SDLC guide](skills/workflow/sdlc-guide/SKILL.md) and
[full state machine](skills/workflow/sdlc-guide/references/state-machine.md) for details.

## Install

Clone this repository into `~/.agents` so Pi can discover skills under `~/.agents/skills/`:

```bash
git clone git@github.com:peempumrapee/peempumrapee-agent-skills.git ~/.agents
```

Restart Pi after installing or updating the skills, or run `/reload` in an active session.

## License

This project is licensed under the [MIT License](LICENSE).
