# Runtime Playbook - Answer-First Variant

This is the operating sequence for the answer-first runtime variant.

## Trigger rule

- If the user sends only a screenshot, treat it as `start tutoring`.
- If the user sends a text-only academic question, treat it as a new answer-first run.
- If the user sends copied classroom transcript blocks for the same active problem, treat it as same-problem classroom continuation.
- If the user sends plain normal-language operator text, treat it as controller input, not a student classroom line.
- Keep child packets production-clean.
- Do not tell child agents that a run is a simulation unless simulation behavior itself is under audit.

## File-backed runtime

- Every run must use:
  - `runtime/answer-first-run/turn-XXX/`
- Save the exact packet before each handoff step.
- `verifier1` should receive the exact current packet, not inferred thread state.
- Required verifier stages must be executed by spawned child agents that are visible and inspectable in the UI.
- Controller-authored or synthetic verifier packets are invalid.
- Minimum turn files:
  - `00-turn-input.json`
  - `00a-current-packet-manifest.json`
  - `00b-repair-ticket.json` when a repair loop exists
  - `01-controller-answer-packet.json`
  - `02-verifier1-baseline-verification.json`
  - `03-verified-answer-packet.json`
  - `04-live-tutor-output.json`
  - `05-verifier1-turn-verification.json`
  - `06-final-cleanup-report.json`
  - `stage-state.json`
- If a repair loop occurs, save repaired verifier packets as new exact files.
- If a required packet file is missing or stale, stop and report the exact missing stage or file.
- For classroom transcript continuation, still write a fresh `01-controller-answer-packet.json` for the current turn.
- Prior verified answer packets may be reused only as controller context and verifier dossier context, not as a substitute for the current turn packet.

## Execution order

1. Controller answer packet
   - Prompt: `01 CONTROLLER - persistent orchestration v2.txt`
   - Input: screenshot only, screenshot plus transcript, or text-only question
   - Output: exact visible text, scope binding, session mode, latest student line / latest tutor line when applicable, full internal answer packet, controller-cited sources, one visible tutor-send cycle
   - Rule: controller may browse and web search directly
   - Rule: no pedagogical baseline generation
   - Rule: copied classroom transcript continuation is parsed into the current-turn controller answer packet
   - Rule: live tutor wording should stay guided, brief, and directly sendable; congratulate briefly when right and use `Quite close.` or `You're almost there.` when nearly right

2. Verifier1 baseline pass
   - Prompt: `03 VERIFIER1 - adversarial falsifier v2.txt`
   - Mode: `BASELINE_ANSWER_VERIFICATION`
  - Input: saved controller answer packet
  - Output: answer-focused proof bundle
  - Rule: run this stage through a spawned, UI-visible child agent and record its id in `active-agent-registry.json` before waiting
  - Rule: falsify controller claims and open controller URLs directly
   - Rule: keep the search agent behavior intact; search remains live and adversarial
   - Rule: approved support must expose `checked_urls` and `supporting_sources`
   - Rule: a current-file defect verdict must include `mismatch_evidence`

3. Controller-built live tutor output
   - Source: `visible_tutor_cycle` inside the saved controller answer packet
   - Saved file: `04-live-tutor-output.json`
   - Rule: for classroom continuation, this is the next directly sendable tutor reply to the latest student line
   - Rule: once the live tutor output is approved, echo these exact saved fields inline in the operator-facing status:
     - `message_to_tutor_now`
     - `next_expected_student_reply`
     - `if_on_track`
     - `if_confused`
     - `silence_breaker`
     - `current_step`
   - Rule: the inline echo is copied from `04-live-tutor-output.json`; the file remains the source of truth

4. Verifier1 turn pass
  - Prompt: `03 VERIFIER1 - adversarial falsifier v2.txt`
  - Mode: `TURN_CONSISTENCY_CHECK`
  - Input: `LIVE_TUTOR_OUTPUT` + `VERIFIED_ANSWER_PACKET`
  - Output: final approval or rejection of the one visible send cycle
  - Rule: run this stage through a spawned, UI-visible child agent and record its id in `active-agent-registry.json` before waiting
  - Rule: do at least one fresh adversarial live retrieval during turn verification even when the live-send content appears dossier-covered
  - Rule: confirm the reply stays tied to the parsed latest student line and does not jump to a full answer unless the verified answer packet supports a close step

5. Cleanup
   - Close every tracked child agent
   - Write `06-final-cleanup-report.json`

## Source policy

- Two independent directly opened live sources are the default for substantive answer claims.
- Packet evidence is context, not enough by itself for substantive answer claim approval.
- Controller-cited URLs are leads until `verifier1` opens them directly.
- A third source is allowed only for conflict resolution or failed independence.
- Fresh browsing is delta-aware.
- Canon-style audit strictness stays on:
  - `checked_urls` should enumerate what was actually checked
  - `supporting_sources` should be non-empty for approved or blocked substantive claims
  - `mismatch_evidence` is required for current-file defect claims

## Repair rule

- Do one repair loop only after a failed or incomplete verification pass.
- Repair exact claim ids and exact field paths.
- Do not open a repair loop for missing pedagogical scaffolding in this variant.

## Cleanup

- Every run ends with explicit cleanup.
- Verify close by exact agent id.
- Report any lingering agent ids exactly.
- For approved runs, include the inline echo of the saved live tutor fields in the final operator-facing status.
- For approved runs, also include the exact baseline and turn verifier child ids in the final operator-facing status.
