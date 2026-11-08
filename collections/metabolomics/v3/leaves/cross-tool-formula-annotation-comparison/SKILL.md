---
name: cross-tool-formula-annotation-comparison
description: Use when evaluating whether a data-driven formula prediction model (e.g.,
  a formula transformer trained on MS/MS spectra) outperforms a classical tree-based
  competitor on the same spectra, and you need accuracy, coverage, and concordance
  metrics on identical inputs.
when_to_use_negative:
- 'Input spectra are negative-mode data: MIST-CF models were trained only on positive-mode
  data, so this comparison is invalid for negative ionization.'
- The goal is Prospective structure or fingerprint prediction (e.g., CSI:FingerID
  or MIST fingerprint scoring) rather than de novo chemical formula annotation accuracy
  comparison.
- No NIST20 license is available and only the public NPLIB1/CANOPUS-trained model
  is used, but the reference claims Orbitrap or higher-resolution performance parity
  — the public model is explicitly less performant than the NIST20-trained model.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MIST-CF
  role: Data-driven neural network model for inferring chemical formulae from MS/MS
    spectra, trained using an energy-based modeling framework; scored on the shared
    test set to obtain the learned-model side of the comparison.
  repo: https://github.com/samgoldman97/mist-cf
- name: SIRIUS
  role: Baseline fragmentation-tree-based formula annotation tool; run headless at
    version 5.6.3 with 300 s tree/compound timeouts to produce the classical competitor's
    predictions on the identical spectra.
  repo: https://github.com/boecker-lab/sirius/releases/download/
- name: FastFilter
  role: Lightweight neural network formula filtering model that pre-ranks SIRIUS decomp
    candidates down to the top-256 per precursor before MIST-CF scoring.
  repo: https://github.com/samgoldman97/mist-cf
- name: Chemical Formula Transformer
  role: Underlying architecture of MIST-CF that replaces SIRIUS fragmentation tree
    kernels during the annotation step.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jcim.3c01082
    title: 'MIST-CF: Chemical Formula Inference from Tandem Mass Spectra'
  source_content_hash: sha256:e06edd13a06f23e2fd8cf8b5a9448b2ab46d085b3e5756245723ca3ba6f09904
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/samgoldman97/mist-cf
  tools:
  - Chemical Formula Transformer
  - FastFilter
  - MIST-CF
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# cross-tool-formula-annotation-comparison

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Benchmark a learned de novo chemical formula annotation model (MIST-CF, 10 ppm candidate masses with FastFilter top-256 down-selection) head-to-head against a fragmentation-tree baseline (SIRIUS 5.6.3, 300 s tree/compound timeouts) on a shared test set such as NPLIB1, reporting top-1 formula accuracy, joint formula+adduct accuracy, failure rate, and per-spectrum concordance.

## When to use

Use when evaluating whether a data-driven formula prediction model (e.g., a formula transformer trained on MS/MS spectra) outperforms a classical tree-based competitor on the same spectra, and you need accuracy, coverage, and concordance metrics on identical inputs.

## When NOT to use

- Input spectra are negative-mode data: MIST-CF models were trained only on positive-mode data, so this comparison is invalid for negative ionization.
- The goal is Prospective structure or fingerprint prediction (e.g., CSI:FingerID or MIST fingerprint scoring) rather than de novo chemical formula annotation accuracy comparison.
- No NIST20 license is available and only the public NPLIB1/CANOPUS-trained model is used, but the reference claims Orbitrap or higher-resolution performance parity — the public model is explicitly less performant than the NIST20-trained model.

## Inputs

- Preprocessed NPLIB1 test spectra (canopus_train directory)
- Trained MIST-CF model checkpoint
- Trained FastFilter model checkpoint
- SIRIUS 5.6.3 headless binary with default fragmentation-tree configuration
- Reference formula/adduct ground truth for each test spectrum

## Outputs

- Per-spectrum top-1 formula predictions from MIST-CF
- Per-spectrum SIRIUS 5.6.3 predictions (or explicit failure records for the ~10.77% unresolved spectra)
- results_table.csv containing top-1 formula accuracy, joint formula+adduct accuracy, SIRIUS failure rate, and MIST-CF-correct-where-SIRIUS-fails coverage for both tools
- Flagged discrepancies if reproduced metrics diverge from the paper's reported values

