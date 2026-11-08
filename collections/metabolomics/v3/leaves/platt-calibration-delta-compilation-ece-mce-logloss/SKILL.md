---
name: platt-calibration-delta-compilation-ece-mce-logloss
description: Use when Nightingale Health 1H-NMR metabolomics surrogate scores projected
  into a new cohort (e.g. LLS_PAROFFs) must be Platt-calibrated to a reference dataset
  and pre/post calibration error deltas (ΔECE, ΔMCE, ΔlogLoss) must be compiled.
when_to_use_negative:
- The input metabolomics data are not Nightingale Health 1H-NMR features (MiMIR's
  24 pre-trained models expect that platform's metabolite column names)
- No reference calibration dataset with observed phenotypes is available — Platt scaling
  requires paired surrogate values and true labels (r.calib, p.calib)
- The goal is raw score projection only; calibration is a separate step after QCprep_surrogates
  and calculate_surrogate_scores have run
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: MiMIR
  role: Provides plattCalibration, getECE, getMCE, roc_surro, LOBOV_accuracies and
    PARAM_surrogates used to calibrate surrogate scores and compile the delta table
  repo: https://github.com/DanieleBizzarri/MiMIR
- name: R
  role: Runtime environment (version 3.6+) in which the MiMIR calibration and calibration-error
    functions are executed
  repo: https://github.com/DanieleBizzarri/MiMIR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btac388
    title: 'MiMIR: R-shiny application to infer risk factors and endpoints from Nightingale
      Health’s 1H-NMR metabolomics data'
  source_content_hash: sha256:b86857767590e4166e044a3de2651accb47f4b7bfe99ee9fb2793a45299c1502
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DanieleBizzarri/MiMIR
  tools:
  - MiMIR
  - R
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# platt-calibration-delta-compilation-ece-mce-logloss

## Summary

Recalibrates MiMIR metabolomics-based surrogate scores (e.g. 'diabetes', 'high hscrp') to a target cohort via Platt scaling and quantifies the improvement as deltas of Expected Calibration Error (ECE), Maximum Calibration Error (MCE), and logLoss. This turns raw projected surrogate values into cohort-appropriate probabilities with measurable calibration gain.

## When to use

Use when Nightingale Health 1H-NMR metabolomics surrogate scores projected into a new cohort (e.g. LLS_PAROFFs) must be Platt-calibrated to a reference dataset and pre/post calibration error deltas (ΔECE, ΔMCE, ΔlogLoss) must be compiled.

## When NOT to use

- The input metabolomics data are not Nightingale Health 1H-NMR features (MiMIR's 24 pre-trained models expect that platform's metabolite column names)
- No reference calibration dataset with observed phenotypes is available — Platt scaling requires paired surrogate values and true labels (r.calib, p.calib)
- The goal is raw score projection only; calibration is a separate step after QCprep_surrogates and calculate_surrogate_scores have run

## Inputs

- Nightingale Health 1H-NMR metabolomics matrix (CSV/TSV) with column names matching the MiMIR example dataset
- Projected surrogate scores for the target cohort (e.g. LLS_PAROFFs), produced via calculate_surrogate_scores with PARAM_surrogates
- Reference binary phenotype labels from a calibration dataset (e.g. binarize_all_pheno output) for Platt fitting
- LOBOV validation accuracies (acc_LOBOV reference dataset) for comparison of ROC AUCs

## Outputs

- Platt-calibrated surrogate probability values per sample
- Calibration delta metrics: ΔECE, ΔMCE, ΔlogLoss (case-study values -0.063, -0.013, -0.05)
- ROC AUC table rows for surrogates ('diabetes' 0.85, 'high hscrp' 0.811) with mean LOBOV AUCs (0.8, 0.81)
- Machine-readable CSV/TSV results table with columns metric_name, value, applies_under

## How to apply

First project the pre-trained surrogate scores onto the target cohort's Nightingale metabolomics features (after QCprep_surrogates preprocessing with PARAM_surrogates thresholds Nmax_miss, Nmax_zero). Then apply MiMIR's plattCalibration(r.calib, p.calib, nbins, pl) function, passing the surrogate logits/probabilities and the reference binary phenotype from the calibration dataset, selecting a bin count (nbins) for reliability estimation. Evaluate calibration before and after using getECE and getMCE plus logLoss, and record the deltas (in the LLS_PAROFFs case study: ΔECE=-0.063, ΔMCE=-0.013, ΔlogLoss=-0.05). In parallel, assess discriminative performance with roc_surro and the LOBOV_accuracies reference data (diabetes AUC 0.85 vs mean LOBOV 0.8; high hscrp AUC 0.811 vs mean LOBOV 0.81) to confirm calibration did not degrade ranking. Finally assemble metric rows (metric_name, value, applies_under) into a machine-readable CSV/TSV following MiMIR's data-frame conventions, verifying every cell against the reported values.

## Related tools

- **MiMIR** (Provides plattCalibration, getECE, getMCE, roc_surro, LOBOV_accuracies and PARAM_surrogates used to calibrate surrogate scores and compile the delta table) — https://github.com/DanieleBizzarri/MiMIR
- **R** (Runtime environment (version 3.6+) in which the MiMIR calibration and calibration-error functions are executed) — https://github.com/DanieleBizzarri/MiMIR

## Examples

```
cal <- plattCalibration(r.calib, p.calib, nbins, pl); dECE <- getECE(cal) - getECE(p.calib); dMCE <- getMCE(cal) - getMCE(p.calib)
```

## Evaluation signals

- Compiled delta values match the article-reported numbers exactly: ΔECE=-0.063, ΔMCE=-0.013, ΔlogLoss=-0.05
- Discriminative rows match: 'diabetes' AUC 0.85 (mean LOBOV 0.8), 'high hscrp' AUC 0.811 (mean LOBOV 0.81), with no missing metric rows
- ECE and MCE are computed via the MiMIR getECE/getMCE functions on the same reliability bins (nbins) used inside plattCalibration
- Output table follows the metric_name, value, applies_under schema in MiMIR's data-frame CSV/TSV conventions
- Post-calibration ROC AUC (via roc_surro) does not fall below the pre-calibration surrogate AUC or its LOBOV reference

## Limitations

- Platt calibration deltas depend on the chosen number of reliability bins (nbins); different bin counts change ECE/MCE magnitudes
- Calibration quality is judged against a reference dataset (e.g. BBMRI-nl); deltas are cohort-specific and not transferable without re-fitting
- The article provides no discussion-section content or changelog, so caveats about calibration behaviour must be inferred from the methods only

## Evidence

- [methods] plattCalibration function signature: "`plattCalibration(r.calib, p.calib, nbins, pl)`"
- [intro] calibration purpose in article intro: "and calibrate the metabolic surrogate values to a desired dataset"
- [readme] surrogate score reference citation in README: "‘1H-NMR metabolomics-based surrogates to impute common clinical risk factors and endpoints’, EBioMedicine"
