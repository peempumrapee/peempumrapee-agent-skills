---
name: pi-orchestrator
description: Orchestrates independent pi sessions in supervised sequential or parallel workflows, including separate frontend/backend validators and bounded implementation → validation → feedback → repair loops grounded in intent and design.
---

# Pi Orchestrator

Use this skill only when the user invokes `/skill:pi-orchestrator`.

## Goal

Orchestrates multiple independent pi sessions to handle complex, multi-stage tasks. Acts as a Manager to dispatch agents in Sequential Pipeline Mode (with supervised handoffs) or Parallel Mode, ensuring continuity and quality control across a chain of agents. For implementation workflows, manage independent tool-driven acceptance validation and route consolidated feedback back to implementation until the approved criteria pass or a stop condition is reached.

## Environment Setup

To avoid PATH issues in tmux, always resolve the absolute path of the `pi` binary before dispatching:
```bash
PI_BIN=$(which pi)
```

## Automatic Run Records

For every approved orchestration run, follow [Run Records and Retrospectives](references/retrospective.md). Before dispatch, create a private `.pi/msp/<run-id>/` directory and `manifest.json` linking the parent session, grounding artifacts, code identity, and agent/skill versions. Register each worker before launch, then update its actual transcript path and outcome as evidence becomes available. Use unique worker IDs for replacement sessions and repair rounds; never overwrite an earlier worker's record.

Keep Pi session persistence enabled for every worker: **do not use `--no-session`**. Pi JSONL transcripts are the primary conversation record; tmux terminal logs supplement them with startup/crash evidence. Attach `pipe-pane` before starting Pi. Verify both records exist after the first completed turn and before teardown; record missing evidence explicitly rather than claiming complete capture. Do not close a live ephemeral parent/worker before offering an export of its available history.

These records are collected by the orchestrator during future runs, not by a global hook for arbitrary tmux panes. Existing ordinary parent Pi sessions already persist automatically unless launched with `--no-session`.

## Execution Workflow

### Primary: Sequential Pipeline Mode (Default)
Execute tasks in a strict chain. This is the default behavior for continuous workflows where Agent B depends on Agent A's decisions.

For authorized implementation runs, apply [To-Todo](../../planning/to-todo/SKILL.md) directly before dispatch: maintain `todo.md` as the primary execution checklist, with the parent as sole writer. Follow the shared rules for transitions, blockers, evidence-based completion, and local ticket synchronization; no separate user invocation is needed. Read-only review/validation runs do not mutate the checklist.

**The Sequential Workflow:**
1. **Dispatch Agent A**: For implementation, mark its assigned checklist item in-progress; then spawn the first session.
2. **Supervise**: Use the Supervision Loop (see below) to approve Agent A's plan.
3. **Wait for Completion**: Monitor for a sentinel file (e.g., `.pi/msp/task-1.done`).
4. **Synthesize Handoff**:
   - Read the logs of Agent A.
   - Extract **Key Decisions**, **Architectural Choices**, and **Unresolved Questions**.
   - Generate a concise "Handoff Summary."
   - For implementation, reconcile its checklist item and relevant local ticket state under the shared To-Todo rules. A sentinel alone cannot complete the item; required validation/review may still be pending.
5. **Dispatch Agent B**: Spawn the next session, injecting the Handoff Summary and original design-phase context into the prompt.

### Alternative: Parallel Mode
Spawn all approved sessions simultaneously. Use only for independent tasks when explicitly requested by the user. Separate frontend/backend validators may run in parallel only with explicit user approval and isolated fixtures, browser sessions, ports, and other mutable resources; otherwise run them sequentially against the same frozen target.

### Implementation–Validation Feedback Loop

Use this workflow when the approved chain includes implementation and acceptance validation. Read [Validator](../../reviewing/validator/SKILL.md) before planning or dispatching validation. Resolve the skill path to an absolute path and require every validator worker to read it. Keep read-only `review` separate: review findings may support the handoff but cannot replace executed behavioral validation.

