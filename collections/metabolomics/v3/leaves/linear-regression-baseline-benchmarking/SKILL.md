---
name: linear-regression-baseline-benchmarking
description: Use when validating a learned predictor for an auxiliary molecular property
  (RT or CCS) by demonstrating it beats a trivial univariate regression baseline on
  identical data splits, e.g. reproducing Fig. 5-style parity plots.
when_to_use_negative:
- When benchmarking spectral intensity prediction against ICEBERG or CFM-ID — this
  skill covers only the auxiliary RT and CCS property tasks, not MS/MS spectra similarity
- When no retention time or CCS annotations exist for the compounds (e.g. NIST'17
  does not provide RT and CCS values), since neither the model nor the linear baselines
  can be trained or evaluated
- When a tuned baseline is required — hyperparameters were not specifically tuned
  for the RT and CCS task and the feature is still in prototype stage
edam_operation: http://edamontology.org/operation_2945
tools:
- name: FIORA
  role: Provides the RT and CCS prediction submodules (fiora-train, fiora-predict
    with --rt/--ccs) whose performance is benchmarked against the linear regressions
  repo: https://github.com/BAMeScience/fiora
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-57422-4
    title: 'FIORA: Local neighborhood-based prediction of compound mass spectra from
      single fragmentation events'
  source_content_hash: sha256:969c8ff91dcae55edd122f066b0bdb5cff8bb963776d876fd3f4af3c514defec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BAMeScience/fiora
  tools:
  - FIORA
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# Linear Regression Baseline Benchmarking

## Summary

Fits simple property-specific linear regression baselines (logP-based for retention time, precursor-m/z-based for CCS) on the same train/val/test splits used for the neural model, so that FIORA's RT and CCS submodules can be judged by R² deltas rather than absolute values.

## When to use

Use when validating a learned predictor for an auxiliary molecular property (RT or CCS) by demonstrating it beats a trivial univariate regression baseline on identical data splits, e.g. reproducing Fig. 5-style parity plots.

## When NOT to use

- When benchmarking spectral intensity prediction against ICEBERG or CFM-ID — this skill covers only the auxiliary RT and CCS property tasks, not MS/MS spectra similarity
- When no retention time or CCS annotations exist for the compounds (e.g. NIST'17 does not provide RT and CCS values), since neither the model nor the linear baselines can be trained or evaluated
- When a tuned baseline is required — hyperparameters were not specifically tuned for the RT and CCS task and the feature is still in prototype stage

## Inputs

- MS-DIAL spectral library in .msp format with RT and CCS annotations
- CASMI 2016 benchmark compounds
- FIORA model checkpoint (e.g. checkpoints/fiora.pt) and fiora-train training history (--history-out, . or .csv)
- Training/validation/test split assignments (80/10/10)

## Outputs

- R² values for FIORA and the logP-based RT baseline and m/z-based CCS baseline per split
- Predicted-vs-observed parity plots for RT and CCS annotated with R² (reproduction of Fig. 5)
- RT and CCS predictions for validation/test and CASMI 16 compounds from fiora-predict

## How to apply

Extract annotated compounds from the MS-DIAL .msp spectral library, forming a 409-compound RT subset and a 1346-compound CCS subset, and split them 80% training / 10% validation / 10% testing to match the FIORA splits. Train FIORA's RT and CCS submodules via fiora-train (200 epochs, ADAM optimizer, weighted MSE loss, plateau-reducing learning-rate scheduler on validation loss), then generate predictions with fiora-predict using --rt and --ccs on held-out and CASMI 2016 compounds. Fit the baselines on the same training splits: linear regression of RT against logP, and linear regression of CCS against precursor m/z. Compute R² for model vs. baseline on each split — expect RT R² ≈ 0.70 for FIORA, and CCS R² ≈ 0.93 vs. 0.9 on the test split and ≈ 0.86 vs. 0.80 on CASMI 16. Render predicted-vs-observed parity plots annotated with the R² values as the final figure; a valid application shows FIORA exceeding its baseline on every split.

## Related tools

- **FIORA** (Provides the RT and CCS prediction submodules (fiora-train, fiora-predict with --rt/--ccs) whose performance is benchmarked against the linear regressions) — https://github.com/BAMeScience/fiora

## Examples

```
fiora-predict -i casmi16_input.csv -o casmi16_predictions.mgf --rt --ccs
```

## Evaluation signals

- RT R² for FIORA on the test split is approximately 0.70
- CCS R² on the test split reaches ≈ 0.93 for FIORA vs. 0.9 for the precursor-m/z linear regression
- CCS R² on CASMI 16 compounds reaches ≈ 0.86 for FIORA vs. 0.80 for the baseline
- Both FIORA and the baselines are fitted/evaluated on identical 80/10/10 splits so R² values are directly comparable
- Parity plots show points distributed around the diagonal with annotated R² for each split

## Limitations

- Hyperparameters were not specifically tuned for the RT and CCS prediction task; the feature is still in prototype stage
- The linear baselines are weak by design (logP for RT, precursor m/z for CCS), so R² gains quantify added value but do not indicate performance ceilings
- NIST'17 MS/MS library does not provide RT and CCS values, restricting annotation-rich training data to other libraries such as MS-DIAL

## Evidence

- [discussion] Split rationale: "FIORA was trained on 80% of the molecular structures, with the remaining 20% split evenly between validation and testing."
- [discussion] Training protocol: "The training process consisted of 200 epochs using the ADAM optimizer, a weighted MSE loss and a scheduler that reduces the learning rate upon reaching a plateau in validation loss."
- [readme] README RT/CCS capability: "Using a graph neural network, FIORA models bond cleavages, fragment intensities, and estimates retention times (RT) and collision cross sections (CCS)."
- [readme] CLI invocation: "fiora-predict [-h] -i INPUT -o OUTPUT [--model MODEL] [--rt | --no-rt] [--ccs | --no-ccs] [--annotation | --no-annotation]"
