---
name: lcms-feature-quality-filtering
description: Use when an extracted LC–MS feature matrix (mzML-derived peaks, TidyMS
  Data Container, MZmine/XCMS output) needs cleaning before statistical analysis and
  you have pooled/intrastudy QC, blank, and study samples to drive reproducibility
  and background filters.
when_to_use_negative:
- The input is already a cleaned, curated feature matrix with only analytically robust
  features retained — re-running the filters would unnecessarily discard features.
- The dataset lacks pooled/intrastudy QC samples or blanks, making LOESS-correction-presence
  and blank-based filters impossible to apply as described.
- The goal is targeted metabolomics or feature annotation rather than removing biologically
  non-relevant features (retention time, m/z pairs) for untargeted QC; TidyMS's curation
  pipeline does not provide annotation.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: TidyMS
  role: Python package implementing the sequential curation pipeline (Rt, QC-presence,
    blank, RSD, prevalence, D-ratio filters) on a Pandas-based Data Container
  repo: https://github.com/griquelme/tidyms
- name: pyopenms
  role: Reads raw mzML mass spectrometry files to build the extracted feature table
    used as input
- name: scikit-learn
  role: PCA implementation used to build score plots that verify the effect of curation
- name: Pandas
  role: Back-end of the TidyMS Data Container that stores the data matrix and metadata
    during filtering
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo10100416
    title: A Python-Based Pipeline for Preprocessing LC–MS Data for Untargeted Metabolomics
      Workflows
  source_content_hash: sha256:75e49e57180a75badfb2fb6b992b718bf9c7267c6a0b26ce218f8d418128bc53
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/griquelme/tidyms
  tools:
  - Pandas
  - TidyMS
  - pyopenms
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# lcms-feature-quality-filtering

## Summary

Curates an LC–MS untargeted metabolomics feature table by sequentially removing low-quality features (early elution, blank contamination, poor QC repeatability, low prevalence, high drift) so only robust features feed downstream statistical analysis, as implemented in TidyMS for NIST plasma reference materials.

## When to use

Use when an extracted LC–MS feature matrix (mzML-derived peaks, TidyMS Data Container, MZmine/XCMS output) needs cleaning before statistical analysis and you have pooled/intrastudy QC, blank, and study samples to drive reproducibility and background filters.

## When NOT to use

- The input is already a cleaned, curated feature matrix with only analytically robust features retained — re-running the filters would unnecessarily discard features.
- The dataset lacks pooled/intrastudy QC samples or blanks, making LOESS-correction-presence and blank-based filters impossible to apply as described.
- The goal is targeted metabolomics or feature annotation rather than removing biologically non-relevant features (retention time, m/z pairs) for untargeted QC; TidyMS's curation pipeline does not provide annotation.

## Inputs

- Extracted LC–MS feature table (retention time, m/z, peak areas) from feature detection and correspondence (e.g. mzML raw data read with pyopenms, MetaboLights MTBLS1919 deposit)
- Sample-class annotations distinguishing study samples (NIST candidate RM 8231, SRM 1950 plasma), pooled/intrastudy QC samples, and solvent, zero-volume-injection, and process blanks
- TidyMS Data Container (Pandas-based) holding the data matrix and metadata

## Outputs

- Curated data matrix containing only features passing all filter criteria (Rt >= 90 s, blank ratio > 10x, pooled-QC RSD <= 20%, 100% intraclass prevalence at area >= 5, D-ratio < 10%)
- Filter statistics for each curation step (e.g. number of features removed per filter)
- PCA score plots (scikit-learn) to visualize the effect of curation on pooled/intrastudy QC samples

## How to apply

Load the extracted feature table (e.g. from MetaboLights MTBLS1919, mzML read with pyopenms, handled as a TidyMS Data Container built on Pandas) together with sample-class annotations for study samples, pooled/intrastudy QC samples, and solvent, zero-volume-injection, and process blanks. Apply the filters in a fixed order: (1) retention-time filter discarding features eluting below 90 s because the system dead time was ~0.8 min; (2) discard features absent from pooled/intrastudy QC samples, which is a precondition for LOESS batch correction; (3) blank-based filter that sets a feature's peak area to zero in a plasma sample when it is 10-fold or less than the maximum peak area of the same feature in the blanks; (4) RSD filter removing features with RSD larger than 20% across pooled/intrastudy QC injections; (5) 100% intraclass prevalence filter with a threshold area value of 5, setting all area values below this threshold to zero; (6) D-ratio filter, defined as the MAD of pooled/intrastudy QC samples relative to the MAD of all study samples, removing features with a D-ratio of 10% or higher. Each step is a decision point whose thresholds can be tuned for fit-for-purpose curation, and the order matters because later filters (e.g. LOESS correction, D-ratio) assume features present in QC samples. Assemble the surviving features into the curated data matrix and monitor the effect of curation with PCA score plots built with the scikit-learn implementation; for the NIST candidate RM 8231 and SRM 1950 plasma study this pipeline yielded a final matrix of 665 metabolic features.

