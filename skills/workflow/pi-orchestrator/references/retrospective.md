# Run Records and Retrospectives

## Purpose

Collect evidence during each approved `pi-orchestrator` run so a later, separate Pi session can review the manager, workers, and handoffs. This is an orchestrator obligation, not a background daemon or an automatic change to all Pi invocations. No new package or logging extension is required.

Pi JSONL files are the primary source: they preserve messages, tool calls/results, and branches. Terminal logs supplement them with shell startup and crash output; they contain ANSI redraws, may repeat text, and do not reliably preserve all tool details. Reports are worker claims unless backed by actual evidence.

## Before dispatch

1. Choose a unique run ID, and resolve the absolute project and run directories. Use `.pi/msp/<run-id>/` with `logs/` and `reports/` subdirectories. Set `umask 077` before creating records; restrict the run directory to its owner. Ensure `.pi/msp/` is excluded from Git in the target project (project ignore rules or local `.git/info/exclude`); never stage these records.
2. Identify the current parent Pi session file and ID from available session metadata. If unavailable, ask the user for the `/session` output. Record unknown values as `null` with an evidence gap; never infer the parent from the most recently modified JSONL file. Ordinary Pi sessions already persist automatically. If the parent is ephemeral, obtain an explicit JSONL export of its available history before shutdown and label it as an export, not a live transcript.
3. Record grounding artifact paths, the project revision and dirty state, and relevant agent/context/skill paths and their SHA-256 hashes. Record the skills repository revision and dirty state when available. Hashes identify versions but do not reconstruct lost contents; raw skill reads in the transcript or separately approved snapshots provide that evidence. Record newly loaded skills when they become relevant. Use `null` plus a gap note for unavailable identities rather than presenting current files as historical versions.
4. Create `manifest.json` before starting a worker. Use absolute artifact paths. Keep all worker records when updating the manifest; write updates through a temporary file and rename it to avoid partial JSON. The parent is the only manifest writer.

## Manifest shape

This example shows a worker registered before launch. Replace example values with observed values. Timestamps are UTC ISO 8601. `null` means unknown/not yet observed, not a successful verification.

```json
{
  "schema_version": 1,
  "run_id": "run-20261012-001",
  "started_at": "2026-10-12T12:00:00Z",
  "finished_at": null,
  "project_dir": "/absolute/project",
  "code_identity": {
    "revision": null,
    "dirty": null,
    "target_snapshot": null
  },
  "grounding_artifacts": [],
  "instruction_versions": [
    {
      "path": "/absolute/skills/workflow/pi-orchestrator/SKILL.md",
      "sha256": null,
      "repository_revision": null,
      "repository_dirty": null
    }
  ],
  "parent": {
    "session_id": null,
    "transcript_path": null,
    "record_kind": "live-session",
    "transcript_verified": false
  },
  "workers": [
    {
      "worker_id": "implementation-r0-a1",
      "role": "implementation",
      "round": 0,
      "attempt": 1,
      "task": "Approved task objective and acceptance criteria references",
      "session_id": "run-20261012-001-implementation-r0-a1",
      "transcript_path": null,
      "transcript_verified": false,
      "tmux_session": null,
      "tmux_pane": null,
      "terminal_log_path": "/absolute/project/.pi/msp/run-20261012-001/logs/implementation-r0-a1.terminal.log",
      "terminal_log_verified": false,
      "report_paths": [],
      "sentinel_path": null,
      "started_at": null,
      "finished_at": null,
      "status": "registered",
      "outcome": null,
      "evidence_gaps": []
    }
  ],
  "evidence_gaps": []
}
```

## Launch and verification

- Register each worker first. Use unique session IDs and log paths for every role, round, and replacement attempt. Independent tmux workers are not automatically branches or children of the parent Pi session; the manifest supplies that relationship.
- Use the skill's launch sequence: create a new owned pane running `/bin/zsh -l`, attach output logging to that exact pane ID, record its identity, then launch persistent interactive Pi. Never use `--no-session`, `--print`, or a `tee` pipeline for these workers. Failures during creation/log setup must stop launch and be recorded.
- Resolve the actual worker transcript after its first completed turn. Use `/session` only while the worker is idle, or find the JSONL whose first-line session header exactly matches the assigned session ID in its configured session storage. The default root is `~/.pi/agent/sessions/`; CLI/environment/settings can override it. Do not assume a filename from an ID or select a file by modification time. An unknown storage location is an evidence gap, not permission to search unrelated private directories.
- Verify that the linked file exists, is readable JSONL, and has the expected session header ID. Record the observed path and verification status. For the parent, verify its header against the observed parent ID too. A file may be changing during a live run: do not rewrite it, and record the review cutoff if inspecting it before completion.
- Verify the terminal log exists and contains Pi startup or task output after the first completed turn. Record unavailable logging as a gap; do not silently substitute captured scrollback for a full log. Terminal logs start when attached, so they cannot recover earlier output.
- If a worker crashes or times out, preserve the transcript and log, update its status/gaps, and register a new attempt only under the existing orchestration approval and budget. A `.done` sentinel proves neither transcript capture nor a successful task outcome.

## Completion and retention

After workers finish, verify the transcript/log links again, add actual report and sentinel paths, outcomes and finish timestamps, and finalize the run manifest. Keep the parent transcript marked as live if the parent session continues after the run.

Before closing any owned pane, confirm its evidence is saved. Disable its pipe with `tmux pipe-pane -t <owned-pane-id>` when no more output is needed; allow pending output to reach disk before final log checks. Do not disable another pane's pipe, delete logs, or remove Pi transcripts during cleanup. A failed pipe may not be detected by tmux command success alone, so inspect the saved log. Existing `--no-session` runs cannot be recovered beyond surviving exports, reports, or terminal history.

Manifests link Pi's existing session files instead of duplicating them. Deleting those sessions breaks the links. No automatic deletion, retention period, upload, or external backup is introduced; ask the user before adding those policies. Logs, transcripts, and manifests may contain secrets or private file contents: keep them local, review/redact before sharing, and do not assume a private gist is safe.

## Separate retrospective session

Start a fresh Pi session after the run, rather than appending retrospective commentary to a worker's source transcript. Invoke the explicitly user-triggered [retro skill](../../../reviewing/retro/SKILL.md):

```text
/skill:retro /absolute/project/.pi/msp/<run-id>/manifest.json
```

The retro skill defines evidence analysis, improvement categories, approval boundaries, and report output (`<run-directory>/retro.md`). For several runs, supply their manifests together and compare recurring patterns. Do not rely only on `/clone` for a whole-run retrospective: it copies the active branch, not every abandoned branch or independent worker.
