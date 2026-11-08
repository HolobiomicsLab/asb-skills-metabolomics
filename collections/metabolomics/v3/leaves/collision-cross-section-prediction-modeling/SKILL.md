---
name: collision-cross-section-prediction-modeling
description: Use when a spectral library (e.g. MS-DIAL .msp) contains RT or CCS annotations
  and you need structure-based predicted RT/CCS values plus quantified prediction
  quality (R²) against m/z- or logP-based baselines.
when_to_use_negative:
- 'NIST''17 MS/MS library data: it does not provide RT and CCS values of the measured
  compounds, so it cannot supply training labels for this task'
- Predicting MS/MS peak intensities or fragment structures rather than RT/CCS — the
  dedicated spectral-prediction skill covers that
- 'Tasks requiring production-grade RT/CCS accuracy: hyperparameters were not tuned
  for this task and the feature is explicitly prototype stage'
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: FIORA
  role: Graph neural network whose RT and CCS submodules are trained with fiora-train
    and applied with fiora-predict --rt --ccs
  repo: https://github.com/BAMeScience/fiora
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-57422-4
    title: 'FIORA: Local neighborhood-based prediction of compound mass spectra from
      single fragmentation events'
  source_content_hash: sha256:3743731d2c4e239b4a2264ea72a11d9ff50a0257ebdf94d83c2dd5f053dc25f0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BAMeScience/fiora
  tools:
  - FIORA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# collision-cross-section-prediction-modeling

## Summary

Train and evaluate FIORA graph-neural-network submodules that predict retention time (RT) and collision cross section (CCS) from molecular structure, benchmarking them against simple linear regressions on logP (RT) and precursor m/z (CCS) to reproduce Fig. 5 parity plots with R² values.

## When to use

Use when a spectral library (e.g. MS-DIAL .msp) contains RT or CCS annotations and you need structure-based predicted RT/CCS values plus quantified prediction quality (R²) against m/z- or logP-based baselines.

## When NOT to use

- NIST'17 MS/MS library data: it does not provide RT and CCS values of the measured compounds, so it cannot supply training labels for this task
- Predicting MS/MS peak intensities or fragment structures rather than RT/CCS — the dedicated spectral-prediction skill covers that
- Tasks requiring production-grade RT/CCS accuracy: hyperparameters were not tuned for this task and the feature is explicitly prototype stage

## Inputs

- MS-DIAL spectral library in .msp format with RT and CCS annotations (409-compound RT subset; 1346-compound CCS subset)
- CASMI 2016 benchmark compounds
- CSV query files with columns 'Name', 'SMILES', 'Precursor_type', 'CE', 'Instrument_type'

## Outputs

- Trained FIORA RT/CCS model checkpoint (e.g. checkpoints/fiora.pt)
- Per-epoch training history file (. or .csv) via --history-out
- RT and CCS predictions for validation/test and CASMI 16 compounds
- R² values per split and predicted-vs-observed parity plots reproducing Fig. 5

## How to apply

Extract compounds with RT and CCS annotations from the MS-DIAL spectral library in .msp format, forming the 409-compound RT subset and 1346-compound CCS subset, and split 80% for training with the remaining 20% split evenly between validation and testing. Train the FIORA RT and CCS submodules with the fiora-train CLI for 200 epochs using the ADAM optimizer, a weighted MSE loss, and a plateau-reducing learning-rate scheduler on validation loss, saving the checkpoint (e.g. checkpoints/fiora.pt) and optionally per-epoch history via --history-out. Run fiora-predict with --rt and --ccs enabled to predict values for validation/test compounds and CASMI 2016 compounds. Fit baselines on the same training splits: a logP-based linear regression for RT and a precursor-m/z-based linear regression for CCS. Compute R² on each split, targeting CCS R² ≈ 0.93 on the test split and ≈ 0.86 on CASMI 16 (FIORA vs 0.90 and 0.80 for the m/z baseline), then render predicted-vs-observed parity plots annotated with R² to reproduce Fig. 5.

## Related tools

- **FIORA** (Graph neural network whose RT and CCS submodules are trained with fiora-train and applied with fiora-predict --rt --ccs) — https://github.com/BAMeScience/fiora

## Examples

```
fiora-predict -i examples/example_input.csv -o predictions.mgf --rt --ccs
```

## Evaluation signals

- CCS R² ≈ 0.93 on the test split (vs 0.90 for the precursor-m/z linear regression baseline)
- CCS R² ≈ 0.86 for CASMI 16 compounds (vs 0.80 for the baseline)
- RT R² ≈ 0.70 on the test split (vs the logP-based linear regression)
- Correct split structure: 80% training, remaining 20% evenly split between validation and testing
- Parity plot points fall near the diagonal with annotated R² values matching Fig. 5

## Limitations

- Hyperparameters were not specifically tuned for the RT/CCS task; the feature is still in its prototype stage
- NIST'17 MS/MS library does not provide RT and CCS values, restricting training labels to other sources such as the MS-DIAL library
- Exact model specification appears only in the Supplementary Information

## Evidence

- [discussion] training procedure: "The training process consisted of 200 epochs using the ADAM optimizer, a weighted MSE loss and a scheduler that reduces the learning rate upon reaching a plateau in validation loss."
- [abstract] feature scope: "also facilitates the prediction of additional features, such as retention time and collision cross section"
- [discussion] missing RT/CCS in NIST: "The NIST’17 MS/MS library does not provide information about RT and CCS values of the measured compounds."
- [discussion] prototype-stage caveat: "Hyperparameters were not specifically tuned for this task"
- [readme] training CLI: "To (re)train a new FIORA model, use `fiora-train`."
