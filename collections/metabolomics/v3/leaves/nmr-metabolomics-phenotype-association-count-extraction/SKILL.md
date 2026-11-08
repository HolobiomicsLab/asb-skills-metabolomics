---
name: nmr-metabolomics-phenotype-association-count-extraction
description: Use when the goal is to quantify how many 1H-NMR metabolomics features
  associate with a clinical endpoint and tabulate surrogate-model performance (AUC,
  LOBOV, calibration deltas) for a cohort such as LLS_PAROFFs against a reference
  like BBMRI-nl.
when_to_use_negative:
- The input metabolomics data do not use Nightingale Health 1H-NMR column names compatible
  with the MiMIR example dataset, so surrogates cannot be projected.
- The target phenotype has no pre-trained surrogate in PARAM_surrogates, so no ROC/LOBOV
  accuracy can be computed and no validated feature count exists.
- The goal is raw metabolite QC or data exploration rather than extracting phenotype-association
  counts and surrogate performance metrics.
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MiMIR
  role: Provides calculate_surrogate_scores, roc_surro/LOBOV_accuracies, plattCalibration,
    getECE/getMCE and the acc_LOBOV reference data used to compute and tabulate the
    surrogate metrics and feature counts
  repo: https://github.com/DanieleBizzarri/MiMIR
- name: R
  role: Runtime (R 3.6+) for executing MiMIR functions and assembling the data-frame/CSV
    results table
- name: R-shiny
  role: GUI front-end of MiMIR for uploading metabolite CSV/TSV files, viewing predicted
    scores, and downloading results
  repo: https://github.com/DanieleBizzarri/MiMIR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btac388
    title: 'MiMIR: R-shiny application to infer risk factors and endpoints from Nightingale
      Health’s 1H-NMR metabolomics data'
  source_content_hash: sha256:861705579ba9446bf63872731606029ae6d48ead825f5094a06d4d52749e5ad2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DanieleBizzarri/MiMIR
  tools:
  - MiMIR
  - R
  - R-shiny
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# nmr-metabolomics-phenotype-association-count-extraction

## Summary

Extracts, from the MiMIR surrogate-score framework, the counts of Nightingale Health 1H-NMR metabolic features significantly associated with a phenotype (e.g. 110 features for type 2 diabetes in the Ahola-Olli comp.T2D_Ahola_Olli context) and assembles the surrogate performance metrics (ROC AUC, mean LOBOV AUC, Platt-calibration deltas) into a machine-readable results table.

## When to use

Use when the goal is to quantify how many 1H-NMR metabolomics features associate with a clinical endpoint and tabulate surrogate-model performance (AUC, LOBOV, calibration deltas) for a cohort such as LLS_PAROFFs against a reference like BBMRI-nl.

## When NOT to use

- The input metabolomics data do not use Nightingale Health 1H-NMR column names compatible with the MiMIR example dataset, so surrogates cannot be projected.
- The target phenotype has no pre-trained surrogate in PARAM_surrogates, so no ROC/LOBOV accuracy can be computed and no validated feature count exists.
- The goal is raw metabolite QC or data exploration rather than extracting phenotype-association counts and surrogate performance metrics.

## Inputs

- Nightingale Health 1H-NMR metabolomics dataset (CSV/TSV, column names matching the MiMIR example dataset)
- Phenotypic dataset with clinical endpoints (e.g. diabetes, hsCRP), binarized via binarize_all_pheno
- PARAM_surrogates parameter set shipped with MiMIR
- acc_LOBOV reference dataset of surrogate validation accuracies
- Article-reported case-study values (LLS_PAROFFs AUCs, calibration deltas, feature counts)

## Outputs

