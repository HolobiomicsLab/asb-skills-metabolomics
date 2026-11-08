---
name: false-peak-characterization-from-feature-tables
description: Use when a machine learning peak classifier (DNN or logistic regression)
  still flags some false positives under more restrictive settings and you need to
  characterize their peak shapes and parameter distributions to explain covariate
  shift.
when_to_use_negative:
- If the goal is to train or recalibrate a peak classifier from scratch rather than
  diagnose residual false positives from an already-trained model.
- If the input data are not non-targeted HILIC LC-MS plasma feature tables with peak
  group parameters and raster-image rows — e.g., targeted peak tables or raw MS vendor
  files.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MZmine 2
  role: Generates the non-filtered peak lists and stacked raster-image peak-group
    parameters that feed the false-peak characterization
- name: R
  role: Used with custom scripts to compute logistic-regression-selected peak group
    parameters and statistical summaries of retained false peaks
- name: LC-MS
  role: Data source instrument producing the HILIC non-targeted plasma metabolomics
    peaks being characterized
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.9b02983
    title: Deep Neural Networks for Classification of LC-MS Spectral Peaks
  source_content_hash: sha256:23f538d52d8fd3ad3aecacc247b1bbd594b909156667c83efd0040d7d98fc74d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - LC-MS
  - MZmine 2
  - R
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# false-peak-characterization-from-feature-tables

## Summary

Characterizes retained false positive peaks in LC-MS feature tables by tabulating their peak group parameters (e.g., medRelFWHM) and stacked raster-image sample rows, to diagnose covariate shift between cohorts.

## When to use

Use when a machine learning peak classifier (DNN or logistic regression) still flags some false positives under more restrictive settings and you need to characterize their peak shapes and parameter distributions to explain covariate shift.

## When NOT to use

- If the goal is to train or recalibrate a peak classifier from scratch rather than diagnose residual false positives from an already-trained model.
- If the input data are not non-targeted HILIC LC-MS plasma feature tables with peak group parameters and raster-image rows — e.g., targeted peak tables or raw MS vendor files.

## Inputs

- Deposited Cohort 1 test-set and Cohort 2 evaluation peak lists (from https://doi.org/doi:10.25345/C5FD2F)
- Ground-truth true/false peak labels for both cohorts
- Stacked raster-image sample rows for each candidate peak (built from the 63 most-intense samples)
- The 15 logistic-regression-selected peak group parameters (medRelFWHM et al.)

## Outputs

- cohort_covariate_shift_analysis.csv with per-parameter median and IQR summaries
- Cohort 1 vs Cohort 2 DNN metric comparison table (98% true peak retention, >50% false peak reduction, 0.988 AUC)
- Tabulated retained false-peak categories (peak splitting outside window, shoulder/double peaks, spurious high-background peaks, low signal-to-noise peaks)

## How to apply

Load the deposited Cohort 1 test and Cohort 2 evaluation peak lists from the Synapse deposit (doi:10.25345/C5FD2F) with pandas, recompute the DNN outcome at the published score threshold of 0.4725 (true positive rate ~0.945, false positive rate ~0.04), and verify Cohort 2 retains 98% of true peaks while removing >50% of false peaks. Then subset peaks classified as true by the DNN but labeled false in the ground truth (the retained false peaks), tabulate the 15 logistic-regression-selected peak group parameters including medRelFWHM, and extract their stacked raster-image sample rows built from the 63 samples where the candidate peak was most intense. Quantify frequency, intensity, and parameter distributions of these retained false peaks relative to Cohort 1 test-set false peaks; visually inspect for peak splitting outside the window, shoulder or double peaks, spurious high-background peaks, and low signal-to-noise peaks as the retained false-peak categories illustrating covariate shift. Write a Cohort 1 vs Cohort 2 comparison table to CSV.

## Related tools

- **MZmine 2** (Generates the non-filtered peak lists and stacked raster-image peak-group parameters that feed the false-peak characterization)
- **R** (Used with custom scripts to compute logistic-regression-selected peak group parameters and statistical summaries of retained false peaks)
- **LC-MS** (Data source instrument producing the HILIC non-targeted plasma metabolomics peaks being characterized)

## Examples

```
python EDML_deep_learning2.py --data_path ./datasets/doi:10.25345/C5FD2F --threshold 0.4725 --cohort 2
```

## Evaluation signals

- Recomputed Cohort 2 metrics must match the reported values: 98% true positive peak retention and >50% false peak reduction at the 0.4725 score threshold.
- Cohort 1 test set should reproduce the published 0.988 AUC when re-run at the same threshold.
- Output CSV must contain one row per peak-group parameter summary with median and interquartile-range fields populated.
- Retained false-peak subset should be non-empty and dominated by the four named peak-shape categories (splitting, shoulder/double, high-background, low S/N).
- Total peaks captured under more restrictive settings should be ~11,026 features with ~50% true / ~50% false positives before DNN filtering.

## Limitations

- The retained false peaks on Cohort 2 reflect covariate shift caused by slightly different peak shapes in the independent dataset relative to training data, so characterization is cohort-specific and may not generalize to new cohorts.
- Characterization depends on the fixed more-restrictive peak extraction settings and the 0.4725 score threshold; changing either changes which peaks are retained as false positives.
- Only the 15 logistic-regression-selected peak group parameters (plus raster-image rows) are tabulated; the remaining 44 peak group parameters are not summarized here.

## Evidence

- [results] identify retained false-peak categories: [evidence span withheld — anchor status: not_found]
