---
name: pretrained-metabolic-score-projection
description: Use when you have a Nightingale Health 1H-NMR metabolomics matrix (CSV/TSV
  with example-matching column names) and need per-sample values for MiMIR's 24 pre-trained
  metabolic scores/surrogates.
when_to_use_negative:
- Your data are not Nightingale Health 1H-NMR metabolomics measurements (e.g. targeted
  LC-MS panels or non-Nightingale NMR platforms) — the models are trained on Nightingale
  biomarkers only
- You need to train new metabolomics-based risk models rather than project existing
  published ones — MiMIR only projects and calibrates pre-trained models
- Your metabolite matrix is missing required Nightingale metabolites so that the name-mapping
  check cannot find all necessary features
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MiMIR
  role: R package providing the pre-trained model parameters, QC pre-processing, projection,
    and calibration functions for the 24 scores
  repo: https://github.com/DanieleBizzarri/MiMIR
- name: R
  role: Runtime environment; MiMIR requires R version 3.6+
- name: devtools
  role: Used to install MiMIR from GitHub via devtools::install_github
- name: R-shiny
  role: Graphical interface (startApp) for uploading data, checking metabolite name
    matching, and downloading projected scores
  repo: https://github.com/DanieleBizzarri/MiMIR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btac388
    title: 'MiMIR: R-shiny application to infer risk factors and endpoints from Nightingale
      Health’s 1H-NMR metabolomics data'
  source_content_hash: sha256:32a54e59bfbca8e54147aeb06ccf5bb6c7da7af0d1dcd7cb53e02794dc7b96aa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DanieleBizzarri/MiMIR
  tools:
  - MiMIR
  - R
  - R-shiny
  - devtools
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# pretrained-metabolic-score-projection

## Summary

Projects 24 pre-trained metabolomics-based models (metaboAge, surrogate clinical variables, and published CVD/T2D/COVID/mortality scores) onto new Nightingale Health 1H-NMR metabolomics data using the MiMIR R package, after model-specific QC pre-processing, normalization, and imputation.

## When to use

Use when you have a Nightingale Health 1H-NMR metabolomics matrix (CSV/TSV with example-matching column names) and need per-sample values for MiMIR's 24 pre-trained metabolic scores/surrogates.

## When NOT to use

- Your data are not Nightingale Health 1H-NMR metabolomics measurements (e.g. targeted LC-MS panels or non-Nightingale NMR platforms) — the models are trained on Nightingale biomarkers only
- You need to train new metabolomics-based risk models rather than project existing published ones — MiMIR only projects and calibrates pre-trained models
- Your metabolite matrix is missing required Nightingale metabolites so that the name-mapping check cannot find all necessary features

## Inputs

- Nightingale Health 1H-NMR metabolomics dataset in CSV or TSV with column names matching the MiMIR example (synthetic_metabolic_dataset)
- MiMIR pre-trained model parameters (PARAM_metaboAge, PARAM_surrogates) and published score beta vectors
- Optional phenotypic dataset for surrogate score calibration (synthetic_phenotypic_dataset-style)

## Outputs

- sample-by-score matrix with the full set of 24 metabolomics-based score/surrogate columns per retained sample
- surrogate_scores.csv file
- Optional figures and predicted score plots from the Shiny app

## How to apply

Load the Nightingale-format dataset and MiMIR's bundled parameters (PARAM_metaboAge, PARAM_surrogates, published score beta vectors), then map metabolite columns to each model's required feature IDs with find_BBMRI_names / prep_met_for_scores (applying the plus-one offset where a model requires it). Run model-specific QC in parallel per model with QCprep and QCprep_surrogates: drop samples exceeding the Nmax_miss and Nmax_zero thresholds, remove SD-based outliers via subset_samples_sd (against the bundled MEAN/SD values), and impute remaining missing values with impute_miss after scaling. Project each cleaned matrix with apply.fit / apply.fit_surro for the metaboAge and surrogate models, and comp.CVD_score, comp.T2D_Ahola_Olli, comp_covid_score, and comp.mort_score for the published clinical scores, keeping each model's own normalization as published. Optionally calibrate surrogate values to your dataset with plattCalibration. Assemble per-sample predictions into a sample-by-score matrix and write surrogate_scores.csv.

## Related tools

- **MiMIR** (R package providing the pre-trained model parameters, QC pre-processing, projection, and calibration functions for the 24 scores) — https://github.com/DanieleBizzarri/MiMIR
- **R** (Runtime environment; MiMIR requires R version 3.6+)
- **devtools** (Used to install MiMIR from GitHub via devtools::install_github)
- **R-shiny** (Graphical interface (startApp) for uploading data, checking metabolite name matching, and downloading projected scores) — https://github.com/DanieleBizzarri/MiMIR

## Examples

```
library(devtools); devtools::install_github("DanieleBizzarri/MiMIR"); library(MiMIR); MiMIR::startApp()
```

## Evaluation signals

- surrogate_scores.csv exists and is valid CSV
- Output contains the full set of 24 score/surrogate columns populated for all retained samples
- No unimputed NA cells remain in the output matrix (impute_miss ran after QC filtering)
- Name-mapping check confirms the App/tool found all necessary metabolites in the dataset
- Retained sample count is consistent with samples removed by Nmax_miss/Nmax_zero thresholds and subset_samples_sd outlier removal

## Limitations

- Samples exceeding the Nmax_miss or Nmax_zero missing/zero-value thresholds are discarded rather than imputed
- SD-based outliers are removed using the bundled MEAN/SD values, so samples far from the reference distribution are excluded
- Models expect Nightingale-format column names (with a plus-one offset for some models); mismatched names break projection
- Calibration of surrogate values is dataset-dependent and requires a separate calibration (phenotypic) dataset

## Evidence

- [abstract] 24 pre-trained models: "allows for the projection and calibration of 24 pre-trained metabolomics-based models, without any pre-required programming knowledge"
- [intro] calibration scope: "project previously published metabolic scores; and calibrate the metabolic surrogate values to a desired dataset"
- [methods] QCprep signature: "`QCprep(mat, PARAM_metaboAge, quiet, Nmax_zero, Nmax_miss)`"
- [methods] surrogate QC: "`QCprep_surrogates(mat, PARAM_surrogates, Nmax_miss, Nmax_zero, quiet)`"
- [methods] feature mapping: "`prep_met_for_scores(dat, featID, plusone, quiet)`"
- [methods] calibration: "`plattCalibration(r.calib, p.calib, nbins, pl)`"
- [readme] column naming: "Upload your metabolites with the same column names as in the example dataset (both CSV and TSV are accepted)."