- Machine-readable CSV/TSV results table with columns metric_name, value, applies_under
- ROC AUC rows for projected surrogate scores (e.g. diabetes 0.85, high hscrp 0.811)
- Mean LOBOV validation AUC rows (e.g. 0.8 and 0.81)
- Platt-calibration delta rows (ΔECE=-0.063, ΔMCE=-0.013, ΔlogLoss=-0.05)
- Count of metabolic features significantly associated with the phenotype (110 for T2D)

## How to apply

Read the MiMIR case-study results (e.g. LLS_PAROFFs) and the package's acc_LOBOV reference dataset for validated surrogate accuracies. Project the surrogate scores with calculate_surrogate_scores (using PARAM_surrogates, roc=TRUE) and evaluate them with the roc_surro/LOBOV_accuracies functions to obtain ROC AUCs — e.g. 'diabetes' AUC=0.85 with mean LOBOV AUC=0.8, and 'high hscrp' AUC=0.811 with mean LOBOV AUC=0.81. Apply plattCalibration and compute calibration deltas via getECE/getMCE and logLoss: ΔECE=-0.063, ΔMCE=-0.013, ΔlogLoss=-0.05. For association counts, extract the number of metabolic features significantly associated with the target phenotype (110 features for type 2 diabetes in the comp.T2D_Ahola_Olli / four-Finnish-cohorts context) and record any qualitative findings (e.g. lipid-medication accuracy limitations) as text-valued rows. Assemble all values into a CSV with columns metric_name, value, and applies_under, following R/MiMIR data-frame output conventions.

## Related tools

- **MiMIR** (Provides calculate_surrogate_scores, roc_surro/LOBOV_accuracies, plattCalibration, getECE/getMCE and the acc_LOBOV reference data used to compute and tabulate the surrogate metrics and feature counts) — https://github.com/DanieleBizzarri/MiMIR
- **R** (Runtime (R 3.6+) for executing MiMIR functions and assembling the data-frame/CSV results table)
- **R-shiny** (GUI front-end of MiMIR for uploading metabolite CSV/TSV files, viewing predicted scores, and downloading results) — https://github.com/DanieleBizzarri/MiMIR

## Examples

```
res <- calculate_surrogate_scores(met, pheno, PARAM_surrogates, bin_names, Nmax_miss, Nmax_zero, post=TRUE, roc=TRUE, quiet=FALSE)
```

## Evaluation signals

- Extracted AUCs match article-reported values exactly: diabetes 0.85 (mean LOBOV 0.8) and high hscrp 0.811 (mean LOBOV 0.81).
- Calibration deltas match: ΔECE=-0.063, ΔMCE=-0.013, ΔlogLoss=-0.05, computed via plattCalibration with getECE/getMCE and logLoss.
- Feature-count row equals 110 for the diabetes (comp.T2D_Ahola_Olli) context, with no missing metric rows.
- Output table validates against the required schema: metric_name, value, applies_under; qualitative findings appear as text-valued rows.
- LOBOV accuracies in the assembled table match MiMIR's acc_LOBOV reference dataset entries for the corresponding surrogates.

## Limitations

- Lipid-medication-related surrogate accuracy is qualitatively limited (recorded as a text-valued finding rather than a numeric metric).
- MiMIR projects only 24 pre-trained metabolomics-based models; phenotypes without a published surrogate cannot be scored.
- Metabolite uploads must match the example dataset's column names exactly or the App cannot find all necessary metabolites.
- No discussion-section content or changelog is available in the extracted article context.

## Evidence

- [methods] surrogate scoring call: "calculate_surrogate_scores(met, pheno, PARAM_surrogates, bin_names, Nmax_miss, Nmax_zero, post, roc, quiet)"
- [methods] calibration call: "plattCalibration(r.calib, p.calib, nbins, pl)"
- [readme] T2D association cohort: "Circulating metabolites and the risk of type 2 diabetes: a prospective study of 11,896 young adults from four Finnish cohorts"
- [readme] input format requirement: "Upload your metabolites with the same column names as in the example dataset (both CSV and TSV are accepted)."
