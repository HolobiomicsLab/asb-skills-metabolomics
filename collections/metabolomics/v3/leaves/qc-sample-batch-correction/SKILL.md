---
name: qc-sample-batch-correction
description: Decision support for when batch-level signal drift is suspected in pooled/intrastudy
  QC samples across an LC–MS run, and those QC samples represent the full analytical
  batch.
when_to_use_negative:
- When the input is already a curated or normalized feature table that has already
  undergone drift correction, the LOESS-based batch correction step should not be
  re-applied.
- When the dataset lacks pooled/intrastudy QC injections that span the full analytical
  batch, drift correction cannot be reliably estimated.
- When the goal is feature annotation or compound identification rather than correcting
  drift-related systematic variance, this skill is not the appropriate step.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: TidyMS
  role: Provides the mzML reader, CWT-based peak picking, and QC feature extraction
    pipeline used to assemble QC injection intensities for drift evaluation and batch
    correction.
  repo: https://github.com/griquelme/tidyms
- name: Python
  role: Runtime environment in which the TidyMS pipeline, scikit-learn PCA, and statsmodels
    LOESS smoother are executed.
- name: scikit-learn
  role: Implements the PCA models used to evaluate drift-related systematic variance
    across pooled/intrastudy QC samples.
- name: statsmodels
  role: Provides the LOESS smoother used to model and correct drift over injection
    time in the batch correction step.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo10100416
    title: A Python-Based Pipeline for Preprocessing LC–MS Data for Untargeted Metabolomics
      Workflows
  source_content_hash: sha256:32627c504e8a0c63dacce301c3dd0f20e1c143e72216392fad5ecbe9d15d5edd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/griquelme/tidyms
  tools:
  - Python
  - TidyMS
  - scikit-learn
  - statsmodels
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# qc-sample-batch-correction

## Summary

This skill applies the TidyMS workflow for evaluating and correcting LC–MS signal drift using pooled/intrastudy QC samples, using PCA-based drift detection followed by LOESS-fitted batch correction.

## When to use

Decision support for when batch-level signal drift is suspected in pooled/intrastudy QC samples across an LC–MS run, and those QC samples represent the full analytical batch.

## When NOT to use

- When the input is already a curated or normalized feature table that has already undergone drift correction, the LOESS-based batch correction step should not be re-applied.
- When the dataset lacks pooled/intrastudy QC injections that span the full analytical batch, drift correction cannot be reliably estimated.
- When the goal is feature annotation or compound identification rather than correcting drift-related systematic variance, this skill is not the appropriate step.

## Inputs

- Raw mzML files of pooled/intrastudy QC sample injections from an LC–MS run (e.g. NIST SRM 1950 / candidate RM 8231 plasma dataset, MTBLS1919)
- Sample metadata table mapping each injection to its sample type and acquisition order
- Extracted feature table with intensity values for all detected features across QC and study samples

## Outputs

- Corrected feature intensity matrix with LOESS-fitted drift removed
- Signal drift evaluation artifact (plot/table of QC signal versus injection order with the fitted drift trend, e.g. signal_drift_evaluation.png)
- QC outlier flag list (injections identified as potential outliers via PCA)

## How to apply

First, order the raw mzML injections of pooled/intrastudy QC samples by acquisition time and extract feature intensities with the TidyMS CWT peak-picking algorithm (for chromatographic data). Next, build PCA models on the pooled/intrastudy QC intensities to evaluate time-related systematic variance or signal drift, and flag potential outlier injections. Then, for each feature present in pooled/intrastudy QC samples, fit a LOESS smoother (statsmodels implementation) to the QC feature intensities versus injection time, as described in Supporting Text 1, and use it to correct drift across the batch. Retain only features that were present in the pooled/intrastudy QC samples as a precondition for the LOESS correction, and discard features with retention time below 90 s (system dead time ≈ 0.8 min) before correction. The corrected intensities are validated by confirming that drift-related systematic variance (e.g. the PC1 trend in QC samples) is removed after correction.

## Related tools

- **TidyMS** (Provides the mzML reader, CWT-based peak picking, and QC feature extraction pipeline used to assemble QC injection intensities for drift evaluation and batch correction.) — https://github.com/griquelme/tidyms
- **Python** (Runtime environment in which the TidyMS pipeline, scikit-learn PCA, and statsmodels LOESS smoother are executed.)
- **scikit-learn** (Implements the PCA models used to evaluate drift-related systematic variance across pooled/intrastudy QC samples.)
- **statsmodels** (Provides the LOESS smoother used to model and correct drift over injection time in the batch correction step.)

## Examples

```
from tidyms.utils import plot_qc_drift; plot_qc_drift(signal_drift_evaluation.png)
```

## Evaluation signals

- After correction, a PCA model on the corrected pooled/intrastudy QC intensities should no longer show a monotonic drift trend along PC1 corresponding to injection order.
- A generated drift-evaluation artifact (plot/table of QC signal versus injection order with the fitted LOESS trend) should be produced and visually/numerically summarized for the batch.
- Features absent from pooled/intrastudy QC samples must be excluded from the LOESS correction step, consistent with the precondition for applying the correction.
- Only features with retention time ≥ 90 s (system dead time ≈ 0.8 min) should be included in the drift-evaluation and correction pipeline.
- QC outlier injections flagged by PCA should be identified and documented before applying the correction.

## Limitations

- Drift correction via LOESS requires that pooled/intrastudy QC samples were injected regularly throughout the batch; sparse or clustered QC injections will degrade the drift estimate.
- The 90 s retention-time cutoff (system dead time ≈ 0.8 min) is specific to this LC system; applying it to other platforms without checking the actual dead time can discard valid features.
- The pipeline was developed and demonstrated on plasma reference materials (NIST SRM 1950 and candidate RM 8231); matrices with very different feature prevalence may require retuning the QC-presence precondition and RSD thresholds.
- Feature annotation is not provided by TidyMS, so corrected intensities are not automatically mapped back to annotated compounds.

## Evidence

- [abstract] PCA score plots built from pooled/intrastudy QC samples revealed the presence of signal drift along PC1, and this trend was removed after applying the LOESS batch correction during data curation.: [evidence span withheld — anchor status: not_found]
- [results] The second step allowed discarding features that were not present in pooled /intrastudy QC samples (see Supporting Text 1) as a condition for performing the LOESS batch correction: "The second step allowed discarding features that were not present in pooled /intrastudy QC samples (see Supporting Text 1) as a condition for performing the LOESS batch correction"
- [results] The first step was implemented to discard features with retention time values lower than 90 s, as the system dead time was approximately 0.8 min.: "The first step was implemented to discard features with retention time values lower than 90 s, as the system dead time was approximately 0.8 min."
- [methods] The mass spectrometry data have been deposited to the Metabo Lights public repository [38] with the data set identifier MTBLS1919.: "The mass spectrometry data have been deposited to the Metabo Lights public repository [38] with the data set identifier MTBLS1919."
- [methods] Source code, documentation, and installation instructions are available online at Github: https://github.com/griquelme/tidyms: "Source code, documentation, and installation instructions are available online at Github: https://github.com/griquelme/tidyms"
