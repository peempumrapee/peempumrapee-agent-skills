---
name: debugging
description: Debug actual output against incident reports using tool-driven reproduction, evidence-based root cause analysis, and minimal fixes. Use native browser tools for frontend issues, curl/xh for APIs, and appropriate runtime tools for build, test, or configuration failures; verify fixes by replaying the incident.
---

## Purpose

Provide a disciplined, reproducible debugging workflow. The goal is to find the simplest possible fix for the reported issue with the minimum number of changes, and to verify that the fix actually works before declaring success.

## When to Use

Activate this skill when the user presents:
1. A runtime error, exception, or crash (with or without a stack trace)
2. A failing test or test suite
3. A build or compilation failure
4. A configuration or environment issue that prevents normal operation
5. Any unexpected behavior that can be observed and described
6. An incident report, bug ticket, or validator finding whose reported behavior must be compared with actual runtime output

Do NOT activate this skill for:
- Feature requests or new functionality
- Code review without a specific failing symptom
- General refactoring or architectural changes

## Scope, Safety, and User Input

- Read project instructions and the incident report supplied in a file, ticket, validator feedback, or conversation. Use relevant intent/design/API contracts to confirm expected behavior; if the report conflicts with them or omits material details, ask rather than silently inventing a requirement. Keep debugging scoped to the reported incident, not a full acceptance audit.
- Before execution, state the reproduction plan, target, tools, test data, permitted mutations, and cleanup. Use the existing authorization when it covers this scope; ask before scope expansion, source instrumentation, or consequential side effects. During delegated debugging, honor the parent's diagnostic/repair budget and timeouts; new hypotheses or replacement workers do not reset them. Stop on cancellation, unsafe effects, or exhaustion of the approved budget.
- Apply the **local-only safety gate** before starting services, tests, browser navigation, or requests. Verify the app and its databases, queues, proxies, redirects, authentication, webhooks, analytics, and other dependencies are isolated local resources. Loopback URLs alone do not prove safety; local tunnels/proxies to remote systems count as remote testing. Inspect scripts/configuration without disclosing secrets, use disposable fixtures, and block unapproved outbound browser requests. If external effects cannot be ruled out or safely blocked, stop the affected check.
- Do not probe remote targets, including read-only endpoints, unless the **user** explicitly supplies the target and a safe testing procedure covering permitted operations, test accounts/data, safeguards, limits, and cleanup. An incident's production URL, available credentials, a staging label, or delegated request is not sufficient. Manual login does not waive this restriction. Bound request counts, retries, and timeouts; do not perform destructive actions, migrations, load tests, real payments/messages, or use production data without explicit scope and safeguards.
- If browser reproduction needs login, MFA, CAPTCHA, or other user-specific input, emit **USER INPUT REQUIRED** and pause affected actions. Tell the user the incident ID, safe URL, exact browser tab/session, fields/actions to complete, and to reply “done”. Ask them to enter sensitive values directly in the browser, never in chat; do not guess credentials, bypass authentication, or persist secrets in screenshots, DOM dumps, or logs. If manual access is unavailable, ask for a safe way to expose the controlled browser rather than assuming a separate browser shares its session.
- When delegated, relay the input request through the parent. Do not write a completion sentinel or consume a repair attempt just because input is pending. After confirmation, use the browser tool to verify the resulting state before continuing; record user-performed prerequisites separately from tool evidence. Unavailable input or tools are blockers, not evidence of a defect or resolution.

## Workflow

Follow these phases in strict order. Do not skip steps. Source inspection or historical reports cannot replace executed reproduction and verification.

### Phase 1: Incident & Error Capture

