---
name: codex-orchestrator
description: Coordinate Codex sub-agents for ticket implementation, debugging, feasibility experiments, acceptance validation, and domain-focused code review. Use for delegated engineering workflows or when explicitly asked to orchestrate workers, validators, and reviewers.
---

# Codex Orchestrator

Act as the parent coordinator. Delegate scoped work through native Codex agent tools, preserve task dependencies and evidence, and return a consolidated result. Use the requested roles only where they add value; a review-only assignment does not require implementation or acceptance validation.

## Roles and skill routing

Read the applicable role reference before dispatch and give its absolute path to the agent. Each agent must read both its role reference and its assigned companion skill before acting.

| Role | Reference | Companion skill and purpose |
| --- | --- | --- |
| Worker | [worker.md](references/worker.md) | `software-engineering` for production implementation/refactoring; `debugging` for observed failures; `proof-of-concept` for bounded empirical feasibility experiments |
| Validator | [validator.md](references/validator.md) | `validator` for intent/design alignment and executed acceptance checks |
| Reviewer | [reviewer.md](references/reviewer.md) | `review` for evidence-based read-only review of a diff or assigned codebase domain |

Resolve companion skills through the installed skill catalog. If a catalog path is stale, search configured skill roots, including `~/.agents/skills` and `~/.codex/skills`; do not assume sibling folders. Resolve references relative to this skill. A missing required skill blocks that assignment; report the exact missing dependency and continue independent work.

Role references are portable Markdown instructions passed in dispatch prompts. They are not automatically registered custom agent types. `agents/openai.yaml` is skill UI metadata, not a sub-agent definition. Do not create or change global agent configuration just to use this workflow.

## 1. Establish scope and ownership

Read applicable project instructions, the authorized request/conversation, assigned tickets, relevant code, tests, and available intent/design/todo artifacts. Use explicit artifact paths when supplied. Current user decisions take precedence over stale artifacts; identify the discrepancy in the handoff. Do not invent documents or demand a complete document set when the authorized request already establishes the task. Missing material criteria block the affected comparison or decision, not unrelated checks.

Build a compact assignment map: task/criterion IDs, source references, dependencies, worker ownership, validation coverage, and review domains. Preserve ticket IDs and give criteria stable IDs when needed. For an implementation workflow, assign every required acceptance criterion to a validator, including cross-layer flows. An integration owner must exercise the actual local cross-layer flow; independently mocked layer checks are not end-to-end evidence.

Choose reviewers from the codebase's actual domain boundaries and affected behavior, such as frontend, API, storage, data pipelines, ML, or AI orchestration. Use multiple reviewers when distinct scopes warrant them; pass shared contracts to the responsible reviewers. Do not spawn a fixed specialist roster or expand ordinary review into an exhaustive security scan.

State the assignment sequence, concurrency, allowed effects, cleanup owner, and budgets briefly. Existing request/ticket authorization covers in-scope dispatch and repairs; do not add routine approval gates. Ask the user only for unresolved material decisions, scope expansion, or effects requiring new authorization. Respect Plan Mode: delegate read-only exploration/planning only, with no implementation, repairs, or experiments.

Defaults: at most two consolidated repair rounds after the initial checks; explicitly assign three experiment attempts to supporting or standalone POC work unless the authorized task specifies another bound. Declare a finite active-work timeout per assignment (default 20 minutes, adjusted before dispatch for task size). Track wall-clock active work using actual time; suspend the execution timeout while waiting for user input. New findings, skill switches, and replacement agents do not reset budgets.

Include these task-specific adaptations explicitly in worker handoffs when applicable, so generic companion instructions do not create conflicting gates:

- Available ticket/conversation grounding replaces the mandatory intent/design/todo reading sequence when those files do not exist and expectations are complete. Still read existing applicable artifacts and escalate material gaps.
- An already authorized implementation/bug-fix request covers a minimal in-scope repair. Require the debugging fix proposal and evidence, but do not repeat its Phase 4 approval request when existing user authorization covers the fix. Diagnosis-only authorization does not permit edits.
- For a POC comparison, success requires results for all requested candidates/criteria, not just the first successful hypothesis. Specify isolated scratch and report destinations in the handoff; the authorized report destination replaces the companion's generic root `POC_FINAL_REPORT.md` location. Budget exhaustion still stops experiments.

These adaptations do not waive reproduction, test-first evidence, local-only execution, or authorization for new effects.

## 2. Dispatch and supervise native agents

Discover the available native collaboration interface and use its actual schema. In runtimes exposing these operations:

- `spawn_agent` starts an assignment. Record the returned agent ID/task name.
- `send_message` supplies context or corrections to an active agent; it does not start a new turn for an idle agent.
- `followup_task` resumes an idle agent for an authorized repair, recheck, or continuation.
- `list_agents` inspects lifecycle status; `wait_agent` waits for updates; `interrupt_agent` stops an active turn when needed.

Use equivalent native lifecycle operations when the runtime names differ. If delegation is unavailable, report the limitation instead of falling back to tmux, Pi subprocesses, or silently claiming delegated execution. Do not invent a custom `agent_type` field: use a role prompt when the spawn interface does not support named agent types. Inherit the parent model/settings unless the user or applicable project instructions require an override. Agents must not delegate further without parent assignment and budget.

Pass every dispatch this minimum context:

