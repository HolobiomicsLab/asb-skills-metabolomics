---
name: molecular-formula-ground-truth-comparison
description: Use when a benchmark provides known molecular formulas for challenges
  (e.g., CASMI category 1) and you need to verify that the top-ranked formula from
  your annotation pipeline matches the ground truth for each challenge.
when_to_use_negative:
- When the goal is structure identification (CASMI category 2 candidate ranking),
  not formula assignment — use the candidate-ranking skill instead
- When no ground-truth formulas are available for the challenges, since the comparison
  step requires provided reference formulas
- When the candidate sets have not been generated/scored by the upstream MAGMa substructure-annotation
  workflow (no top-ranked formula exists yet to compare)
edam_operation: http://edamontology.org/operation_3663
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: MAGMa
  role: Produces the ranked candidate list whose top-ranked formula is compared against
    the ground truth; run in command line mode or via the web interface
  repo: https://github.com/NLeSC/MAGMa
- name: PubChem
  role: Source of candidate molecules (local 2 December 2013 snapshot, C/H/N/O/P/S
    only, mass <= 1200 Da) providing the formulas being validated
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.5702/massspectrometry.s0033
    title: Automatic Compound Annotation from Mass Spectrometry Data Using MAGMa
  source_content_hash: sha256:e16aebab361e9036b9da4db5b3bbe8fb8750afa1d6cf70b4e4fdb49a8b696ce1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/NLeSC/MAGMa
  tools:
  - MAGMa
  - PubChem
  edam_operation: http://edamontology.org/operation_3663
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# molecular-formula-ground-truth-comparison

## Summary

Compares the top-ranked molecular formula produced by a candidate-scoring workflow (here MAGMa's penalty-score ranking over PubChem candidates) against a contest- or reference-provided ground-truth formula, recording a per-challenge match status to validate formula assignment accuracy.

## When to use

Use when a benchmark provides known molecular formulas for challenges (e.g., CASMI category 1) and you need to verify that the top-ranked formula from your annotation pipeline matches the ground truth for each challenge.

## When NOT to use

- When the goal is structure identification (CASMI category 2 candidate ranking), not formula assignment — use the candidate-ranking skill instead
- When no ground-truth formulas are available for the challenges, since the comparison step requires provided reference formulas
- When the candidate sets have not been generated/scored by the upstream MAGMa substructure-annotation workflow (no top-ranked formula exists yet to compare)

## Inputs

- Ranked candidate list per challenge with top-ranked candidate's molecular formula (MAGMa penalty-score ranking output)
- CASMI 2013 category 1 challenge peak lists (precursor peak with nested fragment peaks, converted to MAGMa hierarchical input format)
- Ground-truth molecular formula provided by the CASMI contest for each category-1 challenge

## Outputs

- Per-challenge table listing challenge ID, top-ranked molecular formula, CASMI ground-truth formula, and match status
- Summary validation statement (e.g., number of correct top-ranked formulas out of 8 category-1 challenges)

## How to apply

First run the upstream annotation workflow: convert each CASMI peak list to the MAGMa hierarchical input format (precursor peak with fragment peaks nested in parentheses), retrieve candidate molecules from the local PubChem snapshot (2 December 2013, C/H/N/O/P/S elements only, mass <= 1200 Da) within a 10 ppm mass deviation (absolute minimum 0.002 Da; 4 ppm for challenges 4-6), generate substructures with up to 3 bond dissociations expanded with single losses of hydroxyl or primary amine groups, and rank candidates by reversed intensity-weighted average penalty score, breaking ties by PubChem refscore. Then read off the molecular formula of the top-ranked candidate for each challenge and compare it element-by-element with the CASMI-provided ground-truth formula. Record a per-challenge table with the top-ranked formula, the ground-truth formula, and a match/mismatch status. Validation requires the match-status table to reproduce the reported result; in this study all 8 category-1 top-ranked formulas were correct.

## Related tools

- **MAGMa** (Produces the ranked candidate list whose top-ranked formula is compared against the ground truth; run in command line mode or via the web interface) — https://github.com/NLeSC/MAGMa
- **PubChem** (Source of candidate molecules (local 2 December 2013 snapshot, C/H/N/O/P/S only, mass <= 1200 Da) providing the formulas being validated)

## Evaluation signals

- Every challenge in the per-challenge table has a recorded match status (correct/incorrect) against the CASMI-provided ground-truth formula
- The top-ranked formula is taken from the candidate ranked first under the reversed penalty-score ordering, with ties broken by PubChem refscore
- Reproducing the category-1 table should show all 8 top-ranked formulas correct, matching the reported finding
- Candidate retrieval used the correct mass windows (10 ppm with absolute minimum 0.002 Da; 4 ppm for challenges 4, 5, and 6) so the true formula's candidate set was not excluded

## Limitations

- Correctness of the formula comparison depends on the ground-truth formula being present in the PubChem candidate set; too-restrictive mass tolerance (10 ppm / 0.002 Da) caused misses in retrospective analysis of some challenges (e.g., challenge 11 tautomers)
- Only formula identity is checked, not structural identity — a correct formula does not guarantee the correct candidate structure ranked first (category 2 showed only 6 of 12 correct candidates ranked first)
- Ranking among equally scoring candidates depends on the PubChem refscore tiebreaker and on refscore filtering thresholds (5, or stepwise <1000/<500/<200), which can affect which candidate's formula is read off

## Evidence

- [results] we also submitted lists of unique molecular formulas to the category 1 contest: [evidence span withheld — anchor status: not_found]
- [readme] MAGMa is released under the Apache License Version 2.0.: [evidence span withheld — anchor status: external_doc]
