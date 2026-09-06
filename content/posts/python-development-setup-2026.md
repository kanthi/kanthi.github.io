+++
title = "Python Setup for Everybody in 2026"
date = "2026-09-06T10:30:00+05:30"
draft = true
author = "Kanthi"
authorTwitter = "kanthi"
cover = ""
tags = ["python", "setup", "uv", "ruff", "ty", "pytest", "fastapi", "django", "data-science", "ai", "devops", "vscode", "tools"]
keywords = ["python setup 2026", "uv", "ruff", "ty", "vscode", "pytest", "fastapi", "data science", "devops"]
description = "A follow-along Python setup for 2026: from first script to data science, AI, and DevOps, plus a full VS Code setup wired to uv, Ruff, and ty."
showFullContent = false
readingTime = true
hideComments = false
Toc = true
+++

This is a **draft**. One toolchain, used the same way whether you are writing a 20-line script, a FastAPI service, a notebook, or a container that goes to CI. Sections 0–8 are the terminal. Section 9 wires the same stack into VS Code. Sublime and Zed are a different family — later.

If you'd told me a few years ago that I'd stop typing `pip install` on new projects, drop Black, and barely open mypy, I'd have assumed you were talking about somebody else's machine. Rust happened. A handful of tools ate a whole drawer of one-purpose utilities.

