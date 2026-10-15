# Working rules

Read these first, in this order, before changing anything:

1. [README.md](README.md) — what this project is for, how to run it
2. [PLAN.md](PLAN.md) — where things stand, what the next step is

Keep both honest as you work. If you change the goal or the plan, update the file in
the same commit. `RUNS.md` logs what was asked and how it was checked.

## Python: use uv

All Python work goes through the local `uv` in `tools/`. It is not on `PATH`, so use
the relative path from the repo root:

```powershell
.\tools\uv.exe run hello.py        # run a script
.\tools\uv.exe add requests        # add a dependency
.\tools\uv.exe run pytest          # run a tool inside the project env
```

From Git Bash, `./tools/uv.exe run hello.py`.

- **Never `pip install` globally.** No `pip install`, `python -m pip`, or
  `pip3 install` against the system interpreter, and no `pip install --user`.
  Dependencies belong in `pyproject.toml` via `uv add`.
- Install packages into the project environment, never into a system or
  user-level site-packages.
- One-off tools run without adding them to the project:
  `.\tools\uvx.exe ruff check .`, or `.\tools\uv.exe run --with ruff ruff check .`.
  There is no `uv exec` subcommand — don't reach for it.
- Commit `uv.lock`. It is deliberately not gitignored — that is what makes builds
  reproducible.
- `uv add` needs a `pyproject.toml` (it errors with `No pyproject.toml found`
  otherwise). Create one with `.\tools\uv.exe init --bare --name mlr-agent-lab` —
  `--bare` writes only `pyproject.toml`. Plain `uv init` also creates `.python-version`,
  a `src/mlr_agent_lab/` package with a sample `main()`, and runs `git init` on branch
  `master`; it leaves existing `README.md` and `.gitignore` alone, but that extra
  scaffolding is not wanted here.
- `uv run` on a bare script works without a `pyproject.toml`, so nothing is needed just
  to run `hello.py`.

See [tools/README.md](tools/README.md) for where the binaries come from and how to
reinstall them.

## Run checks before saying it is done

Nothing counts as finished until it has been checked. State what you ran and what
it printed; if a check failed, say so and show the output.

- It has to actually run. Execute it — do not reason about whether it would work.
- After an edit, confirm the failure you set out to fix is gone, and that nothing
  that worked before broke.
- Verify the observable result, not just a clean exit code. `--version` proves an
  install; it does not prove the tool works. For `uv`, that means creating a venv
  and importing something in it.
- After pushing or publishing, read back the server's state. Command output is not
  confirmation.
- Check the thing itself, not its wrapper: compare bytes or content rather than
  trusting a filename, and confirm upstream checksums before running downloaded
  binaries.
- A test that passes without ever touching the code under test proves nothing.
  Make sure your check can fail.
- Never claim a check passed that you did not run. If you skipped one, name it.

If a claim survives this, state it plainly — no hedging.
