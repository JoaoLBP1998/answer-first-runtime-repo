# Answer-First Project Package

This folder is a consolidated project package for the corrected `v2-answer-first` runtime as of `2026-04-21`.

## What is inside

- `portable-runtime/`
  - the portable bootstrap
  - the portable `v2-answer-first` prompt family
  - the portable answer-first runtime README
- `main-runtime/`
  - the main bootstrap
  - the snapshot mirror of the same `v2-answer-first` prompt family
  - the main answer-first runtime README
- `validated-simulation/turn-005/`
  - the saved file-backed simulation that validated:
    - classroom transcript continuation
    - fresh per-turn controller packet creation
    - canon-style verifier traces
    - guided live tutor wording with near-correct phrasing
- `patch-bundle/`
  - the corrected answer-first diff
  - the manifest
  - the rewrite bundle zip

## Recommended source of truth

For actual runtime use, prefer:
- `portable-runtime/PORTABLE NEW CHAT EXECUTION PROMPT - ANSWER FIRST.txt`
- `portable-runtime/prompts/v2-answer-first/`
- `portable-runtime/runtime/answer-first-run/README.md`

Use `validated-simulation/turn-005/` as the reference example of expected saved packet behavior.

## Important design choices preserved

- controller-side search remains enabled
- verifier search remains adversarial and live
- every continuation turn still writes a fresh `01-controller-answer-packet.json`
- canon-style audit strictness is kept for:
  - `checked_urls`
  - `supporting_sources`
  - `mismatch_evidence`
- plain normal-language operator text is not treated as a student classroom line

## Notes

- This package is scoped to `v2-answer-first`.
- It does not replace `v2-canon` or `v2-answer-first-plus`.
- The older generic runtime redesign patch is intentionally not included as the preferred artifact; the corrected answer-first patch bundle here supersedes it for this variant.
