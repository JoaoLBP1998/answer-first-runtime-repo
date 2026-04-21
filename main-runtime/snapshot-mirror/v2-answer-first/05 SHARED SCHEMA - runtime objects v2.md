# Shared Runtime Objects V2 - Minimal File Surface

Use these objects across the controller and `verifier1`.

## PRECEDENCE ORDER

Resolve prompt and workflow conflicts in this order:
1. runtime contract and shared schema
2. role-specific prompt
3. skill-level behavior
4. human-facing wording

If wording conflicts with packet structure, preserve packet structure.

## FAIL_STRINGS

```json
{
  "ocr_binding_failure": "EXECUTION FAILED, REQUEST AGAIN OR TRY WITH A BACKUP AGENT",
  "verification_failure": "UNVERIFIED - DO NOT USE"
}
```

## SOURCE_POLICY_DEFAULTS

```json
{
  "baseline_default_required_source_count": 2,
  "conflict_or_independence_escalation_count": 3,
  "turn_consistency_minimum_fresh_live_retrieval_count": 1
}
```

Two independent directly opened live sources are the default for substantive answer or method claims during baseline verification.
Turn consistency checking still requires at least one fresh adversarial live retrieval even when the current verified dossier appears sufficient.
Packet evidence is context, not enough by itself for new substantive answer claim approval.

## CONTROLLER_OCR_CHECK

```json
{
  "exact_visible_text": "string",
  "visible_subparts": [],
  "artifact_quality": "clear | usable_with_risk | insufficient",
  "notes": "string"
}
```

## CONTROLLER_ANSWER_PACKET

```json
{
  "exact_visible_text": "string",
  "session_mode": "NEW_PROBLEM | CLASSROOM_TRANSCRIPT_CONTINUATION | CONTROL_MESSAGE",
  "text_input_classification": "REAL_QUESTION | TUTORING_TURN | CONTROL_INPUT | INSUFFICIENT_ARTIFACT",
  "certainty_score": 0,
  "claim_delta_status": "NEW_PROBLEM | NEW_SUBSTANTIVE_CLAIM | NO_NEW_SUBSTANTIVE_CLAIMS | CONTRADICTION | CONTROL_ONLY",
  "baseline_verification_required": true,
  "search_required": true,
  "search_scope": "EXACT_MATCH | EQUIVALENT_MATCH | METHOD_ONLY | NONE_REUSED_BOUNDARY | NONE_CONTROL_ONLY",
  "search_skip_reason": "NOT_SKIPPED | NO_NEW_SUBSTANTIVE_CLAIMS_LEGAL_REUSE | CONTROL_ONLY_NO_SUBSTANTIVE_CLAIM | NONE",
  "search_decision_basis": "string",
  "match_status": "LOCAL_BOUND | SEARCH_BOUND | LOCAL_AMBIGUOUS | INSUFFICIENT_ARTIFACT",
  "question_match_notes": "string",
  "visible_subparts": [],
  "active_subpart_status": {
    "status": "BOUND | AMBIGUOUS | INCOMPLETE",
    "safely_bound_subpart": "string or NONE"
  },
  "ambiguity_reason": "string or NONE",
  "latest_student_line": "string or NONE",
  "latest_tutor_line": "string or NONE",
  "classroom_transcript_context": {
    "raw_transcript_text": "string or NONE",
    "latest_student_name": "string or NONE",
    "latest_student_line": "string or NONE",
    "latest_tutor_name": "string or NONE",
    "latest_tutor_line": "string or NONE"
  },
  "answer_scope": {
    "solved_subparts": [],
    "unsolved_subparts": [],
    "scope_notes": "string"
  },
  "answer_claims": [],
  "solved_answers": [],
  "controller_supporting_sources": [
    {
      "id": "C1",
      "title": "string",
      "url": "string",
      "opened_live": true,
      "url_status": "LIVE | DEAD | REDIRECTED | MISMATCHED",
      "copied_text": "string",
      "locator": "string",
      "why_this_source_matters": "string"
    }
  ],
  "visible_tutor_cycle": {
    "message_to_tutor_now": "string",
    "next_expected_student_reply": "string",
    "if_on_track": "string",
    "if_confused": "string",
    "silence_breaker": "string",
    "current_step": "string",
    "supporting_claim_ids": []
  },
  "blocking_gaps": []
}
```

