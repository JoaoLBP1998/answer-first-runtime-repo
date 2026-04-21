# Runtime Playbook - Answer-First Variant - Minimal File Surface

This is the operating sequence for the answer-first runtime variant.

## Trigger rule

- If the user sends only a screenshot, treat it as a tutoring trigger.
- If the user sends a text-only academic question, treat it as a tutoring trigger.
- If the user sends copied classroom transcript blocks for the same active problem, treat it as `SAME_PROBLEM_CONTINUATION`.
- If the user sends plain normal-language operator text, treat it as `CONTROL_MESSAGE`.
- Keep child packets production-clean.
- Do not tell child agents that a run is a simulation unless simulation behavior itself is under audit.

## Route selection

Every incoming turn must be routed as exactly one of:
- `NEW_PROBLEM`
- `SAME_PROBLEM_CONTINUATION`
- `CONTROL_MESSAGE`
- `REPAIR`

## File-backed runtime

Every run must use:
- `runtime/answer-first-run/turn-XXX/`

Save the exact packet before each handoff step.
`verifier1` should receive the exact current packet, not inferred thread state.
Required verifier stages must be executed by one persistent spawned child agent that is visible and inspectable in the UI.
Controller-authored or synthetic verifier packets are invalid.

Current-turn files:
- always: `00-turn-input.json`, `turn-state.json`
- when tutoring work is performed: `01-controller-answer-packet.json`
- when a verifier child is spawned: `active-agent-registry.json`
- when baseline verification runs: `02-verifier1-baseline-verification.json`
- when a verified answer boundary exists for the current turn: `03-verified-answer-packet.json`
- when a live reply exists: `04-live-tutor-output.json`
- when a live reply is checked: `05-verifier1-turn-verification.json`

Deprecated files that must not exist:
- `00a-current-packet-manifest.json`
- `00b-repair-ticket.json`
- `stage-state.json`
- `06-final-cleanup-report.json`
- `ACTIVE_RUN.json`

## Execution order for `NEW_PROBLEM`

1. Controller answer packet
   - prompt: `01 CONTROLLER - persistent orchestration v2.txt`
   - input: screenshot, screenshot plus transcript, or text-only question
   - output: exact visible text, scope binding, session mode, claim delta status, baseline decision, explicit search decision, full answer packet, controller-cited sources, one visible tutor-send cycle
   - rule: controller-side search follows `search_required`, `search_scope`, `search_skip_reason`, and `search_decision_basis`
   - rule: no pedagogical baseline generation

2. Verifier1 baseline pass when required
   - prompt: `03 VERIFIER1 - adversarial falsifier v2.txt`
   - mode: `BASELINE_ANSWER_VERIFICATION`
   - input: saved controller answer packet
   - output: answer-focused proof bundle
   - rule: run this stage through the run's single persistent, UI-visible `verifier1` child and record its id in `active-agent-registry.json` before the first wait
   - rule: falsify controller claims and open controller URLs directly
   - rule: approved support must expose `checked_urls` and `supporting_sources`
   - rule: a current-file defect verdict must include `mismatch_evidence`

3. Controller-built current-turn verified answer packet
   - saved file: `03-verified-answer-packet.json`
   - rule: controller builds this from the current-turn baseline verifier packet

4. Controller-built live tutor output when needed
   - source: `visible_tutor_cycle` inside the saved controller answer packet
   - saved file: `04-live-tutor-output.json`
   - rule: keep the reply guided, brief, and directly sendable

5. Verifier1 turn pass when the live reply is checked
   - prompt: `03 VERIFIER1 - adversarial falsifier v2.txt`
   - mode: `TURN_CONSISTENCY_CHECK`
   - input: current-turn `LIVE_TUTOR_OUTPUT` plus current-turn `VERIFIED_ANSWER_PACKET`
   - output: final approval or rejection of the one visible send cycle
   - rule: reuse the same persistent, UI-visible `verifier1` child here; do not spawn a second verifier child just because the mode changes
   - rule: do at least one fresh adversarial live retrieval during turn verification even when the live-send content appears dossier-covered

