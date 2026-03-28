# Sprint 001 — Quick Wins + CI Foundation

**Status**: Planning
**Goal**: Establish CI pipeline for the fork, fix Gemini reasoning model detection (F2), and validate the fork development workflow end-to-end.
**Upstream base**: v0.2.27

---

## Source

- Revolver demo feedback: `/home/mathi/revolver-demos/beamforming/demos/demo-v0.9.0/feedback.md` (F2)
- Upstream issue: Gemini thinking models not detected as reasoning models in `llm/openai.py`

---

## Tasks

| # | Task | Layer | PR | Status | Upstream PR? |
|---|---|---|---|---|---|
| OE-01 | CI workflow: `.github/workflows/fork-ci.yml` — triggers on push/PR to `develop`, runs unit tests + black + isort + mypy. Uses self-hosted runner. | infra | PR-A | [ ] | No |
| OE-02 | Add Gemini prefixes (`"gemini-2.5-"`, `"gemini-3-"`) to `OPENAI_REASONING_MODEL_PREFIXES` tuple in `openevolve/llm/openai.py`. | code | PR-B | [ ] | Yes |
| OE-03 | Add optional `reasoning_model_prefixes: list[str] | None` field to `LLMModelConfig` in `config.py`. When set, override the global prefixes for that specific model. This future-proofs prefix detection for arbitrary providers. | code | PR-B | [ ] | Yes |
| OE-04 | Unit tests: `tests/test_gemini_reasoning_detection.py` — test that Gemini 2.5 Pro/Flash are detected as reasoning models, that `max_completion_tokens` is used instead of `max_tokens`, that per-model prefix override works. | test | PR-B | [ ] | Yes |

---

## PR Decomposition

| PR | Tasks | Base branch | Target | Upstream-able? |
|---|---|---|---|---|
| PR-A | OE-01 | develop | develop | No (CI is fork-specific) |
| PR-B | OE-02, OE-03, OE-04 | develop | develop | Yes (cherry-pick OE-02 + OE-03 + OE-04 to `upstream/gemini-reasoning-prefixes` from `main`) |

PR-A and PR-B are fully independent — can be developed in parallel.

---

## Acceptance Criteria

- [ ] CI runs on push to `develop` and passes (unit tests + formatting + mypy)
- [ ] `is_reasoning_model("gemini-2.5-pro")` returns `True`
- [ ] `is_reasoning_model("gemini-2.5-flash")` returns `True`
- [ ] `is_reasoning_model("gemini-2.5-flash-lite")` returns `False` (not a thinking model)
- [ ] Per-model `reasoning_model_prefixes` override works in config
- [ ] All existing tests still pass
- [ ] Upstream PR branch (`upstream/gemini-reasoning-prefixes`) created from `main`, clean (no fork docs)

---

## Release

After merge to `develop`:
- Tag: `v0.2.27-rev.1`
- Install in Revolver: `pip install "openevolve @ git+https://github.com/MateoTTR/openevolve@v0.2.27-rev.1"`
- Validate: run Revolver test suite with fork version

---

## Sprint Review

_Filled at sprint end._

### Tests
- X / N new tests passing
- X / N existing tests passing (no regression)

### Upstream PR
- [ ] `upstream/gemini-reasoning-prefixes` branch created
- [ ] PR submitted to `algorithmicsuperintelligence/openevolve`
