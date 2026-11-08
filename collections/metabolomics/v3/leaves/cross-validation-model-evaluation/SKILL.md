---
name: cross-validation-model-evaluation
description: Use when comparing versions of an in silico MS/MS spectral prediction
  model on a shared experimental dataset and you need fold-matched, per-energy-level
  metrics (Dice, Precision, Recall, Dot Product) with significance testing.
when_to_use_negative:
- The task is compound identification (MS2C) benchmarking rather than spectral-prediction
  accuracy comparison — the CASMI 2016 identification evaluation uses a different
  scoring setup.
- Spectra come from non-QTOF instruments, which CFM-ID does not currently handle.
- Only a single train/test split is needed and fold-matched variance/significance
  estimation is not required — a simple holdout evaluation suffices.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: CFM-ID 4.0
  role: Model version under evaluation; predicts ESI-MS/MS spectra using connectivity-matrix
    topology features
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
- name: CFM-ID 2.0
  role: Machine-learning baseline version for the improvement comparison
- name: CFM-ID 3.0
  role: Rule-extended baseline version for the improvement comparison
- name: Docker
  role: Deployment mechanism to run CFM-ID consistently across folds via the wishartlab/cfmid
    image
  repo: https://hub.docker.com/r/wishartlab/cfmid
- name: Metlin
  role: Source of the experimental QTOF ESI-MS/MS training/testing spectra used in
    the cross-validation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c01465
    title: 'CFM-ID 4.0: More Accurate ESI-MS/MS Spectral Prediction and Compound Identification'
  source_content_hash: sha256:9f4839f788f73738842f7912d554738d2f41c292f9f7f785e10195e689f0578a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CFM-ID 2.0
  - CFM-ID 3.0
  - CFM-ID 4.0
  - Docker
  - Metlin
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# cross-validation-model-evaluation

## Summary

Benchmark protocol for comparing spectral-prediction model versions (e.g., CFM-ID 4.0 vs 2.0/3.0) under a matched 10-fold cross-validation framework on Metlin ESI-MS/MS data, scoring Dice, Precision, Recall, and Stein's Dot Product per energy level and ionization mode.

## When to use

Use when comparing versions of an in silico MS/MS spectral prediction model on a shared experimental dataset and you need fold-matched, per-energy-level metrics (Dice, Precision, Recall, Dot Product) with significance testing.

## When NOT to use

- The task is compound identification (MS2C) benchmarking rather than spectral-prediction accuracy comparison — the CASMI 2016 identification evaluation uses a different scoring setup.
- Spectra come from non-QTOF instruments, which CFM-ID does not currently handle.
- Only a single train/test split is needed and fold-matched variance/significance estimation is not required — a simple holdout evaluation suffices.

## Inputs

- Metlin 2015 [M+H]+ dataset (4473 spectra, 1491 non-peptide metabolites, mass ≤ 1000 Da, QTOF ESI-MS/MS at 10/20/40 eV)
- Metlin 2015 [M-H]− dataset (2928 spectra, 976 metabolites, same restrictions)
- Configured model versions (CFM-ID 4.0 via wishartlab/cfmid Docker image; CFM-ID 2.0/3.0 baselines) with matched hyperparameters

## Outputs

- Per-energy-level and averaged metrics table (Dice, Precision, Recall, re-weighted Stein's Dot Product) e.g., written to table1_metrics.csv
- Percent improvement of CFM-ID 4.0 over CFM-ID 2.0/3.0 per ionization mode with two-sided t-test p-values

## How to apply

Partition the dataset (e.g., Metlin 2015 [M+H]+: 4473 spectra / 1491 metabolites; [M-H]−: 2928 spectra / 976 metabolites, non-peptide, mass ≤ 1000 Da, QTOF ESI-MS/MS at 10, 20, and 40 eV) into 10 folds and run all model versions under the same fold assignments with matched hyperparameters: fragmentation-graph depth 2, maximum ring breaks 2, model depth 2, and a fully connected neural network with two hidden layers of 128 nodes each. For the newer model (CFM-ID 4.0), enable connectivity-matrix features (10-atom cap on the nearest-N neighborhood) alongside Break Atom Pair, BrokenOrigBondType, Gasteiger Charges, and Hydrogen Movement features. Predict spectra for each held-out fold at each collision energy, then post-process by iteratively keeping the most intense peaks until the summed intensity reaches 80% of total or 30 peaks are selected. Score each predicted spectrum against the experimental spectrum with peak matches defined as m/z difference < 0.01 Da or 10 ppm (whichever comes first), computing Dice coefficient, Precision, Recall, and re-weighted Stein's Dot Product per energy level and ionization mode. Average metrics across energy levels and folds, compute percent improvement of the new version over the baseline, and test significance with two-sided t-tests.

## Related tools

- **CFM-ID 4.0** (Model version under evaluation; predicts ESI-MS/MS spectra using connectivity-matrix topology features) — https://bitbucket.org/wishartlab/cfm-id-code/
- **CFM-ID 2.0** (Machine-learning baseline version for the improvement comparison)
- **CFM-ID 3.0** (Rule-extended baseline version for the improvement comparison)
- **Docker** (Deployment mechanism to run CFM-ID consistently across folds via the wishartlab/cfmid image) — https://hub.docker.com/r/wishartlab/cfmid
- **Metlin** (Source of the experimental QTOF ESI-MS/MS training/testing spectra used in the cross-validation)

## Examples

```
docker run wishartlab/cfmid cfm-predict --input fold_test.smi --depth 2 --ringbreak 2 --energy_levels 10,20,40
```

## Evaluation signals

- Reproduced Dice improvements of CFM-ID 4.0 over 2.0/3.0 must be positive on both ionization modes and directionally consistent with reported values (≈+31.8% Dice / +26.7% Dot Product on [M+H]+; ≈+8% Dice / +20.6% Dot Product on [M-H]−).
- Two-sided t-test p-values for the Dice improvement should be highly significant (reported 1E-43 on [M+H]+ and 3E-29 on [M-H]−).
- All model versions must share identical fold assignments and hyperparameters (graph depth 2, ring breaks 2, model depth 2, two hidden layers of 128 nodes) so differences are attributable to model version only.
- Post-processed predicted spectra must contain at most 30 peaks with cumulative intensity ≈80% of the total.
- Peak matching must apply the < 0.01 Da or 10 ppm (whichever first) tolerance consistently across all metric computations.

## Limitations

- CFM-ID does not currently handle MS spectra collected using instruments other than QTOF instruments.
- Deeper fragmentation graphs improve accuracy but introduce extra computational complexity, and runtime still has room for improvement.
- Pure in silico approaches could only correctly identify about 25% of testing compounds without metadata help, so spectral-prediction metrics should not be conflated with identification success.

## Evidence
