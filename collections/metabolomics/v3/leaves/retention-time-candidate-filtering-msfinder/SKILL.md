---
name: retention-time-candidate-filtering-msfinder
description: Use when MS-FINDER compound identification returns many isomeric candidate
  structures per formula and you have predicted RTs (Retip) plus experimental RTs
  to exclude candidates that are chromatographically implausible.
when_to_use_negative:
- Predictions outside the chemical landscape of the training set (e.g., high molecular
  weight lipids such as triacylglycerols or phosphatidylcholines, which were largely
  excluded from training) — errors may be larger
- When a narrow RT search window is needed to discriminate closely eluting positional
  isomers in typical ~12 min UPLC methods — 1 min prediction errors preclude this
- 'When the goal requires near-zero false-positive exclusion: ±1 min windows excluded
  about 25% of true positives'
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: Retip
  role: R package providing predicted retention times (fit.keras model) used as the
    ±1 min filter for candidate structures
  repo: https://github.com/olobion/Retip
- name: MS-FINDER 3.30
  role: Compound identification software whose isomer search output (formulas and
    candidate structures) is filtered by predicted RT
- name: R
  role: Environment for loading the spectra/candidate tables, applying the ±1 min
    filter, and computing exclusion percentages
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.9b05765
    title: 'Retip: Retention Time Prediction for Compound Annotation in Untargeted
      Metabolomics'
  source_content_hash: sha256:50cc20d3fc00b1cb00613dd8059e03ffcfeecc03bc4984780633f8ac819583b4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/olobion/Retip
  tools:
  - MS-FINDER 3.30
  - R
  - Retip
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# retention-time-candidate-filtering-msfinder

## Summary

A skill for reducing MS-FINDER isomer-search candidate structures using Retip-predicted retention times: candidates whose predicted RT falls outside a ±1 min window of the experimental RT are excluded as false positives, yielding large candidate-list reductions (68% on average) in untargeted metabolomics compound identification.

## When to use

Use when MS-FINDER compound identification returns many isomeric candidate structures per formula and you have predicted RTs (Retip) plus experimental RTs to exclude candidates that are chromatographically implausible.

## When NOT to use

- Predictions outside the chemical landscape of the training set (e.g., high molecular weight lipids such as triacylglycerols or phosphatidylcholines, which were largely excluded from training) — errors may be larger
- When a narrow RT search window is needed to discriminate closely eluting positional isomers in typical ~12 min UPLC methods — 1 min prediction errors preclude this
- When the goal requires near-zero false-positive exclusion: ±1 min windows excluded about 25% of true positives

## Inputs

- Deconvoluted MS/MS spectra list from HILIC plasma dataset (e.g., ST001154; 5,036 spectra)
- MS-FINDER formula and candidate-structure table (2,002 formulas, 8,685 candidate structures)
- Retip HILIC predicted retention times for candidate structures (fit.keras model in the Retip R package)
- Experimental retention times per feature

## Outputs

- Per-candidate keep/exclude annotations based on the ±1 min RT window
- Aggregated summary table of reduction percentages (retention_filter_results.csv): overall (~68%), single-candidate (~48%), seven-isomer groups (~84%)

## How to apply

Load the deconvoluted MS/MS spectra list (e.g., 5,036 deconvoluted HILIC plasma spectra from ST001154) and the MS-FINDER formula/candidate-structure table (2,002 formulas, 8,685 candidates) into an R data frame. Predict RTs for each candidate structure with the Retip R package (fit.keras model; Keras had the best performance with MAE 0.78 min for HILIC). Mark candidates whose |predicted RT − experimental RT| > 1 min as excluded — the 1 min threshold is grounded in the models' mean absolute error of about 1 min for HILIC/RPLC. Compute overall exclusion (target ≈68% average), then subset single-candidate formulas (580 formulas, target ≈48% removed) and seven-isomer formula groups (87 formulas / 609 candidates, target ≈84% excluded leaving 98 structures). Aggregate the three percentages into a summary table (retention_filter_results.csv).

## Related tools

- **Retip** (R package providing predicted retention times (fit.keras model) used as the ±1 min filter for candidate structures) — https://github.com/olobion/Retip
- **MS-FINDER 3.30** (Compound identification software whose isomer search output (formulas and candidate structures) is filtered by predicted RT)
- **R** (Environment for loading the spectra/candidate tables, applying the ±1 min filter, and computing exclusion percentages)

## Examples

```
devtools::install_github('olobion/Retip'); keep <- subset(candidates, abs(rt_pred - rt_exp) <= 1); mean(excluded_count / total_candidates)
```

## Evaluation signals

- Overall candidate-structure exclusion after the ±1 min filter matches the reported ~68% average reduction
- Single-candidate formula subset (580 formulas) shows ~48% removal due to large RT differences
- Seven-isomer formula subset (87 formulas, 609 candidates) is reduced to 98 surviving structures (~84% excluded)
- Retip Keras model prediction quality is consistent with MAE ≈0.78 min for HILIC before filtering
- Expected false-positive cost: roughly 25% of true positives fall outside the ±1 min window, so surviving-candidate counts should not exceed true values

## Limitations

- 1 min prediction errors preclude discriminating closely eluting compounds in typical 12 min UPLC methods
- Prediction accuracy is limited by the number and chemical diversity of training compounds; models had MAEs no better than ±1 min, and molecular descriptors may not fully capture LC buffer/microenvironment interactions
- Retention time predictions could not be tested for adjustment to small changes in chromatographic methods; results may be more erroneous outside the training chemical landscape

## Evidence
