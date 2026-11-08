---
name: classifier-independent-cohort-validation
description: Use when a trained classifier must be independently tested on a new cohort
  — e.g., sPRM/Skyline peak areas for the 15 HCC features in 23 HCC-R2 samples with
  known early-recurrence labels — to verify generalizability against a reported AUC.
when_to_use_negative:
- Input is raw LC-MS data rather than already-quantified peak areas — feature extraction
  and DeepRTAlign alignment must be run first
- The cohort overlaps the training set (C1); this skill is only for independent non-training
  cohorts
- A classifier is needed de novo for a new cohort without an existing trained model
  and feature-group mapping
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: DeepRTAlign
  role: Produces the RT-aligned feature groups (information_target.csv after QC at
    FDR 0.01) that the 15 targeted sPRM features are mapped to before classifier scoring
  repo: https://github.com/PHOENIXcenter/deeprtalign
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-43909-5
    title: 'DeepRTAlign: toward accurate retention time alignment for large cohort
      mass spectrometry data analysis'
  source_content_hash: sha256:aa30e6d7a5846248f62b9bd817627506d0a7f58ed2967986c9ffd9354bb869af
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PHOENIXcenter/deeprtalign
  tools:
  - DeepRTAlign
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# classifier-independent-cohort-validation

## Summary

Applies a previously trained MS-feature classifier (e.g., the 15-feature early-recurrence SVM for hepatocellular carcinoma) to an independent, non-training cohort and computes its ROC AUC on held-out labels, as demonstrated with the sPRM-quantified HCC-R2 cohort (N=23).

## When to use

Use when a trained classifier must be independently tested on a new cohort — e.g., sPRM/Skyline peak areas for the 15 HCC features in 23 HCC-R2 samples with known early-recurrence labels — to verify generalizability against a reported AUC.

## When NOT to use

- Input is raw LC-MS data rather than already-quantified peak areas — feature extraction and DeepRTAlign alignment must be run first
- The cohort overlaps the training set (C1); this skill is only for independent non-training cohorts
- A classifier is needed de novo for a new cohort without an existing trained model and feature-group mapping

## Inputs

- Skyline peak-area export table for the independent sPRM cohort (23 HCC-R2 samples)
- information_target.csv from DeepRTAlign (target aligned features after QC, default FDR 0.01)
- Mapping of the 15 targeted features to DeepRTAlign-aligned feature groups
- C1-trained 15-feature early-recurrence SVM classifier
- Independent-test early-recurrence labels for each cohort sample

## Outputs

- hcc_r2_sprm_validation.csv containing per-sample classifier scores and the computed independent-test AUC
- ROC curve and AUC value (expected 0.833) computed with scikit-learn

## How to apply

Load the Skyline peak-area export for the independent cohort (23 HCC-R2 sPRM samples) with pandas and extract peak areas for the 15 targeted features. Map each targeted feature to its DeepRTAlign-aligned feature group recorded in information_target.csv (the target results after QC, generated with the default FDR cutoff 0.01), so that feature-group identifiers match those the C1-trained 15-feature classifier was built on. Apply the C1-trained 15-feature SVM classifier to the mapped peak areas to score early recurrence for each sample, then compute the ROC curve and AUC with scikit-learn on the independent-test labels. Write per-sample prediction scores plus the AUC to hcc_r2_sprm_validation.csv; validation succeeds when the computed independent-test AUC reproduces the reported value of 0.833.

## Related tools

- **DeepRTAlign** (Produces the RT-aligned feature groups (information_target.csv after QC at FDR 0.01) that the 15 targeted sPRM features are mapped to before classifier scoring) — https://github.com/PHOENIXcenter/deeprtalign

## Examples

```
import pandas as pd; from sklearn.metrics import roc_auc_score; auc = roc_auc_score(y_r2, clf.predict_proba(X_r2_15feat)[:,1]); print(auc)  # expect 0.833
```

## Evaluation signals

- Computed independent-test ROC AUC on the 23 HCC-R2 sPRM samples equals the reported 0.833
- hcc_r2_sprm_validation.csv contains per-sample prediction scores for all 23 samples plus the AUC
- All 15 targeted features successfully map to a DeepRTAlign-aligned group in information_target.csv under the default QC score workflow (FDR 0.01)
- Classifier feature set matches the C1-trained 15-feature model (same feature-group identifiers, no missing features)

## Limitations

- The mapping depends on DeepRTAlign alignment quality; when samples are quite different or many features share almost the same m/z, the default QC workflow can report very few results, requiring FDR=1 and manual score > 0.5 selection
- Generalizability of the classifier was demonstrated on the HCC-R2 cohort (N=23), a small independent test set; broader applicability bounds were not fully established
- Do not run different DeepRTAlign projects under the same folder, as results (including information_target.csv) will be overwritten

## Evidence

- [abstract] robust classifier trained on aligned features: "using the MS features aligned by DeepRTAlign, we trained and validated a robust classifier to predict the early recurrence of hepatocellular carcinoma."
- [readme] FDR cutoff for QC of target results: "fdr (float, 0-1, default 0.01) is the threshold used in quality control part."
- [readme] information_target.csv is the target results after QC: "If you set kt to 0 (default), there will be only one file in the mass_align_all_information folder: information_target.csv (target results after QC)."
- [readme] aligned features share the same group name, enabling feature-group mapping: "group: the group name corresponding to this feature, aligned features share the same group name."