**Approved chain:** Implementation → freeze target → frontend/backend validation → gather feedback → targeted implementation repairs → freeze new target → revalidation.

#### 1. Establish coverage, owners, and limits

Before the first dispatch, present the full chain, applicable validator roles, local environment, permitted side effects/cleanup, and repair budget. Default to **at most two repair rounds after the initial validation**, unless the user approves a different limit. Each dispatch of consolidated fixes consumes one round; new findings or replacement sessions do not reset the budget. Include a bounded timeout for each worker.

Build a shared alignment matrix using stable checklist task/ticket IDs and criterion IDs with source references to intent, design, and ticket acceptance criteria. Keep run-specific criterion ownership in the run records, not another competing execution checklist. Assign every required criterion to a validator; record any non-applicable role explicitly rather than spawning unnecessary workers.

| Role | Scope | Required runtime evidence |
|------|-------|---------------------------|
| Frontend validator | User journeys, UI states, relevant accessibility/responsive requirements | `browser-control` or an installed equivalent; actual interactions and visible/DOM assertions |
| Backend validator | API contracts, authorization, errors, persistence and other specified side effects | `curl`/`xh` or an appropriate installed protocol client; request/response assertions and approved local side-effect reads |
| Integration owner (one of the validators, or a separate worker when necessary) | Required cross-layer flows and shared contracts | Actual frontend → local API flow with observed results; independently mocked frontend/backend passes are not end-to-end evidence |

Validators assess intent ↔ design ↔ implementation within their assigned scope, not just whether a server starts. The parent checks that the combined assignments leave no required criterion unowned.

Apply the validator's **local-only safety gate** to all validation setup and checks, including indirect outbound dependencies. The parent cannot authorize remote testing on the user's behalf. Only pass a remote exception when the user explicitly supplied the target, safe procedure, permitted operations, test data/accounts, safeguards, limits, and cleanup; quote that authorization in the handoff. Missing tools or safe local dependencies are blockers, not reasons to use a deployed service.

#### 2. Finish implementation and freeze the target

Require the implementation worker to hand back changed files, actual test results, artifact paths, known gaps, and local startup/setup instructions. Confirm the implementation worker is finished and cannot edit during validation. Record a target identity including revision plus a snapshot/hash of relevant uncommitted and untracked implementation files, build identity, and safe environment configuration identity. A commit hash alone is insufficient for a dirty tree.

Start/rebuild only approved local services after the safety gate. Give one worker or the parent ownership of startup, fixtures, and cleanup; validators must not restart shared services or reset shared data independently. Use isolated test fixtures or sequential checks where mutations could interfere. If the target changes during validation, stop affected workers, reject stale results, and re-establish the target before continuing.

#### 3. Dispatch independent validators

Use separate sessions for frontend and backend when both are applicable. Inject the following in addition to the normal design context and sentinel obligation:

```text
Read and follow: <absolute validator/SKILL.md path>.
Role: <frontend | backend | integration>; validate only, do not repair code.
Artifacts: <absolute intent/design/ticket paths and relevant sections>.
Assigned criteria: <stable IDs, expected outcomes, and source references>.
Frozen target: <revision/snapshot, build and environment identity>.
Safe targets: <approved local URLs/dependencies>; network restrictions: <...>.
Fixtures/resource ownership: <accounts/data, ports/browser session, permitted mutations, cleanup owner>.
Remote exception: none, or <the user's explicit safe procedure and limits>.
Budget: <worker timeout, bounded requests/retries>; parent repair rounds remaining: <N>.
Report: <absolute .pi/msp/<run-id>/round-<n>/<role>-validation.md path>.
Return a criterion matrix with PASS/FAIL/BLOCKED/UNVERIFIED, actual tool evidence,
expected/actual results, reproducible findings with stable IDs, and cleanup state.
Write the assigned unique sentinel only after saving the report, including on FAIL or BLOCKED.
Escalate missing artifacts, ambiguous criteria, unsafe targets, or unavailable tools; do not invent passes.
If browser input/login is needed, emit USER INPUT REQUIRED with criterion, safe URL,
browser tab/session, fields/actions, and resume instructions. Pause affected actions
until the parent relays the user's confirmation; never request secrets in chat.
```

