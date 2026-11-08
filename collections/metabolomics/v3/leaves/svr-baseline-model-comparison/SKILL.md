---
name: svr-baseline-model-comparison
description: Use when training a new chemical shift prediction model and needing to
  benchmark it against a legacy SVR predictor per atom type (1H, 13C, 15N) via cross-validated
  RMSD.
when_to_use_negative:
- When evaluating the protein shift prediction model on the 61-protein ShiftX2 test
  dataset (separate evaluation path in Table S1 of the SI)
- When input chemical shifts come from paramagnetic proteins or non-standard residues
  outside the 1480-protein training cohort
edam_operation: http://edamontology.org/operation_0325
edam_topics:
- http://edamontology.org/topic_3420
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_1317
tools:
- name: NMRFx
  role: Hosts the FM chemical shift prediction models being benchmarked and the SVR
    baseline being compared
  repo: github.com/nanalysis/nmrfx
- name: Tribuo
  role: Implements stochastic gradient descent training of the FM models evaluated
    against the SVR baseline
- name: BMRB
  role: Source of experimental chemical shift assignments for the 371 RNA training
    molecules
- name: PDB
  role: Source of RNA structures from which structural features are computed for training
    records
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42004-025-01812-8
    title: NMR data processing, visualization, analysis and structure calculation
      with NMRFx
  source_content_hash: sha256:16c43ca30c15f21305e3f659563a35be236e1a3d1a13bd5e144c48f551bb33dd
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/nanalysis/nmrfx
  tools:
  - BMRB
  - NMRFx
  - PDB
  - Tribuo
  edam_operation: http://edamontology.org/operation_0325
  edam_topics:
  - http://edamontology.org/topic_3420
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_1317
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# svr-baseline-model-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the cross-validated RMSD comparison between NMRFx's FM-based RNA chemical shift predictors and the prior NMRViewJ SVR models on the 371-molecule BMRB/PDB RNA dataset, validating that FM handles more training data while staying performant.

## When to use

Use when training a new chemical shift prediction model and needing to benchmark it against a legacy SVR predictor per atom type (1H, 13C, 15N) via cross-validated RMSD.

## When NOT to use

- When evaluating the protein shift prediction model on the 61-protein ShiftX2 test dataset (separate evaluation path in Table S1 of the SI)
- When input chemical shifts come from paramagnetic proteins or non-standard residues outside the 1480-protein training cohort

## Inputs

- BMRB chemical shift assignments for 371 RNA molecules with matching PDB structures
- Structural features computed from PDB coordinates
- Trained SVR models (legacy NMRViewJ RNA shift predictors)
- Trained FM models per atom class (1H, 13C, 15N)

## Outputs

- Per-atom-type cross-validated RMSD for the FM model
- Per-atom-type cross-validated RMSD for the SVR baseline
- Side-by-side comparison table (e.g. rna_fm_crossvalidated_rmsd.csv) demonstrating FM-vs-SVR performance

## How to apply

Assemble the training set by pairing BMRB chemical shift assignments with structural features from matching PDB structures for the 371 RNA molecules used by NMRFx. Train both the factorization-machine model (via stochastic gradient descent in Tribuo) and the legacy SVR model on the same records, one per atom class. Run cross-validation over the dataset to obtain out-of-fold predictions for each atom type. For both model families, compute the cross-validated RMSD between predicted and experimentally assigned shifts per atom type and tabulate the results side by side. The comparison is judged successful if the FM model matches or exceeds the SVR baseline's cross-validated RMSD while accommodating the larger training corpus, consistent with the article's finding that the updated FM model 'robustly handles a significant increase in training data points while maintaining high performance'.

## Related tools

- **NMRFx** (Hosts the FM chemical shift prediction models being benchmarked and the SVR baseline being compared) — github.com/nanalysis/nmrfx
- **Tribuo** (Implements stochastic gradient descent training of the FM models evaluated against the SVR baseline)
- **BMRB** (Source of experimental chemical shift assignments for the 371 RNA training molecules)
- **PDB** (Source of RNA structures from which structural features are computed for training records)

## Examples

```
# Train FM and SVR per atom type, cross-validate, compare
python scripts/train_rna_shift_predictors.py --model fm svr --atoms 1H 13C 15N --dataset bmrb_rna_371 --cv 5 --out rna_fm_crossvalidated_rmsd.csv
```

## Evaluation signals

- Cross-validated RMSD per atom type (1H, 13C, 15N) is computed for both FM and SVR models on identical folds
- FM model's cross-validated RMSD matches the values reported in the SI evaluation table
- Comparison supports the finding that FM handles more training data while maintaining high performance
- Both model families are trained on the same 371-molecule dataset to ensure a fair comparison

## Limitations

- RNA model cross-validated RMSD evaluation table is only in the SI, not reproduced in the main text
- Performance on proteins or other atom classes beyond 1H/13C/15N RNA evaluation is not addressed here
- Legacy SVR comparison assumes the SVR baseline was trained on the same data distribution; differences in feature sets or preprocessing may confound the comparison

## Evidence
