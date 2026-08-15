---
name: verify-proof
description: Verify candidate proofs with the local proof verification MCP service. Use only when a full candidate proof of the entire problem has been assembled in markdown, and before publishing the final verified blueprint.
---

# Verify Proof

Use the local proof verification service as the canonical verifier before accepting a solution.
Do not use this skill for partial proofs, isolated subgoals, or branches that have not yet produced a full proof draft of the whole problem.

## Input Contract

Read:

- target theorem statement
- assembled proof blueprint candidate from `results/{problem_id}/blueprint.md` as pure markdown text
- relevant prior failure reports and branch context
- semantically relevant previous logs and results under `data/logs/` and `data/results/`, when available; these may come from differently named problems

## Procedure

1. Read the current `results/{problem_id}/blueprint.md` draft as pure text.
2. Search `data/logs/` and `data/results/` for semantically relevant prior verification failures, counterexamples, and repair attempts, without restricting the search to the current problem name. Make sure the current draft does not repeat a known gap.
3. For every `[prior-result: ...; locator: ...]` citation, open the cited file under `data/results/`, resolve the locator, and check that it establishes the consequence claimed in the current proof with compatible hypotheses and definitions. Do not copy the cited statement or proof into the current blueprint merely to make it self-contained. Logs cannot discharge proof obligations.
4. First check that `blueprint.md` contains a full proof draft of the entire target theorem rather than a partial proof, fragment, or exploratory notes. If it does not, do not call the verifier yet.
5. Call MCP tool `verify_proof_service` with:
   - `statement`: target informal statement
   - `proof`: the raw markdown text from `blueprint.md`
6. Read `verification_report.summary`, `critical_errors`, `gaps`, `verdict`, and `repair_hints`.
7. Return and persist exactly what the verification service returns. Do not rename keys, add keys, or change the JSON structure.
8. Treat the proof as failed if any of the following hold:
   - `verdict` is `"wrong"`
   - `verification_report.critical_errors` is non-empty
   - `verification_report.gaps` is non-empty
9. Only treat the proof as passed when none of the failure conditions above hold.
10. If the proof passes, rename `results/{problem_id}/blueprint.md` to `results/{problem_id}/blueprint_verified.md`.

## Output Contract

Append to `verification_reports`:

```json
{
  "verification_report": {
    "summary": "string",
    "critical_errors": [
      {"location": "", "issue": "detailed description of the issue"}
    ],
    "gaps": [
      {"location": "", "issue": "detailed description of the gap"}
    ]
  },
  "verdict": "string",
  "repair_hints": "string"
}
```

Persist the verification service response exactly as returned.

If verification fails, revise `blueprint.md` directly and append to `failed_paths` when a branch is invalidated.

## MCP Tools

- `verify_proof_service`
- `memory_append`
- `memory_search`
- `branch_update`
- Codex built-in web search and `search_arxiv_theorems` when the verifier identifies a missing lemma or gap

## Failure Logging

Always persist verification output, including successful checks.