Review validator plans through the Supervision Loop. Require meaningful tool-driven assertions, coverage of assigned criteria, and compliance with the local-only gate; source inspection, screenshots alone, unit tests, or implementer claims are not substitutes for runtime behavior evidence.

#### 4. Gather and reconcile feedback

Wait for every required validator to finish, then read each report and verify its target identity and evidence. A sentinel means the worker finished, **not** that validation passed. A missing report, crash, timeout, or unsupported PASS is incomplete validation, not success.

Persist a consolidated report under `.pi/msp/<run-id>/round-<n>/feedback.md`. Merge criterion coverage and deduplicate findings without dropping distinct triggers or conflicting observations. Keep stable finding IDs across rounds. Each actionable finding must include criterion/source, owner, target, expected/actual behavior, reproduction steps, tool evidence, impact, and status. Mark old findings resolved, remaining, or unverified based on fresh evidence.

Investigate conflicting frontend/backend results against the same target with a bounded targeted check; do not resolve by majority vote or by preferring the implementation claim. Stop and ask the user for intent/design conflicts, missing authorization, material ambiguity, or persistent environment/tool blockers. Continue safe independent checks where possible, but do not enter an automatic code-repair loop for a blocked prerequisite or an unsupported suspicion.

#### 5. Dispatch targeted repairs, then revalidate

If confirmed failures are within the approved implementation scope and budget remains, send the implementation worker the consolidated findings, original grounding material, affected regression paths, and remaining budget. Require the worker to read [Software Engineering](../../development/software-engineering/SKILL.md), follow its test-first requirements for production behavior fixes, make minimal fixes, and return an ID-by-ID disposition with changed files and actual test results. Do not allow validators to fix the code or implementations to rewrite criteria solely to obtain PASS.

Serialize repairs in a shared workspace; parallel implementation requires separately approved independent ownership/workspaces. Wait for all repairs to finish before validating again. Freeze the new target, rebuild/restart affected local services safely, and dispatch validators again with the prior feedback and repair summary. Retest original failures, affected regression paths, and cross-layer contracts. Do not carry forward stale passes: every required criterion needs evidence applicable to the final target, rerunning checks where target/configuration/fixtures changed or applicability cannot be demonstrated.

#### 6. Stop and hand off truthfully

- **PASS:** Every required in-scope criterion has valid PASS evidence for the final target, including assigned cross-layer checks, and no required worker/report is missing.
- **FAIL:** Confirmed failures remain when the repair budget is exhausted or repairs cannot proceed within scope. Return unresolved IDs and evidence; ask for authorization before another round or expanded work.
- **INCOMPLETE:** Required criteria are BLOCKED/UNVERIFIED, evidence is conflicting/stale, or a worker failed without a conclusive result. Include any confirmed failures rather than hiding them.

Stop on user cancellation or unsafe effects immediately. Do not silently respawn workers or extend the repair budget. Reconcile `todo.md` and relevant local ticket state with final-target evidence under the shared To-Todo rules, leaving blocked or unverified work unchecked. Retain round reports, provide final per-criterion status and finding dispositions, and clean up only processes/fixtures owned by the workflow. Never claim implementation completion is validated merely because implementation tests pass.

---

## Interactive Supervision Loop

The Parent Agent must maintain a watchdog state for every active sub-agent:

1. **Poll the Pane**: Periodically capture the end of the sub-session's pane output (the pane *is* the log):
   ```bash
   tmux capture-pane -t "$SESSION" -p | tail -n 15
   ```
   Only treat the pane as awaiting supervision when its output is **stable across consecutive polls** (e.g., 2–3 polls a few seconds apart) *and* contains a gate signal (`"Proceed?"`, `"Confirm?"`, a proposed `Plan:`, or `"USER INPUT REQUIRED"`). Do not race the model's streaming output — keystrokes sent mid-generation are lost or corrupt the turn.
