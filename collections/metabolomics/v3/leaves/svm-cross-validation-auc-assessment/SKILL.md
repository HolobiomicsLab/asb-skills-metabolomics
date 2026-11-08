---
name: svm-cross-validation-auc-assessment
description: Use when a corrected/processed metabolomics feature table (samples ×
  peaks) and a candidate biomarker list need an unbiased predictive-performance estimate
  distinguishing phenotypic classes such as CRC vs CE.
when_to_use_negative:
- When the goal is assessing batch effect removal itself (QCS gathering, PCC similarity,
  RSD curves) rather than downstream classification performance
- When no phenotype class labels exist (e.g., purely QC-oriented data with only QCSs),
  since AUC for a class comparison like CRC vs CE cannot be computed
- When biomarkers have not yet been selected — running the SVM on all 6,461 peaks
  instead of the FDR < 0.05 and VIP > 1 subset changes the evaluated question
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: MetaboAnalystR
  role: Metabolomics analysis pipeline in which WaveICA is incorporated; used for
    the metabolomics data analysis (e.g., PLS-DA-based biomarker identification feeding
    the SVM evaluation)
- name: R
  role: Implementation environment for the SVM cross-validation workflow and comparison
    methods
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11306-021-01839-7
    title: 'WaveICA 2.0: a novel batch effect removal method for untargeted metabolomics
      data without using batch information'
  source_content_hash: sha256:17128a9c2072ef1d8f2bbe3f6876a574fdb888f531892dd1de319cb27aade736
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MetaboAnalystR
  - R
  - WaveICA_2.0
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# svm-cross-validation-auc-assessment

## Summary

Evaluates the predictive ability of metabolomics biomarkers (e.g., CRC vs CE peaks selected by PLS-DA with FDR < 0.05 and VIP > 1) using five-fold cross-validation of a radial-basis-kernel SVM repeated 100 times, reporting mean AUC ± standard deviation as the performance metric.

## When to use

Use when a corrected/processed metabolomics feature table (samples × peaks) and a candidate biomarker list need an unbiased predictive-performance estimate distinguishing phenotypic classes such as CRC vs CE.

## When NOT to use

- When the goal is assessing batch effect removal itself (QCS gathering, PCC similarity, RSD curves) rather than downstream classification performance
- When no phenotype class labels exist (e.g., purely QC-oriented data with only QCSs), since AUC for a class comparison like CRC vs CE cannot be computed
- When biomarkers have not yet been selected — running the SVM on all 6,461 peaks instead of the FDR < 0.05 and VIP > 1 subset changes the evaluated question

## Inputs

- Corrected (or original) metabolomics feature matrix of peak intensities (samples × 6,461 metabolite peaks in the reference study)
- Class labels for the subjects (e.g., 497 CRC vs 71 CE patients)
- Candidate biomarker peak list selected by PLS-DA with FDR < 0.05 and VIP > 1 (e.g., 172 peaks for WaveICA 2.0)

## Outputs

- Mean AUC ± standard deviation across 100 repetitions of five-fold cross-validated RBF-SVM (e.g., 0.8831 ± 0.0170)
- Comparison row for a results table (e.g., results_table.csv) alongside QCS average distance, subject-sample average distance, and aPCC metrics

## How to apply

After batch effect correction (e.g., with WaveICA 2.0 or WaveICA) and biomarker identification by PLS-DA on data standardized to mean 0 and variance 1 (selecting peaks with FDR < 0.05 and VIP > 1), fit an SVM with radial basis function kernel on the selected biomarker features only. Assess predictive ability with five-fold cross-validation repeated 100 times, recording the AUC for the class comparison of interest (CRC vs CE). Report the AUC as mean ± standard deviation across the 100 repeats (e.g., WaveICA 2.0: 0.8831 ± 0.0170) and compare against the original data and alternative correction methods (e.g., QC-RLSC 0.8585 ± 0.0201, QC-SVRC 0.8476 ± 0.0203) to judge whether correction preserved or improved class-discriminative signal.

## Related tools

- **MetaboAnalystR** (Metabolomics analysis pipeline in which WaveICA is incorporated; used for the metabolomics data analysis (e.g., PLS-DA-based biomarker identification feeding the SVM evaluation))
- **R** (Implementation environment for the SVM cross-validation workflow and comparison methods)

## Examples

```
R: AUC <- sapply(1:100, function(i) cv_svm_auc(biomarker_data, class_labels, folds=5, kernel='radial')); mean(AUC); sd(AUC)
```

## Evaluation signals

- AUC should be reported as mean ± SD over 100 repeated five-fold CV runs (e.g., '0.8831 ± 0.0170' format)
- WaveICA 2.0's AUC (0.8831) should be comparable to or better than original WaveICA and baseline methods (QC-RLSC 0.8585 ± 0.0201, QC-SVRC 0.8476 ± 0.0203)
- The number of biomarkers entered into the SVM should match the PLS-DA selection count (e.g., 172 peaks for WaveICA 2.0 in the supplementary table)
- The same protocol must be applied identically to original, WaveICA 2.0, and WaveICA conditions for a fair comparison

## Limitations

- AUC reflects predictive ability for the specific class comparison (CRC vs CE) and depends on the biomarker selection step (FDR < 0.05, VIP > 1); it does not by itself quantify batch effect removal
- Repeated 100× five-fold CV is computationally intensive and results can vary slightly between runs without fixed random seeds

## Evidence

- [methods] The predictive ability of identified biomarkers for CRC are evaluated using five-fold cross-validation of the SVM model with radial basis function that is repeated 100 times.: [evidence span withheld — anchor status: not_found]
- [methods] Biomarkers are identified using partial least squares discriminant analysis (PLS-DA) and the false discovery rate (FDR) method.: [evidence span withheld — anchor status: not_found]
- [methods] Peaks with an FDR value less than 0.05 and variable importance in the projection (VIP) of PLS-DA greater than 1 are considered as potential biomarkers.: [evidence span withheld — anchor status: not_found]
