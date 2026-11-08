---
name: roc-auc-evaluation-clinical-biomarker
description: Use when a trained MS-feature-based classifier (e.g., HCC early-recurrence
  SVM) must be independently tested on a held-out cohort to reproduce or report a
  ROC/AUC generalization metric.
when_to_use_negative:
- The data are raw LC-MS runs that have not yet been feature-extracted and RT-aligned
  by DeepRTAlign — alignment (information_target.csv) must exist before mapping targeted
  features
- There is no independent held-out cohort — in-cohort resubstitution would not reproduce
  the generalization estimate of 0.833
- The classifier requires peptide/protein-level features rather than aligned MS-feature
  groups; the paper reports aligned MS features contained more effective information
  than peptides and proteins, but the input schema must match the classifier
edam_topics:
- http://edamontology.org/topic_3361
- http://edamontology.org/topic_3520
tools:
- name: DeepRTAlign
  role: Produces the aligned feature groups (information_target.csv, QC at FDR 0.01)
    into which the 15 targeted sPRM features are mapped before classification
  repo: https://github.com/PHOENIXcenter/deeprtalign
- name: scikit-learn
  role: Computes the ROC curve and AUC from per-sample classifier scores and independent-test
    labels
- name: pandas
  role: Loads the Skyline peak-area export table and extracts peak areas for the 15
    targeted features
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-43909-5
    title: 'DeepRTAlign: toward accurate retention time alignment for large cohort
      mass spectrometry data analysis'
  source_content_hash: sha256:7897a6865897b873c3233e8254c98fde8d26ec5ca0d8d8731af969c4e00bff93
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PHOENIXcenter/deeprtalign
  tools:
  - DeepRTAlign
  - pandas
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3361
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# roc-auc-evaluation-clinical-biomarker

## Summary

Independently validates a trained clinical classifier (e.g., the 15-feature HCC early-recurrence SVM) by scoring held-out samples and computing ROC curves and AUC — the DeepRTAlign paper's HCC classifier reached AUC 0.833 on the independent sPRM-quantified HCC-R2 cohort (N = 23).

## When to use

Use when a trained MS-feature-based classifier (e.g., HCC early-recurrence SVM) must be independently tested on a held-out cohort to reproduce or report a ROC/AUC generalization metric.

## When NOT to use

- The data are raw LC-MS runs that have not yet been feature-extracted and RT-aligned by DeepRTAlign — alignment (information_target.csv) must exist before mapping targeted features
- There is no independent held-out cohort — in-cohort resubstitution would not reproduce the generalization estimate of 0.833
- The classifier requires peptide/protein-level features rather than aligned MS-feature groups; the paper reports aligned MS features contained more effective information than peptides and proteins, but the input schema must match the classifier

## Inputs

- Skyline peak-area export table for the independent sPRM cohort (HCC-R2, N = 23 samples)
- Feature-to-group mapping from DeepRTAlign output information_target.csv (target results after QC, default FDR 0.01)
- Pre-trained 15-feature early-recurrence classifier (e.g., the C1-trained SVM)
- Independent-test early-recurrence labels for each cohort sample

## Outputs

- Per-sample classifier prediction scores (hcc_r2_sprm_validation.csv)
- ROC curve and AUC value for the independent test cohort (expected AUC = 0.833)

## How to apply

Load the independent cohort's quantification table (e.g., a Skyline peak-area export for 23 HCC-R2 sPRM samples) with pandas and extract peak areas for the classifier's targeted features (the 15 HCC early-recurrence features). Map each targeted feature to its DeepRTAlign-aligned feature group recorded in information_target.csv, the target results after quality control at the default FDR cutoff of 0.01. Apply the pre-trained classifier (e.g., the C1-trained 15-feature SVM) to the mapped peak areas to score each sample, then compute the ROC curve and AUC with scikit-learn against the independent-test labels. Write per-sample predicted scores plus the AUC to an output file such as hcc_r2_sprm_validation.csv, and treat the run as validated when the computed AUC reproduces the reported value (0.833 for the HCC-R2 cohort).

## Related tools

- **DeepRTAlign** (Produces the aligned feature groups (information_target.csv, QC at FDR 0.01) into which the 15 targeted sPRM features are mapped before classification) — https://github.com/PHOENIXcenter/deeprtalign
- **scikit-learn** (Computes the ROC curve and AUC from per-sample classifier scores and independent-test labels)
- **pandas** (Loads the Skyline peak-area export table and extracts peak areas for the 15 targeted features)

## Examples

```
import pandas as pd; areas = pd.read_csv('skyline_hcc_r2_peaks.csv'); scores = clf.predict_proba(areas[['f1','f2',...,'f15']])[:,1]; from sklearn.metrics import roc_auc_score; auc = roc_auc_score(labels, scores)
```

## Evaluation signals

- Computed independent-test AUC equals the reported value of 0.833 on the HCC-R2 cohort (N = 23)
- Every one of the 15 targeted features is successfully mapped to a DeepRTAlign feature group present in information_target.csv
- hcc_r2_sprm_validation.csv contains exactly one prediction score per sample for all 23 HCC-R2 samples
- AUC falls in the [0, 1] range and the ROC curve is monotone non-decreasing with decreasing threshold
- Mapped peak areas show no missing values after QC-filtered alignment (default FDR cutoff 0.01, keep_best_feature=1, keep_best_group=1)

## Limitations

- Generalizability of the DeepRTAlign-aligned-feature classifier was bounded; its limits of applicability remain, so AUC may not transfer beyond HCC cohorts similar to the training and HCC-R2 test cohorts
- The alignment itself has limited influence on quantification — future work planned to improve quantification accuracy by jointly optimizing feature extraction and alignment
- When DeepRTAlign's default QC yields very few results (very different samples or many features with nearly the same m/z), the FDR cutoff must be relaxed to 1 and features selected by score > 0.5, which changes the feature set fed to the classifier

## Evidence

- [abstract] using the MS features aligned by DeepRTAlign, we trained and validated a robust classifier to predict the early recurrence of hepatocellular carcinoma: "using the MS features aligned by DeepRTAlign, we trained and validated a robust classifier to predict the early recurrence of hepatocellular carcinoma."
- [discussion] aligned MS features more informative than peptides/proteins: "The results showed that aligned MS features have more effective information compared with peptides and proteins."
- [readme] information_target.csv is the QC-passed result file: "If you set kt to 0 (default), there will be only one file in the mass_align_all_information folder: information_target.csv (target results after QC)."
- [readme] fdr default 0.01 for quality control: "fdr (float, 0-1, default 0.01) is the threshold used in quality control part."
- [readme] low-score fallback workflow when default QC yields few results: "In this case, set FDR to 1, and select features with score greater than 0.5 are recomended."
