# Shared Runtime Objects V2 - Controller Search + Verifier1

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
  "default_required_source_count": 2,
  "conflict_or_independence_escalation_count": 3,
  "dossier_covered_required_source_count": 0
}
```

Two independent directly opened live sources are the default for substantive answer or method claims.
Packet evidence is context, not enough by itself for substantive answer claim approval.

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
  "text_input_classification": "REAL_QUESTION | TUTORING_TURN | INSUFFICIENT_ARTIFACT",
  "certainty_score": 0,
  "search_required": true | false,
  "match_status": "LOCAL_BOUND | SEARCH_BOUND | LOCAL_AMBIGUOUS | INSUFFICIENT_ARTIFACT",
  "question_match_notes": "string",
  "visible_subparts": [],
  "active_subpart_status": {
    "status": "BOUND | AMBIGUOUS | INCOMPLETE",
    "safely_bound_subpart": "string or NONE"
  },
  "ambiguity_reason": "string or NONE",
  "artifact_quality": {},
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

`certainty_score` must be an integer from 0 to 100, not a 0-1 float.

`candidate_interaction_tree`, `draft_normal_output`, `cycle_2`, and pedagogical claim typing are not part of this variant.
`session_mode = CLASSROOM_TRANSCRIPT_CONTINUATION` means `latest_student_line` must be actionable for the current turn.
`session_mode = CONTROL_MESSAGE` means plain operator language must not be stored as a student-authored classroom line.

## VERIFIED_ANSWER_PACKET

```json
{
  "status": "READY | PARTIAL | BLOCKED",
  "approved_claim_ids": [],
  "approved_answer_fields": [],
  "verified_source_ids": [],
  "proof_dossier_copy": [],
  "risk_flags": []
}
```

## SESSION_STATE

```json
{
  "active_problem": "string",
  "active_target": "string",
  "session_status": "OPEN | CLOSED",
  "current_stage": "string",
  "problem_anchor": "exact text | OCR | screenshot + chat",
  "trigger_legend_shown": false,
  "verified_claim_cache": [],
  "accepted_source_cache": [],
  "policy_flags": {
    "assessment_risk": false,
    "artifact_ambiguity": false,
    "time_pressure": false
  }
}
```

## RUNTIME_STAGE_STATE

```json
{
  "run_id": "string",
  "turn_id": "string",
  "current_stage": "turn_input_saved | controller_answer_saved | verifier1_baseline_saved | verified_answer_saved | live_tutor_saved | verifier1_turn_saved | approved | blocked | cleanup_complete",
  "thread_state_reliable": true,
  "last_completed_stage": "string",
  "blocked_on_stage": "string or NONE"
}
```

When cleanup completes, both `current_stage` and `last_completed_stage` must be `cleanup_complete`.

## TURN_PACKET_PATHS

```json
{
  "turn_root": "runtime/answer-first-run/turn-XXX",
  "turn_input": "string or NONE",
  "current_packet_manifest": "string or NONE",
  "repair_ticket": "string or NONE",
  "controller_answer_output": "string or NONE",
  "verifier1_baseline_output": "string or NONE",
  "verified_answer_packet": "string or NONE",
  "live_tutor_output": "string or NONE",
  "verifier1_turn_output": "string or NONE",
  "final_cleanup_report": "string or NONE"
}
```

## CURRENT_PACKET_MANIFEST

```json
{
  "stage": "string",
  "allowed_packets": [
    {
      "role": "turn_input | controller_answer_output | verifier1_baseline_output | verified_answer_packet | live_tutor_output | verifier1_turn_output",
      "path": "string",
      "sha256": "string",
      "allowed_as_evidence": true
    }
  ],
  "repair_history_only_packets": [
    {
      "path": "string",
      "reason": "prior audit | superseded proof | interaction history"
    }
  ],
  "clean_audit_mode": true
}
```

For a completed approved run, the final manifest should normally expose `verified_answer_packet`, `live_tutor_output`, and `verifier1_turn_output` as the approved evidentiary boundary.

## REPAIR_TICKET

```json
{
  "owner_role": "controller | verifier1",
  "instruction_only": true,
  "source_audit_packet": "string",
  "repair_items": [
    {
      "claim_id": "string or NONE",
      "affected_fields": [],
      "failure_reason": "string",
      "packet_path": "string or NONE",
      "field_path": "string or NONE",
      "quoted_fragment": "string or NONE"
    }
  ]
}
```

## ACTIVE_AGENT_REGISTRY

```json
{
  "run_id": "string",
  "tracked_agents": [
    {
      "agent_role": "verifier1 | backup_agent",
      "agent_id": "string",
      "modes_run": [],
      "model": "string",
      "reasoning_effort": "xhigh | high | medium | low | minimal | none",
      "is_backup": false,
      "spawn_order": 0,
      "status": "RUNNING | COMPLETED | CLOSED | NOT_FOUND | UNKNOWN"
    }
  ],
  "open_agent_ids": [],
  "closed_agent_ids": [],
  "not_found_agent_ids": []
}
```

Persist this object in the exact file `active-agent-registry.json`.
Prefixed variants such as `00c-active-agent-registry.json` are invalid for this runtime.

The same persistent `verifier1` child should normally cover both baseline and turn verification in a successful run.

## FINAL_CLEANUP_REPORT

```json
{
  "run_id": "string",
  "attempted_agent_ids": [],
  "closed_agent_ids": [],
  "not_found_agent_ids": [],
  "still_open_agent_ids": [],
  "cleanup_complete": true
}
```

## LIVE_TUTOR_OUTPUT

```json
{
  "message_to_tutor_now": "string",
  "next_expected_student_reply": "string",
  "if_on_track": "string",
  "if_confused": "string",
  "silence_breaker": "string",
  "current_step": "string",
  "approval_state": "PROVISIONAL | APPROVED | BLOCKED"
}
```

For classroom transcript continuation, `LIVE_TUTOR_OUTPUT` should be directly sendable tutor wording tied to the latest student line.
Keep it guided, brief, and warm.
If the student is right, congratulate briefly before the next step.
If the student is nearly right, prefer `Quite close.` or `You're almost there.` before the correction or next hint.

