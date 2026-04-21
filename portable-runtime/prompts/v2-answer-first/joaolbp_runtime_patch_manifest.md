# JoaoLBP runtime patch manifest

This bundle contains the rewritten files and a unified diff generated from `UNIVERSAL_LLM_RUNTIME_MODIFICATION_PACKET_2026-04-20.txt`.

Patch paths use POSIX separators for patch applicability. The `Exact corpus path` column preserves the source packet path strings.

The two `ACTIVE_RUN.json` files are seeded with neutral defaults because the redesign treated them as runtime-generated pointers; if preferred, they can be removed from version control and created on first run.

| Change type | Exact corpus path | Patch path |
|---|---|---|
| modify | `AGENTS.md` | `AGENTS.md` |
| modify | `NEW CHAT EXECUTION PROMPT - JOAOLBP - ANSWER FIRST.txt` | `NEW CHAT EXECUTION PROMPT - JOAOLBP - ANSWER FIRST.txt` |
| modify | `NEW CHAT EXECUTION PROMPT - JOAOLBP.txt` | `NEW CHAT EXECUTION PROMPT - JOAOLBP.txt` |
| modify | `PROJECT_ROLE_MAP.md` | `PROJECT_ROLE_MAP.md` |
| modify | `backup 1.0\project_snapshot\JoaoLBP\usable prompts\00 INDEX - usable prompts.md` | `backup 1.0/project_snapshot/JoaoLBP/usable prompts/00 INDEX - usable prompts.md` |
| modify | `backup 1.0\project_snapshot\JoaoLBP\usable prompts\v2-answer-first\01 CONTROLLER - persistent orchestration v2.txt` | `backup 1.0/project_snapshot/JoaoLBP/usable prompts/v2-answer-first/01 CONTROLLER - persistent orchestration v2.txt` |
| modify | `backup 1.0\project_snapshot\JoaoLBP\usable prompts\v2-answer-first\03 VERIFIER1 - adversarial falsifier v2.txt` | `backup 1.0/project_snapshot/JoaoLBP/usable prompts/v2-answer-first/03 VERIFIER1 - adversarial falsifier v2.txt` |
| modify | `backup 1.0\project_snapshot\JoaoLBP\usable prompts\v2-answer-first\05 SHARED SCHEMA - runtime objects v2.md` | `backup 1.0/project_snapshot/JoaoLBP/usable prompts/v2-answer-first/05 SHARED SCHEMA - runtime objects v2.md` |
| modify | `backup 1.0\project_snapshot\JoaoLBP\usable prompts\v2-answer-first\07 EVAL BATTERY - multi-agent runtime v2.md` | `backup 1.0/project_snapshot/JoaoLBP/usable prompts/v2-answer-first/07 EVAL BATTERY - multi-agent runtime v2.md` |
| modify | `backup 1.0\project_snapshot\JoaoLBP\usable prompts\v2-answer-first\08 RUNTIME PLAYBOOK - always full multi-agent.md` | `backup 1.0/project_snapshot/JoaoLBP/usable prompts/v2-answer-first/08 RUNTIME PLAYBOOK - always full multi-agent.md` |
| modify | `portable-clean-runtime\AGENTS.md` | `portable-clean-runtime/AGENTS.md` |
| modify | `portable-clean-runtime\PORTABLE NEW CHAT EXECUTION PROMPT - ANSWER FIRST.txt` | `portable-clean-runtime/PORTABLE NEW CHAT EXECUTION PROMPT - ANSWER FIRST.txt` |
| modify | `portable-clean-runtime\PORTABLE NEW CHAT EXECUTION PROMPT.txt` | `portable-clean-runtime/PORTABLE NEW CHAT EXECUTION PROMPT.txt` |
| modify | `portable-clean-runtime\README.md` | `portable-clean-runtime/README.md` |
| modify | `portable-clean-runtime\prompts\v2-answer-first\01 CONTROLLER - persistent orchestration v2.txt` | `portable-clean-runtime/prompts/v2-answer-first/01 CONTROLLER - persistent orchestration v2.txt` |
| modify | `portable-clean-runtime\prompts\v2-answer-first\03 VERIFIER1 - adversarial falsifier v2.txt` | `portable-clean-runtime/prompts/v2-answer-first/03 VERIFIER1 - adversarial falsifier v2.txt` |
| modify | `portable-clean-runtime\prompts\v2-answer-first\05 SHARED SCHEMA - runtime objects v2.md` | `portable-clean-runtime/prompts/v2-answer-first/05 SHARED SCHEMA - runtime objects v2.md` |
| modify | `portable-clean-runtime\prompts\v2-answer-first\07 EVAL BATTERY - multi-agent runtime v2.md` | `portable-clean-runtime/prompts/v2-answer-first/07 EVAL BATTERY - multi-agent runtime v2.md` |
| modify | `portable-clean-runtime\prompts\v2-answer-first\08 RUNTIME PLAYBOOK - always full multi-agent.md` | `portable-clean-runtime/prompts/v2-answer-first/08 RUNTIME PLAYBOOK - always full multi-agent.md` |
| modify | `portable-clean-runtime\runtime\answer-first-run\README.md` | `portable-clean-runtime/runtime/answer-first-run/README.md` |
| modify | `runtime\answer-first-run\README.md` | `runtime/answer-first-run/README.md` |
| add | `runtime\answer-first-run\ACTIVE_RUN.json` | `runtime/answer-first-run/ACTIVE_RUN.json` |
| add | `portable-clean-runtime\runtime\answer-first-run\ACTIVE_RUN.json` | `portable-clean-runtime/runtime/answer-first-run/ACTIVE_RUN.json` |
