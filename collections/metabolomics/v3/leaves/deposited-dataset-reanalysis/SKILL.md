---
name: deposited-dataset-reanalysis
description: Use when a deposited dataset (e.g. https://doi.org/doi:10.25345/C5FD2F)
  must be independently reprocessed to confirm published metrics (AUC, true positive
  retention, false peak reduction) or to investigate cross-cohort covariate shift
  in peak shape distributions.
when_to_use_negative:
- Input is raw vendor LC-MS data that has not yet been peak-picked with MZmine 2 into
  non-filtered peak lists — run the extraction workflow first
- Goal is training a new DNN model from scratch rather than evaluating/reproducing
  published metrics on the deposited cohorts
- Data are targeted LC-MS features or non-HILIC metabolomics assays outside the human
  plasma peak-classification scope of this study
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MZmine 2
  role: Source of non-filtered peak lists from raw data under more restrictive and
    less restrictive peak extraction parameters; defines the peak windows evaluated
    on both cohorts
- name: R
  role: Used with custom R scripts to generate peak plots and peak group parameters
    used in the reanalysis of the deposited datasets
- name: LC-MS
  role: Non-targeted HILIC LC-MS metabolomics platform that produced the human plasma
    data for Cohort 1 (N=78) and Cohort 2 (N=526)
- name: EDML_deep_learning2.py
  role: Script for training/applying the DNN model to the downloaded deposited datasets
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.9b02983
    title: Deep Neural Networks for Classification of LC-MS Spectral Peaks
  source_content_hash: sha256:bd298c14527b4fb3728a18a13830723aaec14773006c0e040106b98c24a38fa3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - EDML_deep_learning2.py
  - LC-MS
  - MZmine 2
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# deposited-dataset-reanalysis

## Summary

Reanalysis of a deposited LC-MS peak classification dataset (Cohort 1 and Cohort 2 human plasma data) to independently reproduce reported DNN performance metrics and characterize covariate shift via retained false peaks.

## When to use

Use when a deposited dataset (e.g. https://doi.org/doi:10.25345/C5FD2F) must be independently reprocessed to confirm published metrics (AUC, true positive retention, false peak reduction) or to investigate cross-cohort covariate shift in peak shape distributions.

## When NOT to use

- Input is raw vendor LC-MS data that has not yet been peak-picked with MZmine 2 into non-filtered peak lists — run the extraction workflow first
- Goal is training a new DNN model from scratch rather than evaluating/reproducing published metrics on the deposited cohorts
- Data are targeted LC-MS features or non-HILIC metabolomics assays outside the human plasma peak-classification scope of this study

## Inputs

- Deposited Cohort 1 and Cohort 2 evaluation datasets from https://doi.org/doi:10.25345/C5FD2F
- Cohort 1 test set DNN results (726 manually reviewed peak examples) for baseline comparison
- Peak group parameter tables (59 parameters; 15 logistic-regression-selected including medRelFWHM)
- Stacked raster-image representations of candidate peaks (rows = samples, columns = elution time, pixel value = intensity)

## Outputs

- Comparison table of Cohort 1 vs Cohort 2 DNN metrics (AUC, true positive retention, false peak reduction)
- cohort_covariate_shift_analysis.csv with one row per peak-group parameter summary (median and IQR fields)
- Tabulation of retained false peaks with peak group parameters and raster-image sample rows

## How to apply

Download the deposited Cohort 1 and Cohort 2 evaluation datasets from https://doi.org/doi:10.25345/C5FD2F and load them with Python/pandas or R. Recompute the DNN classification outcome under the more restrictive settings (score threshold 0.4725, corresponding to true positive rate ~0.945 and false positive rate ~0.04) to confirm the reported 98% true positive peak capture and >50% false peak reduction on Cohort 2 against the 0.988 AUC on the Cohort 1 test set. Subset peaks classified as true by the DNN on Cohort 2 but labeled false in ground truth (retained false peaks), and tabulate their 15 peak group parameters (including medRelFWHM, the top predictive parameter) and their stacked raster-image rows. Compare the frequency, intensity, and parameter distributions of these retained false peaks against Cohort 1 test-set false peaks to quantify covariate shift (e.g. peak splitting outside the window, shoulder/double peaks, spurious high-background peaks, low signal-to-noise peaks). Write per-parameter median and interquartile-range summaries to a comparison CSV and validate that recomputed Cohort 2 metrics match the published values.

## Related tools

- **MZmine 2** (Source of non-filtered peak lists from raw data under more restrictive and less restrictive peak extraction parameters; defines the peak windows evaluated on both cohorts)
- **R** (Used with custom R scripts to generate peak plots and peak group parameters used in the reanalysis of the deposited datasets)
- **LC-MS** (Non-targeted HILIC LC-MS metabolomics platform that produced the human plasma data for Cohort 1 (N=78) and Cohort 2 (N=526))
- **EDML_deep_learning2.py** (Script for training/applying the DNN model to the downloaded deposited datasets)

## Examples

```
python EDML_deep_learning2.py  # applied after downloading datasets from https://doi.org/doi:10.25345/C5FD2F
```

## Evaluation signals

- Recomputed Cohort 2 metrics match reported values: 98% true positive peak retention and >50% false peak reduction at score threshold 0.4725 (TPR ~0.945, FPR ~0.04)
- Recomputed Cohort 1 test-set AUC matches the reported 0.988
- cohort_covariate_shift_analysis.csv contains one row per peak-group parameter with median and interquartile-range fields present
- Retained false peak subsetting is correct: peaks classified true by the DNN on Cohort 2 that are labeled false in ground truth, with all 15 logistic-regression-selected parameters populated
- Retained false peak categories are consistent with covariate shift: peak splitting outside the window, shoulder or double peaks, spurious high background peaks, and low signal-to-noise peaks

## Limitations

- Cohort 2 is an independent dataset with slightly different peak shapes relative to training datasets (covariate shift), so some false positives persist even under restrictive settings
- No specific deep learning framework or software version is named for the neural network implementation; only EDML_deep_learning2.py without a version
- Repository files are covered under the copyright language in LICENSE.txt, which may constrain reuse of the code

## Evidence