## APPROVED_TARGET_FIELDS

```json
{
  "exact.field.path": true | false
}
```

For baseline verification these paths should point into the answer packet.
For turn verification these paths should point into the 6 `LIVE_TUTOR_OUTPUT` fields.

## PROOF_BUNDLE

```json
{
  "mode": "BASELINE_ANSWER_VERIFICATION | TURN_CONSISTENCY_CHECK | TRANSCRIPT_AUDIT",
  "proof_status": "DONE | INCOMPLETE | FAIL",
  "status": "PASS | INCOMPLETE | FAIL",
  "coverage_status": "ANSWER_READY | ANSWER_PARTIAL | COVERED_BY_DOSSIER | FRESH_SEARCH_REQUIRED | SELF_AUDIT_REPAIR_REQUIRED",
  "claims": [],
  "sources": [],
  "checked_urls": [],
  "proof_dossier_copy": [],
  "turn_level_checks": [],
  "claim_results": [],
  "approved_claim_ids": [],
  "blocked_claim_ids": [],
  "supporting_sources": [],
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
- Do not treat thread memory as the authoritative source of the latest controller answer packet, repair ticket, or proof bundle.
- If `verifier1` says the required packet is missing or stale, the controller must resend the exact packet explicitly.
- `checked_urls` must enumerate every URL or local packet actually checked for the current verdict.
- `supporting_sources` must be non-empty for approved or blocked substantive claims.
- If a verdict alleges a current-file defect, `mismatch_evidence` is required.
- Prior audits, superseded proof bundles, and repair history are instruction-only context, not evidentiary support.
- Under normal operation, the source of truth for `baseline_verifier_agent_id` and `turn_verifier_agent_id` should point to the same persistent verifier child id in `active-agent-registry.json`.

## REPAIR LOOP RULE

Allow only one repair loop after a failed or incomplete verification pass.
After the second failure, fail closed with `UNVERIFIED - DO NOT USE`.
