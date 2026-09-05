# examples_notebook

OpenAI ChatGPT examples in Jupyter notebooks.

## Setup

```powershell
cd courses/02_BuildingWithTheClaudeAPI/projects/examples_notebook
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe -m ipykernel install --user --name=examples_notebook --display-name="Python (examples_notebook)"
```

Add your key to `.env`:

```text
OPENAI_API_KEY=sk-...
```

## Launch (classic Notebook)

```powershell
.\.venv\Scripts\jupyter.exe notebook
```

Open `notebooks/01_getting_started.ipynb` and select kernel **Python (examples_notebook)**.

## Tests

```powershell
.\.venv\Scripts\python.exe -m unittest discover -s tests -v
```
