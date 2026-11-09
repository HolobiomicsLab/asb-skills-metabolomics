---
name: iqr-outlier-filtering
description: Use when computing CV% or R2 repeatability metrics across blind duplicate
  NMR biomarker measurements, or when extreme non-normal outliers would otherwise
  inflate variance estimates for a biomarker panel.
when_to_use_negative:
- When running ukbnmr::remove_technical_variation() itself — the README explicitly
  warns that no prefiltering of samples or columns should be performed prior to this
  function, as the algorithms expect all data present
- When the analysis requires retaining all measurements regardless of extremity (e.g.
  sensitivity analyses to inclusion/exclusion of flagged samples and measurements)
- When analyzing the 61 composite biomarkers and 81 biomarker ratios, which should
  be recomputed post-adjustment from the 107 non-derived biomarkers rather than filtered
  directly
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: ukbnmr
  role: Provides extract_biomarkers() to load NMR data and remove_technical_variation()
    to produce the post-QC concentrations being compared; the package itself performs
    no outlier filtering
  repo: https://github.com/sritchie73/ukbnmr
- name: data.table
  role: Used with fread()/fwrite() to load the exported field data and write per-biomarker
    metrics and summary rows to repeatability_metrics.csv
- name: R
  role: Implementation environment for the per-biomarker median/IQR outlier exclusion
    and CV%/R2 computation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41597-023-01949-y
    title: Quality control and removal of technical variation of NMR metabolic biomarker
      data in ~120,000 UK Biobank participants
  source_content_hash: sha256:68a732c5ce8e31431556855c88def54a5e8f4b84c164812912a5d6fe31e32c55
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sritchie73/ukbnmr
  tools:
  - R
  - data.table
  - ukbnmr
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# iqr-outlier-filtering

## Summary

A per-biomarker outlier exclusion step that removes NMR biomarker measurements lying beyond 4x the interquartile range from the median before computing reproducibility metrics (CV%, R2) on blind duplicate samples, preventing extreme outliers from distorting repeatability summaries.

## When to use

Use when computing CV% or R2 repeatability metrics across blind duplicate NMR biomarker measurements, or when extreme non-normal outliers would otherwise inflate variance estimates for a biomarker panel.

## When NOT to use

- When running ukbnmr::remove_technical_variation() itself — the README explicitly warns that no prefiltering of samples or columns should be performed prior to this function, as the algorithms expect all data present
- When the analysis requires retaining all measurements regardless of extremity (e.g. sensitivity analyses to inclusion/exclusion of flagged samples and measurements)
- When analyzing the 61 composite biomarkers and 81 biomarker ratios, which should be recomputed post-adjustment from the 107 non-derived biomarkers rather than filtered directly

## Inputs

- UK Biobank NMR biomarker data.frame extracted with ukbnmr::extract_biomarkers(), restricted to the 107 non-derived biomarkers
- QC-processed biomarker data.frame from ukbnmr::remove_technical_variation() (processed$biomarkers)
- Subset of 3,169 blind duplicate samples identified among the ~121,000 measured participants

## Outputs

- Outlier-filtered concentration matrices (original and post-QC) for the 107 non-derived biomarkers
- Per-biomarker CV% (root mean square) and R2 values for original and post-QC data
- Median and IQR summaries of CV% and R2 across the 107 biomarkers, written to repeatability_metrics.csv

## How to apply

For each of the 107 non-derived NMR biomarkers, compute the median and interquartile range (IQR) of its measurements, then exclude individual measurements that fall more than 4x the IQR away from the median. Apply this filter identically to the original UK Biobank NMR concentrations and to the post-QC concentrations produced by ukbnmr::remove_technical_variation(), restricted to the 3,169 blind duplicate samples, so that CV% (root mean square approach) and R2 are computed on comparable, outlier-cleaned data. The rationale is that many NMR biomarkers have extreme outliers and highly non-normal distributions, which would otherwise dominate within-sample variance; note the ukbnmr package itself performs no outlier treatment, so this filtering must be done explicitly in the analysis script. After filtering, summarize the median and IQR of CV% and R2 across biomarkers and benchmark against the reported values (original CV% 4.55, IQR 3.21-6.03; R2 0.928, IQR 0.886-0.947; post-QC CV% 4.03, IQR 2.86-5.38; R2 0.937, IQR 0.913-0.953).

## Related tools

- **ukbnmr** (Provides extract_biomarkers() to load NMR data and remove_technical_variation() to produce the post-QC concentrations being compared; the package itself performs no outlier filtering) — https://github.com/sritchie73/ukbnmr
- **data.table** (Used with fread()/fwrite() to load the exported field data and write per-biomarker metrics and summary rows to repeatability_metrics.csv)
- **R** (Implementation environment for the per-biomarker median/IQR outlier exclusion and CV%/R2 computation)

## Examples

```
library(ukbnmr); library(data.table); processed <- remove_technical_variation(fread("exported_ukbiobank_phenotype_data.csv")); fwrite(processed$biomarkers, "nmr_biomarker_data.csv")
```

## Evaluation signals

- Median CV% across the 107 biomarkers reproduces ~4.55% (IQR 3.21-6.03) for original data and ~4.03% (IQR 2.86-5.38) post-QC
- Median R2 reproduces ~0.928 (IQR 0.886-0.947) for original data and ~0.937 (IQR 0.913-0.953) post-QC
- Exactly 107 non-derived biomarker columns remain after filtering, and only blind duplicate samples (n≈3,169, 2.6% of samples) are retained
- Post-QC CV% should decrease and R2 should increase relative to original data for the majority of biomarkers, demonstrating improved reproducibility
- repeatability_metrics.csv contains per-biomarker rows plus summary rows with median and IQR for both original and post-QC datasets

## Limitations

- Only a small fraction of samples (2.6%, the blind duplicates) can be used to assess reproducibility, limiting statistical power
- The 4x-IQR cutoff is a heuristic for repeatability analysis; many biomarkers have extreme outliers and extremely non-normal distributions whose treatment must be considered for downstream statistical modelling
- The ukbnmr R package itself performs no filtering or treatment of outlier samples when removing technical variation, so the choice of outlier rule is the analyst's responsibility
- Findings derive from the ~120,000-participant phase of the data; algorithm versions for later releases covering ~500,000 participants may differ

## Evidence

- [methods] Blind duplicate reproducibility improved by outlier-filtered CV%/R2 computation: [evidence span withheld — anchor status: not_found]
- [discussion] Outlier filtering is necessary because biomarkers are non-normal: "Many biomarkers have extreme outliers whose treatment will need to be carefully considered for statistical modelling and many biomarkers have extremely non-normal distributions."
- [discussion] The ukbnmr package performs no outlier filtering itself: "no filtering or treatment of outlier samples is performed by the ukbnmr R package when removing technical variation."
- [discussion] QC improved reproducibility of blind duplicates: "our procedure improved the reproducibility of biomarker measurements amongst blind duplicate samples; with the coefficient of variation (CV%) decreasing and coefficient of determination (R2)"
- [methods] The removal algorithm filters to 107 non-derived biomarkers: "First biomarker data is filtered to the 107 biomarkers that cannot be derived from any combination of other biomarkers."
- [readme] README warns against prefiltering before remove_technical_variation: "Note that the no prefiltering of samples or columns should be performed prior to running this function: the algorithms used for removing technical variation expect all the data to be present."
