---
name: start-python-notebook-project
description: >-
  Scaffolds a Python Jupyter notebook project after asking setup questions
  first, then creates notebooks/, optional src/, venv, starter .ipynb, installs
  jupyter + python-dotenv (and extras), and registers an ipykernel. Use when the
  user wants to start a Python notebook project, Jupyter Lab/Notebook scaffold,
  or bootstrap a course-style notebooks workspace.
disable-model-invocation: false
---

# Start Python Notebook Project

Quick-start a **Python + Jupyter** project in this repo. **Ask questions first. Do not create anything until answers are complete.** Then execute.

Modeled on `start-python-project`, but notebook-first.

## Phase 1 — Questions (required)

Ask these before any file or command. Prefer one short message with all questions.

1. **Path (always required)** — Exact folder where the project should live (absolute or repo-relative). Never assume `projects/` or a course folder.
2. **Project name** — Folder/kernel display name (snake_case preferred).
3. **Purpose** — One sentence: what the notebooks explore or build.
4. **Notebook UI (always ask)** — `jupyterlab` or `notebook` (classic). Never assume.
5. **Helpers** — Create `src/<name>/` with `__init__.py` for shared Python helpers? (yes/no)
6. **Extra dependencies** — Packages beyond the defaults (or "none").
7. **Secrets** — Create empty `.env` + document keys? (yes/no)
8. **Tests** — Create `tests/` with a placeholder? (yes/no)

**Always include in `requirements.txt`:** `jupyter`, `python-dotenv`, `ipykernel`.  
If UI is `jupyterlab`, also add `jupyterlab`. Then append any extra dependencies from the user.

If the user already answered some items in the same message, do not re-ask those. Still ask for any missing required item, especially **path** and **Notebook UI**.

## Phase 2 — Confirm briefly

Restate in 3–5 bullets: path, name, UI, helpers, deps, extras. Then execute immediately. Only pause if something is ambiguous or the target path already exists and is non-empty.

## Phase 3 — Execute

Work inside the answered path. On Windows use PowerShell-friendly commands.

### Create structure

Default layout (option A):

```text
<project-path>/
  notebooks/
    01_getting_started.ipynb
  requirements.txt
  README.md
  .gitignore
  .env                      # only if secrets=yes
  src/<name>/               # only if helpers=yes
    __init__.py
  tests/test_smoke.py       # only if tests=yes
```

Do **not** put notebooks at the project root unless the user later asks to change the layout.

### Default file contents

**.gitignore**
```text
.venv/
__pycache__/
*.pyc
.env
.DS_Store
.ipynb_checkpoints/
```

**requirements.txt** — at minimum:
```text
jupyter
python-dotenv
ipykernel
```
Add `jupyterlab` when UI is `jupyterlab`. Add extras from the user, one package per line.

**.env** (if requested) — placeholders only; never invent real secrets.

**README.md** — name, purpose, setup (venv + install), how to launch the chosen UI, how to select the project kernel.

**`src/<name>/__init__.py`** (if helpers=yes) — empty or a one-line package docstring.

**`notebooks/01_getting_started.ipynb`** — valid notebook with:
1. Markdown cell: project name + purpose
2. Code cell: `from dotenv import load_dotenv; load_dotenv(); print("Notebook ready")`
3. If helpers=yes, a code cell that imports from `src` (document `PYTHONPATH` or sys.path note in README)

**`tests/test_smoke.py`** (if tests=yes) — simple unittest that `import dotenv` succeeds (or imports a tiny helper if present).

### Environment and install

From `<project-path>`:

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install --upgrade pip
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe -m ipykernel install --user --name=<name> --display-name="Python (<name>)"
```

Prefer invoking `.\.venv\Scripts\python.exe` over relying on shell activation.

### Verify

1. Confirm `notebooks/01_getting_started.ipynb` exists and is valid JSON.
2. Confirm packages import:

```powershell
.\.venv\Scripts\python.exe -c "import jupyter, dotenv, ipykernel; print('ok')"
```

3. Do **not** leave a long-running Jupyter server open unless the user asks. Tell them how to start it:

```powershell
# jupyterlab
.\.venv\Scripts\jupyter.exe lab
# notebook
.\.venv\Scripts\jupyter.exe notebook
```

If tests=yes, run:

```powershell
.\.venv\Scripts\python.exe -m unittest discover -s tests -v
```

Fix any immediate failure, then stop.

## Rules

- Python + Jupyter only — do not scaffold other languages with this skill.
- Always ask for path — no default parent folder.
- Always ask Notebook UI — `jupyterlab` or `notebook`.
- Always install `jupyter` and `python-dotenv` (plus `ipykernel` for the kernel).
- Do not commit, push, or touch git unless the user asks.
- Do not overwrite non-empty existing projects without explicit confirmation.
- Keep the scaffold minimal — no extra frameworks unless requested as dependencies.
- Never put real API keys in files; `.env` is placeholders only.

## Done response

After execution, report:
- Created path
- Notebook UI chosen
- How to launch Jupyter and pick the kernel
- Deps installed (defaults + extras)