**The stack:** [uv](https://docs.astral.sh/uv/) (Python + packages + venv + lockfile), [Ruff](https://docs.astral.sh/ruff/) (lint + format), [ty](https://docs.astral.sh/ty/) (types), [pytest](https://docs.pytest.org/) (tests), Git, Docker when you ship.

## How to read this

Don't install everything on day one. Walk the levels. Stop when it matches what you actually do.

| You want | Go to |
|---|---|
| Install Python once | [0](#0-install-once) |
| Run a script / REPL | [1](#1-first-ten-minutes) |
| A real project (everyone) | [2](#2-a-real-project-the-shared-foundation) |
| HTTP APIs | [3](#3-web-apis-fastapi) |
| Full web app | [4](#4-django-when-you-need-the-whole-app) |
| Data science / notebooks | [5](#5-data-science) |
| ML / AI | [6](#6-machine-learning-and-ai) |
| CI, Docker, lockfiles | [7](#7-devops) |
| CLIs and glue scripts | [8](#8-clis-and-glue) |
| VS Code | [9](#9-vs-code) |

Same commands at every level: `uv add`, `uv run`, `uv sync`. You do **not** activate a venv by hand.

## The toolbox

| Tool | Replaces | Job |
|---|---|---|
| **uv** | pip, venv, pip-tools, pipx, pyenv, Poetry | Install Python, create the env, add deps, lock, run |
| **Ruff** | Flake8, isort, Black, pyupgrade, autoflake | Lint and format, one config |
| **ty** | mypy / Pyright on new work | Type check (still beta; keep mypy in old CI if you must) |
| **pytest** | unittest for daily work | Tests |
| **Git** | — | History |
| **Docker** | "works on my machine" | Ship the same env |

I still use [mise](/posts/mise-version-manager/) for Go/Node/etc. on a machine. For a *Python project*, uv owns Python.

---

## 0. Install once

You need a compiler toolchain only if you later build packages with native extensions. For most people, uv is enough.

**Linux / macOS:**

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
# then restart the shell, or:
source "$HOME/.local/bin/env"
uv --version
```

On macOS you can also `brew install uv`. I install Git from the OS package manager (`sudo pacman -S git`, `sudo apt install git`, `brew install git`).

**Windows (PowerShell):**

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
uv --version
```

Install the Python versions you actually use. uv fetches official builds; you do not need the python.org installer, pyenv, or the Microsoft Store Python.

```bash
uv python install 3.12 3.13
uv python list
```

3.12 is the boring default for new work in this post. Pin per project later; don't fight a global default.

Sanity check:

```bash
uv run python -c "import sys; print(sys.version)"
```

That command created a throwaway env, ran Python, and quit. That is the whole mental model.

---

## 1. First ten minutes

You do not need a project yet.

**REPL:**

```bash
uv run python
```

**One-off script** (`hello.py` anywhere):

```python
def main() -> None:
    print("hello")


if __name__ == "__main__":
    main()
```

```bash
uv run python hello.py
```

**Need a library for one shot**, without polluting a project:

```bash
uvx ruff --version          # run a published CLI
uv run --with httpx python -c "import httpx; print(httpx.get('https://example.com').status_code)"
```

`uvx` is pipx: run a tool, throw the env away. Good for `ruff`, `ty`, `cookiecutter`, `httpie`.

When the script grows a second file or a dependency you want to keep, make a project. That is the next section. Skip it and you will regret it the first time a notebook and a web app share one global site-packages.

---

## 2. A real project (the shared foundation)

This layout is what I use for *everything* after "hello". Data science, APIs, CLIs — same skeleton, extra packages.

```bash
uv init myproject --python 3.12
cd myproject
uv python pin 3.12
```

`uv init` gives you `pyproject.toml`, a `.python-version`, a `.gitignore`, and a sample script. I then nudge it toward a `src/` layout and a tests folder:

```bash
mkdir -p src/myproject tests
mv hello.py src/myproject/__init__.py 2>/dev/null || true
```

Add the quality tools once:

```bash
uv add --dev pytest pytest-cov ruff
uv tool install ty@latest
```

Ruff can live as a dev dependency (`uv add --dev ruff`) so `uv run ruff` uses the project pin. ty is still easiest as a user tool (`uv tool install`) until it settles.

### `pyproject.toml` — the bits I always set

Keep `[project]` as uv wrote it, then add:

```toml
[tool.ruff]
line-length = 88
target-version = "py312"

[tool.ruff.lint]
select = ["E", "F", "I", "UP", "B", "SIM"]

[tool.ruff.format]
quote-style = "double"

[tool.pytest.ini_options]
testpaths = ["tests"]
pythonpath = ["src"]
```

Daily loop:

```bash
uv add httpx                 # runtime dep → pyproject.toml + uv.lock
uv add --dev pytest          # test-only
uv sync                      # create/update .venv from the lock
uv run python -m myproject
uv run ruff check --fix .
uv run ruff format .
ty check src/
uv run pytest --cov=myproject
```

`uv.lock` is the reproducibility story. Commit it. Teammates and CI run `uv sync --frozen` and get the same bytes.

A first test so pytest has something to do:

```python
# tests/test_smoke.py
def test_smoke() -> None:
    assert 1 + 1 == 2
```

```bash
uv run pytest
```

Git, now, not later:

```bash
git init
git add pyproject.toml uv.lock .python-version src tests
git commit -m "start"
```

That is the whole "professional Python" baseline. Everything below is extra packages on this project, or a second project with the same shape.

---

## 3. Web APIs: FastAPI

New project, or `uv add` into the one you already have.

```bash
uv add fastapi "uvicorn[standard]" pydantic httpx
uv add --dev pytest pytest-asyncio
```

```python
# src/myproject/api.py
from fastapi import FastAPI

app = FastAPI()


@app.get("/health")
def health() -> dict[str, str]:
    return {"status": "ok"}
```

```bash
uv run uvicorn myproject.api:app --reload --app-dir src
```

Open `/docs` — OpenAPI is free. [Pydantic v2](https://docs.pydantic.dev/) is how request/response shapes stay honest.

Flask still works. I reach for it less. FastAPI covers the small service case with better defaults.

---

## 4. Django, when you need the whole app

Admin, auth, ORM, templates, "this is a product not an API" — still [Django](https://www.djangoproject.com/).

```bash
uv init mysite --python 3.12
cd mysite
uv add django
uv run django-admin startproject config .
uv run python manage.py migrate
uv run python manage.py runserver
```

Same Ruff / ty / pytest story as section 2. I have an older [Django env](/posts/django-dev-environment-setup/) post that still talks `venv`; treat that as stale until I rewrite it against uv.

---

## 5. Data science

Do **not** `pip install pandas` onto system Python. Same project model as section 2. Notebooks live *inside* the project so the kernel matches the lockfile.

```bash
uv init analysis --python 3.12
cd analysis
uv python pin 3.12
uv add pandas numpy scipy matplotlib seaborn jupyterlab
uv add --dev ruff pytest
```

I keep data and notebooks out of `src/`:

```text
analysis/
  pyproject.toml
  uv.lock
  src/analysis/
  notebooks/
  data/raw/          # not committed if it is big
  data/processed/
  tests/
```

Start Jupyter **through uv**, not from a random `jupyter` on `$PATH`:

```bash
uv run jupyter lab
```

In the notebook, the kernel should be this project's `.venv`. If Jupyter offers a "Python 3" that isn't, pick the one named after the project or register it:

```bash
uv add ipykernel
uv run python -m ipykernel install --user --name analysis --display-name "Python (analysis)"
```

Prefer moving anything that matters out of the notebook into `src/analysis/` and importing it. Notebooks are for exploration; tests don't run notebooks well.

Plotting: Matplotlib/Seaborn for the usual charts, Plotly when I need to poke at the figure. Add them when you need them:

```bash
uv add plotly
```

Lockfile still matters here — especially here. "It worked on my laptop" in data work is usually a NumPy/Pandas wheel mismatch.

A slightly longer writeup lives in [uv + data science](/posts/uv-python-data-science-setup/) (also a draft). This section is the version I actually follow.

---

## 6. Machine learning and AI

Start small. sklearn covers a lot of "I have a table and I want a model". Deep learning stacks are heavy; don't put PyTorch and TensorFlow in the same env unless you enjoy dependency archaeology.

**Classical ML:**

```bash
uv add scikit-learn
```

**PyTorch** (CPU is fine to start):

```bash
uv add torch torchvision --index https://download.pytorch.org/whl/cpu
```

For CUDA, use the index from [pytorch.org](https://pytorch.org/get-started/locally/) that matches your driver. Don't copy a random `pip install torch` line from a blog — the index URL is the whole trick, and uv honors it.

**Working with APIs instead of local weights** (often the right first AI step):

```bash
uv add openai httpx pydantic
```

Talk to OpenAI-compatible endpoints (including local ones) with the official client; keep keys in the environment, not in notebooks.

**Local models later:** llama.cpp / Ollama as *separate* processes; your Python project stays a thin client. Mixing a 7B download into the same venv as pandas is how laptops melt.

**Training hygiene** that is just section 2 again: pin Python, commit `uv.lock`, put real code in `src/`, pytest for the data plumbing, Ruff on everything you will re-read. GPU drivers and CUDA are OS packages, not `uv add` lines.

---

## 7. DevOps

If section 2 is the project, this is how it leaves your laptop.

### Lock and sync

```bash
uv lock                  # refresh uv.lock
uv lock --upgrade        # bump everything (consciously)
uv sync --frozen         # CI / prod: do not re-resolve
uv sync --no-dev         # runtime image: skip pytest/ruff
```

### GitHub Actions

```yaml
# .github/workflows/ci.yml
name: ci
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v6
      - run: uv sync --frozen --all-groups
      - run: uv run ruff check .
      - run: uv run ruff format --check .
      - run: ty check src
      - run: uv run pytest
```

`setup-uv` caches the uv binary and the wheels. First CI run is slow-ish; the rest are not.

Cross-version (3.12 and 3.13): matrix the `uv python install` / `uv python pin` rather than bringing back tox unless you already like tox. I use [nox](https://nox.thea.codes/) when the matrix is more than "two Pythons, same tests".

### Docker

Multi-stage, uv in the builder, slim runtime, **frozen** lock:

```dockerfile
FROM python:3.12-slim AS builder
COPY --from=ghcr.io/astral-sh/uv:latest /uv /bin/uv
WORKDIR /app
COPY pyproject.toml uv.lock ./
RUN uv sync --frozen --no-dev --no-install-project
COPY src ./src
RUN uv sync --frozen --no-dev

FROM python:3.12-slim
COPY --from=builder /app/.venv /app/.venv
COPY src /app/src
ENV PATH="/app/.venv/bin:$PATH"
WORKDIR /app
CMD ["python", "-m", "myproject"]
```

Same image locally and in CI. Compose for Postgres/Redis next to the app; that is a service graph, not a Python problem.

### What I skip until it hurts

- **pre-commit** — Ruff + ty in CI already catch the cheap mistakes. I'll add hooks if people keep pushing unformatted files.
- **Poetry export, requirements.txt** — only if a downstream tool cannot read `uv.lock`. `uv export --frozen -o requirements.txt` is the escape hatch.

---

## 8. CLIs and glue

Python is still the best tape on a homelab.

```bash
uv add rich typer
```

```python
from rich.console import Console
from rich.table import Table

console = Console()
table = Table(title="Project Dependencies")
table.add_column("Package", style="cyan")
table.add_column("Version", style="green")
table.add_row("fastapi", "0.115.0")
console.print(table)
```

[Typer](https://typer.tiangolo.com/) (or Click) for arguments; [Rich](https://github.com/Textualize/rich) for output; [Textual](https://github.com/Textualize/textual) only if you are actually building a TUI.

Install *your* CLI the way you install Ruff:

```bash
uv tool install .
# or, while iterating:
uv run mycli --help
```

---

## What I stopped reaching for

| Old | Now |
|---|---|
| pip + `python -m venv` + activate | `uv add` / `uv run` / `uv sync` |
| pyenv | `uv python install` / `uv python pin` |
| pipx | `uvx` / `uv tool install` |
| Poetry | uv |
| Black + isort | `uv run ruff format` + Ruff's `I` rules |
| Flake8 + plugins | `uv run ruff check` |
| mypy on new repos | `ty check` |
| Anaconda as a default | uv project + conda only if a shop already standardized on it |

Anaconda is not *wrong* for some scientific shops. For everyone else it is a second package universe you have to keep in your head. I don't start there anymore.

---

## A default I copy

New folder, any kind of Python work:

```bash
uv init "$NAME" --python 3.12
cd "$NAME"
uv python pin 3.12
uv add --dev pytest pytest-cov ruff
uv tool install ty@latest
# then only the extras you need:
# uv add fastapi "uvicorn[standard]"
# uv add pandas numpy jupyterlab
# uv add scikit-learn
```

Daily:

```bash
uv sync
uv run ruff check --fix . && uv run ruff format .
ty check src
uv run pytest
```

That is enough. The best setup is the one that disappears.

---

## 9. VS Code

The tools in sections 0–8 do not care which editor you type in. VS Code has one job here: use **this project's** `.venv`, run **Ruff** on save, run **ty** for types, run **pytest** in the testing sidebar. If the status bar says some other Python, it is wrong.

Put the config in the **repo** (`.vscode/`), not only in user settings, so the same project opens the same way on another machine.

[Cursor](https://cursor.com/) and [Antigravity](https://antigravity.google/) are VS Code under the hood. They read the same `.vscode/extensions.json`, `settings.json`, and `launch.json`. Configure VS Code as below; copy the folder, don't invent a second Python setup. The AI layer is extra — it is not a package manager. Point the agent at `uv add` / `uv run`, not `pip install`.

User vs workspace (workspace wins):

| | Linux | macOS |
|---|---|---|
| VS Code user | `~/.config/Code/User/settings.json` | `~/Library/Application Support/Code/User/settings.json` |
| **This project** | `.vscode/settings.json` | same |

I keep user settings thin (font, theme, vim keys). Python lives in `.vscode/`.

### Extensions (install once)

Command Palette → **Extensions: Show Recommended Extensions** after you commit `.vscode/extensions.json`.

```json
{
  "recommendations": [
    "ms-python.python",
    "ms-python.debugpy",
    "charliermarsh.ruff",
    "astral-sh.ty",
    "ms-toolsai.jupyter",
    "ms-toolsai.jupyter-keymap",
    "ms-azuretools.vscode-docker",
    "tamasfe.even-better-toml"
  ]
}
```

What each one is for:

- **Python** (`ms-python.python`) — interpreter picker, testing, terminals. It is *not* the linter anymore.
- **Ruff** (`charliermarsh.ruff`) — lint, format, organize imports. Official Astral extension; ships `ruff server`.
- **ty** (`astral-sh.ty`) — types + (by default) the language server. Official Astral extension; different publisher than Ruff (`astral-sh` vs `charliermarsh`).
- **debugpy** — breakpoints. Don't debug with a random system `python`.
- **Jupyter** — notebooks in the editor for section 5. Kernel = the same `.venv`.
- **Even Better TOML** — `pyproject.toml` is now the config file; syntax highlighting is not optional.

**Pylance:** still excellent on Microsoft VS Code. The ty extension **turns the Python language server off** so you do not run two. That is the setup I want on new work: ty for hover/go-to/completions, Ruff for lint/format.

If you miss Pylance, keep it and tell ty to stay in its lane:

```json
{
  "python.languageServer": "Pylance",
  "ty.disableLanguageServices": true
}
```

Don't run Pylance *and* ty language services at the same time. Two servers on the same buffer is a crowd.

```bash
code --install-extension ms-python.python
code --install-extension ms-python.debugpy
code --install-extension charliermarsh.ruff
code --install-extension astral-sh.ty
code --install-extension ms-toolsai.jupyter
```

### Workspace settings I copy

`.vscode/settings.json` — this is the file that makes the editor disappear:

```json
{
  "python.defaultInterpreterPath": "${workspaceFolder}/.venv/bin/python",
  "python.terminal.activateEnvironment": true,
  "python.testing.pytestEnabled": true,
  "python.testing.unittestEnabled": false,
  "python.testing.pytestArgs": ["tests"],
  "python.analysis.autoImportCompletions": true,

  "ruff.nativeServer": "on",
  "ruff.importStrategy": "fromEnvironment",
  "ruff.configuration": "${workspaceFolder}/pyproject.toml",

  "ty.disableLanguageServices": false,
  "python.languageServer": "None",

  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.ruff": "explicit",
    "source.organizeImports.ruff": "explicit"
  },
  "[python]": {
    "editor.defaultFormatter": "charliermarsh.ruff",
    "editor.tabSize": 4,
    "editor.insertSpaces": true
  },
  "[toml]": {
    "editor.defaultFormatter": "tamasfe.even-better-toml"
  },
  "files.exclude": {
    "**/__pycache__": true,
    "**/.ruff_cache": true,
    "**/.pytest_cache": true
  },
  "search.exclude": {
    "**/.venv": true,
    "**/uv.lock": false
  }
}
```

Windows interpreter path is `${workspaceFolder}\\.venv\\Scripts\\python.exe`. I only bother with that on a Windows-heavy repo; otherwise I pick the interpreter once via **Python: Select Interpreter**.

`ruff.importStrategy: fromEnvironment` means "use the `ruff` inside `.venv`" — the one `uv add --dev ruff` pinned. The extension's bundled binary is the fallback, not the source of truth. Same idea as CI.

After `uv sync`, reload the window if the interpreter picker is empty. The `.venv` directory has to exist *before* Code will see it.

### Debug and tests

`.vscode/launch.json`:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Python: current file",
      "type": "debugpy",
      "request": "launch",
      "program": "${file}",
      "console": "integratedTerminal",
      "justMyCode": true
    },
    {
      "name": "Python: module",
      "type": "debugpy",
      "request": "launch",
      "module": "myproject",
      "cwd": "${workspaceFolder}",
      "justMyCode": true
    },
    {
      "name": "FastAPI",
      "type": "debugpy",
      "request": "launch",
      "module": "uvicorn",
      "args": ["myproject.api:app", "--reload", "--app-dir", "src"],
      "jinja": true,
      "justMyCode": true
    },
    {
      "name": "pytest",
      "type": "debugpy",
      "request": "launch",
      "module": "pytest",
      "args": ["tests"],
      "justMyCode": false
    }
  ]
}
```

Testing sidebar: enable pytest in the settings above, then **Python: Configure Tests**. It should pick up `tests/` and the `pythonpath = ["src"]` from `pyproject.toml`. If it doesn't, the interpreter is not `.venv`.

### Notebooks inside Code

Same rule as section 5: kernel = this project.

1. `uv add jupyterlab ipykernel` (or `uv add jupyter`)
2. Open the `.ipynb`
3. Kernel picker → **Python Environments…** → `.venv`
4. If it isn't listed: Command Palette → **Python: Select Interpreter** → `.venv`, then pick the kernel again

Don't install ipykernel into a user-level Python "to make Jupyter work". That is how you get pandas from 2023 and sklearn from last week in the same notebook.

### Devcontainers (optional)

If the project already has Docker from section 7, a thin `.devcontainer/devcontainer.json` keeps "Open in Container" on the same uv workflow:

```json
{
  "name": "python-uv",
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu",
  "features": {},
  "postCreateCommand": "curl -LsSf https://astral.sh/uv/install.sh | sh && . $HOME/.local/bin/env && uv sync",
  "customizations": {
    "vscode": {
      "extensions": [
        "ms-python.python",
        "charliermarsh.ruff",
        "astral-sh.ty",
        "ms-toolsai.jupyter"
      ],
      "settings": {
        "python.defaultInterpreterPath": "${workspaceFolder}/.venv/bin/python"
      }
    }
  }
}
```

Not required on a laptop where uv is already installed.

### Checklist when it feels "off"

1. Status bar Python path ends in `.venv/bin/python` (or `Scripts\\python.exe`).
2. Save a file → Ruff formats it. If Black or autopep8 runs, uninstall those extensions.
3. Hover on a function → ty (or Pylance, if you chose that split), not a random Jedi.
4. Testing sidebar runs `pytest` from the project, not `unittest`.
5. Integrated terminal: `which python` is the venv. If not, you opened a shell before `uv sync`.

Still later: Sublime Text 4 and Zed. They are not VS Code. Until then, this `.vscode/` folder is the whole editor story.