## Related tools

- **TidyMS** (Python package implementing the sequential curation pipeline (Rt, QC-presence, blank, RSD, prevalence, D-ratio filters) on a Pandas-based Data Container) — https://github.com/griquelme/tidyms
- **pyopenms** (Reads raw mzML mass spectrometry files to build the extracted feature table used as input)
- **scikit-learn** (PCA implementation used to build score plots that verify the effect of curation)
- **Pandas** (Back-end of the TidyMS Data Container that stores the data matrix and metadata during filtering)

## Examples

```
from tidyms.data import DataContainer; dc = DataContainer.from_mzml('MTBLS1919', ms_level=1); dc.apply_filters(rsd_qc=0.2, d_ratio=0.1, rt_min=90, blank_factor=10)
```

## Evaluation signals

- After filtering, every retained feature must satisfy all thresholds: Rt >= 90 s, peak area in sample > 10x maximum area in blanks, RSD <= 20% in pooled/intrastudy QC, 100% prevalence at area >= 5, D-ratio < 10%.
- Features absent from pooled/intrastudy QC samples must be removed before LOESS batch correction is attempted.
- For the NIST candidate RM 8231 / SRM 1950 study, the pipeline should reproduce the reported outcome of 665 curated metabolic features.
- PCA score plots built on pooled/intrastudy QC samples after curation should show the expected clustering/tightening consistent with the article's post-curation plots.
- Per-step filter statistics should be inspectable and reproducible from the pipeline output.

## Limitations

- Thresholds (90 s Rt, 10x blank ratio, 20% RSD, area 5, 10% D-ratio) were tuned for this LC–MS plasma study and system dead time (~0.8 min); they should be re-validated or customized for other platforms.
- TidyMS as described does not perform feature annotation — annotation capability was noted as absent in the comparative table.
- The pipeline requires pooled/intrastudy QC samples (LOESS-correction precondition) and blank samples; datasets lacking these classes cannot fully reproduce the workflow.
- Data normalization strategies applied before statistical analysis can dramatically influence results, so curation choices interact with upstream/downstream processing decisions.

## Evidence

- [results] First, the retention-time filter removed early-eluting features; the Rt threshold was 90 s because the system dead time was approximately 0.8 min.: "The first step was implemented to discard features with retention time values lower than 90 s, as the system dead time was approximately 0.8 min."
- [results] The blank-based filter zeroed a sample's peak area when it was 10-fold or less than the maximum blank peak area.: [evidence span withheld — anchor status: not_found]
- [results] The RSD filter eliminated all features with RSD larger than 20% in pooled/intrastudy QC sample injections.: "all features with an RSD larger than 20% in pooled /intrastudy QC sample injections were eliminated."
- [results] A 100% intraclass prevalence filter was applied with a threshold area value of 5, zeroing all area values below this threshold.: "A 100% intraclass prevalence filter was subsequently applied with a threshold area value of 5, i.e., all area values below this threshold were set to zero."
- [results] The D-ratio filter was estimated as the MAD of pooled/intrastudy QC samples relative to the MAD of all study samples.: "The prevalence filter was followed by the D-ratio filter [19], estimated as the MAD of pooled /intrastudy QC samples relative to the MAD of all study samples."
- [results] The second step discarded features not present in pooled/intrastudy QC samples as a condition for performing the LOESS batch correction.: "The second step allowed discarding features that were not present in pooled /intrastudy QC samples (see Supporting Text 1) as a condition for performing the LOESS batch correction"
- [intro] Source code, documentation, and installation instructions are available on Github at https://github.com/griquelme/tidyms.: "Source code, documentation, and installation instructions are available online at Github: https://github.com/griquelme/tidyms"
- [results] Feature annotation is absent from the TidyMS capability table.: "Feature annotation ❌ ✅ ❌ ✅ ❌ ❌ ❌"
- [abstract] Preprocessing LC–MS untargeted metabolomics data involves the removal of biologically non-relevant features (retention time, m/z pairs) to retain only high-quality data.: "the removal of biologically non-relevant features (retention time, m/z pairs) to retain only high-quality data"
