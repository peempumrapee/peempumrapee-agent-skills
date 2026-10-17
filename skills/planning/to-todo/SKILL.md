---
name: to-todo
description: Maintain a todo.md file to track actionable items from designs, tickets, or discussions.
---

# To-Todo

## Purpose

Maintain the project's `todo.md` as the primary execution checklist, using the project root unless an existing convention or user specifies another path. Keep current work, blockers, and source links here; tickets own detailed scope, acceptance criteria, and evidence, while intent/design own requirements and contracts. Do not copy these into a second plan.

## Ownership and authorization

Both orchestrators apply these shared rules directly during authorized implementation runs; a separate user invocation is not required. The parent coordinator is the sole checklist writer. Workers read their assigned items and linked detail, then report results and evidence to the parent. Concurrent runs sharing a checklist must agree one owning coordinator before writing; other coordinators relay updates. Outside orchestration, the agent handling the authorized task owns updates.

Do not create or mutate a checklist for read-only review/validation or in Plan Mode. Missing tickets do not require inventing a document suite: derive bounded items from the authorized conversation and label that source. Missing consequential decisions still block the affected task.

## Workflow

### 1. Extraction
When reviewing a design specification (from `to-design`), a set of implementation tickets (from `to-ticket`), or a conversation with the user, identify specific, concrete tasks that need to be completed.

### 2. Updating `todo.md`
- **Initialization**: Read the existing checklist before changing it; create it only when authorized work needs tracking. Preserve unrelated tasks and human edits.
- **Adding Tasks**: Use stable task/ticket IDs, short outcome descriptions, and exact source links:
  `- [ ] T-001 Implement <outcome> — pending; source: [ticket](docs/tickets/001-task.md#acceptance-criteria)`
  This link is illustrative; use verified project paths or an identified conversation decision, not a fabricated ticket.
- **Organization**: Keep dependency-first execution order, adding priority/category headings only when useful. Unfinished items remain unchecked and may be pending, in-progress, or blocked with a brief reason. Link prerequisite IDs instead of copying their details.
- **Avoid Duplicates**: Reuse matching items and ticket IDs. Keep run-specific assignment/criterion mappings in existing run records when needed; do not create another competing execution checklist.

### 3. Tracking and Completion
- **Progress**: The parent marks the item in-progress before dispatch, records blockers or changed scope after reports, and checks `- [x]` only after verifying required acceptance evidence and any required review gates. A worker's completion signal is not PASS; failed, blocked, or unverified work remains unchecked.
- **Updates**: Link the final evidence/report from completed items. Keep implementation and review outcomes distinct; reflect explicit user deferrals without claiming deferred work passed.
- **Syncing**: Reconcile relevant local ticket/index status and checklist state at handoff and completion, preserving prior evidence. Do not update external trackers without authorization. A mismatch requires inspecting the evidence, not copying whichever status looks newest.

### 4. Maintenance
- **Archiving**: Use a `## Completed` section when it improves readability; retain IDs, links, and evidence.
- **Supersession**: Mark obsolete tasks superseded with the reason/source instead of deleting history. Do not prune unrelated work.

## Guidelines
- **Actionable**: Ensure every item starts with a verb and describes a clear outcome.
- **Traceable**: Always include a reference to where the task came from.
- **Concise**: Keep descriptions brief; use the reference for detailed context.
- **Visible**: Mention the update to `todo.md` in your summary when you've added or completed tasks.
