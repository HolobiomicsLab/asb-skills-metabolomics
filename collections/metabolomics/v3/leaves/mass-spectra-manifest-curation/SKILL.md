---
name: mass-spectra-manifest-curation
description: Use when preparing held-out EI-MS test sets from spectral libraries and
  you must prevent train/test compound overlap, remove deuterium-labeled compounds,
  or enforce the inference model's input limits (peak count, m/z range, SMILES length).
when_to_use_negative:
- The spectra are training or validation data, not a held-out test set — overlap removal
  against the training split is only needed for evaluation sets.
- The downstream model accepts arbitrary peak counts, m/z ranges, or SMILES lengths
  — applying SpecTUS-specific thresholds (300 peaks, m/z 500, 100-char SMILES) would
  unnecessarily exclude valid spectra (≈3% of NIST was dropped by these thresholds
  alone).
- You are evaluating on the CASMI 2017 MS/MS competition set, which was filtered with
  its own model-specific filters (243 → 93 spectra) and follows a different protocol.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: Jupyter Notebook
  role: 'Environment for the filtering pipeline: canonicalizing SMILES, de-leaking
    test sets against training splits, applying peak/m/z/SMILES-length thresholds,
    and writing the filtered manifests.'
- name: SpecTUS
  role: Source of the model-specific input thresholds (max 300 peaks, m/z ≤ 500, max
    SMILES length 100) that the test sets must satisfy for downstream inference and
    evaluation.
  repo: https://github.com/hejjack/SpecTUS
- name: hejjack/SpecTUS
  role: Provides the NIST splits, example data, and preprocessing notebooks referenced
    for the curation workflow.
  repo: https://github.com/hejjack/SpecTUS
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:e9f3f53dcf15a3d75d16c073868a082590f2b3fa42fe7af024b11f1877ecc65d
  source_papers:
  - doi: 10.48550/arxiv.2502.05114
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hejjack/SpecTUS
  tools:
  - Jupyter Notebook
  - SpecTUS
  - hejjack/SpecTUS
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# mass-spectra-manifest-curation

## Summary

Filter held-out mass-spectral test libraries (NIST 20, SWGDRUG, Cayman, MONA) by canonical-SMILES de-leaking against training splits, deuterium removal, and model-specific thresholds (max 300 peaks, m/z ≤ 500, SMILES ≤ 100 chars) to produce leak-free evaluation manifests.

## When to use

Use when preparing held-out EI-MS test sets from spectral libraries and you must prevent train/test compound overlap, remove deuterium-labeled compounds, or enforce the inference model's input limits (peak count, m/z range, SMILES length).

## When NOT to use

- The spectra are training or validation data, not a held-out test set — overlap removal against the training split is only needed for evaluation sets.
- The downstream model accepts arbitrary peak counts, m/z ranges, or SMILES lengths — applying SpecTUS-specific thresholds (300 peaks, m/z 500, 100-char SMILES) would unnecessarily exclude valid spectra (≈3% of NIST was dropped by these thresholds alone).
- You are evaluating on the CASMI 2017 MS/MS competition set, which was filtered with its own model-specific filters (243 → 93 spectra) and follows a different protocol.

## Inputs

- NIST 20 held-out test split spectra records (msp-style EI-MS peak lists plus compound SMILES)
- SWGDRUG, Cayman, and MONA test library spectra records
- NIST 20 training-split compound SMILES list (with the removed synthetic-set compound list)
- Model-specific input thresholds: max 300 peaks, max m/z 500, max SMILES length 100 characters

## Outputs

- filtered_test_manifests.csv — retained per-library spectra manifests
- Original vs. retained compound counts for each library (NIST 20, SWGDRUG, Cayman, MONA)

## How to apply

In a Jupyter notebook, load the spectra records from the four test libraries (NIST 20 held-out test split, SWGDRUG, Cayman, MONA) plus the NIST 20 training-split SMILES, including the removed synthetic-set compound list. Canonicalize every SMILES with stereochemical nuances removed, then delete any test compound whose canonical SMILES matches a compound in the experimental or synthetic training sets, removing all NIST 20 training overlaps. Drop any remaining compounds containing deuterium atoms, since deuterium was not represented in the training data. Finally apply the model-specific thresholds — maximum m/z value of 500, maximum of 300 peaks per spectrum, and maximum SMILES length of 100 characters — writing the retained per-library spectra manifests to filtered_test_manifests.csv with original-vs-retained counts per library.

## Related tools

- **Jupyter Notebook** (Environment for the filtering pipeline: canonicalizing SMILES, de-leaking test sets against training splits, applying peak/m/z/SMILES-length thresholds, and writing the filtered manifests.)
- **SpecTUS** (Source of the model-specific input thresholds (max 300 peaks, m/z ≤ 500, max SMILES length 100) that the test sets must satisfy for downstream inference and evaluation.) — https://github.com/hejjack/SpecTUS
- **hejjack/SpecTUS** (Provides the NIST splits, example data, and preprocessing notebooks referenced for the curation workflow.) — https://github.com/hejjack/SpecTUS

## Examples

```
jupyter notebook notebooks/data_preprocessing.ipynb  # de-leak NIST20/SWGDRUG/Cayman/MONA test sets vs. training splits, apply 300-peak/mz-500/SMILES-100 filters
```

## Evaluation signals

- Retained counts must reproduce the reported values: NIST test 29,218 → 28,267; SWGDRUG 3,589 → 1,640; Cayman 2,262 → 469; MONA 18,914 → 5,015.
- No canonical SMILES in the filtered test manifests appears in the experimental or synthetic training-set SMILES lists (zero-overlap check).
- Every retained compound contains no deuterium atoms (deuterium-free check on the atom set).
- Every retained spectrum satisfies max m/z ≤ 500, ≤ 300 peaks, and SMILES length ≤ 100 characters.
- filtered_test_manifests.csv exists with one manifest per library and per-library original/retained counts matching the report.

## Limitations

- Applying the model-specific thresholds alone excludes approximately 3% of the NIST dataset, so some spectra are discarded purely for exceeding inference-model limits.
- Canonical SMILES comparison strips stereochemical nuances, so stereochemical variants present in a test library but absent from training are treated as potential overlap.
- The method relies on complete training-set compound lists (including the removed synthetic-set list); missing lists would allow residual leakage.
- SpecTUS's demonstrated sensitivity to input-spectrum quality means curation alone cannot guarantee comparable performance on lower-quality libraries like Cayman or MONA.

## Evidence
