---
name: svm-classifier-training
description: Use when you have a sample-by-analyte intensity matrix (aligned MS features,
  peptides, or proteins) with binary labels (e.g., HCC early recurrence) and need
  to build and compare predictive classifiers via cross-validated AUC.
when_to_use_negative:
- Input data has not been retention-time aligned (no information_target.csv from DeepRTAlign);
  run alignment first to obtain group-defined features
- The outcome is continuous or multi-class rather than a binary label such as early
  recurrence vs. non-recurrence
- The cohort is too small after subsetting to support five-fold cross-validation reliably
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: DeepRTAlign
  role: Produces the RT-aligned feature matrix (information_target.csv with group
    identity) whose top-200 mRMR features train the highest-performing SVM
  repo: https://github.com/PHOENIXcenter/deeprtalign
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-43909-5
    title: 'DeepRTAlign: toward accurate retention time alignment for large cohort
      mass spectrometry data analysis'
  source_content_hash: sha256:d28160a39ad7e1fcf0ceda02b0886c8622c2935c823ba0a519d3cd61c0ea9e4b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PHOENIXcenter/deeprtalign
  tools:
  - DeepRTAlign
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# svm-classifier-training

## Summary

Train support vector machine (SVM) classifiers on mRMR-selected top-200 analyte matrices (DeepRTAlign-aligned MS features, peptides, or proteins) and score them by five-fold cross-validated AUC, as done to predict early recurrence of hepatocellular carcinoma on the C1 training cohort (N=52).

## When to use

Use when you have a sample-by-analyte intensity matrix (aligned MS features, peptides, or proteins) with binary labels (e.g., HCC early recurrence) and need to build and compare predictive classifiers via cross-validated AUC.

## When NOT to use

- Input data has not been retention-time aligned (no information_target.csv from DeepRTAlign); run alignment first to obtain group-defined features
- The outcome is continuous or multi-class rather than a binary label such as early recurrence vs. non-recurrence
- The cohort is too small after subsetting to support five-fold cross-validation reliably

## Inputs

- Sample-by-analyte intensity matrix pivoted from mass_align_all_information/information_target.csv (group column as aligned feature identity)
- Peptide-level abundance matrix
- Protein-level abundance matrix
- Binary early-recurrence label vector for the C1 training cohort (N=52 samples)

## Outputs

- Mean five-fold cross-validation AUC per analyte type (aligned features, peptides, proteins)
- Trained SVM classifier models on each top-200 mRMR matrix
- Results comparison table (hcc_classifier_auc_results.csv)

## How to apply

Restrict all analyte matrices and labels to the training cohort (e.g., C1, N=52 samples). For each matrix, run minimum-redundancy maximum-relevance (mRMR) feature selection independently and retain the top 200 analytes. Train scikit-learn SVM classifiers on each top-200 matrix using the binary early-recurrence labels, and score each with five-fold cross-validation, recording the mean area under the ROC curve. In the reference study, the classifier on the top 200 DeepRTAlign-aligned MS features reached AUC 0.998 versus 0.931 for the top 200 peptides and 0.757 for the top 200 proteins, supporting that aligned MS features carry more effective information. Compare reproduced AUCs against these reported values and export a results table (e.g., hcc_classifier_auc_results.csv).

## Related tools

- **DeepRTAlign** (Produces the RT-aligned feature matrix (information_target.csv with group identity) whose top-200 mRMR features train the highest-performing SVM) — https://github.com/PHOENIXcenter/deeprtalign

## Examples

```
from sklearn.svm import SVC; from sklearn.model_selection import cross_val_score; auc = cross_val_score(SVC(), X_mrmr_top200, y_early_recurrence, cv=5, scoring='roc_auc').mean()
```

## Evaluation signals

- Five-fold CV AUC of the aligned-feature SVM is close to 0.998, peptide SVM close to 0.931, and protein SVM close to 0.757
- Each mRMR selection retains exactly 200 analytes per matrix type
- All matrices and labels contain the same N=52 C1 training samples with matching sample identifiers
- The ranking aligned features > peptides > proteins is reproduced, consistent with the finding that aligned MS features carry more effective information
- hcc_classifier_auc_results.csv contains one AUC per classifier and can be re-derived from the pivoted information_target.csv matrix

## Limitations

- DeepRTAlign alignment has limited influence on quantification; improving quantification accuracy by jointly optimizing feature extraction and alignment remains future work
- When samples are very different or many features share nearly identical m/z, default QC yields few results; FDR may need to be set to 1 and features filtered at score > 0.5
- If keep_best_feature/keep_best_group defaults are changed, multiple features per sample per group may enter the matrix and alter classifier input

## Evidence

- [discussion] Aligned MS features contained more effective information than peptides and proteins in HCC early recurrence prediction: "The results showed that aligned MS features have more effective information compared with peptides and proteins."
- [abstract] Robust classifier for HCC early recurrence: "using the MS features aligned by DeepRTAlign, we trained and validated a robust classifier to predict the early recurrence of hepatocellular carcinoma."
- [readme] Output file location for the aligned feature matrix: "The results will output to the mass_align_all_information folder.If you set kt to 0 (default), there will be only one file in the mass_align_all_information folder: information_target.csv"
- [readme] Group column semantics for pivoting the feature matrix: "group: the group name corresponding to this feature, aligned features share the same group name."
