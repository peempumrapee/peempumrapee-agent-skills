---
name: to-todo
description: Maintain a todo.md file to track actionable items from designs, tickets, or discussions.
---

# To-Todo

## Purpose

Maintain a central `todo.md` file in the project root to track pending actionable items derived from designs, tickets, or general discussions. In the standard engineering pipeline, `to-todo` is the final synchronization point that converts implementation tickets into a sequential checklist for the worker.

## Workflow

### 1. Extraction
When reviewing a design specification (from `to-design`), a set of implementation tickets (from `to-ticket`), or a conversation with the user, identify specific, concrete tasks that need to be completed.

### 2. Updating `todo.md`
- **Initialization**: If `todo.md` does not exist in the project root, create it.
- **Adding Tasks**: Append new tasks using the Markdown checklist format:
  `- [ ] Task description (Ref: #ticket-id, design-section, or conversation-date)`
- **Organization**: As the list grows, organize tasks by:
  - **Priority**: (High/Medium/Low)
  - **Category**: (e.g., Frontend, Backend, Docs, Tests)
  - **Status**: (Pending, In Progress, Blocked)
- **Avoid Duplicates**: Check if a similar task already exists before adding a new one.

### 3. Tracking and Completion
- **Progress**: Mark tasks as `- [x]` once the work is implemented and verified.
- **Updates**: If a task's scope changes based on new information, update the description in `todo.md`.
- **Syncing**: Periodically cross-reference `todo.md` with the project's official issue tracker or design docs to ensure alignment.

### 4. Maintenance
- **Archiving**: Move completed tasks to a `## Completed` section at the bottom of the file.
- **Pruning**: Remove obsolete tasks that are no longer relevant due to design changes.

## Guidelines
- **Actionable**: Ensure every item starts with a verb and describes a clear outcome.
- **Traceable**: Always include a reference to where the task came from.
- **Concise**: Keep descriptions brief; use the reference for detailed context.
- **Visible**: Mention the update to `todo.md` in your summary when you've added or completed tasks.
