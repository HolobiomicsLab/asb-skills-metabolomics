---
name: metabolomics-data-matrix-curation
description: Reference for cleaning LC–MS untargeted metabolomics feature tables (retention
  time × m/z peak areas) when raw extracted data matrices contain biologically non-relevant
  or analytically fragile features and QC sample behavior must drive which rows survive.
when_to_use_negative:
- Input is already a curated feature table that has already passed QC filtering and
  LOESS batch correction — re-applying the RSD/D-ratio cascade would double-filter
  and bias prevalence estimates.
- 'The goal is feature detection/correspondence itself (raw mzML/CDF processing: retention
  time alignment, peak picking, deisotoping, adduct grouping) rather than curation
  of an already-extracted data matrix.'
- The data matrix lacks pooled/intrastudy QC injections or blank injections — the
  blank filter, RSD filter, and LOESS batch correction all depend on these QC sample
  classes being present.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: TidyMS
  role: Python package that implements the data curation filter cascade and the surrounding
    QC pipelines (system suitability, conditioning, drift evaluation) on the extracted
    feature matrix
  repo: https://github.com/griquelme/tidyms
- name: Python
  role: Language environment in which TidyMS runs, allowing integration of curated
    results into state-of-the-art model generation tools
- name: scikit-learn
  role: Implementation used to build the PCA models that visually inspect each curation
    step and evaluate time-related systematic variance / signal drift in pooled QC
    samples
- name: statsmodels
  role: Implementation of the LOESS smoother used for batch correction of each retained
    feature after curation
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.3390/metabo10100416
    title: A Python-Based Pipeline for Preprocessing LC–MS Data for Untargeted Metabolomics
      Workflows
  source_content_hash: sha256:c2de286bc41bf6875e0bb4512dba0f62a678c76c384b0f758d897318626bf5d3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/griquelme/tidyms
  tools:
  - Python
  - TidyMS
  - scikit-learn
  - statsmodels
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# metabolomics-data-matrix-curation

## Summary

Curates an LC–MS untargeted metabolomics feature data matrix by applying the TidyMS QC filter cascade — rt filter, pooled/intrastudy QC prevalence filter, blank filtering, RSD filter, 100% intraclass prevalence filter, and D-ratio filter — before any statistical analysis.

## When to use

Reference for cleaning LC–MS untargeted metabolomics feature tables (retention time × m/z peak areas) when raw extracted data matrices contain biologically non-relevant or analytically fragile features and QC sample behavior must drive which rows survive.

## When NOT to use

- Input is already a curated feature table that has already passed QC filtering and LOESS batch correction — re-applying the RSD/D-ratio cascade would double-filter and bias prevalence estimates.
- The goal is feature detection/correspondence itself (raw mzML/CDF processing: retention time alignment, peak picking, deisotoping, adduct grouping) rather than curation of an already-extracted data matrix.
- The data matrix lacks pooled/intrastudy QC injections or blank injections — the blank filter, RSD filter, and LOESS batch correction all depend on these QC sample classes being present.

## Inputs

- Extracted LC–MS feature data matrix (samples × features of peak areas) from TidyMS feature detection and correspondence, for NIST candidate RM 8231 and SRM 1950 plasma samples
- Sample-type mapping distinguishing study samples, pooled/intrastudy QC injections, solvent blanks, zero-volume injection blanks, and process blanks
- Raw centroided or profile LC–MS data files from the MetaboLights MTBLS1919 deposit (SRM 1950 Metabolites in Frozen Human Plasma and candidate RM 8231 Frozen Human Plasma Suite: hypertriglyceridemic, diabetic, and African-American plasma)

## Outputs

- Curated (filtered) feature data matrix retaining only analytically robust features that pass the rt, QC prevalence, blank, RSD (≤20%), intraclass prevalence (area ≥ 5), and D-ratio (≤10%) filters
- Batch-corrected data matrix written as batch_corrected_data_matrix.csv after LOESS drift correction of retained features
- PCA score plots/tables of pooled/intrastudy QC samples and study samples before and after curation, written as pca_scores.csv

## How to apply

