---
name: mass-spectrometry-library-matching
description: Use when untargeted metabolomics MS/MS peak annotations yield too many
  candidate structures per formula (isomers), and predicted retention times from an
  HILIC or RPLC Retip model can be used to exclude unlikely candidates in MS-FINDER
  compound identification.
when_to_use_negative:
- Predicting RTs for chemical classes outside the training set's chemical landscape
  — the models excluded complex, high molecular weight lipids such as triacylglycerols
  or phosphatidylcholines, so predictions there may be more erroneous
- Attempting to discriminate closely eluting compounds in typical 12 min UPLC methods
  — 1 min prediction errors are not adequate for narrow retention time search windows
- Adjusting predicted retention times for small changes in the chromatographic method
  — this was beyond the scope of the study and is unvalidated
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Retip
  role: R package providing machine-learning RT prediction (fit.keras model, MAE 0.78
    min HILIC) used to score and filter candidate structures
  repo: https://github.com/olobion/Retip
- name: MS-FINDER 3.30
  role: Performs the isomer/candidate structure search whose results are filtered
    by predicted retention times; Retip predictions are integrated into it
- name: R
  role: Environment for the Retip package and the filtering/subsetting workflow (single-candidate
    and seven-isomer formula groups)
  repo: https://github.com/olobion/Retip
- name: MS-DIAL
  role: Mass spectrometry software into which Retip RT predictions are integrated
    for annotation workflows
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.9b05765
    title: 'Retip: Retention Time Prediction for Compound Annotation in Untargeted
      Metabolomics'
  source_content_hash: sha256:7e25355f02aaebe3b1d7701f655382fdf0fe46c771d3c56bb769e0218c434305
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/olobion/Retip
  tools:
  - MS-DIAL
  - MS-FINDER 3.30
  - R
  - Retip
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# mass-spectrometry-library-matching

## Summary

Applies machine-learning-predicted retention times (Retip) as an orthogonal filter on MS/MS spectral library and in silico candidate structures in MS-FINDER, removing candidates whose predicted RT deviates from the experimental RT by more than a ±1 min window to cut false positive annotations.

## When to use

Use when untargeted metabolomics MS/MS peak annotations yield too many candidate structures per formula (isomers), and predicted retention times from an HILIC or RPLC Retip model can be used to exclude unlikely candidates in MS-FINDER compound identification.

## When NOT to use

- Predicting RTs for chemical classes outside the training set's chemical landscape — the models excluded complex, high molecular weight lipids such as triacylglycerols or phosphatidylcholines, so predictions there may be more erroneous
- Attempting to discriminate closely eluting compounds in typical 12 min UPLC methods — 1 min prediction errors are not adequate for narrow retention time search windows
- Adjusting predicted retention times for small changes in the chromatographic method — this was beyond the scope of the study and is unvalidated

## Inputs

- List of deconvoluted MS/MS spectra with experimental retention times (e.g., 5,036 spectra from a HILIC plasma dataset such as ST001154)
- MS-FINDER compound identification output: elemental formulas with matched candidate structures (2,002 formulas, 8,685 candidates)
- Retip predicted retention times for candidate structures (fit.keras HILIC or RPLC model)

## Outputs

- Filtered candidate structure list with excluded candidates flagged by RT deviation > ±1 min
- Exclusion percentages at overall, single-candidate-formula, and isomer-group levels
- Summary table (e.g., retention_filter_results.csv) reporting the three reduction percentages

## How to apply

Load deconvoluted MS/MS spectra with experimental retention times (e.g., 5,036 deconvoluted HILIC plasma MS/MS spectra from ST001154) together with the MS-FINDER output of elemental formulas (2,002) and matched candidate structures (8,685). Predict RTs for each candidate structure using the Retip fit.keras model, which achieved a mean absolute error of 0.78 min for HILIC and 0.57 min for RPLC. Exclude any candidate whose |predicted RT − experimental RT| exceeds ±1 min, the threshold justified by the models' ~1 min MAE and chosen for excluding false positives. Compute exclusion statistics at three levels: overall percentage of candidates removed (expect ~68%), percentage of single-candidate formulas removed (expect ~48% of 580 single-candidate formulas), and percentage removed from multi-isomer formula groups (expect ~84% of 609 candidates across 87 seven-isomer formulas, leaving 98 surviving structures). Aggregate the three reduction percentages into a summary table (e.g., retention_filter_results.csv).

## Related tools

- **Retip** (R package providing machine-learning RT prediction (fit.keras model, MAE 0.78 min HILIC) used to score and filter candidate structures) — https://github.com/olobion/Retip
- **MS-FINDER 3.30** (Performs the isomer/candidate structure search whose results are filtered by predicted retention times; Retip predictions are integrated into it)
- **R** (Environment for the Retip package and the filtering/subsetting workflow (single-candidate and seven-isomer formula groups)) — https://github.com/olobion/Retip
- **MS-DIAL** (Mass spectrometry software into which Retip RT predictions are integrated for annotation workflows)

## Examples

```
devtools::install_github('olobion/Retip')
```

## Evaluation signals

- Overall excluded candidate fraction matches the reported 68% average reduction when searching all isomers in MS-FINDER
- About 48% of the 580 single-candidate formulas are removed as unlikely due to large RT differences
- About 84% of the 609 candidates in 87 seven-isomer formula groups are excluded, leaving 98 surviving structures
- Model quality check: Keras test-set MAE ≈ 0.78 min for HILIC (0.57 min RPLC), with small error differences between training, test, and validation sets
- Sanity bound: the ±1 min window should exclude roughly 25% of true positives, so retention of some true positives is expected and 100% exclusion indicates a broken prediction or RT alignment

## Limitations

- 1 min windows excluded about 25% of true positives, so the filter trades sensitivity for annotation confidence
- Prediction accuracy is limited by the number of compounds used for training and their chemical diversity; chemicals not well represented by the training models may be mispredicted
- No current models yield validation MAEs better than ±1 min, and molecular descriptors seem insufficient to fully explain LC buffer, microenvironment, and adsorption/desorption interactions
- Predictions are not yet adequate for narrow retention time search windows but are useful to disclose likely in-source fragmentation MS/MS spectra and to rank positional isomers

## Evidence
