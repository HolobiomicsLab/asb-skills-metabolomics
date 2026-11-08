---
name: median-aggregation-across-candidate-sets
description: Use when comparing candidate structure sets for the same spectra — e.g.
  whether the incorrect de novo prediction is as chemically close to the truth as
  the incorrect database candidate.
when_to_use_negative:
- When candidate sets cover different spectra or different formulas — the medians
  are only meaningful for matched sets of evaluation spectra.
- When the true structure is unavailable for a spectrum — the incorrect-candidate
  identification and Tanimoto-to-truth calculation cannot be done.
- When you only need candidate rankings without a chemical similarity or score comparison
  across sets — use direct ranking metrics instead.
edam_operation: http://edamontology.org/operation_2940
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: MSNovelist
  role: Source of the de novo candidate set and its per-spectrum ranked structures
    (with score_mod_platt ordering) used as one of the compared groups.
  repo: https://github.com/meowcat/MSNovelist
- name: CSI:FingerID
  role: Defines the modified Platt fingerprint-match score that is aggregated; also
    produces the database-search candidate set used as the comparison baseline.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-022-01486-3
    title: 'MSNovelist: de novo structure generation from mass spectra'
  source_content_hash: sha256:0a890b26808389312c8fd8dcb33f1e0b7a745befc2f89241572c638216ca6f26
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/meowcat/MSNovelist
  tools:
  - CSI:FingerID
  - MSNovelist
  edam_operation: http://edamontology.org/operation_2940
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# median-aggregation-across-candidate-sets

## Summary

Aggregates per-spectrum Tanimoto similarities and modified Platt scores across candidate sets (MSNovelist de novo, naive generation, database search, training-set picks) into per-set median statistics, enabling a like-for-like comparison of de novo versus database candidate quality on GNPS spectra.

## When to use

Use when comparing candidate structure sets for the same spectra — e.g. whether the best incorrect de novo prediction is as chemically close to the truth as the best incorrect database candidate.

## When NOT to use

- When candidate sets cover different spectra or different formulas — the medians are only meaningful for matched sets of evaluation spectra.
- When the true structure is unavailable for a spectrum — the incorrect-candidate identification and Tanimoto-to-truth calculation cannot be done.
- When you only need candidate rankings without a chemical similarity or score comparison across sets — use direct ranking metrics instead.

## Inputs

- Per-spectrum candidate sets with candidate structures and their rankings (MSNovelist de novo, naive generation, CSI:FingerID database candidates, best-in-training-set, random-training-set)
- True reference structures for each of the 3,863 GNPS evaluation spectra
- RDKit-computable molecular fingerprints for each candidate and reference structure

## Outputs

- candidate_comparison.csv — per-candidate-set table of median Tanimoto similarity and median modified Platt score for the best incorrect candidates

## How to apply

For each candidate set (MSNovelist de novo, naive generator without fingerprint input, CSI:FingerID database search, best-in-training-set and random-training-set) over the 3,863-spectrum GNPS evaluation set, first identify the best-ranked incorrect candidate: the top candidate whose structure differs from the true reference structure. Compute Tanimoto similarity between that candidate and the true structure using RDKit molecular fingerprints, and the modified Platt score — the fingerprint-match score CSI:FingerID uses to re-rank candidates — for each top candidate. Then compute the median per candidate set. Results should reproduce the reported medians: Tanimoto 0.80 (de novo) vs 0.76 (best-in-training-set) vs 0.84 (database), and modified Platt scores −5,886 vs −5,883 vs −5,928.

## Related tools

- **MSNovelist** (Source of the de novo candidate set and its per-spectrum ranked structures (with score_mod_platt ordering) used as one of the compared groups.) — https://github.com/meowcat/MSNovelist
- **CSI:FingerID** (Defines the modified Platt fingerprint-match score that is aggregated; also produces the database-search candidate set used as the comparison baseline.)

## Examples

```
MSNovelist results ordered by score_mod_platt descendingly (or filtered to rank_score_lim_mod_platt == 1) are used to select the top de novo candidate per spectrum before computing RDKit Tanimoto and modified Platt medians
```

## Evaluation signals

- Reproduce reported medians: Tanimoto 0.80 (de novo best incorrect), 0.76 (best-in-training-set), 0.84 (database).
- Reproduce reported medians: modified Platt score −5,886 (de novo), −5,883 (best-in-training-set), −5,928 (database).
- For each spectrum, the 'best incorrect' candidate must be the top-ranked candidate whose structure is not identical to the true structure.
- Tanimoto similarities must be computed between the candidate and the true structure using the same fingerprint scheme (RDKit) for all candidate sets.

## Limitations

- Comparison is restricted to GNPS evaluation spectra where a true structure is available; it does not generalize to fully unknown compounds without reference structures.
- Naive generation (without fingerprint input) retrieved only 31% of correct structures, so candidate-set sizes may be skewed and influence aggregate comparisons.

## Evidence

- [abstract] The best incorrect de novo predictions (median 0.80) scored higher than the best-in-training set (0.76) and nearly as high as the best incorrect database structure (0.84): "The best incorrect de novo predictions (median 0.80) scored higher than the best-in-training set (0.76) and nearly as high as the best incorrect database structure (0.84)"
- [abstract] the candidate structures are re-ranked by calculating the match to the query fingerprint xF using the modified Platt score8.: "the candidate structures are re-ranked by calculating the match to the query fingerprint xF using the modified Platt score8."
- [abstract] The correct structure was retrieved for 45% and ranked first for 25% of the instances (Fig. 2a).: "The correct structure was retrieved for 45% and ranked first for 25% of the instances (Fig. 2a)."
