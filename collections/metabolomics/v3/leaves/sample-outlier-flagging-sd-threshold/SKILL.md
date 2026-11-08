---
name: sample-outlier-flagging-sd-threshold
description: Use when a raw metabolomics data matrix (e.g. Metabolon OrigScale sheet
  or dummy_data.csv) needs sample-level outlier detection before analysis, and PCA-based
  SD-threshold flagging of samples on top PCs is desired.
when_to_use_negative:
- Data is already a feature-level summary or sample table with no metabolite intensity
  matrix — PCA outlier flagging requires the raw sample x feature data layer
- You only need feature (metabolite) QC rather than sample outlier identification
  — use feature_summary/feature_missingness instead
- 'The dataset has fewer informative PCs than the requested max_num_pcs and you cannot
  accept the automatic truncation with warning (README case: max_num_pcs=10 vs informative
  PCs=2)'
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: metaboprep
  role: Provides Metaboprep(), quality_control(), pc_and_outliers(), and the 'input'/'qc'
    data layers used to impute, standardize, run PCA, and flag sample outliers at
    a 5-SD threshold
  repo: https://github.com/MRCIEU/metaboprep
- name: R
  role: Implementation language for the S7 Metaboprep object and the PC outlier-detection
    workflow
  repo: https://github.com/MRCIEU/metaboprep
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btac059
    title: 'metaboprep: an R package for preanalysis data description and processing'
  source_content_hash: sha256:38c9861c531d62fb5583eba5b5e28cfa318879daa59590f22ea65b5f646c19ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MRCIEU/metaboprep
  tools:
  - R
  - metaboprep
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# sample-outlier-flagging-sd-threshold

## Summary

Flags outlying samples in a metabolomics data matrix by running PCA on the median-imputed, z-standardized data and marking samples that deviate more than a user-defined number of standard deviations (e.g. pc_outlier_sd = 5) from the mean on the top informative PCs. Part of metaboprep's standardized preanalysis QC workflow for untargeted metabolite data.

## When to use

Use when a raw metabolomics data matrix (e.g. Metabolon OrigScale sheet or dummy_data.csv) needs sample-level outlier detection before analysis, and PCA-based SD-threshold flagging of samples on top PCs is desired.

## When NOT to use

- Data is already a feature-level summary or sample table with no metabolite intensity matrix — PCA outlier flagging requires the raw sample x feature data layer
- You only need feature (metabolite) QC rather than sample outlier identification — use feature_summary/feature_missingness instead
- The dataset has fewer informative PCs than the requested max_num_pcs and you cannot accept the automatic truncation with warning (README case: max_num_pcs=10 vs informative PCs=2)

## Inputs

- Metaboprep S7 object with an 'input' data layer (built from dummy_data.csv, dummy_samples.csv, dummy_features.csv or read_metabolon output)
- quality_control parameter set including pc_outlier_sd (e.g. 5), sample_missingness = 0.2, feature_missingness = 0.2, total_peak_area_sd = 5, outlier_udist = 5, tree_cut_height = 0.5
- sample_ids = NULL and feature_ids = NULL to run on all samples/features

## Outputs

- pca_outliers.tsv: data.frame of top-PC eigenvectors (loadings) per sample with 3/4/5-SD outlier count columns
- Attributes 'input_varexp' (PC variance explained), 'input_num_pcs_scree', and 'input_num_pcs_parallel' (number of informative PCs)
- Sample exclusion code 'user_defined_sample_pca_outlier' counts in the Metaboprep object summary

## How to apply

Build a Metaboprep S7 object with Metaboprep() from data, samples, and features tables (e.g. dummy_data.csv, dummy_samples.csv, dummy_features.csv from the package extdata). Run pc_and_outliers() on the 'input' layer with sample_ids = NULL and feature_ids = NULL: the function imputes missing values to the median, z-standardizes, and performs PCA. Determine the number of informative PCs using the nScree acceleration factor (minimum of 2 retained PCs) and parallel analysis; the README warns that a stated max_num_pcs (e.g. 10) can exceed the number of available informative PCs (e.g. 2), which triggers a warning. Count samples as outliers at 3, 4, and 5 standard deviations from the mean on the top PCs, using pc_outlier_sd = 5 as the exclusion threshold in the quality_control() pipeline, and extract the top-PC eigenvector table plus the 'input_varexp', 'input_num_pcs_scree', and 'input_num_pcs_parallel' attributes. Write the eigenvector table with per-sample 3/4/5-SD outlier counts to pca_outliers.tsv.

## Related tools

- **metaboprep** (Provides Metaboprep(), quality_control(), pc_and_outliers(), and the 'input'/'qc' data layers used to impute, standardize, run PCA, and flag sample outliers at a 5-SD threshold) — https://github.com/MRCIEU/metaboprep
- **R** (Implementation language for the S7 Metaboprep object and the PC outlier-detection workflow) — https://github.com/MRCIEU/metaboprep

## Examples

```
mydata <- mydata |> quality_control(source_layer="input", pc_outlier_sd=5, sample_missingness=0.2, feature_missingness=0.2, total_peak_area_sd=5, outlier_udist=5, outlier_treatment="leave_be", winsorize_quantile=1.0, tree_cut_height=0.5, sample_ids=NULL, feature_ids=NULL)
```

## Evaluation signals

- pca_outliers.tsv exists and contains PC eigenvector columns plus the 3-, 4-, and 5-SD outlier count columns
- The reported number of retained PCs (input_num_pcs_scree / input_num_pcs_parallel) is at least 2, the acceleration-factor minimum
- summary(mydata) reports the 'user_defined_sample_pca_outlier' exclusion count consistent with the pc_outlier_sd = 5 threshold
- The input layer was imputed to the median and z-standardized before PCA (no NA-driven PCA failure)
- No warning that max_num_pcs exceeds the number of available informative PCs, or the truncated PC set is explicitly acknowledged

## Limitations

- The number of informative PCs estimated by the acceleration factor can be far below the requested max_num_pcs; in the README example only 2 informative PCs were available when 10 were requested
- Outlier flags depend on median imputation and z-standardization, so extreme missingness patterns distort PC positions; samples with >=80% extreme missingness are excluded before PCA in the QC pipeline
- Flagging is by SD distance on top PCs only (3/4/5 SD evaluated; pc_outlier_sd = 5 used for exclusion), not multivariate distance on all PCs

## Evidence

- [readme] PCA outlier analysis at 5 SD: [evidence span withheld — anchor status: external_doc]
- [readme] max PCs vs informative PCs: [evidence span withheld — anchor status: external_doc]
- [methods] pc_outlier_sd parameter: "pc_outlier_sd       = 5"
- [abstract] standardized workflow: "we introduce metaboprep, a standardized data processing workflow to extract and characterize high quality metabolomics datasets"
