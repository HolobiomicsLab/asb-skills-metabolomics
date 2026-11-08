---
name: nightingale-nmr-metabolomics-data-import
description: Use when the input is a Nightingale Health NMR metabolomics dataset (e.g.
  ALSPAC_F24) that must be loaded into metaboprep before sample/metabolite QC summaries
  and threshold-based filtering can be performed.
when_to_use_negative:
- The data are from a non-Nightingale platform — use the platform-specific readers
  such as read_metabolon() (Metabolon .xlsx), or the Olink/SomaLogic/dummy-data readers
  instead.
- The dataset is already a Metaboprep object or an already QC-filtered feature table
  — proceed directly to quality_control() or summary/export steps rather than re-importing.
- The goal is statistical downstream modelling rather than preanalysis data description
  and QC — metaboprep's scope is reading, summarising, and threshold-based filtering,
  not association analysis.
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: metaboprep
  role: R package providing read_nightingale(), the Metaboprep object, summarise(),
    and quality_control() with user-defined thresholds for Nightingale NMR data
  repo: https://github.com/MRCIEU/metaboprep
- name: R
  role: Runtime environment in which metaboprep is loaded (library(metaboprep)) and
    summaries are exported as tab-delimited files
  repo: https://github.com/MRCIEU/metaboprep
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btac059
    title: 'metaboprep: an R package for preanalysis data description and processing'
  source_content_hash: sha256:bd44a71bfc796d95d2762208de99bb83ead49336099f4bfc0448696d8f8cafe7
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
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# nightingale-nmr-metabolomics-data-import

## Summary

Import a Nightingale Health NMR metabolomics dataset into the metaboprep R package as a Metaboprep object, so that sample/feature summaries, quality-metric filtering with user-defined thresholds, and tab-delimited summary exports can be run downstream.

## When to use

Use when the input is a Nightingale Health NMR metabolomics dataset (e.g. ALSPAC_F24) that must be loaded into metaboprep before sample/metabolite QC summaries and threshold-based filtering can be performed.

## When NOT to use

- The data are from a non-Nightingale platform — use the platform-specific readers such as read_metabolon() (Metabolon .xlsx), or the Olink/SomaLogic/dummy-data readers instead.
- The dataset is already a Metaboprep object or an already QC-filtered feature table — proceed directly to quality_control() or summary/export steps rather than re-importing.
- The goal is statistical downstream modelling rather than preanalysis data description and QC — metaboprep's scope is reading, summarising, and threshold-based filtering, not association analysis.

## Inputs

- Nightingale Health NMR metabolomics data file (e.g. ALSPAC_F24 dataset or the packaged nightingale_v1_example.xlsx)
- Optional sample and feature annotation metadata (sample_id / feature_id tables)

## Outputs

- A Metaboprep object with an 'input' data layer and sample/feature annotation (sample_id, feature_id, reason_excluded, excluded)
- Tab-delimited sample-level summary statistics (missingness, total peak area, outlier count, top-10 PCs)
- Tab-delimited metabolite-level summary statistics (missingness, n, mean, SD, skew, CV, Shapiro-Wilk W, outlier count)
- A 'qc' data layer with samples/features excluded per user-defined thresholds

## How to apply

Read the Nightingale platform data with metaboprep's read_nightingale(), setting return_Metaboprep = TRUE to obtain a Metaboprep object directly (metaboprep also ships a packaged nightingale_v1_example.xlsx for validating the import path). Keep the object on the 'input' source_layer and run summarise() with outlier_udist = 5 and tree_cut_height = 0.5 so both the sample summary (missingness, total peak area, outlier occurrence count, top-10 principal components) and the feature summary (missingness, n, mean, SD, skew, coefficient of variance, Shapiro-Wilk W, outlier count) are computed together. Export the two returned data frames as tab-delimited sample-level and metabolite-level files. For downstream filtering, apply quality_control() with the paper's user-defined thresholds: sample_missingness = 0.2, feature_missingness = 0.2, total_peak_area_sd = 5, outlier_udist = 5, winsorize_quantile = 1.0, tree_cut_height = 0.5, pc_outlier_sd = 5; the package then copies input data to a 'qc' layer and records exclusion reasons per sample/feature.

## Related tools

- **metaboprep** (R package providing read_nightingale(), the Metaboprep object, summarise(), and quality_control() with user-defined thresholds for Nightingale NMR data) — https://github.com/MRCIEU/metaboprep
- **R** (Runtime environment in which metaboprep is loaded (library(metaboprep)) and summaries are exported as tab-delimited files) — https://github.com/MRCIEU/metaboprep

## Examples

```
mydata <- read_nightingale("ALSPAC_F24_nightingale.csv", return_Metaboprep = TRUE); mydata <- mydata |> quality_control(source_layer = "input", sample_missingness = 0.2, feature_missingness = 0.2, total_peak_area_sd = 5, outlier_udist = 5, winsorize_quantile = 1.0, tree_cut_height = 0.5, pc_outlier_sd = 5)
```

## Evaluation signals

- summary(mydata) on the imported object reports the expected sample/feature counts and data layers ('input' before QC; 'input, qc' after quality_control), plus sample/feature annotation columns ending in reason_excluded and excluded.
- The QC console log shows each threshold step ran: 'Assessing for sample missingness at specified level of >=20%', 'Calculating total peak abundance outliers at +/- 5 Sdev', and 'Sample PCA outlier analysis at +/- 5 Sdev'.
- Reproducing the ALSPAC_F24 run with the Table 2 thresholds should exclude 3 samples and no metabolites, versus 15 samples and 24% of metabolites in BiB_MS-1.
- Both exported summary files are present, tab-delimited, and contain the expected metric columns (sample: missingness, total peak area, outlier count, top-10 PCs; feature: missingness, n, mean, SD, skew, CV, Shapiro-Wilk W, outlier count).
- The packaged nightingale_v1_example.xlsx loads without error, confirming the Nightingale reader path matches the platform's expected file layout.

## Limitations

- The package is marked 'experimental' in its lifecycle badge, so APIs may change between versions.
- In PCA outlier analysis, if max_num_pcs (e.g. 10) exceeds the number of available informative PCs, metaboprep warns and falls back to the available informative PCs (only 2 in the README example).
- The README demonstrates read_metabolon() explicitly; the Nightingale reader path is evidenced by the packaged nightingale_v1_example.xlsx and the ALSPAC_F24 workflow, so verify column layout for your Nightingale export version.

## Evidence

- [readme] Read in and process (un)targeted metabolite data, saving datasets in tab-delimited format for use elsewhere: "1.  Read in and processes (un)targeted metabolite data, saving datasets
    in tab-delimited format for use elsewhere"
- [readme] Perform data filtering with standard pipeline and user-defined thresholds: "3.  Perform data filtering on the data set using a standard pipeline and
    according to user-defined thresholds."
- [abstract] we introduce metaboprep, a standardized data processing workflow to extract and characterize high quality metabolomics datasets: "we introduce metaboprep, a standardized data processing workflow to extract and characterize high quality metabolomics datasets"
- [methods] system.file("extdata", "nightingale_v1_example.xlsx", package = "metaboprep"): "system.file("extdata", "nightingale_v1_example.xlsx", package = "metaboprep")"