Load the extracted feature data matrix (peak areas per sample per feature) produced by feature detection and correspondence, then apply the TidyMS filter sequence from the paper's data curation pipeline. First discard features with retention times below 90 s (the LC system dead time was ~0.8 min). Next, remove features not detected in pooled/intrastudy QC sample injections, since presence in pooled/intrastudy QC injections is a condition for the subsequent LOESS batch correction. For blank filtering, set a feature's peak area in a plasma sample to zero when that area is 10-fold or less than the maximum peak area observed for the same feature in the solvent, zero-volume injection, and process blanks. Then eliminate features whose relative standard deviation (RSD) across pooled/intrastudy QC injections exceeds 20%, apply a 100% intraclass prevalence filter that zeroes all area values below a threshold area value of 5, and finally apply the D-ratio filter — the MAD of pooled/intrastudy QC samples relative to the MAD of all study samples — removing features with D-ratio of 10% or higher. Each curation step can be inspected by building PCA models (scikit-learn implementation) and comparing score plots before and after the step; at the end, the LOESS-based batch correction (with per-feature leave-one-out cross-validation of the LOESS fraction, statsmodels implementation) is performed on the surviving features. The final curated data matrix is written to batch_corrected_data_matrix.csv and PCA scores to pca_scores.csv, so downstream scikit-learn models consume only analytically robust features.

## Related tools

- **TidyMS** (Python package that implements the data curation filter cascade and the surrounding QC pipelines (system suitability, conditioning, drift evaluation) on the extracted feature matrix) — https://github.com/griquelme/tidyms
- **Python** (Language environment in which TidyMS runs, allowing integration of curated results into state-of-the-art model generation tools)
- **scikit-learn** (Implementation used to build the PCA models that visually inspect each curation step and evaluate time-related systematic variance / signal drift in pooled QC samples)
- **statsmodels** (Implementation of the LOESS smoother used for batch correction of each retained feature after curation)

## Examples

```
from tidyms import *; curated = filter_rsd(filter_blank(filter_prevalence(filter_rt(matrix))), rsd_threshold=0.20); corrected = loess_batch_correction(curated)
```

## Evaluation signals

- Each filter's thresholds are honored: features with rt < 90 s removed; blank-masked areas zeroed when sample area ≤ 10× max blank area; features with pooled/intrastudy QC RSD > 20% removed; intraclass prevalence filter zeroes areas < 5 at 100% prevalence; features with D-ratio ≥ 10% removed.
- Every retained feature is present in pooled/intrastudy QC sample injections — a required condition for the subsequent LOESS batch correction.
- PCA of pooled/intrastudy QC samples after curation shows tighter grouping and reduced time-related systematic variance (signal drift) compared with the raw data matrix, mirroring the paper's before/after score-plot comparison.
- Corrected feature values in batch_corrected_data_matrix.csv are finite and cover all samples and all features that survived the filter cascade.
- Downstream scikit-learn models receive only curated features, with no zero-imputation artifacts from the area-threshold zeroing propagating into the LOESS-corrected output.

## Limitations

- The blank filter operates on the 10-fold sample-vs-max-blank-area rule; features with low blank signal but weak analytical robustness may still pass unless caught by the RSD or D-ratio filters.
- Removal of features not present in pooled/intrastudy QC injections means those features can never receive LOESS batch correction, so they are discarded rather than retained for downstream analysis.
- TidyMS itself does not perform feature annotation (the paper's tool comparison marks annotation as ❌ for TidyMS), so curated matrices still require external annotation tools.
- The 20% RSD and 10% D-ratio cutoffs are fit-for-purpose for the plasma pooled/intrastudy QC design shown; other study designs may need re-justified thresholds.

## Evidence

- [abstract] Data curation filter cascade: "The present work introduces TidyMS, a package for the Python programming language for preprocessing LC–MS data for quality control (QC) procedures in untargeted metabolomics workflows."
- [results] rt filter threshold: "The first step was implemented to discard features with retention time values lower than 90 s, as the system dead time was approximately 0.8 min."
- [results] QC presence condition for LOESS: "The second step allowed discarding features that were not present in pooled /intrastudy QC samples (see Supporting Text 1) as a condition for performing the LOESS batch correction"
- [results] Blank filter 10× rule: [evidence span withheld — anchor status: not_found]
- [results] RSD filter 20% and D-ratio 10%: "all features with an RSD larger than 20% in pooled /intrastudy QC sample injections were eliminated."
- [results] Prevalence filter area threshold: "A 100% intraclass prevalence filter was subsequently applied with a threshold area value of 5, i.e., all area values below this threshold were set to zero."
- [results] D-ratio definition: "The prevalence filter was followed by the D-ratio filter [19], estimated as the MAD of pooled /intrastudy QC samples relative to the MAD of all study samples."
- [results] PCA as visual QC: [evidence span withheld — anchor status: not_found]
- [discussion] Downstream integration: "The implementation in Python allows easy integration of results into state-of-the-art tools for model generation, such as scikit-learn"
- [results] LOOCV-based LOESS configuration: [evidence span withheld — anchor status: not_found]
