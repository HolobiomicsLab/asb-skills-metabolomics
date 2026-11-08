---
name: lc-ms-signal-drift-evaluation
description: Use when untargeted LC–MS data in mzML format contains pooled/intrastudy
  QC injections acquired over time and you need to detect, visualize, or correct time-dependent
  signal drift before downstream statistical analysis.
when_to_use_negative:
- The input is already a curated feature table with no pooled/intrastudy QC injections
  in acquisition order — drift cannot be assessed without ordered QC injections.
- You need feature annotation (m/z, adduct, or metabolite identification); the article
  notes TidyMS lacks feature annotation capability.
- The analysis target is study-sample biology rather than analytical platform assessment
  — use the data-curation skill's filter steps (Rt, blank, RSD, D-ratio) instead.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: TidyMS
  role: Python package used to load mzML raw data, extract features, run QC pipelines
    including signal drift evaluation, and perform LOESS batch correction during data
    curation
  repo: https://github.com/griquelme/tidyms
- name: Python
  role: Implementation language; TidyMS integrates with state-of-the-art model-generation
    tools such as scikit-learn
- name: scikit-learn
  role: Used to build the PCA models that reveal time-related systematic variance
    in pooled/intrastudy QC samples
- name: statsmodels
  role: Provides the LOESS smoother used to model drift over injection time for batch
    correction in Supporting Text 1
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo10100416
    title: A Python-Based Pipeline for Preprocessing LC–MS Data for Untargeted Metabolomics
      Workflows
  source_content_hash: sha256:f3439fa25d5152e992a015d75aa38bc86fe7216eea47909cbe99cfd3247e3e5c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/griquelme/tidyms
  tools:
  - Python
  - TidyMS
  - scikit-learn
  - statsmodels
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# lc-ms-signal-drift-evaluation

## Summary

Evaluates LC–MS signal drift by tracking feature intensities across ordered pooled/intrastudy QC injections (NIST candidate RM 8231 and SRM 1950 plasma runs, MetaboLights MTBLS1919) and summarizing systematic time-related variance with PCA and LOESS drift modeling.

## When to use

Use when untargeted LC–MS data in mzML format contains pooled/intrastudy QC injections acquired over time and you need to detect, visualize, or correct time-dependent signal drift before downstream statistical analysis.

## When NOT to use

- The input is already a curated feature table with no pooled/intrastudy QC injections in acquisition order — drift cannot be assessed without ordered QC injections.
- You need feature annotation (m/z, adduct, or metabolite identification); the article notes TidyMS lacks feature annotation capability.
- The analysis target is study-sample biology rather than analytical platform assessment — use the data-curation skill's filter steps (Rt, blank, RSD, D-ratio) instead.

## Inputs

- Raw mzML LC–MS data files from the MetaboLights dataset MTBLS1919 (NIST candidate RM 8231 and SRM 1950 plasma runs)
- Sample metadata assigning injections to study-sample vs pooled/intrastudy QC classes and recording acquisition order

## Outputs

- PCA score plots restricted to pooled/intrastudy QC samples showing time-related systematic variance (drift along PC1)
- Plot/table of pooled QC feature signal versus injection order with the fitted LOESS drift trend (signal_drift_evaluation.png)
- Drift-corrected feature intensities when LOESS batch correction is applied during data curation

## How to apply

Load raw mzML files with TidyMS (pyopenms-backed reader) and extract feature intensities with the CWT-based peak-picking algorithm for chromatographic data. Assemble the pooled/intrastudy QC injections (equal aliquots of every plasma sample) in acquisition order and track each feature's intensity across the batch. Build PCA models with the scikit-learn implementation to evaluate time-related systematic variance or signal drift, using QC-only score plots along PC1 to flag drift and potential outlier injections. For correction during data curation, apply the LOESS smoother (statsmodels implementation) to the QC-sample feature intensities so drift is modeled over injection time and removed via batch correction. Running time on 31 raw data files for Application 1 was about 66 s, roughly 2 s per sample for feature detection.

## Related tools

- **TidyMS** (Python package used to load mzML raw data, extract features, run QC pipelines including signal drift evaluation, and perform LOESS batch correction during data curation) — https://github.com/griquelme/tidyms
- **Python** (Implementation language; TidyMS integrates with state-of-the-art model-generation tools such as scikit-learn)
- **scikit-learn** (Used to build the PCA models that reveal time-related systematic variance in pooled/intrastudy QC samples)
- **statsmodels** (Provides the LOESS smoother used to model drift over injection time for batch correction in Supporting Text 1)

## Examples

```
from tidyms import ms_data; sample = ms_data.MSData('sample.mzML'); sample.detect_peaks(use_mz_obj=False)
```

## Evaluation signals

- PCA score plots built from pooled/intrastudy QC samples show systematic variance along PC1 in the uncorrected data, and this trend is removed after LOESS batch correction.
- Each pooled/intrastudy QC feature's intensity track is in acquisition order, so the drift trend can be plotted against injection time.
- The drift-evaluation artifact (signal_drift_evaluation.png) contains the QC signal versus injection-order plot with the fitted LOESS drift trend.
- Feature detection throughput stays in the reported range (~2 s per sample; 31 files analyzed in 66 s for Application 1).
- Only features present in pooled/intrastudy QC samples are carried forward to the LOESS batch correction, per the data-curation filter condition.

## Limitations

- Signal drift detection depends on the presence and regular placement of pooled/intrastudy QC injections across the batch; drift cannot be modeled without them.
- TidyMS lacks feature annotation capability, so the drift-evaluation output is on the feature (retention time, m/z) level without metabolite identification.
- No software version numbers for TidyMS, Python, or scikit-learn are stated in the article, which complicates exact reproduction.

## Evidence

- [results] PCA drift evaluation: [evidence span withheld — anchor status: not_found]
- [results] LOESS drift correction: "The second step allowed discarding features that were not present in pooled /intrastudy QC samples (see Supporting Text 1) as a condition for performing the LOESS batch correction"
- [results] Runtime benchmark: "The running time needed to analyze the 31 raw data files used to generate the results shown in Application 1 was 66 s."
- [abstract] Pipeline scope: "The capabilities of the package are shown with pipelines for an LC–MS system suitability check, system conditioning, signal drift evaluation, and data curation."
- [methods] Dataset accession: "The mass spectrometry data have been deposited to the Metabo Lights public repository [38] with the data set identifier MTBLS1919."
