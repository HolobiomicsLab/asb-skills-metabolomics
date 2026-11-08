---
name: pls-da-discriminant-modeling-with-permutation-testing
description: Use when you have a batch-corrected metabolomics or proteomics feature
  matrix with sample groups (e.g., control vs. disease) and need supervised class-discrimination
  modeling, e.g. after QC-RFSC drift correction in statTarget.
when_to_use_negative:
- Input is still raw, uncorrected MS intensity data with injection-order signal drift
  — apply shiftCor/QC-RFSC batch correction first, since PLS-DA on drifted data confounds
  batch effects with biological group separation.
- The goal is unsupervised data exploration or QC-metric assessment rather than supervised
  class discrimination — PCA, QC RSD distributions, and quality assessment criteria
  are the appropriate alternatives.
- There is only a single sample group or no group label available — PLS-DA is a supervised
  discriminant model and cannot be fit without class labels.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: statTarget
  role: Provides the statAnalysis function that performs the two-component PLS-DA
    fit, Pareto scaling, glog transformation, and permutation testing on the corrected
    feature matrix
  repo: github.com/13479776/statTarget
- name: XCMS
  role: Upstream feature extraction whose output matrix and m/z values feed the PLS-DA
    analysis and the subsequent mummichog pathway table
- name: randomForest
  role: Fits a Breiman random forest classifier (ntree=500) alongside PLS-DA to identify
    the top nvarRF=20 discriminant features via PIMP variable-importance permutation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.aca.2018.08.002
    title: 'statTarget: A streamlined tool for signal drift correction and interpretations
      of quantitative mass spectrometry-based omics data'
  source_content_hash: sha256:becc6384df02d86d59f7fc5a1fe204873e8b6617523e23176f44c634d6053abe
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - XCMS
  - randomForest
  - statTarget
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# PLS-DA discriminant modeling with permutation testing

## Summary

Build a two-component PLS-DA discrimination model on glog-transformed, Pareto-scaled corrected expression matrix data within statTarget's statAnalysis function, then validate the model by a permutation test using 10 random cross-validation segments to confirm class separation is not driven by overfitting.

## When to use

Use when you have a batch-corrected metabolomics or proteomics feature matrix with sample groups (e.g., control vs. disease) and need supervised class-discrimination modeling, e.g. after QC-RFSC drift correction in statTarget.

## When NOT to use

- Input is still raw, uncorrected MS intensity data with injection-order signal drift — apply shiftCor/QC-RFSC batch correction first, since PLS-DA on drifted data confounds batch effects with biological group separation.
- The goal is unsupervised data exploration or QC-metric assessment rather than supervised class discrimination — PCA, QC RSD distributions, and quality assessment criteria are the appropriate alternatives.
- There is only a single sample group or no group label available — PLS-DA is a supervised discriminant model and cannot be fit without class labels.

## Inputs

- Corrected expression matrix (output of statTarget shiftCor) with sample name, group and feature intensity data
- Sample phenotype/annotation file mapping sample IDs to group labels (e.g. control vs. disease)
- statAnalysis parameter set: Frule=0.8, normM='NONE', imputeM='KNN', glog=TRUE, scaling='Pareto'

## Outputs

- Two-component PLS-DA model result file under the statAnalysis output directory (PCA/PLS-DA results)
- PLS-DA score plots and loading plots with group separation visualization
- Permutation-test statistics confirming model validity (10 random CV segments)
- Discriminant feature list optionally cross-referenced with Univariate/Pvalues and Univariate/Fold_Changes tables

## How to apply

Feed the drift-corrected matrix (e.g. from statTarget shiftCor with Frule=0.8 and KNN imputation) into statTarget's statAnalysis function, configured with normM='NONE', imputeM='KNN', glog=TRUE and scaling='Pareto' so the data are generalized-log transformed and Pareto-scaled before modeling. statAnalysis then fits a two-component PLS-DA model on this transformed data, alongside summary statistics (mean, median, quartile, standard deviation), PCA, and a Breiman random forest classifier (ntree=500) with PIMP variable-importance selection (top nvarRF=20 features). Critically, validate the PLS-DA model with a permutation test using 10 random cross-validation segments to ensure the class separation reflects true group structure rather than overfitting. The model results are written to the statAnalysis output directory under a PCA/PLS-DA result file, and the discriminant features can be cross-referenced with univariate test results (Welch's t-test or Mann-Whitney with BH FDR, fold-change cutoff >2 or <0.5, p<0.05) for downstream pathway interpretation via the transX/mummichog m/z-based table.

## Related tools

- **statTarget** (Provides the statAnalysis function that performs the two-component PLS-DA fit, Pareto scaling, glog transformation, and permutation testing on the corrected feature matrix) — github.com/13479776/statTarget
- **XCMS** (Upstream feature extraction whose output matrix and m/z values feed the PLS-DA analysis and the subsequent mummichog pathway table)
- **randomForest** (Fits a Breiman random forest classifier (ntree=500) alongside PLS-DA to identify the top nvarRF=20 discriminant features via PIMP variable-importance permutation)

## Examples

```
statAnalysis(file, Frule = 0.8, normM = "NONE", imputeM = "KNN", glog = TRUE, scaling = "Pareto")
```

## Evaluation signals

- The statAnalysis output directory contains the expected PCA/PLS-DA result files alongside Univariate/Pvalues and Univariate/Fold_Changes tables.
- The PLS-DA permutation test was executed with 10 random cross-validation segments and the model is not overfitted (permutation performance should be below the original model).
- Variables failing the 80 percent rule (Frule=0.8 — non-zero value in at least 80 percent of samples in any one group) have been removed before modeling.
- Data passed glog transformation and Pareto scaling — verify the statAnalysis call includes glog=TRUE and scaling='Pareto'.
- The final pathway-analysis table contains the required columns m.z, p.value and t.score (fold.change) for submission to MetaboAnalyst mummichog.

## Limitations

- PLS-DA is a supervised method prone to overfitting on high-dimensional omics data; the permutation test with 10 random CV segments is the built-in safeguard, but small sample sizes can still yield spuriously confident separations.
- Feature matrix must already be batch-corrected (e.g. via QC-RFSC/shiftCor); PLS-DA on uncorrected drifted data will confound inter- and intra-batch unwanted variations with group separation.
- Only variables passing the 80 percent rule (Frule=0.8) are retained; low-abundance features in fewer samples are dropped and cannot be recovered by the PLS-DA step.
- Requires R (>= 3.3.0) and, on macOS, XQuartz instead of X11 for the graphical user interface, plus RGtk2 binding.

## Evidence
