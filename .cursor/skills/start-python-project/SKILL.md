---
name: start-python-project
description: >-
  Scaffolds a runnable Python project after asking setup questions first, then
  creates folders, venv, starter files, and installs dependencies. Use when the
  user wants to start quickly a Python project, scaffold a new Python app,
  bootstrap a venv, or set up a new project under any path in this repo.
disable-model-invocation: false
---

# Start Python Project

Quick-start a **Python-only** project in this repo. **Ask questions first. Do not create anything until answers are complete.** Then execute.

## Phase 1 — Questions (required)

Ask these before any file or command. Prefer one short message with all questions.

1. **Path (always required)** — Exact folder where the project should live (absolute or repo-relative). Never assume `projects/` or a course folder.
2. **Project name** — Package/folder name (snake_case preferred).
3. **Purpose** — One sentence: what the app does.
4. **Layout** — Choose one:
   - `flat` — `main.py` at project root
   - `package` — `src/<name>/` with `__init__.py` and `main.py`
5. **Dependencies** — Packages to put in `requirements.txt` (or "none").
6. **Secrets** — Create empty `.env` + document keys? (yes/no)
7. **Tests** — Create `tests/` with a placeholder? (yes/no)

If the user already answered some items in the same message, do not re-ask those. Still ask for any missing required item, especially **path**.

## Phase 2 — Confirm briefly

Restate in 3–5 bullets: path, name, layout, deps, extras. Then execute immediately (user already chose execute mode for this skill). Only pause if something is ambiguous or the target path already exists and is non-empty.

## Phase 3 — Execute

Work inside the answered path. On Windows use PowerShell-friendly commands.

### Create structure

**flat**
```text
<project-path>/
  main.py
  requirements.txt
  README.md
  .gitignore
  .env                 # only if secrets=yes
  tests/test_main.py   # only if tests=yes
```

**package**
```text
<project-path>/
  src/<name>/
    __init__.py
    main.py
  requirements.txt
  README.md
  .gitignore
  .env                 # only if secrets=yes
  tests/test_main.py   # only if tests=yes
```

### Default file contents

**.gitignore**
```text
.venv/
__pycache__/
*.pyc
.env
.DS_Store
```

**requirements.txt** — one package per line from the user's answer; empty file if none.

**main.py** — minimal runnable stub that prints a short hello tied to the project purpose.

**README.md** — name, purpose, setup (venv + install), how to run.

**.env** (if requested) — empty or commented placeholder keys only; never invent real secrets.

### Environment and install

From `<project-path>`:

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install --upgrade pip
```

If `requirements.txt` has packages:

```powershell
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

Prefer invoking `.\.venv\Scripts\python.exe` over relying on shell activation.

### Verify

Run the entry point once with the venv Python, e.g.:

```powershell
.\.venv\Scripts\python.exe main.py
# or
.\.venv\Scripts\python.exe -m src.<name>.main
```

Fix any immediate failure, then stop.

## Rules

- Python only — do not scaffold other languages with this skill.
- Always ask for path — no default parent folder.
- Do not commit, push, or touch git unless the user asks.
- Do not overwrite non-empty existing projects without explicit confirmation.
- Keep the scaffold minimal — no extra frameworks unless requested as dependencies.
- Never put real API keys in files; `.env` is placeholders only.

## Done response

After execution, report:
- Created path
- Layout used
- How to activate / run
- Deps installed (or none)