1. Read the incident report and supplied errors, stack traces, logs, screenshots, or test output. Use `inspect_image` when analyzing image evidence; an image alone does not reproduce interactive behavior. If no formal report exists, build a concise incident record from the user's description without requiring a new document.
2. Give each reported symptom a stable ID. Capture the trigger, exact inputs/fixtures, expected output and its source, reported actual output, frequency, and reproduction steps. Redact secrets and personal data.
3. Record the reported environment and the local reproduction target: revision plus relevant working-tree changes, running build, configuration identity, framework/OS/dependencies, and recent changes. Distinguish differences from the incident environment; do not present a different revision or mocked dependency as the same target.
4. Keep a comparison table throughout debugging:

   | Incident ID / source | Trigger and fixture | Expected output / source | Reported actual output | Locally observed output | Tool/actions and evidence | Reproduction / verification status |
   |----------------------|---------------------|--------------------------|------------------------|-------------------------|---------------------------|------------------------------------|
   | I1 | Exact scenario | Contract or user expectation | Supplied symptom | Pending runtime check | Command/browser actions, result/artifact | Pending |

5. **Do NOT guess the root cause at this stage.** Only collect facts. Treat the report as a claim to reproduce, not proof that the same defect exists locally.

### Phase 2: Reproduction (Mandatory Gate)

1. After the safety gate, reproduce the incident using tools suited to the actual failing surface:
   - **Frontend:** Prefer the native `agent_browser` tool when available, rather than direct `agent-browser` shell commands. Open the approved local target, take `snapshot -i`, act on current refs, and re-snapshot after navigation/rerenders. Use native `batch --bail` for fixed same-snapshot sequences and `agent_browser_code` for branches/loops, returning for judgment at new observations. Use `agent_browser_tools` when specialized capabilities are needed. If unavailable, use an installed equivalent browser-control tool; follow tool instructions and targeted documentation without inventing commands or running exploratory help unnecessarily.
   - Trigger the exact reported journey and assert visible/DOM outcomes, relevant console errors, and network behavior. A loaded page, screenshot, or successful click alone does not reproduce the incident. Keep the assigned browser session/tab stable; use isolated sessions or coordinate with other workers before navigation, and never close an unrelated browser.
   - **Backend/API:** Use `curl` or `xh` via the command tool, or an installed equivalent client, against verified local endpoints. Reproduce the method, route, headers, auth context, payload, and fixture. Compare status, relevant headers, response schema/values, and approved local persistence/queue side effects. A status code alone is insufficient. Set timeouts, bypass inherited outbound proxies for loopback where needed, do not automatically follow unverified redirects, and keep secrets out of commands/logs/transcripts.
   - **Tests/build/runtime/configuration:** Run the specific failing test, build command, CLI/script, or relevant local configuration check. Unit/mocked reproduction may isolate a cause, but must not be presented as live browser/service reproduction. When the incident spans frontend and backend, exercise the actual local cross-layer path.
2. Compare observed output to both the reported symptom and expected behavior in the incident table. Record commands or browser actions, target identity, sanitized fixture, output/exit status, and relevant evidence paths. Verify saved browser artifacts exist using the tool's artifact verification before relying on them. Distinguish current tool evidence from historical evidence supplied by others.
3. **Hard Rule: If you cannot reproduce the error, STOP the affected incident.**
   - Report **NOT REPRODUCED** for completed attempts without the reported symptom, **BLOCKED** for missing tools, data, permission, safe environment, or user input, and **INCONCLUSIVE** for partial/intermittent evidence. None means resolved.
   - Do NOT proceed to root-cause claims or fixes for that incident. Report attempts, actual output, environment differences, and the specific missing context. Do not switch to a remote service to force reproduction. Independent reported incidents may be investigated separately within scope.
4. If reproduction succeeds, mark **REPRODUCED** and preserve the exact scenario and assertions for post-fix verification. If a different failure occurs, report it separately rather than silently substituting it for the incident.

### Phase 3: Hypothesis-Driven Root Cause Analysis

Only proceed if the error was successfully reproduced in Phase 2. Do NOT jump to a single root cause. Complex bugs often have multiple plausible explanations — enumerate and eliminate them systematically, like the scientific method.

#### Step 1: Enumerate Hypotheses

1. Based on the reproduction from Phase 2, brainstorm **all plausible root cause hypotheses** — not just the first one that comes to mind.
2. For each hypothesis, state it as a clear, testable assumption:
   > "I assume the root cause is X, because evidence Y suggests it."
