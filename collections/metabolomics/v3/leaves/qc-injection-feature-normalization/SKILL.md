---
name: qc-injection-feature-normalization
description: Use when a quantitative mass-spectrometry feature table (e.
when_to_use_negative:
- Datasets without interspersed pooled-QC injections and injection-order metadata
  — the shiftCor_dQC/Combat path is the provided alternative for QC-free datasets.
- When the input is already a single-batch, drift-free feature table and only group
  comparisons are needed — statAnalysis without shiftCor is the appropriate entry
  point.
- When the goal is untargeted feature discovery requiring no numeric normalization,
  e.g. raw spectra browsing or library matching rather than quantitative interpretation.
edam_operation: http://edamontology.org/operation_3675
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: statTarget
  role: Provides shiftCor (QC-RFSC random-forest signal correction), transX for converting
    XCMS/MZmine2/SIEVE/Skyline tables to statTarget input format, and statAnalysis
    for downstream statistics.
  repo: https://github.com/13479776/statTarget
- name: random forest
  role: 'Core algorithm under QC-RFSC: quality-control-based random forest regression
    predicts drift per feature from injection order and QC responses.'
  repo: https://github.com/13479776/statTarget
- name: ProteoWizard
  role: Converts vendor MS raw data to mzXML before peak extraction.
- name: XCMS
  role: Extracts metabolite peak abundances from mzXML for metabolomics datasets.
- name: Skyline
  role: Extracts SRM transition peak abundances for the proteomics dataset (174 transitions
    / 69 proteins).
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.aca.2018.08.002
    title: 'statTarget: A streamlined tool for signal drift correction and interpretations
      of quantitative mass spectrometry-based omics data'
  source_content_hash: sha256:c65c04073acbb3ad8ad89d779d241c3925ac282ceb99e6b87109c214f258ba77
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/13479776/statTarget
  tools:
  - ProteoWizard
  - Skyline
  - XCMS
  - random forest
  - statTarget
  edam_operation: http://edamontology.org/operation_3675
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# qc-injection-feature-normalization

## Summary

A QC-injection-based signal correction workflow that uses pooled quality-control samples interleaved with real samples to model and remove inter- and intra-batch signal drift at the feature level in LC/GC-MS metabolomics and SRM proteomics data, implemented through the statTarget QC-RFSC random-forest correction algorithm.

## When to use

Use when a quantitative mass-spectrometry feature table (e.g. XCMS, Skyline or MZmine2 peak abundances) contains systematic drift or batch effects and pooled QC injections were acquired at regular intervals during the run, so drift can be modeled and corrected per feature before statistical analysis.

## When NOT to use

- Datasets without interspersed pooled-QC injections and injection-order metadata — the shiftCor_dQC/Combat path is the provided alternative for QC-free datasets.
- When the input is already a single-batch, drift-free feature table and only group comparisons are needed — statAnalysis without shiftCor is the appropriate entry point.
- When the goal is untargeted feature discovery requiring no numeric normalization, e.g. raw spectra browsing or library matching rather than quantitative interpretation.

## Inputs

- Raw mass-spectrometry data converted to mzXML (e.g. via ProteoWizard)
- Feature/peak-abundance tables from XCMS, MZmine2, SIEVE or Skyline
- Sample phenotype metadata with injection order and QC-vs-subject labels

## Outputs

- QC-RFSC-corrected feature matrix alongside the retained raw matrix
- Per-feature RSD (%) values for QC samples before and after correction
- Cumulative RSD-frequency plots and pre/post-correction PCA score plots

## How to apply

Interleave pooled QC injections at a fixed cadence during acquisition (every five samples in dataset 1, every six in dataset 2, every eight in the SRM proteomics dataset 3) and randomize the running order of subjects. Convert raw MS files with ProteoWizard to mzXML, extract peak abundances with XCMS or Skyline, then use the statTarget transX function to generate statTarget-format sample and phenotype files. Preprocess with shiftCor using the 80-percent rule (Frule = 0.8), KNN imputation (imputeM = "KNN"), and MLmethod = "QCRFSC" (ntree = 500) to fit a random-forest correction per feature across injection order; the same call yields both raw and corrected matrices. Verify improvement by computing the relative standard deviation (RSD%) of each transition in the QC samples and building cumulative RSD-distribution curves, plus pre-/post-correction PCA (Pareto scaling, glog via statAnalysis).

## Related tools

- **statTarget** (Provides shiftCor (QC-RFSC random-forest signal correction), transX for converting XCMS/MZmine2/SIEVE/Skyline tables to statTarget input format, and statAnalysis for downstream statistics.) — https://github.com/13479776/statTarget
- **random forest** (Core algorithm under QC-RFSC: quality-control-based random forest regression predicts drift per feature from injection order and QC responses.) — https://github.com/13479776/statTarget
- **ProteoWizard** (Converts vendor MS raw data to mzXML before peak extraction.)
- **XCMS** (Extracts metabolite peak abundances from mzXML for metabolomics datasets.)
- **Skyline** (Extracts SRM transition peak abundances for the proteomics dataset (174 transitions / 69 proteins).)

## Examples

```
shiftCor(samPeno, samFile, Frule = 0.8, MLmethod = "QCRFSC", QCspan = 0, imputeM = "KNN")
```

## Evaluation signals

- Compute RSD% per feature in QC samples; after QC-RFSC correction the proportion of features with RSD < 30% should rise substantially (e.g. 43.9% → 86.2% in dataset 1, 52.1% → 90.9% in dataset 2, 47.7% → 87.9% in dataset 3).
- Cumulative RSD-distribution curves should show a rightward shift (more features at low RSD) after correction relative to raw data.
- PCA score plots on Pareto-scaled, glog-transformed data should show reduced batch/axis separation of injection order after correction.
- Random-forest predictive accuracy (R-square at median and first quartile) should be comparable to or better than Loess/QRF baselines.
- The 80-percent-rule filter (Frule = 0.8) must retain only features with non-zero values in ≥ 80% of samples in at least one group before correction.

## Limitations

- Requires pooled-QC injections at regular intervals and randomized run order; QC-free datasets must instead use the shiftCor_dQC/Combat path.
- Performance depends on adequate QC coverage — article demonstrated cadences of one QC per 5–8 real samples; sparser QC injection may weaken RF prediction.
- Normalization is per-feature and must be followed by quality filtering; low-quality peaks still need to be removed via the quality-assessment criteria after batch integration.
- On macOS the GUI requires XQuartz and a compatible R/RGtk2 combination (R 3.3.0 and RGtk2 2.20.31 were the tested pair).

## Evidence

- [results] the proportion of peaks with RSDs less than 30 % increased significantly from 43.9% to 86.2 % after QC-RFSC method.: [evidence span withheld — anchor status: not_found]
