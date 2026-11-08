---
name: isomer-candidate-ranking
description: Use when MS-FINDER formula assignment returns multiple isomeric candidate
  structures for HILIC MS/MS features and you have (or can predict) Retip retention
  times to exclude RT-inconsistent candidates.
when_to_use_negative:
- Chromatography is a reversed-phase or otherwise altered method for which no Retip
  model was trained, or the analytes (e.g., high-molecular-weight lipids like triacylglycerols
  or phosphatidylcholines) lie outside the training set's chemical landscape
- You need to discriminate closely eluting positional isomers in fast UPLC methods
  (e.g., 12 min runs), where 1 min prediction errors are too coarse
- Each formula already has a single, uniquely matched candidate structure confirmed
  by an authentic standard — RT filtering adds no information and can wrongly exclude
  true positives
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Retip
  role: Predicts HILIC retention times for candidate structures using the fit.keras
    machine-learning model; provides the RT values used for ±1 min filtering
  repo: https://github.com/olobion/Retip
- name: MS-FINDER
  role: Performs the isomer search that produces formula-to-candidate-structure matches
    and can ingest Retip RT models for candidate filtering (MS-FINDER 3.30)
- name: R
  role: Environment for running the Retip package and the per-formula filtering/aggregation
    script
- name: MS-DIAL
  role: Mass spectrometry software with which Retip RT prediction is integrated for
    metabolome-wide annotation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.9b05765
    title: 'Retip: Retention Time Prediction for Compound Annotation in Untargeted
      Metabolomics'
  source_content_hash: sha256:18c1008ab147e0f54ffae91ceda814602f93fa539cea3d46268222d27f1f85d2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/olobion/Retip
  tools:
  - MS-DIAL
  - MS-FINDER
  - R
  - Retip
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# isomer-candidate-ranking

## Summary

Ranks and prunes isomeric candidate structures from MS-FINDER compound identification by comparing Retip-predicted HILIC retention times against experimental RT within a ±1 min window, yielding large reductions (e.g., 68%) in candidate lists.

## When to use

Use when MS-FINDER formula assignment returns multiple isomeric candidate structures for HILIC MS/MS features and you have (or can predict) Retip retention times to exclude RT-inconsistent candidates.

## When NOT to use

- Chromatography is a reversed-phase or otherwise altered method for which no Retip model was trained, or the analytes (e.g., high-molecular-weight lipids like triacylglycerols or phosphatidylcholines) lie outside the training set's chemical landscape
- You need to discriminate closely eluting positional isomers in fast UPLC methods (e.g., 12 min runs), where 1 min prediction errors are too coarse
- Each formula already has a single, uniquely matched candidate structure confirmed by an authentic standard — RT filtering adds no information and can wrongly exclude true positives

## Inputs

- Deconvoluted HILIC plasma MS/MS spectra list with experimental retention times (e.g., 5,036 spectra from ST001154)
- MS-FINDER isomer-search output: elemental formulas with matched candidate structures (e.g., 2,002 formulas / 8,685 candidates)
- Retip HILIC predicted retention times for candidate structures (fit.keras model)

## Outputs

- Per-candidate exclusion flags (RT difference > ±1 min = unlikely)
- Summary table of reduction percentages (overall, single-candidate formulas, multi-isomer groups), e.g., retention_filter_results.csv
- Surviving RT-consistent candidate structures (e.g., 98 structures from the 609 seven-isomer candidates)

## How to apply

Load the deconvoluted MS/MS spectra list (e.g., 5,036 spectra from ST001154 HILIC plasma) together with elemental formulas and their matched candidate structures (e.g., 2,002 formulas / 8,685 candidates) into an R data frame. Predict retention times for each candidate using the Retip R package's fit.keras model (Keras had the lowest HILIC mean absolute error, 0.78 min). For each formula, mark candidates whose predicted RT differs from the experimental RT by more than ±1 min as excluded, since both HILIC and RPLC models had MAEs of about 1 min, which the authors used as thresholds for excluding false positives. Aggregate the reduction across three strata: all candidates (target ~68% average exclusion), formulas with only one candidate structure (e.g., 580 formulas, ~48% removed), and multi-isomer groups (e.g., 87 formulas with 7 isomers each; 84% of 609 candidates excluded, leaving 98 structures). Write the summary table (retention_filter_results.csv) and verify computed exclusion fractions against the reported values before interpreting surviving candidates as higher-confidence annotations.

## Related tools

- **Retip** (Predicts HILIC retention times for candidate structures using the fit.keras machine-learning model; provides the RT values used for ±1 min filtering) — https://github.com/olobion/Retip
- **MS-FINDER** (Performs the isomer search that produces formula-to-candidate-structure matches and can ingest Retip RT models for candidate filtering (MS-FINDER 3.30))
- **R** (Environment for running the Retip package and the per-formula filtering/aggregation script)
- **MS-DIAL** (Mass spectrometry software with which Retip RT prediction is integrated for metabolome-wide annotation)

## Examples

```
devtools::install_github('olobion/Retiplib'); devtools::install_github('olobion/Retip')
```

## Evaluation signals

- Overall excluded-candidate fraction across all formula groups is ~68% on average
- ~48% of candidates from single-candidate formulas (e.g., 580 formulas) are removed by the ±1 min RT window
- ~84% of candidates in seven-isomer formula groups (609 candidates) are excluded, leaving 98 surviving structures
- RT difference for every excluded candidate exceeds ±1 min and every retained candidate is within ±1 min of the experimental RT
- Summary table (retention_filter_results.csv) contains the three stratified reduction percentages and row counts reconcile with input formula/candidate counts

## Limitations

- A ±1 min window excluded about 25% of true positives, so RT filtering trades false positives for some loss of correct annotations
- None of the current models yield validation MAEs better than ±1 min in retention prediction, so molecular descriptors may not fully explain LC buffer, microenvironment, and adsorption/desorption interactions
- Prediction accuracy is limited by the number of compounds used for training and their chemical diversity; predictions outside the training chemical landscape (or for complex high-molecular-weight lipids such as triacylglycerols or phosphatidylcholines) may be more erroneous
- Predictions are not yet adequate for narrow retention time search windows; Retip RTs were not tested for adjustment to small changes in chromatographic methods

## Evidence
