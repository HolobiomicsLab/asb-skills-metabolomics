---
name: sample-quality-metric-computation
description: Use when raw (un)targeted metabolomics data (e.g. Nightingale NMR or
  Metabolon exports) must be characterized with sample- and feature-level QC metrics
  before applying user-defined exclusion thresholds.
when_to_use_negative:
- The data are already a filtered/QC-passed feature table and no preanalysis quality
  description is needed
- The goal is downstream statistical modelling (e.g. association testing) rather than
  data description and sample/metabolite exclusion
- The input is not a supported metabolomics platform format (Metabolon, Nightingale,
  Olink, SomaLogic) or generic data/features/samples tables
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: metaboprep
  role: Core R package providing read_nightingale()/read_metabolon() importers, summarise(),
    sample_summary(), feature_summary(), quality_control(), and html report generation
    for computing QC metrics
  repo: https://github.com/MRCIEU/metaboprep
- name: R
  role: Runtime environment in which the metaboprep workflow is executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btac059
    title: 'metaboprep: an R package for preanalysis data description and processing'
  source_content_hash: sha256:ad14a79b1d0353138f84326251cb1177bf45025869d261eb077579eb748e20a6
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
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# sample-quality-metric-computation

## Summary

Computes per-sample and per-metabolite quality-control summary statistics (missingness, total peak area, PCA-derived outliers, distributional metrics) on metabolomics data using the metaboprep R package, to support transparent, threshold-based sample and metabolite filtering.

## When to use

Use when raw (un)targeted metabolomics data (e.g. Nightingale NMR or Metabolon exports) must be characterized with sample- and feature-level QC metrics before applying user-defined exclusion thresholds.

## When NOT to use

- The data are already a filtered/QC-passed feature table and no preanalysis quality description is needed
- The goal is downstream statistical modelling (e.g. association testing) rather than data description and sample/metabolite exclusion
- The input is not a supported metabolomics platform format (Metabolon, Nightingale, Olink, SomaLogic) or generic data/features/samples tables

## Inputs

- Nightingale Health NMR metabolomics dataset (read via read_nightingale())
- Metabolon xlsx worksheet export (e.g. metabolon_v1.1_example.xlsx, OrigScale sheet)
- Olink txt, SomaLogic adat, or generic data/features/samples CSV triple
- User-defined QC thresholds (sample_missingness, feature_missingness, total_peak_area_sd, outlier_udist, winsorize_quantile, tree_cut_height, pc_outlier_sd)

## Outputs

- Sample-level summary statistics data frame (missingness, total peak area, outlier occurrence count, top-10 PCs)
- Metabolite-level summary statistics data frame (missingness, n, mean, SD, skew, coefficient of variance, Shapiro-Wilk W, outlier count)
- Tab-delimited summary statistic files and an html report
- Metaboprep object with 'input' and 'qc' data layers plus exclusion-code counts

## How to apply

Import the platform data into R with metaboprep's platform-specific reader (e.g. read_nightingale() for Nightingale Health NMR data, read_metabolon() for Metabolon xlsx) setting return_Metaboprep = TRUE to build a Metaboprep object. Run summarise() on the pre-filtering 'input' source_layer with outlier_udist = 5 and tree_cut_height = 0.5, which jointly returns a sample summary (missingness, total peak area, outlier occurrence count, top-10 principal components) and a metabolite summary (missingness, n, mean, SD, skew, coefficient of variance, Shapiro-Wilk W statistic, outlier count). Export both data frames as tab-delimited text files. Downstream, quality_control() applies the standard pipeline with user-defined thresholds — e.g. sample_missingness = 0.2, feature_missingness = 0.2, total_peak_area_sd = 5, outlier_udist = 5, winsorize_quantile = 1.0, tree_cut_height = 0.5, pc_outlier_sd = 5 — copying the 'input' layer into a 'qc' layer and excluding samples/features per exclusion code.

## Related tools

- **metaboprep** (Core R package providing read_nightingale()/read_metabolon() importers, summarise(), sample_summary(), feature_summary(), quality_control(), and html report generation for computing QC metrics) — https://github.com/MRCIEU/metaboprep
- **R** (Runtime environment in which the metaboprep workflow is executed)

## Examples

```
mydata <- mydata |> quality_control(source_layer="input", sample_missingness=0.2, feature_missingness=0.2, total_peak_area_sd=5, outlier_udist=5, winsorize_quantile=1.0, tree_cut_height=0.5, pc_outlier_sd=5)
```

## Evaluation signals

- Both a sample-level and a metabolite-level summary data frame are returned from summarise() on the 'input' layer and export cleanly as tab-delimited files
- Sample summaries contain missingness, total peak area, outlier occurrence count and top-10 principal components; metabolite summaries contain n, mean, SD, skew, coefficient of variance, Shapiro-Wilk W and outlier count
- Exclusion counts in summary(mydata) match the configured thresholds (e.g. ALSPAC_F24 with Table 2 thresholds excludes 3 samples and no metabolites)
- The Metaboprep object reports two layers named 'input' and 'qc', and exclusion codes (e.g. user_defined_sample_missingness) tally with the filtering log messages

## Limitations

- The package is marked experimental (lifecycle badge in the README), so APIs may change
- PCA outlier assessment warns when max_num_pcs exceeds the number of available informative PCs (README example: max 10 PCs requested but only 2 informative)
- Exclusion outcomes are threshold-dependent: the same pipeline excluded 3 samples/no metabolites on ALSPAC_F24 but 15 samples and 24% of metabolites on BiB_MS-1
- The article notes a lack of standardization and reporting transparency in metabolomics preanalysis; metaboprep mitigates this via user-defined thresholds and open-disclosure reports, but thresholds remain user choices

## Evidence

- [abstract] standardized workflow claim: "we introduce metaboprep, a standardized data processing workflow to extract and characterize high quality metabolomics datasets"
- [intro] user-defined threshold filtering: "Perform data filtering on the data set using a standard pipeline and according to user-defined thresholds."
- [intro] summary outputs: "Provide useful summary data in the form of tab-delimited text file and a html report."
- [abstract] quality metrics selection: "The package extracts data from preformed worksheets, provides summary statistics and enables the user to select samples and metabolites for their analysis based on a set of quality metrics"
- [methods] threshold parameters in methods: "sample_missingness  = 0.2
feature_missingness = 0.2
total_peak_area_sd  = 5"
- [readme] README QC invocation: "mydata <- mydata |> quality_control( source_layer        = "input", sample_missingness  = 0.2, feature_missingness = 0.2"
- [readme] README PCA outlier caveat: [evidence span withheld — anchor status: external_doc]
