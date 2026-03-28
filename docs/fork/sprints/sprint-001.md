# Sprint 001 — Quick Wins + CI Foundation

**Status**: Done
**Goal**: Establish CI pipeline for the fork, generalize reasoning model detection with explicit config flag, and validate the fork development workflow end-to-end.
**Upstream base**: v0.2.27

---

## Source

- Revolver demo feedback: `/home/mathi/revolver-demos/beamforming/demos/demo-v0.9.0/feedback.md` (F2)
- Upstream issue: Non-OpenAI reasoning/thinking models (Gemini, DeepSeek-R1, etc.) cannot be configured as reasoning models

---

## Design Decision — Reasoning Model Detection

### Problem

The current code hardcoded `OPENAI_REASONING_MODEL_PREFIXES` as a local variable inside `generate_with_context()`. Adding non-OpenAI prefixes (Gemini, Claude) is fragile: naming conventions vary, thinking is optional per model, future model names are unpredictable.

### Solution

Added `is_reasoning_model: Optional[bool] = None` on `LLMModelConfig`:
- `True` → treat as reasoning model
- `False` → treat as standard model
- `None` (default) → auto-detect via OpenAI prefix list (backward-compatible)

---

## Tasks

| # | Task | Layer | PR | Status | Upstream PR? |
|---|---|---|---|---|---|
| OE-01 | CI workflow: `.github/workflows/fork-ci.yml` — ubuntu-latest, unit tests + black + isort + mypy (all lint as continue-on-error due to upstream formatting) | infra | [PR #3](https://github.com/MateoTTR/openevolve/pull/3) | [x] | No |
| OE-02 | Extract `OPENAI_REASONING_MODEL_PREFIXES` to module-level constant, extract `is_reasoning_model()` function | code | [PR #4](https://github.com/MateoTTR/openevolve/pull/4) | [x] | Yes |
| OE-03 | Add `is_reasoning_model: Optional[bool] = None` to `LLMModelConfig`, wire through `OpenAILLM` | code | [PR #4](https://github.com/MateoTTR/openevolve/pull/4) | [x] | Yes |
| OE-04 | Unit tests: 3-state logic, backward compat, explicit override, updated stale `test_openai_model_detection.py` | test | [PR #4](https://github.com/MateoTTR/openevolve/pull/4) | [x] | Yes |

---

## PR Decomposition

| PR | Tasks | Base branch | Target | Upstream-able? |
|---|---|---|---|---|
| [PR #3](https://github.com/MateoTTR/openevolve/pull/3) | OE-01 | develop | develop | No (CI is fork-specific) |
| [PR #4](https://github.com/MateoTTR/openevolve/pull/4) | OE-02, OE-03, OE-04 | **main** | develop | Yes (`upstream/reasoning-model-config` branch ready) |

---

## Acceptance Criteria

- [x] CI runs on push to `develop` and passes (unit tests pass, lint as warnings)
- [x] `is_reasoning_model("o3-mini")` returns `True` (auto-detect, backward compat)
- [x] `is_reasoning_model("gemini-2.5-flash", config_flag=True)` returns `True` (explicit)
- [x] `is_reasoning_model("gemini-2.5-flash")` returns `False` (no auto-detect for non-OpenAI)
- [x] `is_reasoning_model("claude-sonnet-4-5-20250929")` returns `False`
- [x] Existing configs without `is_reasoning_model` work unchanged
- [x] All 384 tests pass (12 new + 372 existing, 0 failures)
- [x] Upstream PR branch (`upstream/reasoning-model-config`) created from `main`, clean (no fork docs)

---

## Release

After merge to `develop`:
- Tag: `v0.2.27-rev.1` (pending)
- Install in Revolver: `pip install "openevolve @ git+https://github.com/MateoTTR/openevolve@v0.2.27-rev.1"`
- Validate: run Revolver test suite with fork version

---

## Sprint Review

### Session
- Date: 2026-03-28
- Session: 001

### Tests
- 12 new tests passing (test_reasoning_model_detection.py + updated test_openai_model_detection.py)
- 384 / 384 total tests passing (no regression)

### PRs
- [PR #3](https://github.com/MateoTTR/openevolve/pull/3) — CI workflow — merged
- [PR #4](https://github.com/MateoTTR/openevolve/pull/4) — Reasoning model config — merged

### Upstream PR
- [x] `upstream/reasoning-model-config` branch created from `main`
- [ ] PR submitted to `algorithmicsuperintelligence/openevolve` (to do when ready)

### Review findings addressed
- CI: added `timeout-minutes: 15`, `continue-on-error: true` on lint steps (upstream formatting issues)
- Code: cleaned up redundant prefixes, replaced full model names with `gpt-oss-` prefix, rewrote stale test, added `shared_config` exclusion comment