## How to apply

Load the preprocessed NPLIB1 test spectra (canopus_train directory) through the MIST-CF preprocessing pipeline, enumerate formula candidates with SIRIUS decomp dynamic programming using a 10 ppm mass window, and down-select to the top-256 candidates per precursor with the FastFilter model (src/mist_cf/fast_form_score/predict.py). Score the surviving candidates with the trained MIST-CF model (src/mist_cf/mist_cf_score/predict.py) and record the top-1 formula per spectrum; run SIRIUS 5.6.3 headless on the identical spectra with its default fragmentation-tree scoring configuration, imposing 300 s tree and compound timeouts. Evaluate both outputs with analysis/evaluate_pred.py, computing top-1 formula accuracy, joint formula+adduct accuracy, SIRIUS failure rate, and per-spectrum concordance (including the fraction of spectra MIST-CF gets right where SIRIUS fails). Aggregate the metrics into a results_table.csv and verify the reproduced numbers against the paper's reported comparison (MIST-CF ~71% top-1 / 80% joint accuracy vs SIRIUS 48%, SIRIUS failure rate ~10.77%), flagging any divergence directly in the table.

## Related tools

- **MIST-CF** (Data-driven neural network model for inferring chemical formulae from MS/MS spectra, trained using an energy-based modeling framework; scored on the shared test set to obtain the learned-model side of the comparison.) — https://github.com/samgoldman97/mist-cf
- **SIRIUS** (Baseline fragmentation-tree-based formula annotation tool; run headless at version 5.6.3 with 300 s tree/compound timeouts to produce the classical competitor's predictions on the identical spectra.) — https://github.com/boecker-lab/sirius/releases/download/
- **FastFilter** (Lightweight neural network formula filtering model that pre-ranks SIRIUS decomp candidates down to the top-256 per precursor before MIST-CF scoring.) — https://github.com/samgoldman97/mist-cf
- **Chemical Formula Transformer** (Underlying architecture of MIST-CF that replaces SIRIUS fragmentation tree kernels during the annotation step.)

## Examples

```
. run_scripts/benchmarking/eval_models.py && python analysis/evaluate_pred.py --input results_table.csv
```

## Evaluation signals

- Reproduced MIST-CF top-1 formula accuracy on NPLIB1 should approximate 71% and joint formula+adduct accuracy approximately 80%; any divergence beyond tolerance must be flagged in results_table.csv.
- Reproduced SIRIUS 5.6.3 top-1 formula accuracy should approximate 48% on the same NPLIB1 test set.
- SIRIUS failure rate (spectra where SIRIUS 5.6.3 cannot produce a prediction within the 300 s timeouts) should approximate 10.77%.
- Concordance analysis should show MIST-CF correctly predicting roughly 36% of the spectra on which SIRIUS fails outright.
- Per-spectrum candidate counts should not exceed 256 after FastFilter down-selection, confirming the pre-ranking step executed correctly before MIST-CF scoring.

## Limitations

- MIST-CF models used here were trained only on positive-mode data; negative-mode formula annotation is unsupported.
- MIST-CF scores are not integrated with MS1 isotopic scores, and adduct switching in MS2 subformula assignment is not considered.
- The pipeline still relies on SIRIUS's algorithmic decomposition of exact masses into formula candidates due to its fast implementation, so a full SIRIUS-free workflow is not evaluated.
- The public quickstart model (NPLIB1/CANOPUS-trained) may be less performant than the NIST20-trained model, particularly for Orbitrap or higher-resolution data; NIST20-trained model download links are only available upon reasonable request to users with a NIST license.

## Evidence

- [other] Reproduce the SIRIUS head-to-head comparison on the NPLIB1 test set: [evidence span withheld — anchor status: not_found]
- [methods] Pipeline steps: [evidence span withheld — anchor status: not_found]
- [methods] Evaluation metric definitions: [evidence span withheld — anchor status: not_found]
