---
name: proof-of-concept
description: Test technical feasibility through hypothesis-driven experiments with measurable pass/fail criteria, explicit assumptions, and bounded authorization. Use for feasibility POCs, experimental spikes, and empirical technology comparisons; use research for source-reading and prototype for appearance or interaction exploration.
---

# Proof-of-Concept (POC) & Research Skill

## Purpose

Guide disciplined, hypothesis-driven experimentation and research. The agent helps formulate testable hypotheses from the user's goal or question, establishes covered scope and a stopping budget, then tests each hypothesis with isolated experiments — documenting findings at every step and adapting based on accumulated evidence.

## When to Use

Activate when the request requires a technical feasibility experiment:
- "POC", "try this out", "spike on", "experiment with"
- "validate that", "prove that", "I want to test whether..."
- "which approach meets this latency or capacity target?" (empirical evaluation)

Words such as "research", "explore", "compare", or "try this" alone do not select this workflow. Use `research` when reading sources can answer the question, and `prototype` when a disposable artifact helps decide appearance or interaction. The experiment authorization check below applies only after selecting this experimental workflow.

The user may arrive with:
- A **goal** — "I need real-time sync between 3 services"
- A **question** — "Can we use WebSocket for real-time sync?"
- A **vague idea** — "I want to explore serverless for our API"

In all cases, the agent helps formulate the hypothesis document.

## Workflow

In Plan Mode, draft hypotheses only; defer experiment execution and artifact writes. For delegated work, report to the parent and respect its remaining budget. The experiment budget bounds all adaptation and retry instructions below.

### 1. EXPLORE — Understand the Problem Space

Before formulating hypotheses, understand what the user is trying to achieve:

1. **Clarify the goal**: Restate the user's objective in your own words. If ambiguous, ask questions.
2. **Explore the problem space**:
   - Read relevant docs, code, or configs in the project.
   - Search the web for official documentation, existing solutions, approaches, and known constraints.
   - Identify the key technical decisions and trade-offs involved.
3. **Identify candidate approaches**: Based on exploration, list the plausible approaches, technologies, or strategies that could achieve the goal.
4. Ask the user if anything is unclear or if they have constraints not yet mentioned.

### 2. FORMULATE — Build the Hypothesis Document

From the exploration, construct a structured hypothesis document:

| Field | What to write |
|-------|---------------|
| **Goal** | What the user wants to achieve (restated clearly) |
| **Hypotheses** | All plausible approaches to test, each as a testable claim |
| **Success Criteria** | Concrete, measurable pass/fail conditions per hypothesis |
| **Assumptions** | What we believe about tech, data, APIs, constraints |
| **Scope Boundaries** | What is OUT of scope for this POC/research |
| **Initial Ranking** | Hypotheses ranked by likelihood + ease of verification |

### 3. AUTHORIZATION — Present the bounded experiment

Present the hypothesis document in this format:

```markdown
## POC/Research Hypothesis Document

### Goal
[Clear statement of what we are trying to achieve]

### Hypotheses (ranked)
1. **[H1]**: [Approach/claim] — [Why it's likely, ease of verification]
2. **[H2]**: [Approach/claim] — [Why it's likely, ease of verification]
3. **[H3]**: [Approach/claim] — [Why it's likely, ease of verification]

### Success Criteria
- [Criterion 1: specific, measurable outcome]
- [Criterion 2: specific, measurable outcome]

### Assumptions
1. [Assumption about technology or environment]
2. [Assumption about data or inputs]
3. [Assumption about behavior or performance]

### Out of Scope
- [Item 1]
- [Item 2]

Authorization: [existing request/ticket scope, or the specific missing authorization]
```

Proceed when the user request or ticket already covers this bounded, isolated experiment. Do not repeat approval merely because a worker switched skills. Ask the parent/user before new scope, external effects, or consequential design choices. State the hypothesis, measurable success criteria, and a stopping budget (default: up to three experiment attempts for a supporting ticket spike). Stop at the budget and return evidence or uncertainty; a new hypothesis does not reset it.

### 4. EXECUTE — Hypothesis-Driven Experiment Loop

Test hypotheses in ranked order. For each hypothesis, document every step to the user.

#### Step 1: State the Hypothesis

Present the current hypothesis clearly:
> "Testing H[N]: [hypothesis statement]. Success criteria: [criteria]."

#### Step 2: Design the Experiment

Design the **minimal possible test** to validate or invalidate this hypothesis:
- Use temporary directories (e.g., `/tmp`) for scratch work. Do NOT modify the main project workspace unless explicitly authorized.
- Keep experiments isolated and disposable.
- The test should directly address the success criteria — nothing more.

#### Step 3: Run the Experiment

Execute the test. Use bash/code tools as needed. Report what you are doing as you do it.

