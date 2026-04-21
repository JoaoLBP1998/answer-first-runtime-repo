# Eval Battery - Answer-First Runtime V2

Use this battery to test the answer-first runtime before promoting it.

## Core cases

1. Multi-part screenshot
   - Input: screenshot with visible parts `(a)` and `(b)`.
   - Expected: controller answer packet solves all feasible visible parts; visible output contains one sendable cycle only.

2. Simple algebra text-only question
   - Input: one straightforward solve-for-x prompt.
   - Expected: no solver-child stage, no 5-cycle baseline, no `candidate_interaction_tree`, no `cycle_2`.

3. Under-sourced substantive answer claim
   - Input: proof bundle with one substantive approved claim supported by only one live source plus packet context.
   - Expected: controller or verifier rejects it.

4. Ambiguous visible target
   - Input: screenshot with multiple visible subparts and no selected branch.
   - Expected: answer packet solves feasible parts internally; visible send cycle may ask which part to start with; verifier does not lint choreography.

5. Wrong-role output
   - Input: verifier1 returns controller-answer fields instead of proof-bundle fields.
   - Expected: `WRONG_ROLE_OUTPUT`, not stage completion.

6. Live-cycle drift
   - Input: `LIVE_TUTOR_OUTPUT` contains a new substantive claim not covered by the verified answer packet.
   - Expected: turn check runs targeted fresh search or fails closed.

7. Controller-search regression
   - Input: routine answer-first run.
   - Expected: controller may browse, `search_required` may be `true`, and `controller_supporting_sources` is present when web support is used.

8. Verifier URL challenge regression
   - Input: controller answer packet cites a mismatched, dead, or weak URL for an approved claim.
   - Expected: verifier1 flags the URL issue and rejects or downgrades the related claim.

9. Cleanup regression
   - Input: completed run.
   - Expected: final cleanup report exists and no tracked agents remain open.

10. Classroom transcript continuation
   - Input: copied classroom transcript blocks with speaker / timestamp / message lines for the same already-verified problem.
   - Expected: controller saves a fresh `01-controller-answer-packet.json`, parses the latest student line and latest tutor line, reuses prior verified proof only as context, and emits one directly sendable guided next reply.

11. Normal-language control message
   - Input: plain operator text such as ending or redirecting the session.
   - Expected: classified as control input, not a student continuation line.

12. Near-correct student reply
   - Input: classroom continuation where the student is nearly right but not fully correct.
   - Expected: live tutor output stays guided and uses brief wording such as `Quite close.` or `You're almost there.` before the correction or next hint.

13. Current-file defect audit trace
   - Input: verifier packet that alleges a current-file defect.
   - Expected: verifier provides exact `mismatch_evidence`; otherwise the controller rejects the verdict as incomplete.

## Parser checks

Every active schema should be parse-tested as its own artifact:
- `CONTROLLER_ANSWER_PACKET`
- `VERIFIED_ANSWER_PACKET`
- `LIVE_TUTOR_OUTPUT`
- `PROOF_BUNDLE`
- `FINAL_CLEANUP_REPORT`

## Pass criteria

- No pedagogical baseline objects appear in the answer-first variant.
- No solver-child stage remains in the controller flow.
- `candidate_interaction_tree` is absent.
- `draft_normal_output` is absent.
- `cycle_2` is absent.
- `pedagogical` claim typing is absent.
- Controller-side search is allowed and explicit.
- Controller-cited URLs are opened and challenged by `verifier1`.
- Substantive answer claims use two independent live sources by default.
- Classroom transcript continuation stays in the answer-first runtime and does not collapse into plain thread-memory tutoring.
- Every classroom continuation turn still writes a fresh `01-controller-answer-packet.json`.
- `session_mode`, `latest_student_line`, and `latest_tutor_line` are present when classroom transcript continuation is used.
- Normal-language operator commands are not misclassified as student classroom lines.
- Turn verification checks that the live reply is tied to the latest student line.
- `checked_urls` is populated whenever URLs or local packets were checked.
- `supporting_sources` is populated for approved or blocked substantive claims.
- A verdict alleging a current-file defect includes `mismatch_evidence`.
- The visible send cycle remains separate from the raw answer packet and the audit packet.
- Canonical fail strings remain exact:
  - `EXECUTION FAILED, REQUEST AGAIN OR TRY WITH A BACKUP AGENT`
  - `UNVERIFIED - DO NOT USE`
