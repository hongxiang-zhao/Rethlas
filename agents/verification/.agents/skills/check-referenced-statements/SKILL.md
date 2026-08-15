---
name: check-referenced-statements
description: Validate prior local results and externally referenced theorems. Resolve local data/results citations directly; query arXiv theorem search first and Codex's built-in web search second for external papers.
---

# Check Referenced Statements

Validate every previous-result citation and external-paper reference used in the proof.

## Input Contract

For each citation:

- location where it is used,
- the imported consequence,
- either a local `data/results/` path and exact locator or the full external referenced statement text.

## Procedure

1. For `[prior-result: data/results/<relative-file>; locator: <exact locator>]`, resolve the path under `../generation/`, read the cited result at that locator, and verify that it establishes the imported consequence with compatible hypotheses and definitions. Do not require its full statement or proof to be copied into the current proof.
2. Record a critical error if the local path or locator is missing, the prior result is unsupported, or it does not imply the imported consequence; record a gap if the locator is ambiguous.
3. For an external-paper citation, query `search_arxiv_theorems` using the full referenced statement as `query`.
4. Inspect returned results and compare theorem text directly to the referenced statement in reasoning.
5. Expand the definitions and terminology appearing in the cited statement using the cited paper's context before deciding whether the theorem applies.
6. Check whether the same words in the current proof mean the same thing as they do in the cited paper. In mathematics, identical words can carry different definitions in different contexts.
7. Distinguish similar-looking definitions and compare their exact formulas, notation, and quantifiers. Do not collapse two definitions just because the names or formulas look close.
8. Accept an external citation as matched and applicable only when both are true:
   - the result clearly corresponds to the cited statement,
   - the contextual definitions and hypotheses align with the current problem.
9. If the proof uses the referenced statement to obtain further conclusions, verify the transition from the referenced statement to those conclusions.
10. Treat a hand-wavy specialization, instantiation, or downstream deduction as a gap even when the cited theorem itself is valid.
11. Treat a logically invalid transition from the cited theorem to the claimed conclusion as a critical error.
12. If that downstream step deduces one property from another, compare the exact definitions and defining formulas of both properties before accepting the deduction.
13. If the theorem exists but the current proof uses different definitions, hypotheses, ambient objects, or a subtly different defining formula, record a critical error for incorrect application.
14. If no match is found, use Codex's built-in web search with the same statement text.
15. If still not found, emit a critical error:
   - location: where the citation is used,
   - issue: referenced theorem appears non-existent or incorrectly cited.
16. Persist each reference check in `reference_checks`.

Do not rely on dedicated comparison utility code; perform comparison through careful reasoning.

## Output Contract

Append records to `reference_checks` like:

```json
{
  "location": "Lemma 2",
  "referenced_statement": "Exact statement text",
  "context_expansion": "In the cited paper, 'regular' means regular with respect to the valuation topology.",
  "arxiv_match_found": false,
  "web_match_found": false,
  "critical_error": {
    "location": "Lemma 2",
    "issue": "Referenced external theorem was not found in arXiv search or Codex built-in web search."
  }
}
```

## Tools

- `search_arxiv_theorems`
- `memory_append`
- Codex's built-in web search