#### Step 4: Evaluate and Report

For each success criterion, report PASS or FAIL with evidence.
For each assumption, report VALID or INVALID with evidence.

State the verdict clearly:
- **SUCCESS**: Hypothesis is proven. This approach works. Proceed to Step 6 (Final Report).
- **FAILURE**: Hypothesis is disproven. Document findings. Proceed to Step 5.
- **INCONCLUSIVE**: Experiment gave partial results. Not a hard pass or fail. Document what was learned. Decide whether to refine this hypothesis or move to the next.
- **BLOCKED**: Cannot proceed (missing access, dependency unavailable, etc.). Stop immediately. Report blocker to user. Do not auto-proceed.

#### Step 5: Findings Log & Adapt

After each non-success experiment, update the **Findings Log** — a running record across all experiments:

```markdown
## Findings Log

### H1: [Hypothesis name]
- **Verdict**: FAILURE / INCONCLUSIVE
- **What was tested**: [brief description]
- **Key findings**: [what we learned]
- **Implications for future hypotheses**: [how this changes our understanding]

### H2: [Hypothesis name]
- **Verdict**: ...
- **Key findings**: ...
...
```

Then **adapt** using the accumulated findings. The agent should be self-sufficient here — do NOT go back to the user just because the original hypothesis list is exhausted. Use the findings to generate new directions autonomously.

#### Step 5b: Hypothesis Generation Strategies

Use these strategies to design, enhance, or create new hypotheses from accumulated findings. Try them in order before considering escalation:

1. **REFINE — Enhance the current hypothesis**: The hypothesis was close but not quite right. Adjust parameters, constraints, scope, or assumptions based on what was learned.
   > "H1 failed because latency was 500ms. Refining: test Redis Pub/Sub with local batching to reduce effective latency."

2. **COMBINE — Merge elements from multiple hypotheses**: Different disproven hypotheses each had a partially valid insight. Combine the working parts into a new hybrid hypothesis.
   > "H1 (Pub/Sub) had low overhead but high latency. H2 (WebSocket) had good latency but complex setup. Combining: WebSocket with Redis as the message backbone."

3. **PIVOT — Use findings to identify a new approach**: The findings revealed something unexpected that points to an approach not in the original list.
   > "H1 revealed that network latency is the bottleneck, not the protocol. Pivoting: test a local cache with periodic sync instead of real-time push."

4. **DEEPEN — Investigate an inconclusive aspect more closely**: An INCONCLUSIVE result identified a specific unknown. Design a targeted experiment to resolve just that unknown.
   > "H2 was inconclusive because reconnection behavior was unclear. Deepening: test reconnection under controlled network interruptions."

5. **RE-RANK — Reorder remaining original hypotheses**: Findings from a failed experiment may elevate a previously low-ranked hypothesis. Re-rank and test the new top candidate.
   > "H1 showed network latency is the core issue. H3 (SSE) is now higher priority since it has simpler transport."

After generating a new or refined hypothesis:
- State it clearly to the user: > "Based on findings from H[N], I'm forming/enhancing H[M]: [hypothesis]."
- Add it to the findings log with its rationale.
- Return to Step 1 of the experiment loop (state, design, run, evaluate).

**The agent should continue generating and testing hypotheses autonomously through multiple rounds.** Each round of findings should produce new, better-informed hypotheses. This is the core of productive research — not getting stuck, but using each result to get smarter.

#### Step 6: Escalation (last resort, not default)

Escalation to the user is a **last resort**, not the first response to running out of original hypotheses. Only escalate when:

1. You have exhausted all hypothesis generation strategies (REFINE, COMBINE, PIVOT, DEEPEN, RE-RANK) across multiple rounds, AND
2. The accumulated findings do not point to any new promising direction, AND
3. You genuinely cannot form a testable hypothesis from the available evidence.

When escalating:
1. Present the **full Findings Log** to the user.
2. Summarize: what was tested, what was learned, what remains uncertain.
3. Explain specifically why you are stuck — what information is missing that would help you form a new hypothesis.
4. Ask targeted questions, not open-ended ones: "I found that X is the bottleneck but I don't know Y. Can you clarify?"
5. Wait for the user's response before continuing.

**Do NOT escalate just because the original hypothesis list is done.** The original list is a starting point, not a boundary.

### 5. FINAL REPORT — Summarize and Recommend

Once a hypothesis is proven (or all are exhausted with escalation), write `POC_FINAL_REPORT.md` in the workspace root (or `/tmp` if no workspace):

```markdown
# POC/Research Final Report

## Goal
[Restated goal]

## Recommendation
[PROCEED / DO NOT PROCEED / PROCEED WITH CAUTION] — [Approach name]

## Summary
[Brief summary of what was tested and the outcome]

## Findings Log
[Full accumulated findings from all experiments]

## Evidence
- [Experiment log file references]
- [Key data points, metrics, or observations]

## Risks & Caveats
[Known limitations, assumptions that were not fully validated, areas needing further investigation]
```

