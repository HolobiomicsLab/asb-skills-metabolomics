---
name: pca-based-drift-diagnostics
description: Use when a feature matrix contains ordered pooled/intrastudy QC injections
  and you need to quantify or visually assess signal drift, or to verify that LOESS
  batch correction reduced time-related systematic variance.
when_to_use_negative:
- The dataset contains no pooled/intrastudy QC injections ordered by acquisition time
  — drift cannot be separated from sample composition effects.
- Features have not passed QC filtering (e.g., RSD > 20% in pooled QC or D-ratio ≥
  10%), so excessive noise—not drift—may dominate the PCA.
- The goal is drift correction itself rather than drift assessment; use LOESS batch
  correction (per-feature, LOOCV-optimized) and use PCA only afterwards for validation.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: scikit-learn
  role: Provides the PCA implementation used to model pooled/intrastudy QC injections
    and evaluate reduction of time-related systematic variance before and after data
    curation
- name: TidyMS
  role: Loads and manages the QC-filtered feature table and sample mapping in a pandas-backed
    data container from which pooled QC intensities are extracted for PCA
  repo: https://github.com/griquelme/tidyms
- name: statsmodels
  role: Supplies the LOESS smoother used in the batch-correction step whose effectiveness
    this PCA diagnostics skill evaluates
- name: Python
  role: Language environment integrating TidyMS, statsmodels, and scikit-learn in
    the drift-diagnostics workflow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo10100416
    title: A Python-Based Pipeline for Preprocessing LC–MS Data for Untargeted Metabolomics
      Workflows
  source_content_hash: sha256:34e3e92919073c5f077cf374c62d13094783d6b8c3162cde2cb95e35334c6e64
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/griquelme/tidyms
  tools:
  - Python
  - TidyMS
  - scikit-learn
  - statsmodels
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# pca-based-drift-diagnostics

## Summary

Builds PCA models of pooled/intrastudy QC sample injections with scikit-learn to detect and track time-related systematic variance (signal drift) in LC-MS feature matrices, mirroring the TidyMS paper's before/after pooled-QC PCA comparison for evaluating batch correction quality.

## When to use

Use when a feature matrix contains ordered pooled/intrastudy QC injections and you need to quantify or visually assess signal drift, or to verify that LOESS batch correction reduced time-related systematic variance.

## When NOT to use

- The dataset contains no pooled/intrastudy QC injections ordered by acquisition time — drift cannot be separated from sample composition effects.
- Features have not passed QC filtering (e.g., RSD > 20% in pooled QC or D-ratio ≥ 10%), so excessive noise—not drift—may dominate the PCA.
- The goal is drift correction itself rather than drift assessment; use LOESS batch correction (per-feature, LOOCV-optimized) and use PCA only afterwards for validation.

## Inputs

- Feature matrix (CSV / pandas-backed TidyMS data container) of pooled/intrastudy QC sample injections with features retained by the QC filters (e.g., RSD < 20% in pooled QC, D-ratio < 10%)
- Sample-to-type mapping associating each injection with 'pooled/intrastudy QC' and preserving injection order
- Optionally, the batch-corrected (LOESS-corrected) feature matrix for the after-correction comparison

## Outputs

- PCA score plots of pooled QC injections (before and after correction) showing scores along leading PCs versus injection order
- Qualitative assessment of whether time-related systematic variance was reduced after LOESS batch correction
- Optionally, a saved diagnostics artifact (e.g., PCA scores CSV or plot) accompanying batch_corrected_data_matrix.csv

## How to apply

Assemble the feature intensities of pooled/intrastudy QC samples from the (QC-filtered) TidyMS data container, keeping injection order so drift appears along the acquisition-time axis. Fit a PCA model using scikit-learn and project the pooled QC injections onto the leading principal components; systematic drift manifests as scores that trend monotonically with injection order rather than scattering randomly. Run the PCA both before and after data curation steps (e.g., LOESS batch correction, in which the LOESS fraction of samples is optimized per feature by leave-one-out cross-validation) and compare the score-versus-time trends: successful correction removes the time-related systematic variance so corrected pooled QC scores no longer trend with injection order. This qualitative before/after comparison is exactly the acceptance check the paper applies to pooled QC samples.

## Related tools

- **scikit-learn** (Provides the PCA implementation used to model pooled/intrastudy QC injections and evaluate reduction of time-related systematic variance before and after data curation)
- **TidyMS** (Loads and manages the QC-filtered feature table and sample mapping in a pandas-backed data container from which pooled QC intensities are extracted for PCA) — https://github.com/griquelme/tidyms
- **statsmodels** (Supplies the LOESS smoother used in the batch-correction step whose effectiveness this PCA diagnostics skill evaluates)
- **Python** (Language environment integrating TidyMS, statsmodels, and scikit-learn in the drift-diagnostics workflow)

## Examples

```
from sklearn.decomposition import PCA; import pandas as pd; qc = pd.read_csv('qc_features.csv'); scores = PCA(n_components=2).fit_transform(qc.drop(columns=['injection_order']))
```

## Evaluation signals

- Before correction: pooled QC scores on leading PCs trend with injection order (monotonic time-related pattern), confirming drift is present.
- After LOESS correction: pooled QC PCA scores no longer trend with injection order, mirroring the paper's before/after pooled-QC PCA comparison.
- All corrected values feeding the PCA are finite (no NaN/Inf introduced by LOESS fitting or correction).
- PCA is computed only on features present in pooled/intrastudy QC injections, per Supporting Text 1's condition for LOESS batch correction.
- The corrected matrix is exported (e.g., batch_corrected_data_matrix.csv) with all LOESS-corrected features and samples intact, so the PCA input matches the shipped output.

## Limitations

- The drift assessment is qualitative (visual/structural score-trend comparison), not a formal quantitative metric with a defined cutoff.
- Drift evaluation requires pooled/intrastudy QC injections injected in known order across the analytical batch; without ordered QC injections the approach is not applicable.
- PCA of pooled QC reflects only time-related systematic variance; other quality dimensions (e.g., blank contamination, 10-fold sample/blank peak-area rule, 100% intraclass prevalence with area threshold of 5) need separate filters.
- No software version numbers for TidyMS, Python, or scikit-learn are stated in the paper, so exact numeric reproducibility of PCA outputs may vary across library versions.

## Evidence

- [other] Track correction quality by building a PCA model (scikit-learn implementation) of the pooled QC samples to evaluate the reduction of time-related systematic variance: [evidence span withheld — anchor status: not_found]
- [other] PCA of corrected pooled QC shows reduced drift: [evidence span withheld — anchor status: not_found]
- [abstract] TidyMS QC pipelines: "The capabilities of the package are shown with pipelines for an LC–MS system suitability check, system conditioning, signal drift evaluation, and data curation."
- [results] RSD filter in pooled QC: "all features with an RSD larger than 20% in pooled /intrastudy QC sample injections were eliminated."
- [results] LOESS LOOCV configuration: [evidence span withheld — anchor status: not_found]
- [intro] scikit-learn availability: "Python packages include widely used machine learning frameworks, such as scikit-learn [27] and Tensor Flow [28],"
