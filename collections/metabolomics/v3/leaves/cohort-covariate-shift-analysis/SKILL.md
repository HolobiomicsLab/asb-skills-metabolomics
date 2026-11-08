---
name: cohort-covariate-shift-analysis
description: Use when an image-based DNN peak classifier (score threshold 0.4725)
  applied to an independent cohort retains false peaks, and you need to diagnose whether
  peak-shape differences (covariate shift) explain the residual false positives.
when_to_use_negative:
- The input is a single cohort with no independent replication cohort, so no between-cohort
  covariate-shift comparison is possible
- Peak classification has not yet been run (no DNN probability scores at the 0.4725
  threshold), so retained false peaks cannot be identified
- Ground-truth true/false labels are unavailable, making the retained-false-peak subset
  undefined
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: R
  role: Custom R scripts for peak-list processing and comparison of peak group parameter
    distributions between cohorts
- name: MZmine 2
  role: Generates non-filtered peak lists by extracting raw data with more restrictive
    and less restrictive peak extraction parameters
- name: LC-MS
  role: Non-targeted HILIC LC-MS acquisition of human plasma polar metabolites from
    Cohort 1 (N=78) and Cohort 2 (N=526)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.9b02983
    title: Deep Neural Networks for Classification of LC-MS Spectral Peaks
  source_content_hash: sha256:103ebcf82e0a042ab0c3d95069f52c30c0bc6cc386763b2e06e5c08fa87b3ad9
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
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# cohort-covariate-shift-analysis

## Summary

Quantifies covariate shift between LC-MS peak-classification cohorts by comparing a trained DNN's metrics on an independent cohort (e.g., Cohort 2, N=526 human plasma) against the training cohort test set (Cohort 1, N=78), and characterizing retained false peaks whose shapes differ between cohorts.

## When to use

Use when an image-based DNN peak classifier (score threshold 0.4725) applied to an independent cohort retains false peaks, and you need to diagnose whether peak-shape differences (covariate shift) explain the residual false positives.

## When NOT to use

- The input is a single cohort with no independent replication cohort, so no between-cohort covariate-shift comparison is possible
- Peak classification has not yet been run (no DNN probability scores at the 0.4725 threshold), so retained false peaks cannot be identified
- Ground-truth true/false labels are unavailable, making the retained-false-peak subset undefined

## Inputs

- Cohort 2 evaluation dataset and Cohort 1 test set results from https://doi.org/doi:10.25345/C5FD2F (loaded via Python/pandas)
- DNN probability-score assignments per peak group under more restrictive extraction settings (threshold 0.4725)
- Ground-truth true/false peak labels for both cohorts
- Stacked raster-image representations of candidate peaks (rows = samples, columns = retention/scanning dimension, pixel = intensity)
- Peak group parameters including medRelFWHM and the other 14 logistic-regression-selected parameters

## Outputs

- cohort_covariate_shift_analysis.csv with one row per peak-group parameter summary, containing median and interquartile-range fields
- Cohort 1 vs Cohort 2 DNN metric comparison (98% true peak retention, >50% false peak reduction on Cohort 2 vs 0.988 AUC on Cohort 1)
- Tabulated retained false peaks with peak group parameter distributions and raster-image sample rows

## How to apply

Load the deposited Cohort 2 evaluation data and Cohort 1 test results from https://doi.org/doi:10.25345/C5FD2F and recompute DNN outcomes under the more restrictive settings (threshold 0.4725, TPR ~0.945, FPR ~0.04) for both cohorts, confirming 98% true-positive peak retention and >50% false-peak reduction on Cohort 2 versus the 0.988 test AUC on Cohort 1. Subset peaks classified true by the DNN on Cohort 2 but labeled false in ground truth (retained false peaks), then tabulate their 15 peak group parameters (with medRelFWHM as the most predictive) and their stacked raster-image sample rows (each row = one sample, each pixel = intensity). Compare frequency, intensity, and parameter distributions of Cohort 2 retained false peaks against Cohort 1 test-set false peaks to characterize the covariate shift. Expect retained false peaks to show distinct morphologies: peak splitting outside the window, shoulder/double peaks, spurious high-background peaks, and low signal-to-noise peaks.

## Related tools

- **R** (Custom R scripts for peak-list processing and comparison of peak group parameter distributions between cohorts)
- **MZmine 2** (Generates non-filtered peak lists by extracting raw data with more restrictive and less restrictive peak extraction parameters)
- **LC-MS** (Non-targeted HILIC LC-MS acquisition of human plasma polar metabolites from Cohort 1 (N=78) and Cohort 2 (N=526))

## Examples

```
python EDML_deep_learning2.py  # after downloading datasets from https://doi.org/doi:10.25345/C5FD2F
```

## Evaluation signals

- Recomputed Cohort 2 metrics match reported values: 98% of true positive peaks captured and false peak detection reduced by more than 50% under more restrictive settings
- Recomputed Cohort 1 test AUC matches the reported 0.988 (and 0.992 training AUC)
- cohort_covariate_shift_analysis.csv contains one row per peak-group parameter summary with median and interquartile-range fields present
- Retained false peak categories include expected morphologies: splitting outside window, shoulder/double peaks, spurious high-background peaks, and low signal-to-noise peaks
- Retained false peak fraction on Cohort 2 is small but notable despite the 0.4725 score threshold achieving ~0.945 TPR and ~0.04 FPR on training data

## Limitations

- Covariate shift persists even with restrictive settings: a small but notable percentage of Cohort 2 peaks remain false positives because peak shapes differ slightly from training datasets
- Retained false peaks are visually heterogeneous (split, shoulder/double, high-background, low-S/N), so no single peak-group parameter fully explains the shift
- Version details for the neural net implementation are absent; only the script name EDML_deep_learning2.py is given, so reproducing the classifier may require care

## Evidence

- [results] visual inspection revealed slightly different peak shapes in this independent dataset relative to training datasets, illustrating covariate shift: [evidence span withheld — anchor status: not_found]
- [results] Common retained false peaks included those with peak splitting outside of the window, shoulder or double peaks, spurious high background peaks, and peaks with low signal to noise: [evidence span withheld — anchor status: not_found]
- [results] Application of the optimized machine learning based neural network to the identical peak windows captured 98% of the true positive peaks and reduced false peak detection by more than 50%: "Application of the optimized machine learning based neural network to the identical peak windows captured 98% of the true positive peaks and reduced false peak detection by more than 50%"
- [results] the peak group parameter found to be the most predictive of peak quality was medRelFWHM: "the peak group parameter found to be the most predictive of peak quality was medRelFWHM"
- [readme] first download the datasets from https://doi.org/doi:10.25345/C5FD2F. Then use the script EDML_deep_learning2.py: "first download the datasets from https://doi.org/doi:10.25345/C5FD2F. Then use the script EDML_deep_learning2.py."