3. Rank hypotheses by:
   - **Likelihood**: how well does it explain the observed symptom?
   - **Ease of verification**: how quickly can I prove or disprove it?
4. Prefer testing high-likelihood, easy-to-verify hypotheses first.

#### Step 2: Investigation Loop (for each hypothesis)

For each hypothesis, in ranked order:

1. **State the assumption** clearly to the user.
2. **Design a diagnostic** to PROVE or DISPROVE it. This is NOT a fix — it is a test:
   - Run targeted browser state/network inspection, a local API request, or a safe local query to test the causal explanation.
   - Add temporary logging or isolate a variable in an authorized scratch copy, test fixture, or source scope. Ask before unapproved source instrumentation; record these changes and remove only your own instrumentation before final verification.
   - Check a config value, dependency version, or environment variable without exposing secrets.
   - Maintain the incident target identity and safety gate throughout diagnostics. Explain when a stub/mock changes the reproduction environment; evidence from it supports isolation, not proof the actual delivered behavior is fixed.
3. **Run the diagnostic** and observe the result.
4. **Evaluate**:
   - **If DISPROVEN**: Document what you learned in the findings log (see Step 3). Use the findings to generate new hypotheses (see Step 3b). Do NOT attempt a fix for a disproven hypothesis.
   - **If PROVEN**: You have found the root cause. Proceed to Phase 4 (Fix Planning).
   - **If INCONCLUSIVE**: Document the partial findings. Decide whether to refine this hypothesis (see Step 3b: REFINE/DEEPEN) or move to the next one.

#### Step 3: Findings Log

Maintain a running log of investigation results. For each hypothesis tested, record:
- **Hypothesis**: what you assumed
- **Diagnostic**: what you did to test it
- **Result**: proven, disproven, or inconclusive
- **What you learned**: new facts that inform future hypotheses

This log prevents repeating the same investigation and helps form better hypotheses from accumulated evidence.

#### Step 3b: Hypothesis Generation Strategies

When a hypothesis is disproven or inconclusive, do NOT immediately jump to the next pre-ranked hypothesis or escalate to the user. Use the findings to generate new, better-informed hypotheses autonomously. Try these strategies in order:

1. **REFINE — Enhance the current hypothesis**: The hypothesis was close but the diagnostic was too broad or the assumption slightly off. Narrow the scope or adjust based on what was learned.
   > "H1 (DB returns None) was disproven — DB is populated. Refining: test whether the lookup query uses the correct key."

2. **COMBINE — Merge insights from multiple disproven hypotheses**: Different disproven hypotheses each revealed a partial truth. Combine the working parts.
   > "H1 showed the DB is populated. H2 showed the query is correct. Combining: test whether a transaction rollback between insert and query is the issue."

3. **PIVOT — Use findings to identify a new root cause direction**: The findings revealed something unexpected that points to a cause not in the original list.
   > "H1-H3 all disproven, but H2's diagnostic revealed a timing issue. Pivoting: test whether a race condition between the fixture setup and the test execution is the root cause."

4. **DEEPEN — Investigate an inconclusive aspect more closely**: An INCONCLUSIVE result identified a specific unknown. Design a targeted diagnostic to resolve just that unknown.
   > "H2 was inconclusive because the log output was ambiguous. Deepening: add a more specific log at the exact query execution point."

5. **RE-RANK — Reorder remaining original hypotheses**: Findings from a disproven hypothesis may elevate a previously low-ranked one. Re-rank and test the new top candidate.

After generating a new or refined hypothesis:
- State it clearly to the user: > "Based on findings from H[N], I'm forming/enhancing H[M]: [hypothesis]."
- Add it to the findings log with its rationale.
- Return to Step 2 (state, design diagnostic, run, evaluate).

**Continue testing evidence-supported hypotheses only within the authorized scope and diagnostic/repair budget.** Each round should add useful information; user-input waits, safety gates, and stop conditions take precedence.

