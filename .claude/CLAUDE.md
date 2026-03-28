# OpenEvolve Fork — Claude Instructions

## Fork Overview

This is a **fork** of [OpenEvolve](https://github.com/algorithmicsuperintelligence/openevolve) (Apache-2.0), maintained to support the [Revolver](https://github.com/MateoTTR/revolver) MLOps CLI platform.

The fork adds targeted fixes and features discovered during Revolver demos that cannot wait for upstream acceptance.

| Property | Value |
|---|---|
| Upstream | `algorithmicsuperintelligence/openevolve` |
| Fork | `MateoTTR/openevolve` |
| Local path | `/home/mathi/openevolve/` |
| Revolver path | `/home/mathi/revolver/` |
| License | Apache-2.0 (inherited from upstream) |

## How to Use This File

At the start of each session:

1. Read this file (`.claude/CLAUDE.md`)
2. Also read upstream `CLAUDE.md` (root) for architecture reference
3. Read `docs/fork/backlog.md` — identify current in-progress or next task
4. Check the latest session report in `docs/fork/sessions/` for context
5. At end of session:
   - Create a session report in `docs/fork/sessions/YYYY-MM-DD-session-NNN.md`
   - Update `docs/fork/cost-tracker.md`
   - Update the backlog

---

## Documentation Index

| Document | Path | Purpose |
|---|---|---|
| Upstream CLAUDE.md | `CLAUDE.md` | OE architecture, commands, patterns (DO NOT MODIFY) |
| Fork CLAUDE.md | `.claude/CLAUDE.md` | This file — fork conventions and workflow |
| Fork backlog | `docs/fork/backlog.md` | Numbered task list with status |
| Sprint plans | `docs/fork/sprints/` | One file per sprint |
| Session reports | `docs/fork/sessions/` | One file per session |
| Cost tracker | `docs/fork/cost-tracker.md` | Cumulative session duration |

---

## Branching Strategy

```
upstream/main ──sync──> origin/main ──merge──> origin/develop
  (OE pure)              (mirror)            (our patches + docs)
```

| Branch | Purpose | Push to |
|---|---|---|
| `main` | Mirror of upstream — NEVER commit directly | `origin` |
| `develop` | Integration branch — our patches + internal docs | `origin` |
| `feature/*` | One branch per internal PR (base: `develop`) | `origin` |
| `upstream/*` | Clean branches for upstream PRs (base: `main`) | `origin` |

### Sync upstream workflow

```bash
git fetch upstream
git checkout main && git merge upstream/main && git push origin main
git checkout develop && git merge main && git push origin develop
```

### Upstream PR workflow

When contributing back to upstream:

```bash
# Branch from main (clean, no fork artifacts)
git checkout main
git checkout -b upstream/fix-description

# Cherry-pick ONLY code+test commits (NOT doc/sprint commits)
git cherry-pick <commit-hash>

# Push and create PR against upstream
git push origin upstream/fix-description
gh pr create --repo algorithmicsuperintelligence/openevolve \
  --title "Fix: description" --body "..."
```

**Key rule**: upstream PR branches MUST NOT contain any file from `docs/fork/` or `.claude/`. These files only exist on `develop`.

---

## Versioning

Fork versions use a `-rev.N` suffix tied to the upstream version:

```
upstream:  v0.2.27  →  v0.2.28
fork:      v0.2.27-rev.1  →  v0.2.27-rev.2  →  v0.2.28-rev.1
```

- Tags are placed on `develop` (where our patches live)
- `-rev.N` resets to `-rev.1` when syncing to a new upstream version
- Revolver installs via: `pip install "openevolve @ git+https://github.com/MateoTTR/openevolve@v0.2.27-rev.1"`

---

## Development Conventions

### Inherited from upstream (MUST follow for cherry-pick compatibility)

- **Formatter**: Black (line-length 100, target py310)
- **Import sort**: isort (black profile)
- **Type checking**: mypy (strict)
- **Test framework**: unittest for unit tests, pytest for integration tests
- **Python**: >= 3.10
- **Pre-commit hooks**: isort + black (run `pre-commit install` after clone)

### Injected from Revolver

- **Language**: All code, comments, variable names in **English**. Communication with user in **French**.
- **Commit messages**: English, imperative mood (`Add`, `Fix`, `Update`...)
- **PR workflow**: GitHub issue → dev agent (worktree) → draft PR → code review agent → fix → ready → merge
- **Session reports**: One per session in `docs/fork/sessions/`
- **Code review**: Mandatory via code-reviewer agent after every PR creation (triggered by PostToolUse hook)
- **Irreversible actions require explicit user approval**: never merge PRs, tag, or push to main without user confirmation

### Dev commands

```bash
# Install in dev mode
pip install -e ".[dev]"

# Run unit tests (no external deps needed)
OPENAI_API_KEY=test python -m unittest discover -s tests -p "test_*.py" -v

# Run specific test
OPENAI_API_KEY=test python -m unittest tests.test_openai_model_detection -v

# Format
python -m black openevolve tests
python -m isort openevolve tests

# Type check
python -m mypy openevolve

# Pre-commit (all checks)
pre-commit run --all-files
```

### Testing discipline

- Every fix/feature MUST include tests
- Unit tests: `tests/test_*.py` (unittest framework, matching upstream)
- Integration tests: `tests/integration/` (pytest, need optillm or mock server)
- Run `OPENAI_API_KEY=test python -m unittest discover tests` before every commit

---

## CI Pipeline

### Phase 1 (immediate) — Unit tests only

Workflow runs on push/PR to `develop`:
- Install `pip install -e ".[dev]"`
- `OPENAI_API_KEY=test python -m unittest discover tests`
- Black + isort check
- mypy

### Phase 2 (future) — Integration tests

Add optillm service or Ollama backend for integration tests.

---

## Key Source Files (Revolver-relevant)

| File | What | Why we care |
|---|---|---|
| `openevolve/config.py` | All `Config` dataclasses | `build_config()` in Revolver generates these |
| `openevolve/llm/openai.py` | `OPENAI_REASONING_MODEL_PREFIXES` | Missing Gemini prefixes (F2) |
| `openevolve/llm/ensemble.py` | `LLMEnsemble` weighted selection | Revolver ensemble feature (F3) depends on this |
| `openevolve/iteration.py` | `diff_based_evolution` logic | Full-rewrite mode (F4/F5) |
| `openevolve/process_parallel.py` | `ProcessPoolExecutor` wrapper | Post-evolution hang (B1) |
| `openevolve/controller.py` | Main orchestrator | Evolution loop, checkpoint save |
| `openevolve/database.py` | MAP-Elites + islands | Population inspection (B13) |

---

## Fork Roadmap

See `docs/fork/backlog.md` for the full task list. Summary:

| ID | Item | Effort | Upstream PR? |
|---|---|---|---|
| OE-F2 | Add Gemini prefixes to `OPENAI_REASONING_MODEL_PREFIXES` | S | Yes |
| OE-B1 | Fix `ProcessPoolExecutor.shutdown(wait=True)` hang | M | Yes |
| OE-F5 | Hybrid diff/rewrite mode (schedule by iteration) | L | Yes (after validation) |
| OE-LOG | Structured iteration logging (JSON) for Revolver parsing | M | Maybe |
| OE-CI | CI pipeline for fork (unit tests + formatting) | S | No (internal) |

---

## Relation to Revolver

- Revolver source: `/home/mathi/revolver/`
- Revolver installs OE as a dependency via git tag
- Changes in OE's `Config` dataclass may require updates to Revolver's `build_config()`
- Changes in OE's `database.py` may affect Revolver's `run population` command (B13)
- Always test OE changes with Revolver's test suite after installing the fork version

---

## Claude Agents

| Agent | Path | Trigger |
|---|---|---|
| code-reviewer | `.claude/agents/code-reviewer/agent.md` | Auto after `gh pr create` (PostToolUse hook) |
