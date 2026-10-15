# Plan

## Done

- **2026-10-15** — Installed `gh` 2.102.0 and `uv` 0.13.1 into `tools/`, per-user,
  no admin. Verified against upstream SHA-256 digests and byte-compared the installed
  binaries against the verified archives. Both work.
- **2026-10-15** — Published the repo, public, at
  <https://github.com/emafelixova/mlr-agent-lab>. Initial commit `905a9ca`: `hello.py`,
  `.gitignore`, `tools/README.md`. Binaries excluded by design.
- **2026-10-15** — Working rules written: `AGENTS.md` (read `README.md` + `PLAN.md`
  first, uv only, check before claiming done), loaded from `CLAUDE.md`. Added `RUNS.md`
  for the request/verification log.

## Next step

Nothing is specified beyond the placeholder `hello.py`. The first real decision is what
MLR problem to work on and what data to use — **this needs input from the project
owner**, so pick that up before writing code.

When that's settled, the mechanical first steps are:

1. `.\tools\uv.exe init --bare --name mlr-agent-lab` for `pyproject.toml` only — plain
   `uv init` also scaffolds a `src/` package and runs `git init` on `master`.
2. Add the actual dependencies with `.\tools\uv.exe add ...`, commit `uv.lock`.
3. Replace `hello.py` with the real entry point and add a test that can fail.
4. Record each step here, with the check that confirmed it.

## Open questions

- What is the target MLR task, and what does "working" look like for it?
- Where does the data come from, and is it public enough for this repo?
- Should `tools/` be vendored at pinned versions (releases only, no drift) or is
  "latest at install time" fine? Currently pinned in `tools/README.md` by hash.
