---
name: five-fold-cross-validated-auc-evaluation
description: Use when comparing predictive performance of classifiers built on different
  feature representations (aligned MS features vs peptides vs proteins) and you need
  a single cross-validated ranking metric such as mean AUC on a defined training cohort.
when_to_use_negative:
- Input features have not yet been aligned across LC-MS runs (e.g., raw per-run feature
  lists); run DeepRTAlign RT alignment first to obtain the group identities used here
- The cohort is an independent validation set rather than a training set — the reported
  AUCs (0.998/0.931/0.757) refer to five-fold CV within training set C1
- Different top-k feature counts are used without justification; the paper's comparison
  is specifically built on mRMR top-200 analytes per representation
edam_topics:
- http://edamontology.org/topic_3570
- http://edamontology.org/topic_0091
tools:
- name: DeepRTAlign
  role: Produces information_target.csv of RT-aligned LC-MS features whose group column
    defines the aligned-feature matrix evaluated against peptide and protein classifiers
  repo: https://github.com/PHOENIXcenter/deeprtalign
- name: scikit-learn
  role: Trains the SVM classifiers and computes five-fold cross-validation ROC-AUC
    scores
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-43909-5
    title: 'DeepRTAlign: toward accurate retention time alignment for large cohort
      mass spectrometry data analysis'
  source_content_hash: sha256:446e5f87995904bd5891ba27ac2b8be14cc0642b05f0211883909e21537579a8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PHOENIXcenter/deeprtalign
  tools:
  - DeepRTAlign
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3570
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# five-fold-cross-validated-auc-evaluation

## Summary

Evaluates and compares binary classifiers (e.g., SVMs on mRMR-selected top-200 aligned MS features, peptides, and proteins) by mean AUC under five-fold cross-validation, as done for HCC early-recurrence prediction with DeepRTAlign-aligned features on training set C1 (N=52).

## When to use

Use when comparing predictive performance of classifiers built on different feature representations (aligned MS features vs peptides vs proteins) and you need a single cross-validated ranking metric such as mean AUC on a defined training cohort.

## When NOT to use

- Input features have not yet been aligned across LC-MS runs (e.g., raw per-run feature lists); run DeepRTAlign RT alignment first to obtain the group identities used here
- The cohort is an independent validation set rather than a training set — the reported AUCs (0.998/0.931/0.757) refer to five-fold CV within training set C1
- Different top-k feature counts are used without justification; the paper's comparison is specifically built on mRMR top-200 analytes per representation

## Inputs

- Sample-by-group intensity matrix pivoted from information_target.csv (mass_align_all_information folder), group column used as the aligned-feature identity
- Peptide-level abundance matrix restricted to the C1 training cohort (N=52)
- Protein-level abundance matrix restricted to the C1 training cohort (N=52)
- Early-recurrence binary labels for the C1 samples

## Outputs

- Mean five-fold cross-validation AUC per classifier (top-200 aligned MS features, top-200 peptides, top-200 proteins)
- Results table written to hcc_classifier_auc_results.csv

## How to apply

Restrict all feature matrices to the training cohort of interest (here the C1 early-recurrence cohort, N=52 samples) and join them with the binary early-recurrence labels. Apply feature selection independently within each representation — mRMR retaining the top 200 analytes — so the comparison is not confounded by dimensionality. Train a support vector machine from scikit-learn on each top-200 matrix and compute mean area under the ROC curve with five-fold cross-validation, recording one AUC per representation. Judge correctness by reproducing the expected ordering and magnitudes (aligned MS features 0.998 > peptides 0.931 > proteins 0.757) and by writing the comparison to a results table such as hcc_classifier_auc_results.csv.

## Related tools

- **DeepRTAlign** (Produces information_target.csv of RT-aligned LC-MS features whose group column defines the aligned-feature matrix evaluated against peptide and protein classifiers) — https://github.com/PHOENIXcenter/deeprtalign
- **scikit-learn** (Trains the SVM classifiers and computes five-fold cross-validation ROC-AUC scores)

## Examples

```
from sklearn.model_selection import cross_val_score; auc = cross_val_score(SVC(), X_top200, y_early_recurrence, cv=5, scoring='roc_auc').mean()
```

## Evaluation signals

- Reproduced CV AUCs match the reported values: 0.998 for aligned MS features, 0.931 for peptides, 0.757 for proteins
- Ordering invariant holds: aligned-features AUC > peptide AUC > protein AUC in five-fold CV
- All three matrices contain exactly the same 52 C1 samples and identical early-recurrence labels before selection
- Each classifier receives exactly 200 mRMR-selected analytes, confirming a dimensionality-controlled comparison
- Output file hcc_classifier_auc_results.csv exists and contains one AUC row per feature representation

## Limitations

- Aligned MS features are sample-group identifiers without protein-level annotation, so the classifier is harder to interpret biologically than peptide- or protein-based models
- AUCs near 1.0 (0.998) may reflect effective feature selection or small-sample overfitting within the N=52 C1 cohort and require independent validation
- The paper notes alignment has limited influence on quantification compared with feature extraction, so quantitative accuracy of the intensity matrix itself is bounded upstream

## Evidence

- [discussion] aligned MS features outperform peptides and proteins: "The results showed that aligned MS features have more effective information compared with peptides and proteins."
- [abstract] classifier training and validation motivation: "using the MS features aligned by DeepRTAlign, we trained and validated a robust classifier to predict the early recurrence of hepatocellular carcinoma."
- [readme] result file feeding the feature matrix: "The results will output to the mass_align_all_information folder."