2. **Intercept Prompts**: If the captured pane contains `"Proceed?"`, `"Confirm?"`, a proposed `Plan:`, or `"USER INPUT REQUIRED"`, the Parent Agent intercepts. Every intercepted plan is reviewed **against the Grounding Material** (see Prompt Composition & Handoffs): the sub-agent's plan must map to the assigned objective and acceptance criteria and respect applicable intent constraints/anti-goals, or explicit authorized user decisions when formal artifacts are absent — not merely look reasonable in isolation.
   - **TDD Enforcement**: For `software-engineering` tasks, the Parent Agent must verify that the plan starts with "Write a failing test for [behavior]." If it is missing, the Parent Agent must send a correction: *"Your plan is missing the Test-First step. Please start by writing and verifying a failing test for the intended behavior before implementing the change."*
   - **Validator Enforcement**: For `validator` tasks, confirm every assigned criterion maps to a comparison or concrete runtime assertion, the local-only safety gate precedes execution, and the report records the frozen target and actual evidence. Reject plans that repair code, probe unapproved remote environments, or substitute source inspection for behavioral validation.
3. **Review & Respond** — ground every response in applicable artifacts or explicit authorized user decisions. Cite the specific source (file path + section, criterion ID, or identified user decision) rather than giving an opinionated verdict:
   - **Correct Plan**: Send approval **after confirming each assigned acceptance criterion maps to a plan step**. If grounded, keep it terse: `tmux send-keys -t "$SESSION" "y — plan covers all acceptance criteria [list them]. Proceed." Enter`.
   - **Incorrect Plan**: Send a correction that **quotes the violated source requirement** — the intent constraint, design decision, acceptance criterion, anti-goal, or explicit authorized user decision — then the required change: `tmux send-keys -t "$SESSION" "No — violates <source §ref>: <quoted requirement>. Instead do [X]." Enter`.
   - **User Browser Input**: For `USER INPUT REQUIRED`, notify the user immediately with the worker's criterion, safe URL, exact browser tab/session, fields/actions to complete (such as entering a local test account email/password and clicking Sign in), and a request to reply “done”. Never ask for secrets in chat or auto-approve the login. Track the worker as awaiting user input, not hung/failed; suspend its execution-time watchdog while waiting without busy-polling, respawning, dispatching repairs, or consuming a repair round. Keep its browser/session available and do not let another worker interfere. After the user's confirmation, relay it to the worker and require browser-tool verification before resuming. If access is unavailable, the user cancels, or the flow is unsafe, preserve a BLOCKED/INCOMPLETE result. Manual input cannot authorize an unapproved remote authentication flow.
   - **Clarification**: Relay sub-agent questions to the user **together with the relevant design-stage context** (what the ticket/design says about the topic). Send the answer back citing its source: if it comes from an existing artifact, quote it; if it is a genuinely new user decision, mark it as such and note that it should be recorded (via `domain-modeling` / a decision ticket) so later agents inherit it instead of re-deriving it.
   - **No grounding, no response**: if no applicable artifact or explicit authorized user decision establishes a material requirement, do not improvise — surface the gap to the user first.

---

## Prompt Composition & Handoffs

### Design Phase Context
When spawning any agent, include applicable intent constraints, assigned ticket/criteria, and exact relevant design sections/contracts. Summaries orient the worker but do not replace reading technical detail. For implementation, include the `todo.md` path and task ID, and state that only the parent edits the shared checklist; workers return evidence and blockers.

### Grounding Material (required before dispatch and before any supervision response)
Before dispatch and any supervision response, the parent must have the applicable source requirements and technical detail in context. Reuse loaded sources and refresh affected sections when decisions or scope change; do not reread the entire suite on every gate. These are the available outputs of the engineering pipeline skills:

