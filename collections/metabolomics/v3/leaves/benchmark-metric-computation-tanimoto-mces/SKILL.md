---
name: benchmark-metric-computation-tanimoto-mces
description: Use when a generative structure-elucidation model (e.g. MS-BART, DiffMS,
  FRIGID) has produced candidate molecules per spectrum and you must score them on
  MassSpecGym or NPLIB1 with Top-1/Top-10 InChIKey accuracy, MCES distance, and Tanimoto
  similarity.
when_to_use_negative:
- The dataset is NIST23 — access is restricted by commercial licensing, so the metric
  harness cannot be assembled without the test fold
- Evaluating fingerprint-prediction quality itself rather than generated molecular
  structures — use fingerprint bit-level metrics instead of InChIKey/MCES/Tanimoto
  scoring
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: RDKit
  role: Compute Morgan fingerprints (radius 2, 2048 bits) used for Tanimoto similarity
    scoring of candidate vs. reference molecules
  repo: https://github.com/OpenDFM/MS-BART
- name: MS-BART
  role: Generates the ranked candidate SELFIES structures per spectrum that are scored
    with the metric suite
  repo: https://github.com/OpenDFM/MS-BART
- name: MassSpecGym
  role: 'Benchmark test fold providing ground-truth structures and the evaluation
    script (with known issues to correct: test-path filename and NUM_BEAMS/NUM_BEAM
    variable mismatch)'
- name: NPLIB1
  role: Second benchmark test fold (GNPS subset curated for CANOPUS) on which the
    same metrics are computed
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:ee744c5944a46a6e9df4e1d4673bad571dea61b525055c613bd25bd855728141
  source_papers:
  - doi: 10.48550/arxiv.2510.20615
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/OpenDFM/MS-BART
  tools:
  - BART-Base
  - MS-BART
  - MassSpecGym
  - NPLIB1
  - RDKit
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# benchmark-metric-computation-tanimoto-mces

## Summary

Computes the standard structure-elucidation benchmark metrics — Top-1/Top-10 accuracy by full InChIKey exact match, minimum MCES distance, and maximum Tanimoto similarity (RDKit Morgan fingerprints, radius 2) — to evaluate de novo molecular candidate sets against MassSpecGym and NPLIB1 test folds.

## When to use

Use when a generative structure-elucidation model (e.g. MS-BART, DiffMS, FRIGID) has produced candidate molecules per spectrum and you must score them on MassSpecGym or NPLIB1 with Top-1/Top-10 InChIKey accuracy, MCES distance, and Tanimoto similarity.

## When NOT to use

- The dataset is NIST23 — access is restricted by commercial licensing, so the metric harness cannot be assembled without the test fold
- Evaluating fingerprint-prediction quality itself rather than generated molecular structures — use fingerprint bit-level metrics instead of InChIKey/MCES/Tanimoto scoring

## Inputs

- Ranked candidate molecule lists (SMILES/SELFIES) per test spectrum from the aligned model's beam-search decoding (beam widths up to 100)
- Ground-truth molecule structures (InChIKey/SMILES) from the MassSpecGym and NPLIB1 test folds
- Predicted molecular formula per spectrum (for DiffMS-comparison re-ranking)

## Outputs

- Per-dataset metric table (Table-1 style, e.g. results_table.csv) with Top-1/Top-10 accuracy, Top-1/Top-10 MCES distance, and Top-1/Top-10 Tanimoto similarity for MS-Bart and baselines (Spec2Mol, MIST+Neuraldecipher, MIST+MSNovelist, MADGEN, DiffMS)
- Per-spectrum re-ranked candidate lists when using the DiffMS comparison protocol (100 samples, formula-distance re-ranking)

## How to apply

For each test spectrum, take the ranked candidate SELFIES/SMILES list and compare against the ground-truth structure: convert candidates to canonical structures, compute the full InChIKey and report Top-1/Top-10 exact-match accuracy. Compute Morgan fingerprints with RDKit (radius 2, 2048 bits) for candidate and reference molecules to obtain maximum Tanimoto similarity, and compute the minimum MCES distance. When comparing against DiffMS, sample 100 molecules per spectrum and re-rank candidates by formula distance (Eq. 6), breaking ties by model log-probability. Report MS-BART reference values to validate the harness: NPLIB1 Top-1 7.45% / MCES 9.66 / Tanimoto 0.44; MassSpecGym Top-1 1.07% / MCES 16.47 / Tanimoto 0.28.

## Related tools

- **RDKit** (Compute Morgan fingerprints (radius 2, 2048 bits) used for Tanimoto similarity scoring of candidate vs. reference molecules) — https://github.com/OpenDFM/MS-BART
- **MS-BART** (Generates the ranked candidate SELFIES structures per spectrum that are scored with the metric suite) — https://github.com/OpenDFM/MS-BART
- **MassSpecGym** (Benchmark test fold providing ground-truth structures and the evaluation script (with known issues to correct: test-path filename and NUM_BEAMS/NUM_BEAM variable mismatch))
- **NPLIB1** (Second benchmark test fold (GNPS subset curated for CANOPUS) on which the same metrics are computed)

## Examples

```
python evaluate.py --dataset MassSpecGym --model ./logs/aligned-model --num_beams 100 --metrics top1,top10,mces,tanimoto --output results_table.csv
```

## Evaluation signals

- NPLIB1 reference values reproduced: Top-1 accuracy 7.45%, MCES 9.66, Tanimoto 0.44
- MassSpecGym reference values reproduced: Top-1 accuracy 1.07%, MCES 16.47, Tanimoto 0.28
- Gold-fingerprint upper bounds matched: 73.50% (NPLIB1) and 47.56% (MassSpecGym) Top-1 accuracy
- All metrics for MS-Bart and baselines (Spec2Mol, MIST+Neuraldecipher, MIST+MSNovelist, MADGEN, DiffMS) compiled into a Table-1-style comparison saved as results_table.csv

## Limitations

- MCES distance computation is slow for large candidate sets; DiffMS comparison requires sampling 100 molecules per spectrum and re-ranking, adding inference cost
- Evaluation depends on correct use of the MassSpecGym eval script — known issues include a test-path filename error and a NUM_BEAMS/NUM_BEAM variable mismatch that must be patched
- Fingerprint-based Tanimoto scoring inherits the accuracy limits of MIST-predicted fingerprints, which significantly impact overall performance

## Evidence

- [other] Compute Top-1/Top-10 accuracy by full InChIKey exact match, minimum MCES distance, and maximum Tanimoto similarity using RDKit Morgan fingerprints (radius 2, 2048 bits): [evidence span withheld — anchor status: not_found]
- [other] DiffMS comparison re-ranking protocol: [evidence span withheld — anchor status: not_found]
- [other] Validation reference values: [evidence span withheld — anchor status: not_found]
- [methods] Benchmark scope: [evidence span withheld — anchor status: not_found]
- [readme] Known MSG eval script issues: [evidence span withheld — anchor status: not_found]