```text
Assignment ID / role / objective:
Read role reference: <absolute path>
Read companion skill: <absolute path>
Workspace and applicable project instructions:
Grounding: <ticket/conversation and available intent/design sections>
Assigned criteria or incidents: <IDs, expected outcomes, sources>
Dependencies and prior handoff: <verified results, decisions, gaps>
Authorized files, effects, and exclusions:
Applicable workflow adaptations: <grounding, existing repair authorization, POC completion/report location>
Target identity: <revision and relevant working-tree/build/config identity>
Resources: <services, ports, fixtures, browser sessions, ownership/cleanup>
Budget: <active-work timeout, attempts/repair rounds remaining>
Return: <role report requirements>; message parent for material questions.
```

For an implementation assignment, the initial target may precede edits; issue a new frozen target for checks. Keep prompts focused on the assigned scope. Validators/reviewers must independently inspect source material and evidence rather than treating an implementer's claims as the specification.

Use minimal context for independent reviewers/validators to avoid carrying the worker's reasoning as an assumed conclusion. When conversation history is not inherited, include all required user decisions and authorization explicitly. Every supervision correction must cite the requirement or evidence supporting it; routine implementation choices remain with the worker.

Track assignments as pending, running, awaiting input, completed, or interrupted/failed. Wait for completion messages and inspect returned evidence; completion status alone is not PASS. Avoid busy polling and use bounded waits so progress updates and timeouts remain possible. A crash, timeout, missing report, or unsupported success claim is incomplete work. Interrupt on cancellation, unsafe effects, or elapsed timeout. Do not silently restart a failed agent; first account for changes, findings, and remaining budget in the recovery handoff. Verify edits and owned processes have stopped before allowing another writer or freezing the target.

Relay agent questions through the parent. Answer from authorized grounding where possible; ask the user about new material decisions. For `USER INPUT REQUIRED`, preserve the exact safe browser/session and relay the required action without collecting secrets in chat. Keep the assignment waiting, preserve independent safe work, and resume only after confirmation plus the agent's tool verification. Waiting does not consume a repair round or justify respawning.

## 3. Coordinate concurrency and freeze checks

Use available concurrency slots without changing global limits. Parallelize independent reviewers and validators only when their mutable resources are isolated. Serialize workers that share files, dependent tasks, and checks that share browser sessions, fixtures, databases, or ports. Parallel implementation requires disjoint ownership or isolated workspaces plus an explicit integration owner. Never discard another worker's or the user's changes to resolve a collision.

After implementation, collect changed files, actual check results, criteria coverage, gaps, and startup instructions. Confirm all implementation writers have stopped, integrate their results, then freeze the target. Identify it by revision plus a content snapshot/hash of relevant tracked modifications and untracked implementation files, running build, and safe configuration identity. A commit hash alone does not identify a dirty tree. Exclude evidence-output directories from implementation identity and never persist secrets.

Keep the implementation frozen throughout review and acceptance checks. Reviewers are read-only; validators may create approved isolated fixtures/evidence but cannot edit application code. Give one owner responsibility for shared startup/rebuilds, fixtures, and cleanup. Enforce the companion validator/debugging local-only gate before execution, including outbound dependencies; the parent cannot grant a remote exception on the user's behalf.

If implementation, build, or relevant configuration changes during checks, stop affected checks, mark their evidence stale, and freeze the new target. Review and validation may run in parallel when these conditions hold.

## 4. Consolidate findings and repair

Read every required report and verify its target, coverage, and evidence. Persist returned reports and consolidated feedback under a unique run directory, for example `.codex/orchestration/<run-id>/round-<n>/`, unless the user specifies another destination. The parent writes review reports; reviewers only return messages. Keep workflow artifacts distinct from implementation files and do not update unrelated tickets or publish results externally without authorization.

Maintain stable finding IDs across rounds. Include source/criterion, owner, target, expected/actual behavior or code trigger, reproduction/source evidence, severity/impact, confidence, and disposition. Deduplicate shared causes without losing distinct triggers. Keep optional improvements and unsupported questions separate from confirmed defects. Resolve conflicting evidence through a bounded targeted check on the same target, not a vote or preference for the implementer.

Return confirmed, in-scope defects to a worker with the original grounding, finding IDs, affected regression paths, and remaining budget. Route observed runtime failures through `debugging`; route other supported production corrections through `software-engineering`. Do not weaken criteria/tests just to obtain PASS. Scope expansion, material intent/design conflicts, or blocked prerequisites require clarification, not an automatic repair loop.

Each dispatch of a consolidated repair batch consumes one round, even if several workers perform it. Complete repairs before freezing again. Require finding-by-finding dispositions, then re-review causes/regression paths and revalidate failures and affected acceptance criteria. Final evidence must apply to the final target; retain prior passes only when their continuing applicability is demonstrated. Otherwise rerun them. Stop at the budget, retaining unresolved evidence rather than resetting the counter.

## 5. Report outcome and cleanup

Report acceptance separately from code review. Acceptance is PASS only when every required in-scope criterion has valid final-target evidence; FAIL when a confirmed criterion fails; otherwise INCOMPLETE. A review-only run reports review findings and limitations without claiming acceptance validation occurred.

Overall implementation completion requires required acceptance checks to pass, required assignments/reports to be complete, and confirmed in-scope defects to be resolved or explicitly deferred by the user. Optional review suggestions do not block completion. State any deferrals, excluded scope, blocked/unverified checks, exhausted budgets, and unresolved IDs truthfully.

Return a concise summary of delivered work, target, actual checks, review domains/findings, repair rounds used, evidence paths, and remaining actions. Stop or clean up only agents, processes, and disposable fixtures owned by this run and permitted by its authorization. Preserve user data and unrelated sessions.