6. Cleanup
   - close every tracked child agent
   - embed cleanup accounting in `turn-state.json`

## Execution order for `SAME_PROBLEM_CONTINUATION`

1. Read only the exact prior files needed:
   - prior `turn-state.json`
   - prior `03-verified-answer-packet.json`
   - prior `05-verifier1-turn-verification.json` when it exists
   - prior `active-agent-registry.json` only when prior `turn-state.json` says child provenance or cleanup state must be consulted

2. Controller answer packet
   - save a fresh current-turn `01-controller-answer-packet.json`
   - parse latest student line and latest tutor line
   - compute `claim_delta_status`
   - compute `baseline_verification_required`
   - compute `search_required`, `search_scope`, `search_skip_reason`, and `search_decision_basis`

3. Baseline decision
   - if `baseline_verification_required = true`, run the normal baseline verifier stage and save `02-verifier1-baseline-verification.json`
   - if `baseline_verification_required = false`, do not run a current-turn baseline verifier stage and do not create `02-verifier1-baseline-verification.json`

4. Build fresh current-turn `03-verified-answer-packet.json` when the turn reaches live output or live verification
   - if baseline ran, build from the current-turn baseline pass
   - if baseline was legally skipped, build with `reuse_mode = REUSED_PRIOR_APPROVED_BOUNDARY` and record exact prior packet paths and hashes in `reuse_basis`

5. Build fresh current-turn `04-live-tutor-output.json` when a live reply is produced
   - this is the next directly sendable tutor reply to the latest student line

6. Run fresh current-turn `TURN_CONSISTENCY_CHECK` when the live reply is checked
   - save `05-verifier1-turn-verification.json`
   - require at least one fresh adversarial live retrieval
   - confirm the live reply stays tied to the parsed latest student line and does not jump to a full answer unless the verified answer packet supports a close step

7. Cleanup
   - close every tracked child agent
   - embed cleanup accounting in `turn-state.json`

## Execution order for `CONTROL_MESSAGE`

1. Save exact turn input first
2. Route as `CONTROL_MESSAGE`
3. Do not misclassify the text as a student-authored classroom line
4. Do not present a tutor reply as approved unless the operator explicitly requested one
5. If the operator ends the session, clean up tracked agents and mark control handled inside `turn-state.json`
6. Return controller JSON only

## Source policy

- Two independent directly opened live sources are the default for substantive answer claims in baseline verification.
- Packet evidence is context, not enough by itself for new substantive answer claim approval.
- Controller-cited URLs are leads until `verifier1` opens them directly.
- A third source is allowed only for conflict resolution or failed independence.
- Fresh browsing is delta-aware.
- Turn verification must still perform at least one fresh adversarial live retrieval even when the current verified dossier appears sufficient.
- `independence_audit` must explain whether the accepted source set was independent or why a third source was required.

## Wrong-role guard

- If a supposed verifier packet contains controller-answer fields, treat it as `WRONG_ROLE_OUTPUT`.
- Wrong-role output does not complete the stage.
- Close the child, record the failure, and rerun the same verifier stage with the exact verifier prompt and exact mode.

## Repair rule

- Do one repair loop only after a failed or incomplete verifier pass.
- Repair exact claim ids and exact field paths.
- Do not open a repair loop for missing pedagogical scaffolding in this variant.
- After the second failure, fail closed with `UNVERIFIED - DO NOT USE`.
- Record repair state inside `turn-state.json`.

## Cleanup

- Every run ends with explicit cleanup.
- Verify close by exact agent id.
- Report any lingering agent ids exactly.
- For approved tutoring runs, the final controller JSON must contain the exact current-turn live tutor fields and the exact verifier child ids.
- For approved tutoring runs, baseline and turn verifier ids should normally be identical because the same persistent child is reused across both modes.
