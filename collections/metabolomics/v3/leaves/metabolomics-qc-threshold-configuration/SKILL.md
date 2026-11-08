---
name: metabolomics-qc-threshold-configuration
description: Use when a raw metabolomics feature table (e.g. Nightingale NMR, Metabolon
  xlsx, Olink txt, SomaLogic adat) must be quality-controlled and filtered with explicit,
  reproducible sample/feature exclusion thresholds.
when_to_use_negative:
- Input is already a filtered/QCed feature table and no threshold-based exclusion
  audit is required
- Data is not metabolomics (the readers target Metabolon, Nightingale, Olink, and
  SomaLogic worksheet formats)
- You need to alter numeric values rather than select/flag samples and metabolites
  based on quality metrics
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: metaboprep
  role: Provides quality_control(), sample_summary(), feature_summary(), and platform
    readers that apply the user-defined thresholds and produce the summaries
  repo: https://github.com/MRCIEU/metaboprep
- name: R
  role: Runtime environment in which the metaboprep open-source package is executed
    as an R pipeline
  repo: https://github.com/MRCIEU/metaboprep
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btac059
    title: 'metaboprep: an R package for preanalysis data description and processing'
  source_content_hash: sha256:67b355bd6dc4abae94c1fd37d2d6f4bf71bb256ccbcb4e9f81afd25a973d031e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MRCIEU/metaboprep
  tools:
  - R
  - metaboprep
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# metaboprep-metabolomics-qc-threshold-configuration

## Summary

Configure and run the metaboprep quality_control pipeline in R, applying user-defined thresholds (sample_missingness=0.2, feature_missingness=0.2, total_peak_area_sd=5, outlier_udist=5, tree_cut_height=0.5, pc_outlier_sd=5) to filter Nightingale/Metabolon-style metabolomics data and generate sample- and feature-level QC summaries.

## When to use

Use when a raw metabolomics feature table (e.g. Nightingale NMR, Metabolon xlsx, Olink txt, SomaLogic adat) must be quality-controlled and filtered with explicit, reproducible sample/feature exclusion thresholds.

## When NOT to use

- Input is already a filtered/QCed feature table and no threshold-based exclusion audit is required
- Data is not metabolomics (the readers target Metabolon, Nightingale, Olink, and SomaLogic worksheet formats)
- You need to alter numeric values rather than select/flag samples and metabolites based on quality metrics

## Inputs

- Platform metabolomics dataset file (e.g. nightingale_v1_example.xlsx, metabolon_v1.1_example.xlsx, olink_v1_example.txt, somalogic_v1_example.adat, or dummy_data.csv with dummy_samples.csv/dummy_features.csv)
- User-defined QC thresholds (sample_missingness, feature_missingness, total_peak_area_sd, outlier_udist, winsorize_quantile, tree_cut_height, pc_outlier_sd)
- Optional user-specified sample_ids / feature_ids to force exclusions

## Outputs

- Metaboprep object with 'input' and 'qc' data layers plus sample and feature summary layers
- Sample-level and metabolite-level summary statistic data frames exportable as tab-delimited text files
- HTML QC report summarizing quality metrics and batch-variable influence
- Exclusion-code counts per sample/feature (e.g. user_defined_sample_missingness, extreme_feature_missingness)

## How to apply

Import the platform data into a Metaboprep object using the platform-specific reader (read_nightingale(), read_metabolon(), etc.) with return_Metaboprep=TRUE, then run quality_control() on the 'input' source_layer. Set the user-defined thresholds from the paper's Table 2 workflow: sample_missingness=0.2, feature_missingness=0.2, total_peak_area_sd=5, outlier_udist=5, winsorize_quantile=1.0, tree_cut_height=0.5, pc_outlier_sd=5, choosing outlier_treatment (e.g. 'leave_be') and feature_selection='least_missingness' as needed. The pipeline computes sample summaries (missingness, total peak area, outlier counts, top-10 principal components) and feature summaries (missingness, n, mean, SD, skew, coefficient of variance, Shapiro-Wilk W, outlier count), copies input data to a 'qc' layer, and excludes samples/features exceeding thresholds. Rationale: the package exists because there is 'a lack of standardization and reporting transparency for metabolomics preanalytical data processing procedures', and it gives users flexibility in defining selection thresholds. Judge correctness by inspecting summary(mydata) exclusion-code counts and exporting the returned sample- and feature-summary data frames as tab-delimited files (e.g. in the ALSPAC_F24 reproduction, the thresholds excluded 3 samples and no metabolites, versus 15 samples and 24% of metabolites for BiB_MS-1).

## Related tools

- **metaboprep** (Provides quality_control(), sample_summary(), feature_summary(), and platform readers that apply the user-defined thresholds and produce the summaries) — https://github.com/MRCIEU/metaboprep
- **R** (Runtime environment in which the metaboprep open-source package is executed as an R pipeline) — https://github.com/MRCIEU/metaboprep

## Examples

```
mydata <- mydata |> quality_control(source_layer="input", sample_missingness=0.2, feature_missingness=0.2, total_peak_area_sd=5, outlier_udist=5, tree_cut_height=0.5, pc_outlier_sd=5)
```

## Evaluation signals

- summary(mydata) reports two data layers ('input, qc') and Exclusion Codes Summary counts consistent with the chosen thresholds
- Sample summary contains missingness, total peak area, outlier occurrence count, and top-10 principal components for the source_layer
- Feature summary contains missingness, n, mean, SD, skew, coefficient of variance, Shapiro-Wilk W statistic, and outlier count
- QC log messages report per-step exclusion counts (e.g. 'excluding N sample(s)' at >=20% missingness) matching expected exclusions (e.g. 3 samples, 0 metabolites for ALSPAC_F24)
- Exported summary files are tab-delimited and the feature QC tree attribute (dendrogram) is retrievable for plotting

## Limitations

- Package is marked 'Lifecycle: experimental' in the README, so APIs may change
- PCA outlier assessment can warn that the stated max_num_pcs exceeds the number of available informative PCs (README example: max 10 vs 2 informative PCs)
- The package is not a targeted statistical analysis tool — it stops at description, summarization, and threshold-based selection of samples/metabolites

## Evidence

- [abstract] abstract: user-defined threshold flexibility: "Where possible, we provide users flexibility in defining their own selection thresholds"
- [intro] intro: standard pipeline with user-defined thresholds: "Perform data filtering on the data set using a standard pipeline and according to user-defined thresholds."
- [readme] readme: QC pipeline invocation: "mydata <- mydata |> quality_control( source_layer        = "input", 
                                     sample_missingness  = 0.2, 
                                     feature_missingness = 0.2,"
- [readme] readme: outputs are tab-delimited summary and html report: "Provide useful summary data in the form of tab-delimited text file and a html report."