Rules:
- `certainty_score` must be an integer from 0 to 100
- `claim_delta_status` governs whether baseline verification is required on continuation turns
- `claim_delta_status = NEW_PROBLEM | NEW_SUBSTANTIVE_CLAIM | CONTRADICTION` requires `search_required = true`, `search_skip_reason = NOT_SKIPPED`, and `search_scope != NONE_REUSED_BOUNDARY | NONE_CONTROL_ONLY`
- `claim_delta_status = NO_NEW_SUBSTANTIVE_CLAIMS` allows `search_required = false` only when legal reuse conditions are satisfied and no invalidation trigger fires
- `claim_delta_status = CONTROL_ONLY` requires `search_required = false`, `search_scope = NONE_CONTROL_ONLY`, and `search_skip_reason = CONTROL_ONLY_NO_SUBSTANTIVE_CLAIM`
- when `search_required = false`, `search_decision_basis` must explain the legal reuse or control-only basis explicitly
- `session_mode = CLASSROOM_TRANSCRIPT_CONTINUATION` means `latest_student_line` must be actionable for the current turn
- `session_mode = CONTROL_MESSAGE` means plain operator language must not be stored as a student-authored classroom line

## VERIFIED_ANSWER_PACKET

```json
{
  "status": "READY | PARTIAL | BLOCKED",
  "active_problem": "string",
  "active_problem_fingerprint": "string",
  "reuse_mode": "FRESH_BASELINE_PASS | REUSED_PRIOR_APPROVED_BOUNDARY",
  "reuse_basis": {
    "turn_path": "string or NONE",
    "verified_answer_packet_path": "string or NONE",
    "turn_verifier_packet_path": "string or NONE",
    "verified_answer_sha256": "string or NONE",
    "turn_verifier_sha256": "string or NONE",
    "reuse_reason": "string or NONE"
  },
  "approved_claim_ids": [],
  "approved_answer_fields": [],
  "verified_source_ids": [],
  "proof_dossier_copy": [],
  "risk_flags": []
}
```

## TURN_STATE

This object replaces separate manifest, stage-state, repair-ticket, and final-cleanup files.
Persist it in the exact file `turn-state.json`.

```json
{
  "run_id": "string",
  "turn_id": "string",
  "route": "NEW_PROBLEM | SAME_PROBLEM_CONTINUATION | CONTROL_MESSAGE | REPAIR",
  "active_problem": "string or NONE",
  "active_problem_fingerprint": "string or NONE",
  "claim_delta_status": "NEW_PROBLEM | NEW_SUBSTANTIVE_CLAIM | NO_NEW_SUBSTANTIVE_CLAIMS | CONTRADICTION | CONTROL_ONLY",
  "baseline_verification_required": true,
  "current_stage": "turn_input_saved | controller_answer_saved | verifier1_baseline_saved | verified_answer_saved | live_tutor_saved | verifier1_turn_saved | control_handled | blocked | cleanup_complete",
  "last_completed_stage": "string",
  "blocked_on_stage": "string or NONE",
  "child_registry_path": "active-agent-registry.json or NONE",
  "packet_index": [
    {
      "role": "turn_input | controller_answer_output | verifier1_baseline_output | verified_answer_packet | live_tutor_output | verifier1_turn_output",
      "path": "string",
      "sha256": "string",
      "authoritative_for_stage": true,
      "allowed_as_evidence": true
    }
  ],
  "approved_boundary": [
    "03-verified-answer-packet.json",
    "04-live-tutor-output.json",
    "05-verifier1-turn-verification.json"
  ],
  "reuse_basis": {
    "turn_path": "string or NONE",
    "verified_answer_packet_path": "string or NONE",
    "turn_verifier_packet_path": "string or NONE",
    "verified_answer_sha256": "string or NONE",
    "turn_verifier_sha256": "string or NONE",
    "reuse_reason": "string or NONE"
  },
  "repair": {
    "owner_role": "controller | verifier1 | NONE",
    "source_audit_packet": "string or NONE",
    "repair_items": []
  },
  "cleanup": {
    "required": true,
    "attempted_agent_ids": [],
    "closed_agent_ids": [],
    "not_found_agent_ids": [],
    "still_open_agent_ids": [],
    "cleanup_complete": false
  }
}
```

Rules:
- update `turn-state.json` only after the referenced packet has been successfully written
- if a packet hash or path in `packet_index` is wrong, stale, or missing, block the turn
- do not create separate files for manifest, repair, stage-state, or final cleanup when `turn-state.json` can hold that state

## ACTIVE_AGENT_REGISTRY

Persist this object in the exact file `active-agent-registry.json` only when at least one verifier child or backup child is spawned.

