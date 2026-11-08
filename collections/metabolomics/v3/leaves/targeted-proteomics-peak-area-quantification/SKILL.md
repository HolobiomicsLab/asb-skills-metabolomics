---
name: targeted-proteomics-peak-area-quantification
description: Use when validating a discovery-trained MS feature classifier on an independent
  targeted (sPRM/Skyline peak-area) cohort, requiring peak-area extraction, feature-group
  mapping, and AUC reproduction.
when_to_use_negative:
- Input is already identified peptide/protein-level quantification rather than raw
  or aligned MS feature peak areas — the article found aligned MS features carry more
  predictive information than peptides/proteins.
- The targeted features have not been mapped to DeepRTAlign-aligned feature groups;
  applying the classifier to unmapped Skyline areas without group correspondence from
  information_target.csv is out of scope.
- The goal is retention time alignment itself rather than downstream targeted quantification
  and classifier validation — use the DeepRTAlign alignment workflow instead.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: DeepRTAlign
  role: Produces the aligned feature groups (information_target.csv after QC at FDR
    0.01) to which targeted sPRM peak areas are mapped before classifier prediction.
  repo: https://github.com/PHOENIXcenter/deeprtalign
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-43909-5
    title: 'DeepRTAlign: toward accurate retention time alignment for large cohort
      mass spectrometry data analysis'
  source_content_hash: sha256:3323b0f5dc0e5da46cf51780a67b78b3f8fc51378b8c0b88931c97c08946b837
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PHOENIXcenter/deeprtalign
  tools:
  - DeepRTAlign
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# targeted-proteomics-peak-area-quantification

## Summary

Extract and map Skyline-exported sPRM peak areas for a targeted feature panel (e.g., 15 HCC early-recurrence features) onto DeepRTAlign-aligned feature groups, then apply a trained classifier and score it by independent-test ROC/AUC.

## When to use

Use when validating a discovery-trained MS feature classifier on an independent targeted (sPRM/Skyline peak-area) cohort, requiring peak-area extraction, feature-group mapping, and AUC reproduction.

## When NOT to use

- Input is already identified peptide/protein-level quantification rather than raw or aligned MS feature peak areas — the article found aligned MS features carry more predictive information than peptides/proteins.
- The targeted features have not been mapped to DeepRTAlign-aligned feature groups; applying the classifier to unmapped Skyline areas without group correspondence from information_target.csv is out of scope.
- The goal is retention time alignment itself rather than downstream targeted quantification and classifier validation — use the DeepRTAlign alignment workflow instead.

## Inputs

- Skyline peak-area export table for the targeted sPRM cohort (e.g., 23 HCC-R2 samples)
- information_target.csv — DeepRTAlign target results after QC (default FDR 0.01), with sample, time, mz, charge, intensity, score, and group columns
- Trained classifier (C1-trained 15-feature early-recurrence SVM)
- Independent-test labels (early recurrence status per sample)

## Outputs

- Per-sample classifier scores for the targeted cohort
- ROC curve and independent-test AUC (expected ~0.833 for the HCC-R2 cohort)
- hcc_r2_sprm_validation.csv containing per-sample scores plus the AUC

## How to apply

Load the Skyline peak-area export table for the targeted cohort (e.g., 23 HCC-R2 sPRM samples) with pandas and extract peak areas for the targeted features (15 HCC early-recurrence features). Map each targeted feature to its DeepRTAlign-aligned feature group recorded in information_target.csv, the target results after QC at the default FDR cutoff of 0.01; where alignment results are noisy, the README advises setting FDR to 1 and selecting features with score greater than 0.5. Apply the discovery-trained classifier (C1-trained 15-feature SVM) to the mapped peak areas to predict early recurrence per sample. Compute the ROC curve and AUC with scikit-learn on independent-test labels and write per-sample scores plus AUC to a validation CSV (hcc_r2_sprm_validation.csv). Correct application is judged by the computed independent-test AUC reproducing the reported value (0.833 for HCC-R2).

## Related tools

- **DeepRTAlign** (Produces the aligned feature groups (information_target.csv after QC at FDR 0.01) to which targeted sPRM peak areas are mapped before classifier prediction.) — https://github.com/PHOENIXcenter/deeprtalign

## Examples

```
import pandas as pd; areas = pd.read_csv('skyline_hcc_r2_sprm.csv'); info = pd.read_csv('mass_align_all_information/information_target.csv'); # map 15 features to aligned groups, predict, then sklearn.metrics.roc_auc_score(labels, scores)
```

## Evaluation signals

- The computed independent-test AUC on the 23-sample HCC-R2 sPRM cohort reproduces the reported value of 0.833.
- All 15 targeted features are successfully mapped to DeepRTAlign-aligned feature groups in information_target.csv (no unmapped features).
- information_target.csv was generated under the default QC (fdr=0.01); if FDR was relaxed to 1, retained features must have score > 0.5.
- hcc_r2_sprm_validation.csv contains one score row per sample (N = 23) plus the aggregate AUC, with no missing peak areas for the 15 features.
- ROC curve computed with scikit-learn uses independent-test labels, not the training cohort.

## Limitations

- The default QC workflow can report very few results when samples are quite different or many features share nearly the same m/z; in that case set FDR to 1 and select features with score > 0.5.
- The 15-feature classifier was validated on a small cohort (N = 23), so the AUC estimate has limited precision.
- keep_best_feature=1 and keep_best_group=1 defaults (keeping the highest-scoring feature per sample/group and the largest group) constrain which peak areas can be mapped; setting them to 0 yields ambiguous multi-candidate mappings.

## Evidence

- [abstract] using the MS features aligned by DeepRTAlign, we trained and validated a robust classifier to predict the early recurrence of hepatocellular carcinoma.: "using the MS features aligned by DeepRTAlign, we trained and validated a robust classifier to predict the early recurrence of hepatocellular carcinoma."
- [discussion] The results showed that aligned MS features have more effective information compared with peptides and proteins.: "The results showed that aligned MS features have more effective information compared with peptides and proteins."
- [readme] fdr (float, 0-1, default 0.01) is the threshold used in quality control part.: "fdr (float, 0-1, default 0.01) is the threshold used in quality control part."
- [readme] set FDR to 1, and select features with score greater than 0.5 are recomended.: "In this case, set FDR to 1, and select features with score greater than 0.5 are recomended."
- [readme] information_target.csv (target results after QC): "If you set kt to 0 (default), there will be only one file in the mass_align_all_information folder: information_target.csv (target results after QC)."
