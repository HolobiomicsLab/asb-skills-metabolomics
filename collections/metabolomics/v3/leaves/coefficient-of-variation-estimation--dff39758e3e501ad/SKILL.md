---
name: coefficient-of-variation-estimation
description: Use when replicate (blind duplicate) measurements are available for NMR
  metabolic biomarker data and you need to quantify and compare measurement reproducibility
  (CV%, R2) before and after a quality control or technical-variation-removal procedure.
when_to_use_negative:
- No replicate/blind duplicate measurements exist in the dataset, so reproducibility
  cannot be assessed within-sample
- You intend to assess the QC procedure itself rather than quantify reproducibility
  — remove_technical_variation() requires unfiltered input and performs no outlier
  filtering of samples itself
- You are working with derived/composite biomarkers or ratios, which should be recomputed
  post-adjustment from the 107 non-derived biomarkers instead
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3272
tools:
- name: ukbnmr
  role: R package providing remove_technical_variation() to produce the post-QC biomarker
    concentrations whose reproducibility is quantified
  repo: https://github.com/sritchie73/ukbnmr
- name: R
  role: Analysis environment in which the CV% (root mean square approach) and R2 across
    blind duplicates are computed
- name: data.table
  role: Used for fast reading and writing of CSV files (fread/fwrite), including the
    final repeatability_metrics.csv
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41597-023-01949-y
    title: Quality control and removal of technical variation of NMR metabolic biomarker
      data in ~120,000 UK Biobank participants
  source_content_hash: sha256:f3841f60d958580a897dd6dd8e7e09ea0ef276a3a27079baaf24ccea5303dad8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sritchie73/ukbnmr
  tools:
  - R
  - data.table
  - ukbnmr
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# coefficient-of-variation-estimation

## Summary

Quantifies measurement reproducibility of UK Biobank NMR metabolic biomarkers by computing within-sample coefficient of variation (CV%) via the root mean square approach and coefficient of determination (R2) across 3,169 blind duplicate samples, for the 107 non-derived biomarkers, both on original and QC-processed (remove_technical_variation) concentrations.

## When to use

Use when replicate (blind duplicate) measurements are available for NMR metabolic biomarker data and you need to quantify and compare measurement reproducibility (CV%, R2) before and after a quality control or technical-variation-removal procedure.

## When NOT to use

- No replicate/blind duplicate measurements exist in the dataset, so reproducibility cannot be assessed within-sample
- You intend to assess the QC procedure itself rather than quantify reproducibility — remove_technical_variation() requires unfiltered input and performs no outlier filtering of samples itself
- You are working with derived/composite biomarkers or ratios, which should be recomputed post-adjustment from the 107 non-derived biomarkers instead

## Inputs

- Original UK Biobank NMR biomarker data extracted via extract_biomarkers() (data.frame, one column per biomarker, rows keyed by eid and visit_index)
- QC-processed biomarker data from ukbnmr::remove_technical_variation()
- Blind duplicate sample subset (3,169 duplicate sample pairs)
- List of the 107 non-derived NMR biomarkers

## Outputs

- Per-biomarker CV% and R2 metrics for original and post-QC concentrations
- Summary rows with median and IQR of CV% and R2 across the 107 biomarkers
- repeatability_metrics.csv written with per-biomarker metrics and summary rows

## How to apply

Load the original NMR biomarker data and the output of ukbnmr::remove_technical_variation(), filter to the 107 non-derived biomarkers, and restrict to the blind duplicate samples (~2.6% of samples, 3,169 duplicates). For each biomarker, exclude measurements that are outliers beyond 4x the interquartile range from the median to avoid extreme outliers distorting the metrics. Compute the within-sample CV% using the root mean square approach and the coefficient of determination (R2) across duplicate measurements for each biomarker, separately on original and post-QC concentrations. Summarize median and IQR of CV% and R2 across the 107 biomarkers and compare against expected values: CV% 4.55 (IQR 3.21-6.03) original vs 4.03 (IQR 2.86-5.38) post-QC; R2 0.928 (IQR 0.886-0.947) original vs 0.937 (IQR 0.913-0.953) post-QC.

## Related tools

- **ukbnmr** (R package providing remove_technical_variation() to produce the post-QC biomarker concentrations whose reproducibility is quantified) — https://github.com/sritchie73/ukbnmr
- **R** (Analysis environment in which the CV% (root mean square approach) and R2 across blind duplicates are computed)
- **data.table** (Used for fast reading and writing of CSV files (fread/fwrite), including the final repeatability_metrics.csv)

## Examples

```
processed <- ukbnmr::remove_technical_variation(exported); fwrite(processed$biomarkers, file="nmr_biomarker_data.csv")
```

## Evaluation signals

- Median CV% across the 107 non-derived biomarkers on original data matches 4.55% (IQR 3.21-6.03%)
- Median CV% after QC matches 4.03% (IQR 2.86-5.38%), i.e. CV% decreases after remove_technical_variation()
- Median R2 matches 0.928 (IQR 0.886-0.947) original and 0.937 (IQR 0.913-0.953) post-QC, i.e. R2 increases after QC
- repeatability_metrics.csv contains both per-biomarker rows and summary rows, restricted to blind duplicate samples
- Outlier exclusion (beyond 4x IQR from the median) was applied per biomarker before computing metrics

## Limitations

- Only a small fraction of samples (2.6%) could be assessed for reproducibility via blind duplicates, limiting the precision of the reproducibility assessment
- Extreme outliers must be carefully handled; many biomarkers have extreme outliers and extremely non-normal distributions, hence the 4x IQR exclusion step
- The QC procedure regresses out technical covariates rather than calibrating to quality control samples, and could in principle remove biological variation of interest

## Evidence

- [discussion] QC improved blind duplicate reproducibility: "our procedure improved the reproducibility of biomarker measurements amongst blind duplicate samples; with the coefficient of variation (CV%) decreasing and coefficient of determination (R2)"
- [discussion] Only 2.6% of samples assessable via blind duplicates: "although with the caveat that only a small fraction of samples (2.6%) could be assessed in this way."
- [methods] Filtering to 107 non-derived biomarkers: "First biomarker data is filtered to the 107 biomarkers that cannot be derived from any combination of other biomarkers."
- [readme] ukbnmr performs no outlier filtering itself: "Note that the no prefiltering of samples or columns should be performed prior to running this function: the algorithms used for removing technical variation expect all the data to be present."
- [readme] remove_technical_variation usage: "processed <- remove_technical_variation(exported)"
