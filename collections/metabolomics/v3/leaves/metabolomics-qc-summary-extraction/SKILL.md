---
name: metabolomics-qc-summary-extraction
description: Use when you have run metaboprep quality_control() on a metabolomics
  dataset (e.g. Metabolon UPLC-MS/MS or Nightingale NMR) and need the per-cohort sample/metabolite
  exclusion counts and the filtered dataset exported for reporting or verification.
when_to_use_negative:
- Input is already a filtered/QC'd feature table that bypasses the standard filtering
  pipeline
- The platform is not metabolomics (e.g. proteomics Olink/SomaLogic-style data requiring
  different QC semantics than the Metabolon/Nightingale read functions)
- You need bespoke, non-standard filtering logic rather than metaboprep's standard
  pipeline with user-defined thresholds
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: metaboprep
  role: Provides read_metabolon(), Metaboprep object construction, quality_control()
    with user-defined thresholds, summary() exclusion counting, and tab-delimited
    export
  repo: https://github.com/MRCIEU/metaboprep
- name: R
  role: Execution environment in which the metaboprep QC pipeline and summary extraction
    are run
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btac059
    title: 'metaboprep: an R package for preanalysis data description and processing'
  source_content_hash: sha256:ddbec40f980d254bcc55327db4bfcc516f10a1ed1e629d6b1cfc7eea4ade915f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MRCIEU/metaboprep
  tools:
  - R
  - metaboprep
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# metabolomics-qc-summary-extraction

## Summary

Extract per-cohort sample and metabolite exclusion counts from a metaboprep Metaboprep object after running the quality_control() pipeline, and save the filtered dataset and exclusion summary as tab-delimited text for reproducible reporting.

## When to use

Use when you have run metaboprep quality_control() on a metabolomics dataset (e.g. Metabolon UPLC-MS/MS or Nightingale NMR) and need the per-cohort sample/metabolite exclusion counts and the filtered dataset exported for reporting or verification.

## When NOT to use

- Input is already a filtered/QC'd feature table that bypasses the standard filtering pipeline
- The platform is not metabolomics (e.g. proteomics Olink/SomaLogic-style data requiring different QC semantics than the Metabolon/Nightingale read functions)
- You need bespoke, non-standard filtering logic rather than metaboprep's standard pipeline with user-defined thresholds

## Inputs

- Metabolon-format UPLC-MS/MS metabolomics workbook (e.g. BiB_MS-1 .xlsx with 'OrigScale' sheet)
- Nightingale NMR metabolomics dataset (e.g. ALSPAC_F24)
- User-defined QC thresholds as named arguments to quality_control()

## Outputs

- Metaboprep object with input and qc data layers
- summary() output with sample/feature exclusion code counts
- qc_filtered_dataset.txt (tab-delimited filtered dataset)
- qc_exclusion_summary.txt (tab-delimited exclusion count summary)

## How to apply

Load each cohort's data into a Metaboprep object (e.g. read_metabolon with return_Metaboprep=TRUE) and run quality_control(source_layer='input') with the paper's user-defined thresholds: sample_missingness=0.2, feature_missingness=0.2, total_peak_area_sd=5, outlier_udist=5, outlier_treatment='leave_be', winsorize_quantile=1.0, tree_cut_height=0.5, pc_outlier_sd=5. The pipeline drops samples and features exceeding the missingness thresholds, flags total-peak-area and univariate outliers, and performs PCA-based sample outlier detection. Call summary() on the quality-controlled object to read the Exclusion Codes Summary (e.g. user_defined_sample_missingness, user_defined_sample_pca_outlier, user_defined_feature_missingness counts). Save the filtered dataset as tab-delimited text (qc_filtered_dataset.txt) and an exclusion summary (qc_exclusion_summary.txt), then verify the counts match expected values — for example BiB_MS-1 (Metabolon): 15 samples excluded and 24% of metabolites excluded by missingness criteria; ALSPAC_F24 (Nightingale NMR): 3 samples and 0 metabolites excluded.

## Related tools

- **metaboprep** (Provides read_metabolon(), Metaboprep object construction, quality_control() with user-defined thresholds, summary() exclusion counting, and tab-delimited export) — https://github.com/MRCIEU/metaboprep
- **R** (Execution environment in which the metaboprep QC pipeline and summary extraction are run)

## Examples

```
mydata <- quality_control(mydata, source_layer="input", sample_missingness=0.2, feature_missingness=0.2, total_peak_area_sd=5, outlier_udist=5, outlier_treatment="leave_be", winsorize_quantile=1.0, tree_cut_height=0.5, pc_outlier_sd=5); summary(mydata)
```

## Evaluation signals

- summary() Exclusion Codes Summary shows expected counts (e.g. BiB_MS-1: 15 samples; ALSPAC_F24: 3 samples, 0 metabolites)
- Feature missingness exclusions account for ~24% of metabolites in BiB_MS-1 and 0 in ALSPAC_F24
- qc_filtered_dataset.txt is valid tab-delimited text whose row/column counts match the post-QC layer
- qc data layer exists alongside input layer in the Metaboprep object (Layer Names: input, qc)
- Extreme missingness checks (>=80%) run first and exclude 0 samples/features before user-defined thresholds are applied

## Limitations

- Thresholds are user-defined, so exclusion counts are only comparable when the exact same parameter set (sample_missingness=0.2, feature_missingness=0.2, etc.) is applied across cohorts
- PCA outlier analysis can emit warnings when max_num_pcs exceeds the number of available informative PCs
- The paper notes a general lack of standardization and reporting transparency for metabolomics preanalytical processing, which this workflow addresses but does not eliminate
- Package lifecycle is marked experimental

## Evidence

- [intro] Perform data filtering using a standard pipeline with user-defined thresholds: "Perform data filtering on the data set using a standard pipeline and according to user-defined thresholds."
- [methods] QC thresholds in methods: "sample_missingness  = 0.2"
- [readme] summary exclusion counts from README: [evidence span withheld — anchor status: external_doc]
- [readme] read_metabolon invocation from README: "mydata <- read_metabolon(system.file("extdata", "metabolon_v1.1_example.xlsx", package = "metaboprep")"
- [results] exclusion summary counts from task card finding: [evidence span withheld — anchor status: not_found]