| Artifact | Source skill | Typical location | Used for |
|---|---|---|---|
| `intent.md` | `to-intent` | `docs/intent/` | Objective, rationale, constraints, anti-goals; the benchmark for plan approval |
| Design suite | `to-design` | `docs/designs/<name>/overview.md` plus relevant optional detail | Technical decisions and exact contracts the sub-agent must follow |
| Implementation tickets | `to-ticket` | Issue tracker / local tickets | Per-agent objectives, dependencies, acceptance criteria |
| `todo.md` | `to-todo` | project root | Current checklist state the chain is executing |

Missing formal artifacts do not justify inventing requirements or demanding a complete document suite when the authorized ticket/conversation establishes the task. Use explicit user decisions as grounding and let the parent initialize the execution checklist under To-Todo; missing consequential criteria block the affected task. When an artifact is outdated, current user instructions take precedence; identify the discrepancy in the handoff.

### Sentinel Obligation
Instruct every sub-agent, as part of its injected prompt, to write a sentinel file (`.pi/msp/task-n.done`) upon completion. The parent sequence-gates on it; a sub-agent that never writes it is considered failed or hung. For feedback loops, use unique run/round/role paths, such as `.pi/msp/<run-id>/round-0/frontend.done`, so earlier completion cannot trigger a later round. Save the assigned report before writing the sentinel; include the worker outcome, target identity, and report path in it. FAIL/BLOCKED reports also signal completion. The parent must verify the report rather than treating sentinel existence as PASS.

### Handoff Injection (Sequential Mode)
For continuous tasks, the prompt for the next agent must follow this structure:
```text
[Grounding]  <- applicable intent constraints + exact design/contract sections + assigned criteria or authorized user decisions
+
[Implementation Checklist]  <- todo.md path + task ID; parent-only writes; omit for read-only assignments
+
[Handoff Summary from Previous Agent]:
"The previous agent completed [Task A]. Key decisions made:
1. Decision X: [Rationale]
2. Decision Y: [Rationale]
Current State: [Result/Artifact Path]"
+
[Current Task Objective]
+
[Sentinel Obligation]  <- write .pi/msp/task-n.done on completion
```

---

## Tmux Implementation

