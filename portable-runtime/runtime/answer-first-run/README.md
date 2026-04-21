# Answer-First Runtime Packet Contract - Minimal File Surface

Use this folder for file-backed runs of the answer-first runtime variant.

This runtime is separate from any canon or legacy runtime.
Do not mix its turn folders with `runtime/active-run/`.
Do not inspect or reuse `runtime/active-run/*` or any `v2-canon` artifact during live execution of this repo.

## Run root

Per answer-first run:
- `runtime/answer-first-run/turn-XXX/`

## Route types

Every incoming turn must be classified as exactly one route:
- `NEW_PROBLEM`
- `SAME_PROBLEM_CONTINUATION`
- `CONTROL_MESSAGE`
- `REPAIR`

## Run-selection rule

- A new screenshot, new exact student problem text, or a materially different problem means `NEW_PROBLEM`.
- Copied classroom transcript blocks for the same already-verified problem mean `SAME_PROBLEM_CONTINUATION`.
- Plain normal-language operator instructions mean `CONTROL_MESSAGE`.
- Explicit repair after a failed verifier pass means `REPAIR`.
- Explicit `resume` means resume the exact saved stage for the named or current open turn.
- Do not auto-resume an older turn just because it exists.
- `archive/` is evidence-only and not part of live resumable state.

## Minimal file policy

Always write:
- `00-turn-input.json`
- `turn-state.json`

Write only when the stage exists:
- `01-controller-answer-packet.json`
- `active-agent-registry.json`
- `02-verifier1-baseline-verification.json`
- `03-verified-answer-packet.json`
- `04-live-tutor-output.json`
- `05-verifier1-turn-verification.json`

Do not create empty placeholders for skipped stages.

## Deprecated files that must not be created

- `00a-current-packet-manifest.json`
- `00b-repair-ticket.json`
- `stage-state.json`
- `06-final-cleanup-report.json`
- `ACTIVE_RUN.json`
- any prefixed variant of `active-agent-registry.json`

## `turn-state.json` authority rule

`turn-state.json` is the only authoritative local state file for:
- stage tracking
- packet index and hashes
- evidentiary boundary
- reuse basis
- repair state
- cleanup state

Do not split those responsibilities across extra control files.

## `active-agent-registry.json` authority rule

`active-agent-registry.json` exists only when at least one verifier child or backup child is spawned.

Rules:
- each spawned verifier child id must be written to `active-agent-registry.json` before waiting on that child
- controller-authored, synthetic, or locally fabricated verifier packets are invalid
- if a required verifier child cannot be spawned visibly, block the turn
- if a child was spawned, cleanup must resolve its final status by exact id
- the same persistent `verifier1` child should normally cover both baseline and turn verification in a successful run
- the expected persistent `verifier1` settings are `model = gpt-5.4` and `reasoning_effort = xhigh`

## Controller search decision rule

Every current-turn `01-controller-answer-packet.json` must record:
- `claim_delta_status`
- `search_required`
- `search_scope`
- `search_skip_reason`
- `search_decision_basis`

Exact mapping:
- `NEW_PROBLEM | NEW_SUBSTANTIVE_CLAIM | CONTRADICTION` -> `search_required = true`
- `NO_NEW_SUBSTANTIVE_CLAIMS` -> `search_required = false` only when legal reuse conditions are satisfied and no invalidation trigger fires
- `CONTROL_ONLY` -> `search_required = false`, `search_scope = NONE_CONTROL_ONLY`, `search_skip_reason = CONTROL_ONLY_NO_SUBSTANTIVE_CLAIM`

When `search_required = false`, the packet must explain why search was skipped.

## Approval gate

Approve a tutoring run only if all are true:
- `01-controller-answer-packet.json` exists
- baseline verification either passed or was legally reused through the current-turn `03-verified-answer-packet.json`
- `04-live-tutor-output.json` exists when a live tutor reply is approved
- `TURN_CONSISTENCY_CHECK` was executed by the current turn's persistent, UI-visible `verifier1` child
- the saved raw `05-verifier1-turn-verification.json` shows at least one fresh adversarial live retrieval

If any condition fails:
- block or reject the run
- do not present a tutor reply as approved

## Packet rules

- Save the exact packet before the next handoff.
- Pass the exact saved packet or exact pasted packet content to the next stage.
- Do not rely on implicit thread state as the source of truth.
- If a required packet is missing or stale, block the turn and report the exact missing stage or file.
- Keep transport packets, live tutor output, and audit output separate.

## Legal reuse rule for current-turn `03-verified-answer-packet.json`

Current-turn `03-verified-answer-packet.json` is required only when the turn reaches live output generation or live verification.

It may be produced in one of two ways only:

1. `FRESH_BASELINE_PASS`
   - built from the current-turn baseline verifier packet

2. `REUSED_PRIOR_APPROVED_BOUNDARY`
   - built by the controller from the latest approved answer-first boundary
   - allowed only when:
     - `claim_delta_status = NO_NEW_SUBSTANTIVE_CLAIMS`
     - the active problem fingerprint matches
     - the latest approved `03-verified-answer-packet.json` exists
     - the latest approved `05-verifier1-turn-verification.json` exists
     - no invalidation trigger fires
   - the current-turn `03` must record the exact reused paths and hashes in `reuse_basis`

Prior approved packets may be reused only as controller context and verified-boundary context.
They do not replace the requirement to write a fresh current-turn `03` when the current turn reaches live output or live verification.

## Invalidation triggers

A same-problem continuation must rerun baseline verification if any of these fire:
1. the active problem fingerprint changes
2. the active target or subpart changes materially
3. the controller introduces a new substantive answer or method claim
4. the latest approved `03` or `05` is missing, stale, or hash-mismatched
5. the controller source basis changed materially
6. the turn verifier exposes a contradiction requiring upstream repair
7. mismatch evidence shows the current verified boundary no longer matches the live target
8. wrong-role output or packet corruption is detected

## Source independence rule

For substantive approved or blocked claims:
- baseline verification normally needs two independent directly opened live sources
- a third source is allowed only for conflict resolution or failed independence
- two sources are not independent if they are derivative mirrors, shared-lineage reproductions, or thin summaries of the same underlying text when a better independent source is available
- `05-verifier1-turn-verification.json` must expose `independence_audit` whenever substantive claims were approved or blocked

## Minimal continuation read set

Default prior-turn reads:
- prior `turn-state.json`
- prior `03-verified-answer-packet.json`
- prior `05-verifier1-turn-verification.json` when it exists
- prior `active-agent-registry.json` only when prior `turn-state.json` says child provenance or cleanup state must be checked

Read no other prior turn file unless the saved stage or a blocker requires it.

## Approved boundary

For an approved tutoring turn, `turn-state.json` should normally expose these current-turn files as the evidentiary boundary:
- `03-verified-answer-packet.json`
- `04-live-tutor-output.json`
- `05-verifier1-turn-verification.json`

## Completion rule

On successful completion:
- `turn-state.json.current_stage` must be `cleanup_complete`
- `turn-state.json.last_completed_stage` must be `cleanup_complete`
- `turn-state.json.cleanup.cleanup_complete` must be `true`
- all tracked child agents must be marked closed, not found, or explicitly exempted by operator instruction

Do not patch prompts during an active run.
Stop the run, patch, then start the next run.