Present a brief summary in chat.

## Rules

1. **Respect Experiment Authorization**: Establish covered scope and stopping budget before execution; reuse existing authorization and escalate new scope.
2. **Keep Experiments Isolated**: Use `/tmp` for scratch work. No file changes in main workspace unless authorized.
3. **Document Every Step**: For each hypothesis, state it, run the experiment, and report results to the user. No silent testing.
4. **Maintain the Findings Log**: Update it after every experiment. It accumulates across all hypotheses and informs the next steps.
5. **Fail Fast**: If an assumption is invalidated, stop the current hypothesis. Don't apply workarounds.
6. **No Blind Pivoting**: Never re-test a disproven hypothesis without new evidence. When all hypotheses are exhausted, escalate with the full findings log. Do not enter an infinite loop of guessing.
7. **Keep POC Separate from Production**: Use disposable experiments and measurement checks; record evidence. Return to software-engineering for production changes and required validation.
8. **User Can Abort**: If user says "stop", "abort", or "cancel", halt immediately and summarize what was learned.
9. **Blocked Means Stop**: Report BLOCKER to user and await guidance. Do not auto-proceed.
10. **Adapt From Findings**: Each iteration must build on accumulated findings. Use disproven hypotheses to form better ones, not to repeat the same approach.
11. **Be Autonomous, Not Needy**: When the original hypothesis list is exhausted, generate new hypotheses from findings (REFINE, COMBINE, PIVOT, DEEPEN, RE-RANK) before escalating to the user. Escalation is a last resort, not a default. The agent should keep making progress through multiple rounds of hypothesis generation.

## Guidelines

- Keep hypotheses small and focused. One big hypothesis is harder to validate than three small ones.
- Success criteria should be binary (pass/fail) whenever possible.
- If the user provides a new idea mid-loop, incorporate it into the findings log and continue.
- If the user arrives with only a goal (not a question), the agent is responsible for researching the problem space and proposing candidate hypotheses.
- Always link to experiment log files in the final report.
- Re-rank hypotheses after each experiment — new evidence changes priorities.

## Example

**User**: "I need real-time sync between 3 services."

**Step 1 (EXPLORE)**: Clarify goal — sync state changes between 3 microservices in real time. Explore problem space — research WebSocket, Server-Sent Events, message queues (Redis Pub/Sub, RabbitMQ), and polling. Read existing project code to understand current architecture. Identify candidate approaches.

**Step 2 (FORMULATE)**: Present hypothesis document:
- Goal: Real-time state sync between 3 services
- H1: Redis Pub/Sub (high likelihood — already using Redis; easy to verify)
- H2: WebSocket (medium likelihood — more setup; moderate to verify)
- H3: Server-Sent Events (lower likelihood — one-way only; easy to verify)
- Success criteria: <100ms propagation, works across 3 services, survives reconnection
- Assumptions: Redis is already in the stack, services can hold persistent connections
- Out of scope: persistence/durability, historical replay

**Step 3 (AUTHORIZATION)**: Confirm the experiment is covered by the request; ask only for missing authorization.

**Step 4 (EXECUTE)**:

**Testing H1 (Redis Pub/Sub)**:
- State hypothesis: "Redis Pub/Sub can propagate state changes to all 3 services in <100ms."
- Design: Minimal test — 3 small scripts subscribe to a Redis channel, one publishes a state change, measure propagation time.
- Run: Create scripts in `/tmp`, run them.
- Evaluate: Propagation was 5-15ms across all 3 services. Reconnection works (Pub/Sub auto-reconnects).
- Verdict: **SUCCESS**.

**Step 5 (FINAL REPORT)**: Write `POC_FINAL_REPORT.md` — Recommend PROCEED with Redis Pub/Sub. Evidence: 5-15ms propagation, reconnection works. Caveat: durability not tested (out of scope).

---

**Example with adaptation (failed first hypothesis):**

Suppose H1 (Redis Pub/Sub) FAILED — propagation was 500ms due to network configuration.

**Findings Log updated**:
- H1: FAILURE — 500ms propagation. Finding: network latency between services is higher than expected. Implication: approaches requiring persistent connections may have the same issue.

**Adapt**: Re-rank — H3 (SSE) moves up (simpler transport, may have lower overhead). Form new hypothesis H4: "Redis Pub/Sub with local batching reduces effective latency."

**Testing H4**: Design experiment with batching. Run. Evaluate. Continue the loop until proven or all hypotheses exhausted.

If all hypotheses fail, escalate to user with the full findings log.
