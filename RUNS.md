# Runs

Log of what was asked, which model did it, and how the result was checked. One entry
per request, newest last. The check column is the point of this file — an entry without
a check is not finished.

## Environment

| | |
| --- | --- |
| Date | 2026-10-15 |
| Host | Windows 11 (10.0.26200) x64, Git Bash / MINGW64 |
| Model | `agentic` — the `ANTHROPIC_MODEL` value in `~/.claude/settings.json`, sent to proxy `llm.ai.e-infra.cz`. Which upstream model that resolves to is decided by the proxy and is not visible from inside the session, so treat `agentic` as a label, not an identification. |
| Not confirmed | `/status` is a built-in CLI command and cannot be run from here, so this line comes from config, not from `/status`. Confirm the actual model there and correct this row. |
| Tools | `tools/gh.exe` 2.102.0, `tools/uv.exe` 0.13.1 |
| GitHub | account `emafelixova`, scopes `repo`, `workflow`, `gist`, `read:org` |

---

## 1. Install gh and uv without admin, into the user folder

**Asked:** install GitHub CLI and uv into my user folder without administrator rights,
and check that both work.

**Did:** latest releases resolved via redirect (GitHub API was rate-limited) — gh
2.102.0, uv 0.13.1. Downloaded the Windows amd64 archives, verified each against the
upstream published digest, and placed `gh.exe`, `uv.exe`, `uvw.exe`, `uvx.exe` in
`~/.local/bin` (already on both the bash and Windows user PATH).

**Checked:**

- `sha256sum` of each download matched `gh_2.102.0_checksums.txt` and
  `uv-x86_64-pc-windows-msvc.zip.sha256` from the release assets.
- `gh --version`, `uv --version`, `uvx --version` all reported the expected versions.
- Not yet done when superseded: no functional test of gh beyond `auth status`.

**Note:** the first attempt at a checksum line was mis-parsed and no gh digest existed
at the URL I guessed; corrected by fetching the real `checksums.txt`.

## 2. Move the install into the project's `tools/` folder

**Asked:** put the executables in `C:\dev\mlr-agent-lab\tools\` instead, using the
already-verified downloads; don't require admin; don't touch other project files.

**Did:** copied the four staged binaries to `tools/`, and deleted the copies from
`~/.local/bin`.

**Checked:**

- Installed binaries byte-compared against the contents of the checksum-verified
  archives — identical (`diff` of the digest lists returned empty).
- `.\tools\gh.exe --version` and `.\tools\uv.exe --version` run from the project
  directory, in PowerShell as well as Git Bash.
- Functional uv test in a temp dir, not the project: `uv venv --python 3.12`, `uv pip
  install anyio`, then a real `import anyio` from the venv's interpreter; plus
  `uv run --no-project --with httpx` for an ephemeral env. Both imported successfully.
- `~/.local/bin` confirmed back to its original contents (`claude.exe` only).
- Temp files removed afterwards.

**Note:** my first uv test asserted `anyio.__version__`, which anyio 4.x does not
define — that was a bad assertion, not a broken install. Re-checked with a plain import.

## 3. Create a public repo with gh, showing what would be uploaded

**Asked:** create a public GitHub repo called mlr-agent-lab from this folder, and show
what will be uploaded before pushing.

**Did:** `git init -b main`, wrote `.gitignore` (`tools/*`, keeping `tools/README.md`)
and `tools/README.md` with versions, SHA-256 digests, and a verified reinstall script.
Paused for a decision on the 72 MB of binaries — excluded them. Then one commit and
`gh repo create mlr-agent-lab --public --source=. --remote=origin --push`.

**Checked before pushing:**

- `gh repo view` confirmed the name was free; listed existing repos.
- Full `git diff --cached` shown — 3 files, 2,968 bytes.
- `git check-ignore -v` matched all four binaries; no untracked, non-ignored files left.
- Swept the tree for secrets, `.env`, and stray config — none. The gh token lives in
  the Windows credential manager, outside the repo.

**Checked after pushing:**

- Local HEAD and `git ls-remote origin main` both `905a9ca`.
- `gh api .../git/trees/main?recursive=1` — upstream holds exactly `.gitignore`,
  `hello.py`, `tools/README.md`; `tools/` has only `README.md`.
- Unauthenticated `curl` to `raw.githubusercontent.com` returned 200 with the file
  contents, which is what actually proves public visibility.

**Note:** I wrote a `.gitignore` line for `uv.lock` and removed it before committing —
uv.lock should be committed, not ignored. Also declined a denied `env | grep` for
credentials; it would have leaked an API key into this log, and reading
`settings.json` answered the question instead.

## 4. Working rules and project docs

**Asked:** write `AGENTS.md` with the working rules (read README + PLAN first, use uv
and no global pip, run checks before saying done), make `CLAUDE.md` load it, and create
`README.md`, `PLAN.md`, `RUNS.md`.

**Did:** the five files, plus a check of what uv actually accepts here so the rules
match reality.

**Checked:**

- `./tools/uv.exe run hello.py` printed `hello` with no `pyproject.toml` present, and
  created no `pyproject.toml`, `uv.lock`, or venv — so script-mode `uv run` is safe to
  document as-is.
- `uv add anyio` in a bare temp directory failed with `No pyproject.toml found`, so
  `AGENTS.md` documents `uv init` as the prerequisite for `uv add` rather than implying
  it works from a bare checkout.
- `uv --help` has no `exec` subcommand — the first draft of `AGENTS.md` told the reader to
  run `uv exec ruff check .`, which would have failed. Corrected to `uvx` /
  `uv run --with`, both checked to exist.
- End-to-end in a temp dir mirroring this layout: `uv init --bare --name mlr-agent-lab`
  wrote only `pyproject.toml`; `uv add anyio` then worked and produced `uv.lock` and
  `.venv`; `uv run hello.py` printed `hello` in project mode.
- Plain `uv init` (no `--bare`) additionally created `.python-version` and
  `src/mlr_agent_lab/__init__.py` with a sample `main()`, and ran `git init` on branch
  `master`. It did **not** overwrite an existing `README.md` or `.gitignore`. `--bare` is
  documented as the default here because of the extra scaffolding.
- No pre-existing `AGENTS.md`, `CLAUDE.md`, `README.md`, `PLAN.md`, or `RUNS.md` was
  overwritten — all five were absent beforehand.
- Checked the `@AGENTS.md` import against the Claude Code memory docs rather than
  assuming it: `@path` imports are real, relative paths resolve relative to the file
  containing them, and max depth is four hops. Per the docs' precedence table, a repo
  with **both** `AGENTS.md` and `CLAUDE.md` loads `CLAUDE.md` only — so the explicit
  import line is load-bearing, not decorative. Dropping it would silently stop the rules
  from loading.
- Confirmed the one `@` in `AGENTS.md` is inside a backticked code span, where import
  parsing does not apply, so it won't try to import anything.

**Not yet checked:** that a new session actually picks up the rules through the
`CLAUDE.md` import — that needs a fresh session to observe, so it is recorded here as
unverified rather than claimed. Not committed at the time of writing.