```json
{
  "run_id": "string",
  "turn_id": "string",
  "tracked_agents": [
    {
      "agent_role": "verifier1 | backup_agent",
      "modes_run": [],
      "stage": "BASELINE_ANSWER_VERIFICATION | TURN_CONSISTENCY_CHECK | TRANSCRIPT_AUDIT | NONE",
      "agent_id": "string",
      "model": "string",
      "reasoning_effort": "xhigh | high | medium | low | minimal | none",
      "ui_visible": true,
      "registered_before_wait": true,
      "is_backup": false,
      "spawn_order": 0,
      "status": "RUNNING | COMPLETED | CLOSED | NOT_FOUND | UNKNOWN",
      "close_status": "CLOSED | NOT_FOUND | UNKNOWN"
    }
  ],
  "open_agent_ids": [],
  "closed_agent_ids": [],
  "not_found_agent_ids": []
}
```

The same persistent `verifier1` child should normally cover both baseline and turn verification in a successful run.
For this runtime, the expected persistent `verifier1` settings are `model = gpt-5.4` and `reasoning_effort = xhigh`.

## LIVE_TUTOR_OUTPUT

```json
{
  "message_to_tutor_now": "string",
  "next_expected_student_reply": "string",
  "if_on_track": "string",
  "if_confused": "string",
  "silence_breaker": "string",
  "current_step": "string",
  "approval_state": "PROVISIONAL | APPROVED | BLOCKED | NONE"
}
```

Rules:
- for classroom transcript continuation, `LIVE_TUTOR_OUTPUT` should be directly sendable tutor wording tied to the latest student line
- keep it guided, brief, and warm
- if the student is right, congratulate briefly before the next step
- if the student is nearly right, prefer `Quite close.` or `You're almost there.` before the correction or next hint

## SOURCE_RECORD

Use this shape for substantive entries inside `sources` and `supporting_sources`.

```json
{
  "source_id": "string",
  "title": "string",
  "url": "string",
  "opened_live": true,
  "copied_text": "string",
  "locator": "string or NONE",
  "independence_group": "string",
  "independence_status": "INDEPENDENT | SHARED_LINEAGE | DERIVATIVE | UNKNOWN",
  "independence_reason": "string"
}
```

## PROOF_BUNDLE

```json
{
  "mode": "BASELINE_ANSWER_VERIFICATION | TURN_CONSISTENCY_CHECK | TRANSCRIPT_AUDIT",
  "proof_status": "DONE | INCOMPLETE | FAIL",
  "status": "PASS | INCOMPLETE | FAIL",
  "coverage_status": "ANSWER_READY | ANSWER_PARTIAL | COVERED_BY_DOSSIER | FRESH_SEARCH_REQUIRED | SELF_AUDIT_REPAIR_REQUIRED",
  "fresh_live_retrieval_performed": false,
  "fresh_live_source_ids": [],
  "claims": [],
  "sources": [],
  "checked_urls": [],
  "proof_dossier_copy": [],
  "turn_level_checks": [],
  "claim_results": [],
  "approved_claim_ids": [],
  "blocked_claim_ids": [],
  "supporting_sources": [],
  "independence_audit": [
    {
      "claim_id": "string",
      "required_independent_source_count": 2,
      "accepted_independent_source_ids": [],
      "rejected_source_ids": [],
      "independence_verdict": "SATISFIED | ESCALATED_TO_THIRD_SOURCE | FAILED | NOT_APPLICABLE",
      "third_source_reason": "string or NONE"
    }
  ],
  "approved_target_fields": {},
  "mismatch_evidence": [],
  "mismatches": [],
  "repair_instructions": [],
  "transcript_verdict": "CORRECT | WRONG | MIXED | UNVERIFIED | NONE",
  "transcript_findings": [],
  "escalation_recommended": false,
  "unsupported_claim_ids": [],
  "conflicts": [],
  "missing_data": [],
  "rationale": "string"
}
```

## PACKET TRANSPORT RULE

- `verifier1` should receive the exact explicit packet for its current stage.
- Do not treat thread memory as the authoritative source of the latest controller answer packet, verified answer packet, or proof bundle.
- If `verifier1` says the required packet is missing or stale, the controller must resend the exact packet explicitly.
- `checked_urls` must enumerate every URL actually checked for the current verdict.
- `supporting_sources` must be non-empty for approved or blocked substantive claims.
- `independence_audit` must be non-empty for approved or blocked substantive claims.
- if two accepted sources share lineage, the verifier must either mark independence as failed or explain the third-source escalation explicitly.
- If a verdict alleges a current-file defect, `mismatch_evidence` is required.
- Prior audits and superseded proof bundles are instruction-only context, not evidentiary support.

## REPAIR LOOP RULE

Allow only one repair loop after a failed or incomplete verification pass.
After the second failure, fail closed with `UNVERIFIED - DO NOT USE`.
