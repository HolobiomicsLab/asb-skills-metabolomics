---
name: refscore-based-candidate-filtering
description: Use when a mass deviation query against PubChem (e.g., 10 ppm with 0.002
  Da absolute minimum) returns very large candidate sets (challenges 1, 2, and 14)
  for MAGMa penalty-score ranking, and candidate-list reduction by compound reliability
  is desired.
when_to_use_negative:
- When the correct compound is known to have a very low PubChem refscore (like challenge
  3's N2-acetyl glutaminyl leucinamide, refscore 1) — aggressive thresholds (<200)
  will remove it from the candidate set entirely.
- When the initial candidate list is already small (few candidates), filtering provides
  no ranking benefit and only risks excluding the true structure.
- When candidates must be retrieved without a refscore field (e.g., sources other
  than the local PubChem snapshot), since the threshold cannot be applied.
tools:
- name: MAGMa
  role: Produces the penalty-score rankings whose correct-candidate ranks are tracked
    as candidate lists are stepwise reduced by refscore thresholds
  repo: https://github.com/NLeSC/MAGMa
- name: PubChem
  role: Source of candidate molecules and their refscore values used as the filtering
    criterion
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.5702/massspectrometry.s0033
    title: Automatic Compound Annotation from Mass Spectrometry Data Using MAGMa
  source_content_hash: sha256:916108ebbdcf447d8519b0b8fa57f9a35614a57dda11004e5676aeb75b68043e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/NLeSC/MAGMa
  tools:
  - MAGMa
  - PubChem
  - emetabolomics web interface
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# refscore-based-candidate-filtering

## Summary

Prunes large PubChem-derived candidate molecule lists using the PubChem refscore (a database quality/reliability score), either at a fixed threshold (e.g., 5) or stepwise (<1000, <500, <200 candidates) to improve MAGMa structure-ranking results.

## When to use

Use when a mass deviation query against PubChem (e.g., 10 ppm with 0.002 Da absolute minimum) returns very large candidate sets (challenges 1, 2, and 14) for MAGMa penalty-score ranking, and candidate-list reduction by compound reliability is desired.

## When NOT to use

- When the correct compound is known to have a very low PubChem refscore (like challenge 3's N2-acetyl glutaminyl leucinamide, refscore 1) — aggressive thresholds (<200) will remove it from the candidate set entirely.
- When the initial candidate list is already small (few candidates), filtering provides no ranking benefit and only risks excluding the true structure.
- When candidates must be retrieved without a refscore field (e.g., sources other than the local PubChem snapshot), since the threshold cannot be applied.

## Inputs

- Local PubChem database snapshot (all compounds on 2 December 2013, C/H/N/O/P/S only, up to 1200 Da)
- Per-compound PubChem refscore values for the retrieved candidate set
- MAGMa retrospective candidate rankings (penalty-score based) per CASMI challenge

## Outputs

- Filtered candidate lists at <1000, <500, and <200 molecules per challenge
- Table of the correct candidate's absolute ranking at each refscore threshold level (e.g., refscore_filtering_table.csv)
- Counter-example record for correct candidates lost to filtering (challenge 3, N2-acetyl glutaminyl leucinamide, refscore 1)

## How to apply

First retrieve candidates from the local PubChem snapshot (2 December 2013, C/H/N/O/P/S only, up to 1200 Da) using the 10 ppm / 0.002 Da mass deviation window (4 ppm for challenges 4, 5, 6). If the initial set is very large, apply a fixed refscore threshold of 5 to filter it, as done for CASMI challenges 1, 2, and 14. For retrospective analysis, stepwise raise the refscore threshold until the candidate sets shrink below 1000, then below 500, then below 200 molecules, recording the absolute rank of the correct candidate at each level. Rank candidates by reversed MAGMa penalty score, tie-breaking by PubChem refscore. Rationale: filtering removes low-confidence database entries, so the correct structure's absolute rank tends to improve as the list shrinks — but compounds with low refscores that are genuinely correct (challenge 3's N2-acetyl glutaminyl leucinamide, refscore 1) are lost below the <200 threshold.

## Related tools

- **MAGMa** (Produces the penalty-score rankings whose correct-candidate ranks are tracked as candidate lists are stepwise reduced by refscore thresholds) — https://github.com/NLeSC/MAGMa
- **PubChem** (Source of candidate molecules and their refscore values used as the filtering criterion)

## Evaluation signals

- The correct compound remains present in the reduced candidate lists at nearly every threshold level (per the article's reported trend).
- The correct candidate's absolute rank improves monotonically (or near-monotonically) as the candidate set shrinks from <1000 to <500 to <200 molecules.
- Candidate counts after each filtering step actually fall below the targeted sizes (<1000, <500, <200).
- Challenge 3's N2-acetyl glutaminyl leucinamide appears as a counter-example: its rank degrades to exclusion once the list is reduced below 200 molecules.
- The fixed refscore threshold of 5, when applied, substantially reduces the initially large candidate lists for challenges 1, 2, and 14.

## Limitations

- Filtering can exclude the true structure: challenge 3's N2-acetyl glutaminyl leucinamide (refscore 1) was the only loss when lists were reduced below 200 molecules.
- Refscore filtering is only applicable when candidates come from PubChem with refscore annotations; the article's snapshot was limited to C/H/N/O/P/S compounds up to 1200 Da from 2 December 2013.
- The refscore is a database-quality heuristic, not a spectral-evidence measure — it reorders only equally scoring candidates and reduces list size, and does not guarantee the correct structure is ranked first.

## Evidence

- [results] Stepwise threshold reduction procedure: [evidence span withheld — anchor status: not_found]
- [readme] PubChem database component used to build candidate lookup: [evidence span withheld — anchor status: external_doc]
