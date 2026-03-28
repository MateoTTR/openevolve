# OpenEvolve Fork — Task Backlog

## Sprint 001 — Quick Wins + CI Foundation

| # | Task | Status | Sprint | Upstream PR? |
|---|---|---|---|---|
| OE-01 | Set up CI workflow for fork: unit tests + black + isort + mypy on push/PR to develop | [ ] | S1 | No |
| OE-02 | Add Gemini model prefixes to `OPENAI_REASONING_MODEL_PREFIXES` in `llm/openai.py` (F2) | [ ] | S1 | Yes |
| OE-03 | Add configurable `reasoning_model_prefixes` to `LLMModelConfig` so providers can declare prefixes via config (future-proof F2) | [ ] | S1 | Yes |
| OE-04 | Unit tests for Gemini prefix detection (reasoning model detection, `max_completion_tokens` vs `max_tokens`) | [ ] | S1 | Yes |

## Sprint 002 — Post-Evolution Hang Fix (B1)

| # | Task | Status | Sprint | Upstream PR? |
|---|---|---|---|---|
| OE-05 | Investigate `ProcessPoolExecutor.shutdown(wait=True)` hang in `controller.py` / `process_parallel.py` | [ ] | S2 | Yes |
| OE-06 | Implement fix: timeout on shutdown, fallback to `terminate()` if workers don't exit | [ ] | S2 | Yes |
| OE-07 | Add structured logging for iteration completion (JSON-parseable progress lines) | [ ] | S2 | Maybe |
| OE-08 | Unit tests for graceful shutdown + timeout behavior | [ ] | S2 | Yes |

## Sprint 003 — Hybrid Diff/Rewrite Mode (F5)

| # | Task | Status | Sprint | Upstream PR? |
|---|---|---|---|---|
| OE-09 | Design: hybrid mode config schema (`diff_schedule: [{iterations: 0-50, mode: "full_rewrite"}, {iterations: 51+, mode: "diff"}]`) | [ ] | S3 | Yes |
| OE-10 | Implement `diff_schedule` in `Config` dataclass (backward-compat with `diff_based_evolution: bool`) | [ ] | S3 | Yes |
| OE-11 | Implement schedule resolution in `iteration.py` and `process_parallel.py` | [ ] | S3 | Yes |
| OE-12 | Update prompt templates: adapt system prompt based on current mode (diff vs rewrite) | [ ] | S3 | Yes |
| OE-13 | Unit tests for hybrid mode: schedule parsing, mode switching, backward compat | [ ] | S3 | Yes |
| OE-14 | Integration test: run with hybrid schedule, verify mode switch at iteration boundary | [ ] | S3 | Yes |

## Unscheduled

| # | Task | Status | Sprint | Upstream PR? |
|---|---|---|---|---|
| OE-15 | Structured iteration logging: JSON log lines with iteration number, valid_diffs, score, model used | [ ] | TBD | Maybe |
| OE-16 | Expose MAP-Elites database as JSON API for Revolver `run population` parsing | [ ] | TBD | Maybe |
| OE-17 | `valid_iterations` counter accessible after evolution completes (for Revolver B9 status logic) | [ ] | TBD | Maybe |
