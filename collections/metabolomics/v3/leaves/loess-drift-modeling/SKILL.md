---
name: loess-drift-modeling
description: Use when pooled/intrastudy QC samples in an LC–MS batch (e.g., mzML files
  from MetaboLights MTBLS1919) show time-related systematic variance — signal drift
  along PC1 of a PCA score plot — that must be modeled and removed over injection
  order.
when_to_use_negative:
- Features that are not present in the pooled/intrastudy QC samples — the article
  requires discarding these as a condition for performing LOESS batch correction.
- Datasets lacking repeated pooled/intrastudy QC injections or their acquisition-order
  metadata, since the smoother needs QC intensity as a function of injection time.
- Data already drift-corrected or preprocessed by another tool's normalization/batch-correction
  stage — applying LOESS again would double-correct.
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: TidyMS
  role: Python package whose data-curation pipeline performs the LOESS batch correction
    on QC feature intensities
  repo: https://github.com/griquelme/tidyms
- name: statsmodels
  role: Provides the LOESS smoother implementation applied to QC feature intensities
    over injection time
- name: scikit-learn
  role: PCA models built from pooled/intrastudy QC samples to detect time-related
    systematic variance before and after drift correction
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo10100416
    title: A Python-Based Pipeline for Preprocessing LC–MS Data for Untargeted Metabolomics
      Workflows
  source_content_hash: sha256:e8b9643e710d13eb74f54deceb5a767f76e9a4f0cdb2ccb4c859bdd9989c1cff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/griquelme/tidyms
  tools:
  - TidyMS
  - scikit-learn
  - statsmodels
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# loess-drift-modeling

## Summary

Models and corrects instrument signal drift in untargeted LC–MS batches by fitting a LOESS smoother (statsmodels implementation, as used by TidyMS) to each feature's intensity across pooled/intrastudy QC injections in acquisition order, then applying the fitted trend as batch correction during TidyMS data curation.

## When to use

Use when pooled/intrastudy QC samples in an LC–MS batch (e.g., mzML files from MetaboLights MTBLS1919) show time-related systematic variance — signal drift along PC1 of a PCA score plot — that must be modeled and removed over injection order.

## When NOT to use

- Features that are not present in the pooled/intrastudy QC samples — the article requires discarding these as a condition for performing LOESS batch correction.
- Datasets lacking repeated pooled/intrastudy QC injections or their acquisition-order metadata, since the smoother needs QC intensity as a function of injection time.
- Data already drift-corrected or preprocessed by another tool's normalization/batch-correction stage — applying LOESS again would double-correct.

## Inputs

- Feature intensity matrix extracted from mzML raw files (TidyMS, CWT-based peak picking) for pooled/intrastudy QC injections
- Sample metadata (acquisition/injection order) for the QC injections
- PCA assessment (scikit-learn) of QC samples indicating drift-related systematic variance along PC1

## Outputs

- Per-feature LOESS-fitted drift trend of QC signal versus injection order
- Drift-evaluation artifact: plot/table of QC signal versus injection order with fitted drift trend (signal_drift_evaluation.png)
- Drift-corrected feature intensities for study samples via TidyMS batch correction (data curation step)

## How to apply

First process the raw mzML data with TidyMS (CWT-based peak picking) and assemble the pooled/intrastudy QC injections — created from equal aliquots of every plasma sample — in acquisition order, tracking each feature's intensity across the batch. Build PCA models (scikit-learn) on the QC samples to detect drift-related systematic variance along PC1 and flag potential outlier injections. Restrict drift correction to features present in the pooled/intrastudy QC samples (the article's data-curation pipeline explicitly discards features absent from QC as a precondition for LOESS batch correction), then apply the LOESS smoother (statsmodels implementation, as used for TidyMS batch correction in Supporting Text 1) to each feature's QC intensities as a function of injection time. The fitted drift trend is subtracted/applied during data curation to remove the time-related variance from study-sample intensities. Rationale: QC injections repeated throughout the run sample the instrumental drift, and LOESS robustly captures its non-linear time course so batch correction can be performed per feature.

## Related tools

- **TidyMS** (Python package whose data-curation pipeline performs the LOESS batch correction on QC feature intensities) — https://github.com/griquelme/tidyms
- **statsmodels** (Provides the LOESS smoother implementation applied to QC feature intensities over injection time)
- **scikit-learn** (PCA models built from pooled/intrastudy QC samples to detect time-related systematic variance before and after drift correction)

## Examples

```
from statsmodels.nonparametric.smoothers_lowess import lowess; drift_fit = lowess(qc_intensity, injection_order, return_sorted=False)
```

## Evaluation signals

- PCA score plots of pooled/intrastudy QC samples show drift along PC1 before correction, and this trend is removed after LOESS batch correction (the article's Application 1 outcome).
- The drift-evaluation artifact (signal_drift_evaluation.png) visually and numerically summarizes QC signal versus injection order with the fitted LOESS drift trend.
- Features absent from pooled/intrastudy QC samples were excluded prior to LOESS correction, matching the article's precondition filter.
- Drift-related systematic variance in pooled QC samples is detected and summarized for the batch, consistent with the paper's signal drift evaluation pipeline.

## Limitations

- LOESS batch correction is only defined for features detected in the pooled/intrastudy QC samples; features missing from QC must be filtered out first.
- The article does not state LOESS smoothing parameters (e.g., span) or software version numbers for TidyMS, Python, or statsmodels, so exact reproduction depends on Supporting Text 1 / the tidyms-notebooks repository.
- The observed drift behavior is specific to the NIST candidate RM 8231 / SRM 1950 plasma runs (MTBLS1919); drift magnitude and correction performance may differ for other instruments and study designs.

## Evidence

- [results] The second step allowed discarding features that were not present in pooled /intrastudy QC samples (see Supporting Text 1) as a condition for performing the LOESS batch correction: "The second step allowed discarding features that were not present in pooled /intrastudy QC samples (see Supporting Text 1) as a condition for performing the LOESS batch correction"
- [abstract] The capabilities of the package are shown with pipelines for an LC–MS system suitability check, system conditioning, signal drift evaluation, and data curation.: "The capabilities of the package are shown with pipelines for an LC–MS system suitability check, system conditioning, signal drift evaluation, and data curation."
- [results] The running time needed to analyze the 31 raw data files used to generate the results shown in Application 1 was 66 s.: "The running time needed to analyze the 31 raw data files used to generate the results shown in Application 1 was 66 s."
