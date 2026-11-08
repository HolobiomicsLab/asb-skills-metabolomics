---
name: metabolomics-batch-correction-pca-visualization
description: Use when an uncorrected LC-MS metabolomics feature table with QC injections
  at fixed intervals (e.g., every 5–6 samples) shows high RSDs and scattered QC samples
  in PCA, and you need drift/batch correction plus visual validation.
when_to_use_negative:
- The dataset has no repeated QC injections and cannot support QC-based correction
  — use statTarget's shiftCor_dQC with MLmethod='Combat' instead
- Input is already batch-corrected or you only need univariate statistics without
  PCA visualization
- The data are label-free LC-MS proteomics rather than metabolomics and the task targets
  proteomics-specific preprocessing (though QC-RFSC is also applicable there)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: statTarget
  role: R package providing shiftCor (QC-RFSC drift correction) and statAnalysis (Pareto/glog
    PCA scoring) used end-to-end for this skill
  repo: github.com/13479776/statTarget
- name: QC-RFSC
  role: The QC-based random forest signal correction algorithm (MLmethod='QCRFSC',
    ntree=500) that removes inter- and intra-batch unwanted variation at feature level
  repo: github.com/13479776/statTarget
- name: random forest
  role: Ensemble-learning regression engine underlying QC-RFSC signal correction
  repo: github.com/13479776/statTarget
- name: transX
  role: Converts XCMS/MZmine2/SIEVE/SKYLINE output into statTarget input file format
    before correction
  repo: github.com/13479776/statTarget
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.aca.2018.08.002
    title: 'statTarget: A streamlined tool for signal drift correction and interpretations
      of quantitative mass spectrometry-based omics data'
  source_content_hash: sha256:4e1d84b1533ab5ee36713a12593a81f95eb397904062f34c5fc49207887cb7e4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - QC-RFSC
  - random forest
  - statTarget
  - transX
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# metabolomics-batch-correction-pca-visualization

## Summary

Corrects signal drift and batch effects in LC-MS metabolomics feature tables using the statTarget QC-RFSC random forest algorithm, then visualizes the impact via Pareto-scaled, glog-transformed PCA score plots comparing raw vs. corrected QC clustering and biological group separation.

## When to use

Use when an uncorrected LC-MS metabolomics feature table with QC injections at fixed intervals (e.g., every 5–6 samples) shows high RSDs and scattered QC samples in PCA, and you need drift/batch correction plus visual validation.

## When NOT to use

- The dataset has no repeated QC injections and cannot support QC-based correction — use statTarget's shiftCor_dQC with MLmethod='Combat' instead
- Input is already batch-corrected or you only need univariate statistics without PCA visualization
- The data are label-free LC-MS proteomics rather than metabolomics and the task targets proteomics-specific preprocessing (though QC-RFSC is also applicable there)

## Inputs

- Sample intensity matrix (metabolomics feature table, e.g. statTarget samFile from Dataset 2 / Giga Science ID 100108, generated from XCMS or Skyline peak extraction, convertible via transX)
- Sample meta information file (samPeno) with batch and injection order columns
- QC sample annotations identifying aliquoted pooled QC injections

## Outputs

- QC-RFSC-corrected sample intensity matrix from shiftCor
- PCA score coordinates (PC1 vs PC2) for raw and corrected data from statAnalysis
- Two PCA score plots (raw vs corrected) with QC samples and biological groups coloured distinctly
- RSD distribution statistics (proportion of peaks within 15% and 30% RSDs)

## How to apply

Load the sample intensity matrix and meta information (batch, injection order) and run statTarget's shiftCor with MLmethod='QCRFSC', Frule=0.8 (80 percent rule keeping variables with non-zero values in at least 80% of samples in any one group), QCspan=0, ntree=500, and KNN imputation to obtain corrected intensities. Feed raw and corrected matrices separately into statAnalysis with Frule=0.8, normM='NONE', imputeM='KNN', glog=TRUE, and scaling='Pareto' to compute PC1 vs PC2 scores. Render two PCA score plots colouring QC samples and biological groups distinctly with Labels=TRUE. The rationale is that QC-based random forest signal correction (QC-RFSC) removes inter- and intra-batch unwanted variations at feature-level, and PCA of QCs is the visual readout of improved precision.

## Related tools

- **statTarget** (R package providing shiftCor (QC-RFSC drift correction) and statAnalysis (Pareto/glog PCA scoring) used end-to-end for this skill) — github.com/13479776/statTarget
- **QC-RFSC** (The QC-based random forest signal correction algorithm (MLmethod='QCRFSC', ntree=500) that removes inter- and intra-batch unwanted variation at feature level) — github.com/13479776/statTarget
- **random forest** (Ensemble-learning regression engine underlying QC-RFSC signal correction) — github.com/13479776/statTarget
- **transX** (Converts XCMS/MZmine2/SIEVE/SKYLINE output into statTarget input file format before correction) — github.com/13479776/statTarget

## Examples

```
R: shiftCor(samPeno, samFile, Frule = 0.8, MLmethod = "QCRFSC", QCspan = 0, ntree = 500, imputeM = "KNN")
```

## Evaluation signals

- Proportion of peaks with RSDs < 30% rises markedly after correction (e.g. 52.1% raw to 90.9% QC-RFSC in dataset 2; 43.9% to 86.2% in dataset 1)
- Corrected PCA score plot shows QC samples clustering within a markedly smaller variance region than the raw-data plot
- Biological groups regain visible separation along PC1/PC2 after correction
- All variables retained satisfy the 80% rule (non-zero value in at least 80% of samples in at least one group, Frule=0.8)
- QC samples in the corrected plot are tightly clustered consistent with improved precision reported in the paper

## Limitations

- QC-RFSC requires regularly interspersed pooled QC injections (every 5 real samples in Dataset 1, every 6 plasma samples in Dataset 2); datasets lacking this design cannot use QC-based correction
- Low-quality peaks remain and must be filtered using quality assessment criteria after batch integration
- Random forest correction needs ntree tuning (500 trees used here); QCspan=0 disables local LOESS smoothing span fitting
- GUI usage on macOS requires XQuartz and specific R (>=3.3.0, recommended 3.3.0) with RGtk2 2.20.31 versions

## Evidence

- [readme] System requirements from README: [evidence span withheld — anchor status: external_doc]
