---
name: rsd-repeatability-assessment-in-qc-samples
description: Use when curating an untargeted LC–MS feature table where pooled/intrastudy
  QC injections are available and features with poor analytical repeatability (RSD
  > 20%) must be removed before statistical analysis.
when_to_use_negative:
- The dataset has no pooled/intrastudy QC injections or reference-material samples
  to serve as repeated measurements — RSD cannot be estimated without replicate QC
  data
- You are evaluating inter-batch/inter-laboratory robustness rather than within-run
  repeatability; the D-ratio filter (MAD of QC vs. MAD of study samples) targets that
  complementary question
- The input is already a fully curated matrix where low-RSD filtering was previously
  applied; reapplying would not change the result and would lose provenance of earlier
  filter steps
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: TidyMS
  role: Python package that implements the data curation pipeline, including the RSD-based
    repeatability filter on pooled/intrastudy QC injections, using its Pandas-backed
    Data Container
  repo: https://github.com/griquelme/tidyms
- name: pyopenms
  role: Reads the mzML raw MS files from the MetaboLights MTBLS1919 deposit upstream
    of the RSD assessment, producing the feature table on which the filter operates
- name: scikit-learn
  role: Used for post-filter quality inspection (PCA score plots) to verify that curated
    features, including those passing the RSD filter, preserve the pooled-QC clustering
    pattern
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo10100416
    title: A Python-Based Pipeline for Preprocessing LC–MS Data for Untargeted Metabolomics
      Workflows
  source_content_hash: sha256:3473c47cc833378824d90bc239f202e27289e9ed9a38ff47a461950a3ac9eed5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/griquelme/tidyms
  tools:
  - TidyMS
  - pyopenms
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# rsd-repeatability-assessment-in-qc-samples

## Summary

Evaluate repeatability of metabolic features by computing the relative standard deviation (RSD) of peak areas across pooled/intrastudy QC sample injections, and eliminate features with RSD larger than 20% in the LC–MS data curation pipeline demonstrated with NIST candidate RM 8231 and SRM 1950 plasma reference materials.

## When to use

Use when curating an untargeted LC–MS feature table where pooled/intrastudy QC injections are available and features with poor analytical repeatability (RSD > 20%) must be removed before statistical analysis.

## When NOT to use

- The dataset has no pooled/intrastudy QC injections or reference-material samples to serve as repeated measurements — RSD cannot be estimated without replicate QC data
- You are evaluating inter-batch/inter-laboratory robustness rather than within-run repeatability; the D-ratio filter (MAD of QC vs. MAD of study samples) targets that complementary question
- The input is already a fully curated matrix where low-RSD filtering was previously applied; reapplying would not change the result and would lose provenance of earlier filter steps

## Inputs

- Extracted LC–MS feature data matrix (features × samples, peak-area values) produced by feature detection and correspondence, e.g. loaded from the MTBLS1919 MetaboLights deposit (mzML raw files processed with pyopenms)
- Sample-class metadata identifying which injections are pooled/intrastudy QC samples versus study samples (NIST candidate RM 8231 and SRM 1950 plasma samples)

## Outputs

- RSD value per feature computed across pooled/intrastudy QC sample injections
- Filtered feature matrix retaining only features with QC RSD ≤ 20%
- Filter statistics showing how many features were eliminated by the 20% RSD criterion

## How to apply

Assemble the extracted feature matrix (e.g. from mzML files processed with pyopenms and handled in TidyMS's Pandas-based Data Container) and restrict the RSD computation to pooled/intrastudy QC sample injections that bracket the analytical run. For each feature, compute the RSD (standard deviation / mean, typically as a percentage) of its peak areas across those QC injections; this metric quantifies signal drift and injection-to-injection variability under repeated measurement of identical material. Apply the decision threshold from the article: eliminate all features with an RSD larger than 20% in pooled/intrastudy QC injections. The rationale is that features whose peak areas vary more than 20% even when the same pooled plasma reference material is repeatedly injected are not analytically robust enough for downstream univariate or multivariate analysis. Run this filter in sequence after the retention-time (Rt ≥ 90 s), QC-presence, and blank-based (10-fold vs. solvent/zero-volume/process blanks) filters, and before the 100% intraclass prevalence (area threshold 5) and D-ratio (10% cutoff, MAD of QC vs. MAD of study samples) filters, so that the surviving feature count (665 features for the NIST materials in the paper) can be tracked per filter step.

## Related tools

- **TidyMS** (Python package that implements the data curation pipeline, including the RSD-based repeatability filter on pooled/intrastudy QC injections, using its Pandas-backed Data Container) — https://github.com/griquelme/tidyms
- **pyopenms** (Reads the mzML raw MS files from the MetaboLights MTBLS1919 deposit upstream of the RSD assessment, producing the feature table on which the filter operates)
- **scikit-learn** (Used for post-filter quality inspection (PCA score plots) to verify that curated features, including those passing the RSD filter, preserve the pooled-QC clustering pattern)

## Examples

```
from tidyms import *; dm = read_data_matrix('mtbls1919_feature_table.csv'); dm = apply_qc_rsd_filter(dm, qc_class='pooled_qc', rsd_threshold=0.20)
```

## Evaluation signals

- No feature retained after the filter should have a QC RSD greater than 20%; recomputing RSD on the curated matrix must reproduce values ≤ 20% for every surviving feature
- The pipeline's filter statistics must report the number of features eliminated at the RSD step, and running the full curation chain (Rt ≥ 90 s → QC presence → blank 10× → RSD 20% → prevalence ≥ 5 → D-ratio < 10%) must yield 665 features for the NIST candidate RM 8231 / SRM 1950 data
- Only pooled/intrastudy QC injections should enter the RSD computation — verify sample-class labels exclude study samples, solvent blanks, and zero-volume-injection/process blanks
- PCA score plots built with scikit-learn on the post-curation matrix should show pooled QC samples clustering tightly, consistent with the RSD-based repeatability criterion
- Features removed by the RSD filter should not reappear in the final matrix; comparing feature identities (retention time, m/z) before and after the step confirms correct removal

## Limitations

- The 20% RSD cutoff is a threshold calibrated for this plasma reference-material application; applying it to other matrices or acquisition modes may require re-tuning
- RSD reflects repeatability only within the pooled/intrastudy QC injections available in the run — it cannot compensate for features absent from QC samples (those are already discarded by the preceding QC-presence filter required for LOESS batch correction)
- Features with high RSD but genuine biological signal may be lost, since the filter is purely repeatability-based and does not distinguish analytical noise from true inter-injection variability

## Evidence

- [results] RSD filter threshold applied to pooled/intrastudy QC injections: "all features with an RSD larger than 20% in pooled /intrastudy QC sample injections were eliminated."
- [results] Position of RSD filter within the full curation chain and final feature count: [evidence span withheld — anchor status: not_found]
- [intro] Purpose of RSD metric in repeatability assessment of QC samples: "allow the assessment of data quality within a study [6,19,20]."
- [methods] Upstream data source and software context for the RSD computation: "The mass spectrometry data have been deposited to the Metabo Lights public repository [38] with the data set identifier MTBLS1919."