Run the sub-agent **interactively and directly in the pane**. Do **not** use `--print` (non-interactive: the process exits after one turn, making supervision impossible) and do **not** pipe output through `tee` (pi's TUI requires the real tmux TTY; a pipe kills it). Use `tmux pipe-pane` for observe-only logging instead. Start a login shell first, attach logging to its exact pane ID, then launch Pi so startup output is not lost.

The examples below run in Bash. Set `PROJECT_DIR` to the absolute project path, `RUN_ID` to a unique run identifier, `WORKER_ID` to a unique role/round/attempt identifier, and `PROMPT` to the approved handoff. Use letters, numbers, underscores, or hyphens for these IDs and start/end with a letter or number (also safe for tmux session names). Prepare the manifest as described above before running these commands. Stop on any failed command; do not launch Pi after failed log setup.

### Shared Setup
```bash
set -e
umask 077
PI_BIN=$(command -v pi)
RUN_DIR="$PROJECT_DIR/.pi/msp/$RUN_ID"
SESSION_NAME="pi-$RUN_ID-$WORKER_ID"
WORKER_SESSION_ID="$RUN_ID-$WORKER_ID"
LOG_FILE="$RUN_DIR/logs/$WORKER_ID.terminal.log"
mkdir -p "$RUN_DIR/logs" "$RUN_DIR/reports"
# Quote arguments instead of interpolating the prompt into executable shell syntax.
printf -v PI_COMMAND '%q ' "$PI_BIN" --name "$SESSION_NAME" --session-id "$WORKER_SESSION_ID" -- "$PROMPT"
printf -v LOG_COMMAND 'umask 077; exec cat >> %q' "$LOG_FILE"
```

### New Dedicated Session (Recommended)
```bash
PANE_ID=$(tmux new-session -d -P -F '#{pane_id}' -s "$SESSION_NAME" -c "$PROJECT_DIR" /bin/zsh -l)
tmux pipe-pane -O -t "$PANE_ID" "$LOG_COMMAND"
# Save PANE_ID, SESSION_NAME, WORKER_SESSION_ID, and LOG_FILE in the worker record.
tmux send-keys -l -t "$PANE_ID" "$PI_COMMAND"
tmux send-keys -t "$PANE_ID" Enter
```

### New Window in Current Session
```bash
CURRENT_SESSION=$(tmux display-message -p '#S')
PANE_ID=$(tmux new-window -d -P -F '#{pane_id}' -t "$CURRENT_SESSION" -n "$WORKER_ID" -c "$PROJECT_DIR" /bin/zsh -l)
tmux pipe-pane -O -t "$PANE_ID" "$LOG_COMMAND"
# Save PANE_ID, CURRENT_SESSION, WORKER_SESSION_ID, and LOG_FILE in the worker record.
tmux send-keys -l -t "$PANE_ID" "$PI_COMMAND"
tmux send-keys -t "$PANE_ID" Enter
```

Only use these launch commands for newly created, owned panes. Capture the actual Pi transcript path after the first completed turn and update the manifest; a requested session ID alone is not proof of a saved transcript.

## Verification & Monitoring

- **Attach**: `tmux attach -t <session-name>`
- **Watch live**: `tmux capture-pane -t <session-name> -p | tail -n 15`
- **Watch terminal logs**: `tail -f .pi/msp/<run-id>/logs/*.terminal.log` (written by `tmux pipe-pane`; includes terminal control sequences, not a structured transcript).
- **Review full history**: Read the parent and worker Pi JSONL files linked in `.pi/msp/<run-id>/manifest.json`, following entry IDs/parent IDs across branches. A sentinel or terminal log cannot replace this evidence.
- **Sentinel**: Check for `.pi/msp/task-n.done` to trigger the next sequential step.
- **Tearing down a failed sub-session**: only after corrections via `send-keys` are exhausted; kill with `tmux kill-session -t <session-name>` and re-dispatch with a handoff stating what did and did not happen — never silently retry until the sub-agent stops asking for approval.

## Output Summary

At the end of dispatch/pipeline setup, print:
- Tmux session name and attach command.
- Sequence of agents (Parallel vs Sequential).
- Run manifest path, parent/worker Pi transcript paths, terminal log paths, and expected sentinel files; explicitly list missing or unverified records.
- Grounding artifacts loaded (intent.md, design suite, tickets, todo.md) with their paths; flag any that were missing.
- Summary of the context being passed between agents.
- For implementation–validation loops: criterion/role ownership, frozen target, approved local resources, repair budget and current round, validator/consolidated report paths, unresolved finding IDs, and final PASS/FAIL/INCOMPLETE verdict when finished.

## Safety Rules

- **Grounded Supervision**: Never approve, correct, or answer a sub-agent without citing the applicable source artifact or explicit authorized user decision. If neither covers a consequential requirement, surface the gap to the user instead of improvising.
- **No Auto-Approval**: Always review plans via the Supervision Loop.
- **Environment Integrity**: Always use absolute paths and login shells (`zsh -l`).
- **Truthful Handoffs**: Do not hallucinate decisions or results; synthesize only what is supported by the sub-agent's logs, saved reports, and referenced artifacts. Distinguish reported claims from verified tool evidence.
- **User Approval**: Present the full sequential chain to the user before starting the first agent, including validator roles, any proposed parallelism, and the repair budget.
- **Private Records**: Keep run records local and private, exclude `.pi/msp/` from Git, and never automatically upload logs or edit agent/skill instructions from retrospective findings. Stop logging only on owned panes and preserve the files after cleanup.
- **Validation Isolation**: No implementation edits while validators are checking a frozen target; isolate mutable test resources or serialize checks.
- **Local-Only Validation**: Enforce the validator safety gate for every worker; only the user's explicit safe testing procedure can permit remote validation.
- **Bounded Feedback**: Consolidate evidence before dispatching fixes; never reset the repair budget on new findings or worker replacement, or treat missing validation as success.
