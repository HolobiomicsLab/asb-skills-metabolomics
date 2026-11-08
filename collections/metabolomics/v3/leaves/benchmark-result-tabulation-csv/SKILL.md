---
name: benchmark-result-tabulation-csv
description: Use when a study reports surrogate-model performance (ROC AUC, LOBOV
  validation AUC) and Platt-calibration quality (ECE/MCE/logLoss deltas) that must
  be reproduced as a structured, machine-readable results table.
when_to_use_negative:
- Input is raw 1H-NMR metabolite data needing QC and score projection (use QCprep_surrogates/calculate_surrogate_scores
  first, not tabulation)
- The goal is interactive exploration of scores and figures in the MiMIR Shiny app
  rather than a fixed benchmark table
- Metrics are not surrogate performance/calibration quantities (e.g., unvalidated
  scores with no LOBOV or Platt-calibration reference available)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MiMIR
  role: Provides calculate_surrogate_scores, roc_surro/LOBOV_accuracies, plattCalibration,
    getECE/getMCE, and the acc_LOBOV reference dataset from which the tabulated metrics
    are extracted; outputs follow R data-frame/CSV conventions
  repo: https://github.com/DanieleBizzarri/MiMIR
- name: R
  role: Runtime environment (R 3.6+) used to compute and export the results table
    as CSV/TSV
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btac388
    title: 'MiMIR: R-shiny application to infer risk factors and endpoints from Nightingale
      Health’s 1H-NMR metabolomics data'
  source_content_hash: sha256:90039b81165fa4bfb65ccbdfd15515796b1328dc06e57e893e7503f2bcfb04b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DanieleBizzarri/MiMIR
  tools:
  - MiMIR
  - R
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# Benchmark result tabulation into CSV

## Summary

Assembles article-reported surrogate performance and calibration metrics (ROC AUCs, mean LOBOV AUCs, Platt-calibration deltas) for MiMIR metabolomics-based surrogates into a machine-readable CSV table with columns metric_name, value, and applies_under.

## When to use

Use when a study reports surrogate-model performance (ROC AUC, LOBOV validation AUC) and Platt-calibration quality (ECE/MCE/logLoss deltas) that must be reproduced as a structured, machine-readable results table.

## When NOT to use

- Input is raw 1H-NMR metabolite data needing QC and score projection (use QCprep_surrogates/calculate_surrogate_scores first, not tabulation)
- The goal is interactive exploration of scores and figures in the MiMIR Shiny app rather than a fixed benchmark table
- Metrics are not surrogate performance/calibration quantities (e.g., unvalidated scores with no LOBOV or Platt-calibration reference available)

## Inputs

- Case-study surrogate performance results from the article (e.g., LLS_PAROFFs ROC AUCs for 'diabetes' and 'high hscrp')
- MiMIR acc_LOBOV reference dataset of validated surrogate-model accuracies
- Platt-calibration outputs from plattCalibration with ECE/MCE (getECE/getMCE) and logLoss values
- Surrogate parameter specification (PARAM_surrogates), e.g., comp.T2D_Ahola_Olli with 110 significant metabolic features

## Outputs

- Machine-readable CSV/TSV results table with columns metric_name, value, and applies_under
- Rows covering: AUC 0.85 ('diabetes'), AUC 0.811 ('high hscrp'), mean LOBOV AUC 0.8 and 0.81, ΔECE=-0.063, ΔMCE=-0.013, ΔlogLoss=-0.05, feature count 110, and a text-valued lipid-medication qualitative row

## How to apply

Read the case-study results (e.g., LLS_PAROFFs) and the MiMIR package's acc_LOBOV reference dataset holding validated LOBOV accuracies for each surrogate. Extract ROC AUCs of projected surrogate scores — here 'diabetes' AUC=0.85 and 'high hscrp' AUC=0.811 — alongside their mean LOBOV validation AUCs (0.8 and 0.81) computed via roc_surro/LOBOV_accuracies. Then extract Platt-calibration deltas from plattCalibration output using getECE/getMCE and logLoss: ΔECE=-0.063, ΔMCE=-0.013, ΔlogLoss=-0.05, plus any count-based metrics (110 metabolic features associated with diabetes in the comp.T2D_Ahola_Olli context) and text-valued qualitative findings (e.g., lipid-medication accuracy). Assemble all rows into a CSV/TSV with columns metric_name, value, applies_under, following R/MiMIR data-frame output conventions, and verify every cell matches the article-reported values with no missing metric rows.

## Related tools

- **MiMIR** (Provides calculate_surrogate_scores, roc_surro/LOBOV_accuracies, plattCalibration, getECE/getMCE, and the acc_LOBOV reference dataset from which the tabulated metrics are extracted; outputs follow R data-frame/CSV conventions) — https://github.com/DanieleBizzarri/MiMIR
- **R** (Runtime environment (R 3.6+) used to compute and export the results table as CSV/TSV)

## Examples

```
scores <- calculate_surrogate_scores(met, pheno, PARAM_surrogates, bin_names, Nmax_miss=1, Nmax_zero=5, post=TRUE, roc=TRUE, quiet=FALSE)
```

## Evaluation signals

- Every cell of the output table matches article-reported values exactly: AUC 0.85 ('diabetes'), AUC 0.811 ('high hscrp'), LOBOV 0.8 and 0.81
- Calibration deltas match: ΔECE=-0.063, ΔMCE=-0.013, ΔlogLoss=-0.05
- Feature count row equals 110 (comp.T2D_Ahola_Olli diabetes-associated metabolic features)
- CSV parses cleanly with columns metric_name, value, applies_under and no missing metric rows
- Applies_under column correctly attributes each metric to its surrogate/context (LLS_PAROFFs vs BBMRI-nl LOBOV)

## Limitations

- Values are valid only for the LLS_PAROFFs case-study context; comparing against BBMRI-nl LOBOV accuracies assumes the acc_LOBOV reference dataset shipped with MiMIR is used unchanged
- Qualitative findings (e.g., lipid-medication accuracy) must be recorded as text-valued rows, which limits machine comparability of that row

## Evidence

- [methods] plattCalibration signature: "`plattCalibration(r.calib, p.calib, nbins, pl)`"
- [methods] surrogate score computation function: "`calculate_surrogate_scores(met, pheno, PARAM_surrogates, bin_names, Nmax_miss, Nmax_zero, post, roc, quiet)`"
- [abstract] calibration scope of MiMIR: "allows for the projection and calibration of 24 pre-trained metabolomics-based models, without any pre-required programming knowledge"
- [readme] accepted export formats: "Upload your metabolites with the same column names as in the example dataset (both CSV and TSV are accepted)."
