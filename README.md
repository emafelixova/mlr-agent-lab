# mlr-agent-lab

Experiments with MLR (machine-learning-based routing) agents. Working notes, code,
and the record of what was tried and how it was checked.

## Layout

| File | What it's for |
| --- | --- |
| [AGENTS.md](AGENTS.md) | Working rules for anyone (or anything) working in here |
| [PLAN.md](PLAN.md) | Progress so far and the next step |
| [RUNS.md](RUNS.md) | Log of requests and how each was verified |
| [tools/](tools/README.md) | Locally installed `gh` and `uv` binaries (not committed) |
| `hello.py` | Placeholder entry point — replace once there is real code |

Start with [AGENTS.md](AGENTS.md): it says to read this file and `PLAN.md` before
touching anything.

## Setup

Clone, then install the tooling into `tools/` (the binaries are gitignored, so a
fresh clone has only the README there). From the repo root in PowerShell — no
administrator rights needed:

```powershell
Set-Location <path-to-this-repo>
# run the install block in tools/README.md
```

To work on the Python side, give the project an environment:

```powershell
.\tools\uv.exe init --bare --name mlr-agent-lab   # only if pyproject.toml is missing
.\tools\uv.exe add <package>                      # creates uv.lock, .venv
```

## Running

```powershell
.\tools\uv.exe run hello.py    # prints: hello
.\tools\uvx.exe ruff check .   # lint, no install step
.\tools\uv.exe run pytest      # tests, once there are any
```

`tools/` is deliberately not on `PATH`; use the relative paths above. Python
dependencies are managed by uv in `pyproject.toml` — never `pip install` globally.

## Current state

`hello.py` is a one-line placeholder. The repo exists to hold the real work, the
tooling conventions around it, and the log in `RUNS.md`. See `PLAN.md` for what is
next.
