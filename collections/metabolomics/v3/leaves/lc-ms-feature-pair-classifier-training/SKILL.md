---
name: lc-ms-feature-pair-classifier-training
description: 'Use when labeled feature-pair data from aligned LC-MS runs (attributes:
  m/z, charge, RT, intensity) are available and the goal is to train or benchmark
  classifiers, e.g. testing whether a DNN outperforms RF/KNN/SVM/LR by AUC.'
when_to_use_negative:
- The data are raw (unaligned) LC-MS feature lists — retention time alignment with
  DeepRTAlign must be performed first
- The task is peptide/protein identification-based modeling rather than ID-free aligned-feature
  modeling
- There are no labels for true vs. false feature matches, so supervised AUC evaluation
  is impossible
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3172
tools:
- name: DeepRTAlign
  role: Produces the aligned MS features and labeled feature-pair scores (information_target.csv
    / information_all.csv) used to train the classifiers
  repo: https://github.com/PHOENIXcenter/deeprtalign
- name: PyTorch
  role: Framework used to construct and train the deep neural network classifier (3
    hidden layers of 5000 neurons each) on the feature-pair training set
  repo: https://pytorch.org/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-43909-5
    title: 'DeepRTAlign: toward accurate retention time alignment for large cohort
      mass spectrometry data analysis'
  source_content_hash: sha256:18e24ae24bfd1b8cd329c8d79d948f436765ca081e000a8997e6a68f6b69dc9b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PHOENIXcenter/deeprtalign
  tools:
  - DeepRTAlign
  - PyTorch
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# lc-ms-feature-pair-classifier-training

## Summary

Train and benchmark a deep neural network (PyTorch, 3 hidden layers x 5000 neurons) against RF, KNN, SVM, and LR classifiers on labeled LC-MS feature-feature pairs (m/z, charge, RT, intensity) aligned by DeepRTAlign, ranking models by held-out test AUC.

## When to use

Use when labeled feature-pair data from aligned LC-MS runs (attributes: m/z, charge, RT, intensity) are available and the goal is to train or benchmark classifiers, e.g. testing whether a DNN outperforms RF/KNN/SVM/LR by AUC.

## When NOT to use

- The data are raw (unaligned) LC-MS feature lists — retention time alignment with DeepRTAlign must be performed first
- The task is peptide/protein identification-based modeling rather than ID-free aligned-feature modeling
- There are no labels for true vs. false feature matches, so supervised AUC evaluation is impossible

## Inputs

- Labeled feature-feature pairs (~400,000 for HCC-T) with m/z, charge, RT, and intensity attributes from DeepRTAlign-aligned LC-MS data
- Train/held-out test split of the labeled feature-pair table
- DeepRTAlign alignment results (information_target.csv after QC, or information_all.csv before QC containing decoy samples)

## Outputs

- Trained DNN (3x5000-neuron PyTorch MLP) and RF/KNN/SVM/LR models
- Per-model held-out test AUC comparison table with model ranking

## How to apply

Collect labeled true/false feature-feature pairs from DeepRTAlign-aligned LC-MS runs (e.g. the 400,000 pairs from the HCC-T cohort, scored via information_target.csv / information_all.csv outputs). Split into training and held-out test sets. In PyTorch, build a DNN with 3 hidden layers of 5000 neurons each and train it to discriminate true from false feature matches; train random forest, k-nearest neighbors, SVM, and logistic regression on the identical split for a fair comparison. Evaluate all five models on the held-out test set by computing the area under the ROC curve (AUC), optionally using 10-fold cross-validation on the training cohort. Rank the models and verify that the DNN attains the highest test AUC, as reported in Supplementary Table 3 of the paper.

## Related tools

- **DeepRTAlign** (Produces the aligned MS features and labeled feature-pair scores (information_target.csv / information_all.csv) used to train the classifiers) — https://github.com/PHOENIXcenter/deeprtalign
- **PyTorch** (Framework used to construct and train the deep neural network classifier (3 hidden layers of 5000 neurons each) on the feature-pair training set) — https://pytorch.org/

## Examples

```
model = nn.Sequential(nn.Linear(n_features, 5000), nn.ReLU(), nn.Linear(5000, 5000), nn.ReLU(), nn.Linear(5000, 5000), nn.ReLU(), nn.Linear(5000, 1))
```

## Evaluation signals

- The DNN attains the highest held-out test AUC among DNN, RF, KNN, SVM, and LR, reproducing the ranking in Supplementary Table 3
- All five models are trained on the identical training split and evaluated on the same held-out test set (fair comparison invariant)
- 10-fold cross-validation on HCC-T training data shows stable AUC across folds before held-out evaluation
- Feature pairs carry the expected attributes (m/z, charge, RT, intensity) and QC-passing scores from DeepRTAlign output columns

## Limitations

- Result depends on alignment quality; DeepRTAlign's default workflow can report very few results when samples are quite different or many features share almost the same m/z (README recommends setting FDR to 1 and selecting score > 0.5 in that case)
- A robust classifier was demonstrated for HCC early recurrence prediction; utility for low-abundant biomarker discovery is only an expectation, not demonstrated
- Generalizability boundary was determined but limits of applicability remain

## Evidence

- [abstract] using the MS features aligned by DeepRTAlign, we trained and validated a robust classifier to predict the early recurrence of hepatocellular carcinoma.: "using the MS features aligned by DeepRTAlign, we trained and validated a robust classifier to predict the early recurrence of hepatocellular carcinoma."
- [discussion] The results showed that aligned MS features have more effective information compared with peptides and proteins.: "The results showed that aligned MS features have more effective information compared with peptides and proteins."
- [methods] Construct the deep neural network classifier in PyTorch with 3 hidden layers of 5000 neurons each, and train it on the feature-pair training set: [evidence span withheld — anchor status: not_found]
- [readme] DeepRTAlign is a deep learning-based retention time alignment tool for large cohort LC-MS data analysis.: "DeepRTAlign is a deep learning-based retention time alignment tool for large cohort LC-MS data analysis."
