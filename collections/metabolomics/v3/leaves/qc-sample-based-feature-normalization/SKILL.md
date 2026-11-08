---
name: qc-sample-based-feature-normalization
description: Use when MS-based metabolomics or proteomics feature tables show signal
  drift or inter-batch variation, pooled QC samples were injected at regular intervals
  (e.g. every 5–8 samples), and feature-level correction is required before statistical
  analysis.
when_to_use_negative:
- QC samples were not injected at regular intervals across the batch, so no QC-based
  drift model can be fitted — the article notes Combat is provided instead via shiftCor_dQC
  for QC-free datasets
- The goal is univariate normalization per sample rather than feature-level signal
  correction across the injection sequence
- Input is still raw vendor MS data rather than a feature table of peak abundances;
  peak extraction (XCMS/Skyline) must happen first
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3174
tools:
- name: statTarget
  role: R package that implements the QC-RFSC algorithm via shiftCor, plus QC-RLSC,
    Combat, 80% rule filtering (Frule), KNN imputation, and downstream statAnalysis
  repo: https://github.com/13479776/statTarget
- name: random forest
  role: Ensemble learning model fitted on QC-sample intensities versus injection order
    to predict per-feature correction factors (QC-RFSC)
- name: XCMS
  role: Used upstream to extract peak abundances from LC-MS mzXML data into the feature
    table that is then corrected; statTarget's transX function converts XCMS output
    into statTarget input format
- name: Skyline
  role: Used upstream to extract peak abundances of tryptic peptides for the SRM proteomics
    dataset before QC-RFSC correction
- name: ProteoWizard
  role: Converts vendor MS raw files to mzXML format prior to peak extraction with
    XCMS or Skyline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.aca.2018.08.002
    title: 'statTarget: A streamlined tool for signal drift correction and interpretations
      of quantitative mass spectrometry-based omics data'
  source_content_hash: sha256:9f9590640dda773d9d20d514ce6cf0611860911e441a5f43a90fb574b37c9084
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# QC-sample-based feature normalization

## Summary

A quality control (QC) sample–based signal drift and batch correction method in which pooled QC injections, spaced at regular intervals across the injection sequence, are used to learn per-feature correction curves that are then applied to neighboring biological samples at the feature level.

## When to use

Use when MS-based metabolomics or proteomics feature tables show signal drift or inter-batch variation, pooled QC samples were injected at regular intervals (e.g. every 5–8 samples), and feature-level correction is required before statistical analysis.

## When NOT to use

- QC samples were not injected at regular intervals across the batch, so no QC-based drift model can be fitted — the article notes Combat is provided instead via shiftCor_dQC for QC-free datasets
- The goal is univariate normalization per sample rather than feature-level signal correction across the injection sequence
- Input is still raw vendor MS data rather than a feature table of peak abundances; peak extraction (XCMS/Skyline) must happen first

## Inputs

- Feature abundance table with peak intensities for metabolites or tryptic peptides from LC-MS or GC-MS runs (e.g. XCMS, MZmine2, SIEVE, or SKYLINE output), ordered by injection sequence
- Sample phenotype/metadata file mapping each injection to its class (QC vs biological sample) and batch/run order
- Raw MS data converted to mzXML format (e.g. via ProteoWizard) prior to peak extraction
- Pooled QC sample injections placed at regular intervals throughout the batch (e.g. after every 5, 6, or 8 real samples)

## Outputs

- Feature-level corrected abundance table (e.g. shift_all_cor.csv) with signal drift and inter-batch variation removed for all samples
- QC-only corrected table (shift_QC_cor.csv) for precision/RSD assessment
- Sample-only corrected table (shift_sample_cor.csv) ready for downstream statistical analysis (e.g. via statAnalysis with Pareto scaling and glog transform)

## How to apply

Order the feature table (e.g. XCMS-extracted peak abundances from mzXML files) by injection sequence so that QC and biological samples are interleaved. For each feature, fit a regression model on the QC-sample intensities versus injection order — statTarget implements this via a quality control–based random forest signal correction (QC-RFSC) with an ensemble of random forest trees, with alternatives such as QC-RLSC (LOESS, tunable via QCspan) and QC-SVR. Predict the correction factor at the injection position of each biological sample and divide the raw intensity by that factor to remove unwanted variation. Enforce the 80% rule filter (Frule = 0.8): keep features with non-zero values in at least 80% of samples, remove features with >20% missing values, and impute remaining NAs with KNN. The rationale is that pooled QC injections share the same analytical matrix and appear at fixed intervals, so their intensity trends track instrument drift and batch effects that can be extrapolated to the adjacent real samples.

## Related tools

- **statTarget** (R package that implements the QC-RFSC algorithm via shiftCor, plus QC-RLSC, Combat, 80% rule filtering (Frule), KNN imputation, and downstream statAnalysis) — https://github.com/13479776/statTarget
- **random forest** (Ensemble learning model fitted on QC-sample intensities versus injection order to predict per-feature correction factors (QC-RFSC))
- **XCMS** (Used upstream to extract peak abundances from LC-MS mzXML data into the feature table that is then corrected; statTarget's transX function converts XCMS output into statTarget input format)
- **Skyline** (Used upstream to extract peak abundances of tryptic peptides for the SRM proteomics dataset before QC-RFSC correction)
- **ProteoWizard** (Converts vendor MS raw files to mzXML format prior to peak extraction with XCMS or Skyline)

## Examples

```
R: statTarget::shiftCor(samPeno, samFile, Frule = 0.8, MLmethod = "QCRFSC", QCspan = 0, imputeM = "KNN")
```

## Evaluation signals

- Relative standard deviation (RSD) distribution of corrected QC peaks improves markedly: in the article, the proportion of peaks with RSD < 30% rose from 43.9% to 86.2% (dataset 1) and from 52.1% to 90.9% (dataset 2) after QC-RFSC
- The proportion of peaks within 15% RSD increased ~12.7-fold (to 25.1%) in dataset 1 after correction
- QC-RFSC outperforms QC-RLSC (65.5%/80.1% peaks ≤30% RSD) and QC-SVR (61.9%/72.3%) on the same data, so corrected RSD distributions should be at least as good as those baselines
- All features surviving correction should satisfy the 80% rule: non-zero in ≥80% of samples and ≤20% missing values, with remaining NAs KNN-imputed
- Corrected tables (shift_all_cor.csv, shift_QC_cor.csv, shift_sample_cor.csv) are produced with QC and biological samples integrated across batches

## Limitations

- Requires pooled QC samples injected at regular intervals throughout the run; without them the QC-based correction cannot be applied (statTarget offers Combat via shiftCor_dQC for QC-free datasets instead)
- Default Frule = 0.8 removes features with >20% missing values, which may discard genuinely low-abundance features
- The method assumes drift/batch effects can be extrapolated from QC injection positions to neighboring biological samples; performance degrades if QC spacing is too sparse or if drift is nonlinear between QC events
- GUI use on macOS requires XQuartz rather than X11, and the package depends on R (>= 3.3.0) with RGtk2, which can complicate installation

## Evidence

- [results] Correction improved precision substantially (dataset 1): [evidence span withheld — anchor status: not_found]
- [readme] Mac users need XQuartz instead of X11: [evidence span withheld — anchor status: external_doc]
