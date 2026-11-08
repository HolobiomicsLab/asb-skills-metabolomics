---
name: qc-sample-drift-evaluation
description: Use when preprocessing LC–MS untargeted metabolomics raw data (mzML)
  from long batches where pooled/intrastudy QC samples are injected at intervals and
  intensity drift or batch effects must be quantified and corrected before statistical
  analysis.
when_to_use_negative:
- The analysis is limited to a system suitability check on the five SSS standard compounds
  (leucine enkephalin, lysophosphatidylcholine (0:0/18:0), L-phenylalanyl-L-phenylalanine,
  L-tryptophan, alogliptin) rather than drift correction of the full feature table.
- The data are already a curated, drift-corrected feature matrix; re-applying drift
  evaluation would double-correct intensities.
- The study lacks repeated pooled/intrastudy QC injections interleaved with samples,
  so no QC-based drift trend can be estimated.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: TidyMS
  role: Loads mzML chromatograms, performs retention time alignment, CWT peak picking,
    deisotoping, integration, adduct grouping, and QC-based signal drift/batch correction
    of the extracted matrix.
  repo: https://github.com/griquelme/tidyms
- name: Python
  role: Runtime language in which TidyMS drift-evaluation and correction pipelines
    are implemented and scripted.
- name: scikit-learn
  role: Downstream integration for model generation on the drift-corrected feature
    matrix.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo10100416
    title: A Python-Based Pipeline for Preprocessing LC–MS Data for Untargeted Metabolomics
      Workflows
  source_content_hash: sha256:1e6dcc1585d896e2c045dfd9700299a7b0ea05f0ffbb5032b053ffbbcb91d4e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/griquelme/tidyms
  tools:
  - Python
  - TidyMS
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# qc-sample-drift-evaluation

## Summary

TidyMS-based signal drift evaluation for untargeted LC–MS metabolomics, in which pooled/intrastudy QC injections (NIST candidate RM 8231 and SRM 1950 plasma) are used to estimate and correct intensity drift across the analytical run before statistical analysis.

## When to use

Use when preprocessing LC–MS untargeted metabolomics raw data (mzML) from long batches where pooled/intrastudy QC samples are injected at intervals and intensity drift or batch effects must be quantified and corrected before statistical analysis.

## When NOT to use

- The analysis is limited to a system suitability check on the five SSS standard compounds (leucine enkephalin, lysophosphatidylcholine (0:0/18:0), L-phenylalanyl-L-phenylalanine, L-tryptophan, alogliptin) rather than drift correction of the full feature table.
- The data are already a curated, drift-corrected feature matrix; re-applying drift evaluation would double-correct intensities.
- The study lacks repeated pooled/intrastudy QC injections interleaved with samples, so no QC-based drift trend can be estimated.

## Inputs

- mzML raw data files of plasma study samples plus repeated pooled/intrastudy QC injections (MetaboLights MTBLS1919, NIST candidate RM 8231 and SRM 1950 batches)
- Sample metadata distinguishing pooled/intrastudy QC injections from hypertriglyceridemic, diabetic, and African-American plasma study samples
- Chromatographic acquisition parameters defining system dead time (approx. 0.8 min in the reference application)

## Outputs

- Drift-corrected extracted feature data matrix (rows = samples, columns = curated m/z / retention time features)
- Signal drift trend estimated from pooled/intrastudy QC injections across the run
- QC summary of pre- and post-correction RSD and prevalence for retained features

## How to apply

Convert or obtain the raw LC–MS runs as mzML files (e.g. from MetaboLights MTBLS1919 for candidate RM 8231 and SRM 1950 plasma batches) and load them with TidyMS to build chromatograms from consecutive MS scans. Perform retention time alignment, CWT-based peak picking, deisotoping, integration, and adduct grouping to produce an extracted feature matrix where each feature is an (m/z, retention time) pair and rows include pooled/intrastudy QC injections interleaved with study samples. Before correction, apply data curation filters that condition the drift model: discard features below the ~90 s dead time, retain only features present in pooled/intrastudy QC samples, remove features with RSD larger than 20% in the QC injections, and enforce 100% intraclass prevalence with an area threshold of 5. With the curated matrix, use the pooled/intrastudy QC injections as repeat observations to estimate signal drift (e.g. via the LOESS-style batch-correction strategy the article applies to features present in pooled/intrastudy QC samples) and normalize study-sample intensities to the QC-derived drift trend, yielding drift-corrected feature intensities suitable for downstream machine learning with scikit-learn. Validate that corrected QC intensities are stable across the run (RSD within the retained features well below the 20% threshold) and that features absent from QC injections were not silently drift-corrected.

## Related tools

- **TidyMS** (Loads mzML chromatograms, performs retention time alignment, CWT peak picking, deisotoping, integration, adduct grouping, and QC-based signal drift/batch correction of the extracted matrix.) — https://github.com/griquelme/tidyms
- **Python** (Runtime language in which TidyMS drift-evaluation and correction pipelines are implemented and scripted.)
- **scikit-learn** (Downstream integration for model generation on the drift-corrected feature matrix.)

## Examples

```
from tidyms.core import assay_data ; from tidyms import Application1 ; app = Application1() ; app.load_data('mzml_dir') ; app.run()  # drift-evaluation pipeline on MTBLS1919 mzML files
```

## Evaluation signals

- The 31 raw data files used for the QC/batch-processing application process in the ballpark of the reported 66 s runtime (~2 s feature detection per sample).
- Features retained for drift correction are present in pooled/intrastudy QC injections and have QC RSD ≤ 20%; features with RSD > 20% were eliminated.
- The number of features in the corrected matrix is smaller than the raw extracted matrix after the Rt (>90 s), QC-prevalence, and D-ratio filters have been applied.
- Post-correction pooled/intrastudy QC intensities show reduced drift compared with the pre-correction run-order trend.
- No feature absent from the pooled/intrastudy QC injections appears in the drift-corrected matrix.

## Limitations

- Drift correction via the QC strategy is conditional on the feature being present in pooled/intrastudy QC samples; features not detected in QC cannot be corrected.
- TidyMS does not perform feature annotation, so drift-corrected features remain unannotated (m/z, retention time) pairs.
- No software version numbers are stated for TidyMS, Python, or scikit-learn in the source, so reproducibility of exact drift estimates depends on the version obtained from GitHub.
- Normalization strategy choices made before statistical analysis can dramatically influence results.

## Evidence

- [abstract] QC-based signal drift evaluation: "The capabilities of the package are shown with pipelines for an LC–MS system suitability check, system conditioning, signal drift evaluation, and data curation."
- [results] QC-prevalence condition for batch correction: "The second step allowed discarding features that were not present in pooled /intrastudy QC samples (see Supporting Text 1) as a condition for performing the LOESS batch correction"
- [results] RSD filter threshold: "all features with an RSD larger than 20% in pooled /intrastudy QC sample injections were eliminated."
- [results] Runtime benchmark: "The running time needed to analyze the 31 raw data files used to generate the results shown in Application 1 was 66 s."
- [methods] Processing workflow: "The procedure included retention time alignment, peak picking, deisotoping, integration, and grouping together adducts derived from the same compound."
