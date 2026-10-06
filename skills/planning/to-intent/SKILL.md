---
name: to-intent
description: Guide the user through an interactive interview to define the objective, rationale, constraints, and anti-goals of a feature, producing a high-quality intent.md file to prevent agent drift.
---

# To Intent Skill

The To Intent skill is the entry point for all new feature requests, bug fixes, or architectural changes. Its purpose is to transform a vague user request into a rigorous, unambiguous `intent.md` file. This prevents "agent drift" and ensures that subsequent design and implementation phases are grounded in a clear mission.

## Goal
Generate a high-quality `intent.md` file that serves as the "North Star" for the entire feature lifecycle.

## Process

### 1. Interactive Consulting (The Interview)
Do not simply write the file based on the initial prompt. Instead, act as a senior product manager/architect and interview the user to fill in the gaps. Ask about:
- **The Core Objective:** What is the single most important outcome of this work?
- **The Rationale:** Why is this being done? What is the problem it solves?
- **Constraints:** Are there technical, business, or time constraints? (e.g., "must use X library", "cannot change Y API").
- **Anti-Goals:** What are we explicitly NOT doing? (e.g., "we are adding search, but NOT filters yet"). This is critical to prevent scope creep.
- **Definition of Done (DoD):** What are the specific, measurable conditions that must be true for this task to be considered complete?

### 2. Draft & Refine
Present a draft of the `intent.md` to the user. Ask: *"I have captured the following intent. Did I miss any edge cases, constraints, or anti-goals?"*

### 3. Finalize `intent.md`
Write the finalized content to `docs/intent/[YYYYMMDD-HHMM]_[feature-name].md`. Ensure the directory exists before writing.

## `intent.md` Template
The output file must follow this structure:

# Intent: [Feature Name]

## 🎯 Objective
[A one-sentence high-level goal]

## 💡 Rationale
[Why this is necessary and what problem it solves]

## 🛠 Constraints
- [Constraint 1]
- [Constraint 2]

## 🚫 Anti-Goals
- [What we are NOT doing]
- [What is out of scope]

## ✅ Definition of Done
- [ ] [Condition 1]
- [ ] [Condition 2]
- [ ] [Condition 3]

---

## Next Steps (The Hand-off)
Once `intent.md` is finalized, the To Intent skill must guide the user to the next phase:
1. Suggest running `pi describe` to map the intent against the current codebase.
2. Suggest running `pi to-design` to create the `DESIGN.md` based on the `intent.md`.
