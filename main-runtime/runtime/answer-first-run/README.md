# Answer-First Runtime Packet Contract

Use this folder for file-backed runs of the answer-first runtime variant.

This variant is separate from the active `v2-canon` runtime.
Do not mix its turn folders with `runtime/active-run/`.

Per answer-first run:
- `runtime/answer-first-run/turn-XXX/`

Run-selection rule:
- `start tutoring`, a new screenshot, a new problem statement, or an explicit restart means create a brand-new turn.
- Same-problem classroom transcript continuation creates a fresh `turn-XXX/` but reuses the latest approved or incomplete answer-first turn only as same-problem controller context.
- Resume only when the user explicitly says `resume` or names an exact `turn-XXX/`.
- Do not auto-resume an older turn just because it exists.
- `archive/` is evidence-only and not part of live resumable state.

Classroom continuation rule:
- During an open tutoring session, copied classroom transcript text is continuation input by default.
- Expected format is repeated speaker / timestamp / message blocks from the classroom transcript.
- Preserve the exact copied transcript in `00-turn-input.json`.
- Parse the latest student-authored line and latest tutor-authored line for the current turn.
- Treat plain normal-language operator text as control input, not as a student classroom line.
- If the operator ends or redirects the session in normal language, stop continuation handling and follow the control instruction.

Minimum files per turn:
- `00-turn-input.json`
- `00a-current-packet-manifest.json`
- `00b-repair-ticket.json` when a repair loop exists
- `active-agent-registry.json`
- `01-controller-answer-packet.json`
- `02-verifier1-baseline-verification.json`
- `03-verified-answer-packet.json`
- `04-live-tutor-output.json`
- `05-verifier1-turn-verification.json`
- `06-final-cleanup-report.json`
- `stage-state.json`

Rules:
- Save the exact packet before the next handoff.
- Pass the exact saved packet or exact pasted packet content to the next agent.
- Do not rely on implicit thread state as the source of truth.
- The registry filename is exactly `active-agent-registry.json`; prefixed variants such as `00c-active-agent-registry.json` are invalid.
- The controller owns OCR binding, answer construction, and web search in this variant.
- The controller owns classroom transcript parsing and next-reply construction for same-problem continuation in this variant.
- `verifier1` verifies substantive answer claims with two independent live sources by default.
- `verifier1` must directly open controller-cited URLs before accepting the related claims.
- `verifier1` should inherit the stricter canon-style audit requirements for checked URLs, supporting source traces, and mismatch evidence when current-file defects are alleged.
- If a required packet is missing or stale, block the run and report the exact missing stage or file.
- On successful completion, `00a-current-packet-manifest.json` should end at the approved boundary and `stage-state.json` must set both `current_stage` and `last_completed_stage` to `cleanup_complete`.
- On successful completion, the operator-facing final status should echo these exact saved fields from `04-live-tutor-output.json`:
  - `message_to_tutor_now`
  - `next_expected_student_reply`
  - `if_on_track`
  - `if_confused`
  - `silence_breaker`
  - `current_step`
- `04-live-tutor-output.json` remains the source of truth; the inline echo is an operator convenience layer only.
- Do not patch prompts during an active run. Stop the run, patch, then start the next run.
- For a brand-new answer-first run, do not open or reuse `runtime/active-run/*`.
- For a brand-new answer-first run, do not open or reuse `prompts/v2-canon/*`.
- Start a fresh run after this controller-search change; older answer-first turns may use incompatible packet names.
- `archive/` is the only legal location for stale pre-cleanup turns from older packet shapes.
