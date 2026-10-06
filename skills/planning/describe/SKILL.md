---
name: describe
description: "Generate a concise, accurate project overview from the current directory. Use when the user asks to summarize, overview, describe, or understand a project, codebase, or repository."
---

# /describe

Generate a concise, accurate project overview from the current directory. Reports what the project is, what it does, its tech stack, main components, entry points, and key dependencies.

## Usage

```
/describe                 # overview of the current directory
/describe <path>          # overview of a specific directory
/describe --format json   # output as structured JSON
```

## What describe is for

- Quickly understand a codebase you just opened
- Summarize a project for documentation, onboarding, or a README
- Identify the tech stack, build system, and entry points
- Surface architecture and main modules without reading every file

## What You Must Do When Invoked

If no path is given, use `.` (current directory). Do not ask the user for a path unless the directory is ambiguous.

### Step 1 — Determine project root

Find the project root by looking for the deepest directory (closest to `.`) that contains any of:

- `package.json`
- `pyproject.toml`, `setup.py`, `requirements.txt`
- `Cargo.toml`
- `go.mod`
- `Gemfile`
- `pom.xml`, `build.gradle`
- `CMakeLists.txt`
- `.git` directory
- `README.md` or `README*` at the root level

Use that directory as `PROJECT_ROOT`. If none are found, use the directory the user provided.

### Step 2 — List top-level structure

Run:

```bash
ls -la PROJECT_ROOT
```

Then summarize the top-level layout in 3-6 bullet points. Ignore common noise files (`.git`, `node_modules`, `.venv`, `__pycache__`, `dist`, `build`, `.DS_Store`).

### Step 3 — Read key project files

Read these files if they exist, in this order. Stop early only if the README is comprehensive:

1. `README.md` (or `README.*`)
2. `package.json` / `pyproject.toml` / `setup.py` / `Cargo.toml` / `go.mod` / `pom.xml` / `build.gradle` / `CMakeLists.txt`
3. `requirements.txt` / `package-lock.json` / `yarn.lock` / `Gemfile.lock` / `Cargo.lock`
4. One config file that reveals framework or runtime: `next.config.*`, `vite.config.*`, `webpack.config.*`, `tsconfig.json`, `django/settings.py`, `app.py`, `main.py`, `src/index.*`, `lib/main.dart`, etc.
5. `.github/workflows/*.yml` (to understand CI/CD and deployment)

Do not read more than ~10 files. The goal is a high-level overview, not a deep audit.

### Step 4 — Identify architecture and main modules

Look at the directory tree one level below `src/`, `lib/`, `app/`, `packages/`, or `tests/`:

```bash
find PROJECT_ROOT -maxdepth 2 -type d | head -40
```

Map the main folders to roles (e.g. "authentication", "database models", "API routes", "UI components", "tests"). Mention only the meaningful ones.

### Step 5 — Output the overview

Produce a single Markdown response with these sections:

1. **Project** — one-sentence description of what this project does.
2. **Tech Stack** — languages, frameworks, runtime, package manager, database (if obvious).
3. **Structure** — top-level layout and the role of key directories.
4. **Entry Points** — main commands, scripts, or files someone runs to start/use the project.
5. **Key Dependencies** — 3-8 notable runtime dependencies and what they do.
6. **Notable Observations** — optional: anything unusual, missing, or worth flagging (e.g. no tests, no README, mixed languages, legacy patterns).

Keep the entire output concise: 1-2 short paragraphs per section, 6 sections total. Do not paste large file contents verbatim.

### For `--format json`

Return only a JSON object with this shape:

```json
{
  "project": "string",
  "tech_stack": ["string"],
  "structure": {
    "root": "string",
    "directories": [
      {"path": "string", "role": "string"}
    ]
  },
  "entry_points": ["string"],
  "key_dependencies": [
    {"name": "string", "purpose": "string"}
  ],
  "notable_observations": ["string"]
}
```

No Markdown wrapper, no commentary outside the JSON.

## Rules

- Be factual. If you cannot determine something, say "Not obvious from the files read" instead of guessing.
- Do not execute install, build, test, or deploy commands. This skill is read-only.
- Do not read binary files, secrets, `.env` files, or credentials.
- If the project is empty or unsupported, say so plainly and stop.
- If multiple possible project roots exist (e.g. monorepo), pick the root that contains the workspace config and note sub-packages briefly.
