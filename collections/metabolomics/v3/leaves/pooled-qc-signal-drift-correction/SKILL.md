---
name: pooled-qc-signal-drift-correction
description: Use when an LC–MS feature table (e.g. CSV from TidyMS or MZmine2/XCMS/MSDIAL)
  contains time-ordered pooled/intrastudy QC injections and study samples whose intensities
  show signal drift that must be corrected before statistical analysis.
when_to_use_negative:
- The feature table has not yet passed the upstream filters (Rt filter ≥ 90 s, presence
  in pooled/intrastudy QC, 10-fold blank-based area filter, RSD ≤ 20% in pooled/intrastudy
  QC, 100% intraclass prevalence with area threshold 5, D-ratio < 10%) — correction
  on unfiltered data reintroduces non-robust features.
- There are no pooled/intrastudy QC injections in the run, so no time-ordered reference
  series exists to fit the LOESS smoother.
- The goal is feature annotation or de novo peak picking rather than post-extraction
  data curation — TidyMS is noted as lacking feature annotation capability.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: TidyMS
  role: Hosts the LOESS batch-correction step inside its data curation pipeline; loads
    the feature table into a Data Container and applies per-feature LOESS fitted on
    pooled/intrastudy QC injections.
  repo: https://github.com/griquelme/tidyms
- name: statsmodels
  role: Provides the LOESS smoother implementation used to fit feature intensities
    across ordered pooled/intrastudy QC injections.
- name: scikit-learn
  role: Used to build a PCA model of the pooled QC samples to assess reduction of
    time-related systematic variance after LOESS correction.
- name: Python
  role: Runtime language for TidyMS; enables easy integration of results into state-of-the-art
    tools for model generation, such as scikit-learn.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo10100416
    title: A Python-Based Pipeline for Preprocessing LC–MS Data for Untargeted Metabolomics
      Workflows
  source_content_hash: sha256:e58f50d7ce09c61753c8ee4f03c5e8f318a0e5e4c5d5de1d280b33b1f2c8b38a
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
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# pooled-qc-signal-drift-correction

## Summary

Applies a LOESS smoother fitted on pooled/intrastudy QC sample injections (via TidyMS + statsmodels) to remove time-related systematic variance (signal drift) from LC–MS feature intensities in study samples, outputting a batch-corrected data matrix.

## When to use

Use when an LC–MS feature table (e.g. CSV from TidyMS or MZmine2/XCMS/MSDIAL) contains time-ordered pooled/intrastudy QC injections and study samples whose intensities show signal drift that must be corrected before statistical analysis.

## When NOT to use

- The feature table has not yet passed the upstream filters (Rt filter ≥ 90 s, presence in pooled/intrastudy QC, 10-fold blank-based area filter, RSD ≤ 20% in pooled/intrastudy QC, 100% intraclass prevalence with area threshold 5, D-ratio < 10%) — correction on unfiltered data reintroduces non-robust features.
- There are no pooled/intrastudy QC injections in the run, so no time-ordered reference series exists to fit the LOESS smoother.
- The goal is feature annotation or de novo peak picking rather than post-extraction data curation — TidyMS is noted as lacking feature annotation capability.

## Inputs

- QC-filtered LC–MS feature table (CSV, e.g. batch_corrected_data_matrix.csv precursor) with features as columns and samples as rows
- Sample mapping / metadata associating each injection as pooled/intrastudy QC, study sample, or blank (e.g. NIST candidate RM 8231 and SRM 1950 injections)
- Feature list restricted to features present in pooled/intrastudy QC sample injections (condition from Supporting Text 1 for performing LOESS batch correction)

## Outputs

- batch_corrected_data_matrix.csv — CSV of LOESS-corrected feature intensities for all retained features and samples
- PCA model of pooled/intrastudy QC samples (scikit-learn) demonstrating reduced time-related systematic variance after correction

## How to apply

Load the QC-filtered feature table into a TidyMS Data Container together with a sample mapping that tags pooled/intrastudy QC injections versus study samples (NIST candidate RM 8231 and SRM 1950 plasma). Restrict correction to features present in the pooled/intrastudy QC samples, since LOESS batch correction requires them (Supporting Text 1). For each retained feature, fit a LOESS smoother to intensities across the ordered QC injections using the statsmodels implementation; by default the LOESS fraction is optimized per feature via leave-one-out cross-validation (LOOCV). Apply the fitted models to study-sample intensities to remove time-related systematic variance, then verify correction quality by building a scikit-learn PCA model of the pooled QC samples and checking that time-related drift is reduced before/after correction.

## Related tools

- **TidyMS** (Hosts the LOESS batch-correction step inside its data curation pipeline; loads the feature table into a Data Container and applies per-feature LOESS fitted on pooled/intrastudy QC injections.) — https://github.com/griquelme/tidyms
- **statsmodels** (Provides the LOESS smoother implementation used to fit feature intensities across ordered pooled/intrastudy QC injections.)
- **scikit-learn** (Used to build a PCA model of the pooled QC samples to assess reduction of time-related systematic variance after LOESS correction.)
- **Python** (Runtime language for TidyMS; enables easy integration of results into state-of-the-art tools for model generation, such as scikit-learn.)

## Examples

```
from tidyms.dataContainer import DataContainer; dc = DataContainer('data_matrix.csv', sample_meta='sample_map.csv'); dc.loess_batch_correction('pooled_qc')
```

## Evaluation signals

- batch_corrected_data_matrix.csv exists as a CSV containing all LOESS-corrected features and samples (no dropped rows/columns relative to the QC-filtered input).
- All corrected intensity values in the output matrix are finite (no NaN/inf produced by the LOESS fit).
- PCA of the corrected pooled/intrastudy QC injections shows reduced time-related systematic variance (qualitative before/after drift check mirroring the paper's pooled-QC PCA comparison).
- Only features that were present in the pooled/intrastudy QC sample injections appear in the corrected matrix (Supporting Text 1 condition).
- Per-feature LOESS fraction values can be traced to the LOOCV optimization rather than a single global setting.

## Limitations

- LOESS batch correction can only be performed on features that are present in pooled/intrastudy QC sample injections — features absent from QC are excluded from this step.
- No software version numbers are stated for TidyMS, Python, or statsmodels/scikit-learn in the article, so reproducibility requires pinning versions externally.
- The article does not provide a direct repository URL or accession IDs in all sections; verify install instructions at the GitHub link cited (https://github.com/griquelme/tidyms).
- Correction quality is assessed qualitatively via pooled-QC PCA rather than by a numeric acceptance threshold.

## Evidence

- [supplementary] LOESS batch correction configured per feature: [evidence span withheld — anchor status: not_found]
- [results] Features must be present in pooled/intrastudy QC samples for LOESS correction: "The second step allowed discarding features that were not present in pooled /intrastudy QC samples (see Supporting Text 1) as a condition for performing the LOESS batch correction"
- [abstract] TidyMS pipeline capabilities include data curation: "The capabilities of the package are shown with pipelines for an LC–MS system suitability check, system conditioning, signal drift evaluation, and data curation."
- [methods] TidyMS repository and installation instructions: "Source code, documentation, and installation instructions are available online at Github: https://github.com/griquelme/tidyms"
- [discussion] Python enables integration with scikit-learn for model generation: "The implementation in Python allows easy integration of results into state-of-the-art tools for model generation, such as scikit-learn"