#### Step 4: Escalation (last resort, not default)

Do not escalate merely because the initial hypothesis list is exhausted if evidence supports another useful diagnostic within scope and budget. Escalate when:

1. Available findings no longer support a useful, testable diagnostic, OR
2. Required context, user input, tools, permission, or a safe environment is missing, OR
3. The parent/user's budget is exhausted or further work would expand the approved scope.

When escalating:
1. **Do NOT keep guessing blindly.** This leads to infinite loops.
2. Present the full findings log — all hypotheses tested and what was learned.
3. Explain specifically why you are stuck — what information is missing that would help you form a new hypothesis.
4. Ask targeted questions, not open-ended ones: "I ruled out X, Y, Z. The issue seems related to W but I can't confirm without [specific info]. Can you provide that?"
5. Wait for the user's response before continuing.

**Do NOT escalate just because the original hypothesis list is done.** The original list is a starting point, not a boundary.

#### Distinguish Symptoms from Causes

Throughout Phase 3, remember: the first line of a stack trace is often the symptom, not the root cause. Trace backward from the failure point, but verify each step with diagnostics rather than assuming. Note any technical debt, risky assumptions, or environment-specific quirks discovered during investigation.

### Phase 4: Fix Planning

1. Propose the **simplest possible fix**.
   - Prefer: correcting a parameter, fixing a typo, adding a missing check, reordering logic.
   - Avoid: adding new libraries, introducing new abstractions, rewriting large modules, refactoring unrelated code.
2. Explain the proposed fix in plain language: what line(s) will change and why.
3. **Hard Rule: Do NOT apply the fix yet.**
   - Present the plan to the user.
   - Wait for explicit user approval (e.g., "PROCEED", "go ahead", "fix it") before editing any source code.

### Phase 5: Fix Execution & Verification Loop

1. Once approved, capture the reproduced defect in a meaningful failing regression test where applicable and verify that it fails before applying the minimal root-cause fix. Follow the project's test-first requirements; never weaken assertions, rewrite the incident, or change expected behavior solely to hide failure.
2. Record the new revision/working-tree/build identity, remove your own temporary instrumentation, and rebuild/restart affected approved local services as needed. Immediately replay the exact Phase 2 scenario with the same fixture and assertions using the original browser/API/runtime tools. Do not substitute a unit-test pass for the live behavior that originally failed.
3. Evaluate the result against the incident comparison table:
   - **If expected behavior is observed and the reported failure is absent**: Run affected regression tests and relevant cross-layer/error paths. Mark **VERIFIED RESOLVED** only with fresh tool evidence for the corrected target; absence of an exception alone is not sufficient.
   - **If the error persists or output remains wrong**: Record expected/actual results and return to Phase 3, subject to the approved budget. A failed fix is evidence that the causal explanation or correction may be incomplete, not automatically proof that every part of the hypothesis is false. Reassess before proposing another fix.
   - **If replay is blocked or inconclusive**: State that the fix is unverified and what prevents verification. Do not claim resolution or start blind repairs.
4. Continue the analysis → approved fix → tool verification loop only within the authorized scope and budget. Stop on success, blockers, cancellation, unsafe effects, or exhausted budget; never extend a parent's repair limit automatically.

### Final Incident Handoff

Return the incident source/IDs, original and corrected target identities, comparison table, reproduction steps and actual tool results, root-cause evidence, changed files, regression results, and any NOT REPRODUCED/BLOCKED/INCONCLUSIVE or unverified outcomes. Separate reported claims, manual user actions, diagnostics, and verified behavior. Include cleanup performed and anything left running; stop only your own processes and remove only your own disposable fixtures when authorized. Persist a report only when requested or required by the handoff.

## Rules

1. **No Fix Without Reproduction**
   - If the error cannot be reproduced, stop immediately. No root cause analysis, no fix proposals, no code edits.

