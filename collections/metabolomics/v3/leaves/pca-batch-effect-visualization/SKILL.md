---
name: pca-batch-effect-visualization
description: Use when a raw feature table with periodic pooled QC injections (e.g.
  every 5–8 samples) has been drift-corrected and you need to demonstrate batch-effect
  removal before downstream statistics.
when_to_use_negative:
- Data has no pooled QC injections — QC-RFSC/PCA visualization relies on QC samples;
  use Combat via shiftCor_dQC for QC-free datasets instead.
- Input is already a normalized, corrected, or imputed feature table — re-running
  PCA purely for batch visualization adds nothing beyond what the existing analysis
  provides.
- Goal is batch removal rather than visualization — PCA plots are diagnostics; correction
  itself must be done first via shiftCor.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: statTarget
  role: Provides shiftCor for QC-RFSC correction and statAnalysis for the glog + Pareto-scaled
    PCA that produces the score plots.
  repo: github.com/13479776/statTarget
- name: Skyline
  role: Extracts SRM peak abundances for the proteomics dataset (174 transitions,
    69 proteins) that serve as the feature matrix for pre/post PCA.
- name: XCMS
  role: Extracts metabolomics peak abundances that are converted to statTarget input
    format by transX before PCA visualization.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.aca.2018.08.002
    title: 'statTarget: A streamlined tool for signal drift correction and interpretations
      of quantitative mass spectrometry-based omics data'
  source_content_hash: sha256:90f7c0cb9e63b9b273ecea75a7c2dd6b0bb007086d6ea10c2956f7d8814fb810
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Skyline
  - XCMS
  - statTarget
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# pca-batch-effect-visualization

## Summary

Generate pre- and post-correction PCA score plots (Pareto scaling, glog transform) from the raw and QC-RFSC-corrected feature matrices to visually confirm that inter-batch drift and unwanted variation have been removed.

## When to use

Use when a raw feature table with periodic pooled QC injections (e.g. every 5–8 samples) has been drift-corrected and you need to demonstrate batch-effect removal before downstream statistics.

## When NOT to use

- Data has no pooled QC injections — QC-RFSC/PCA visualization relies on QC samples; use Combat via shiftCor_dQC for QC-free datasets instead.
- Input is already a normalized, corrected, or imputed feature table — re-running PCA purely for batch visualization adds nothing beyond what the existing analysis provides.
- Goal is batch removal rather than visualization — PCA plots are diagnostics; correction itself must be done first via shiftCor.

## Inputs

- raw feature abundance matrix (samples x features) from Skyline or XCMS/MZmine2/SIEVE
- sample list and phenotype/injection-order file (statTarget samPeno format)
- QC-corrected sample matrix from shiftCor (QC-RFSC)

## Outputs

- PCA score plot images before and after signal drift correction
- scaled/imputed data matrix used for PCA (glog-transformed, Pareto-scaled)
- pre/post batch-effect comparison figure (e.g. Figure 4 panels)

## How to apply

Take the sample matrix produced by statTarget's shiftCor (QC-RFSC with ntree = 500, Frule = 0.8, imputeM = 'KNN') and the raw matrix, then run the statAnalysis pipeline with normM = 'NONE', imputeM = 'KNN', glog = TRUE, scaling = 'Pareto'. PCA is computed on both raw and corrected data so that the score plots directly show whether batch structure is gone. In the proteomics case the input is the Skyline-derived 174-transition/69-protein SRM table; in metabolomics, XCMS/MZmine/SIEVE peak tables via transX. Inspect the pre/post PCA scores side by side: a successful QC-RFSC run shows that replicate/QC points from different batches collapse onto the same principal-component space rather than clustering by injection order or batch.

## Related tools

- **statTarget** (Provides shiftCor for QC-RFSC correction and statAnalysis for the glog + Pareto-scaled PCA that produces the score plots.) — github.com/13479776/statTarget
- **Skyline** (Extracts SRM peak abundances for the proteomics dataset (174 transitions, 69 proteins) that serve as the feature matrix for pre/post PCA.)
- **XCMS** (Extracts metabolomics peak abundances that are converted to statTarget input format by transX before PCA visualization.)

## Examples

```
R> library(statTarget); shiftCor(samPeno,samFile,Frule=0.8,MLmethod="QCRFSC",QCspan=0,imputeM="KNN"); statAnalysis(file,Frule=0.8,normM="NONE",imputeM="KNN",glog=TRUE,scaling="Pareto")
```

## Evaluation signals

- After correction, the proportion of features with RSD ≤ 30% in QC samples should rise markedly (e.g. from 43.9% to 86.2% in Dataset 1, and to 87.9% in the proteomics Dataset 3).
- PCA score plot of corrected data should no longer separate by batch/injection order while raw-data PCA does.
- The corrected matrix should be the one actually produced by shiftCor with the declared Frule (0.8) and MLmethod ('QCRFSC') settings.
- Pre/post plots should be generated from the same feature set after the 80% rule filter and KNN imputation, so any difference is attributable to drift correction.

## Limitations

- PCA visualization is post-hoc: it cannot remove batch effects, only illustrate them; if correction (shiftCor) is skipped the plots will still show batch structure.
- Interpretation depends on the number of QC injections; sparse QC sampling (e.g. every eight samples in Dataset 3) may under-constrain the RF model.
- Requires the statTarget R stack (R ≥ 3.3.0; RGtk2, XQuartz on macOS), which can be non-trivial to install.

## Evidence

- [results] Proportion of peaks with RSDs less than 30 % increased significantly from 43.9% to 86.2 % after QC-RFSC method.: [evidence span withheld — anchor status: not_found]
- [methods] statTarget is employed for signal drift correction and statistical analysis: [evidence span withheld — anchor status: not_found]
- [readme] README: dependency and citation info: [evidence span withheld — anchor status: external_doc]