2. **Simplicity First**
   - Always prefer the smallest change that corrects the demonstrated root cause and restores the incident's expected behavior; do not merely suppress the symptom.
   - One-line fixes are better than ten-line fixes if they solve the problem.
   - Do not use a bug fix as an excuse to refactor, clean up, or modernize surrounding code.

3. **No Over-Engineering**
   - Do not add new dependencies, libraries, or frameworks to fix a bug.
   - Do not introduce new abstractions, classes, or interfaces unless the user explicitly requests it.
   - Do not rewrite modules or change APIs as part of a bug fix.
   - Stay focused on the reported issue. Minimal changes, maximum clarity.

4. **No Blind Guessing in Loops**
   - When a hypothesis is disproven, document it and move on. Never re-test the same disproven hypothesis without new evidence.
   - When a fix fails, record the actual output and reassess the causal explanation and correction; do not immediately apply another fix without re-analyzing.
   - If all hypotheses are exhausted, escalate to the user with the full findings log. Do not enter an infinite loop of guessing.

5. **Be Autonomous, Not Needy**
   - When the original hypothesis list is exhausted, generate new hypotheses from findings (REFINE, COMBINE, PIVOT, DEEPEN, RE-RANK) before escalating to the user.
   - Escalation is a last resort, not a default. The agent should keep making progress through multiple rounds of hypothesis generation.
   - Escalate sooner for missing authorization, user input, unsafe effects, blockers, or an exhausted parent/user budget. Otherwise escalate when the findings no longer support a useful diagnostic; never keep generating hypotheses to evade a stop condition.

## Example Workflow

**User**: "My test `test_user_login` is failing with `AttributeError: 'NoneType' object has no attribute 'encode'`."

**Phase 1**: Read the test output. The error occurs in `auth.py` line 42, where `user.password.encode()` is called. `user` appears to be `None`.

**Phase 2**: Run `pytest tests/test_auth.py::test_user_login -v`. The test fails with the exact same `AttributeError`. Reproduction confirmed.

**Phase 3**: Read `tests/test_auth.py` and `auth.py`. Enumerate hypotheses for why `user` is `None`:

- **H1**: `create_user()` has a bug that returns `None` instead of a User object.
- **H2**: The test fixture doesn't insert the user into the mock database, so the lookup returns `None`.
- **H3**: `auth.py` line 42 is looking up the wrong user identifier.

Rank: H2 first (high likelihood — mock DB setup is a common omission; easy to verify).

**Investigate H2**: Add a temporary print after `create_user()` in the test to inspect the mock DB state. Run the test. Output shows the mock DB is empty — the user was created but never inserted. **H2 is PROVEN.**

Findings log:
- H2: PROVEN — mock DB empty, user not inserted in fixture.

**Phase 4**: Propose fix: Add `db.session.add(user)` and `db.session.commit()` in the test fixture in `tests/test_auth.py`. This is a one-line change in the test setup. Explain this to the user and wait for approval.

**Phase 5**: User says "PROCEED". Apply the fix. Re-run `pytest tests/test_auth.py::test_user_login -v`. Test passes. Run the full `test_auth.py` suite to check for regressions — all pass. Report success.

---

**Example with a failed first fix (loop):**

Suppose H2 was disproven (the mock DB was correctly populated). The findings log would record this, and the agent would move to H1 or H3. If a fix based on a later hypothesis also fails in Phase 5, the agent returns to Phase 3 with the accumulated findings (not from scratch), forms a new hypothesis informed by what was ruled out, and tries again. If all hypotheses are exhausted, the agent escalates to the user with the full findings log.

## Error-Handling Notes

- **Missing logs or traces**: Ask the user for the full output. Partial traces often hide the real failure point.
- **Flaky or intermittent errors**: Note the inconsistency. Reproduction may require multiple runs. If it cannot be reliably reproduced, report this and stop.
- **Environment-specific issues**: If the bug only appears in the user's environment and not in yours, document the differences (OS, dependency versions, env vars) and ask the user to verify the fix in their environment.
- **Multiple symptoms**: Focus on one reproducible symptom at a time. Fixing the root cause often resolves secondary symptoms automatically.
