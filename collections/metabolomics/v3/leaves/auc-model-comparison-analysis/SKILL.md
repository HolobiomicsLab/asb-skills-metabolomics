---
name: auc-model-comparison-analysis
description: Use when several classifiers have been trained on an identical labeled
  training split and the goal is a fair, ranked comparison of discrimination performance
  via AUC on a held-out test set.
when_to_use_negative:
- When the candidate models were trained on different training splits or feature subsets
  — the comparison requires identical training data for fairness
- When the goal is evaluating the alignment tool itself (DeepRTAlign vs XCMS/MZmine/OpenMS)
  rather than comparing downstream classifiers
- When only training-set performance is available and no independent held-out test
  set exists — AUC ranking would not be an unbiased comparison
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3474
tools:
- name: PyTorch
  role: Framework used to construct and train the 3-hidden-layer (5000 neurons each)
    DNN classifier compared against RF, KNN, SVM and LR
  repo: https://pytorch.org/
- name: DeepRTAlign
  role: Produces the aligned MS features from the HCC-T LC-MS cohort from which the
    labeled feature-pair training/testing data are collected
  repo: github.com/PHOENIXcenter/DeepRTAlign
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-43909-5
    title: 'DeepRTAlign: toward accurate retention time alignment for large cohort
      mass spectrometry data analysis'
  source_content_hash: sha256:c4940af5940be0ae5775fa002687e2e205f884e8e84f04663fd3ee14fecb2380
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepRTAlign
  - PyTorch
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# AUC model comparison analysis

## Summary

Benchmark a deep neural network classifier against reference machine-learning models (RF, KNN, SVM, LR) by computing and ranking test-set AUC values. In the DeepRTAlign paper this establishes that the DNN discriminates true from false aligned feature pairs better than the classical baselines on the HCC-T cohort.

## When to use

Use when several classifiers have been trained on an identical labeled training split and the goal is a fair, ranked comparison of discrimination performance via AUC on a held-out test set.

## When NOT to use

- When the candidate models were trained on different training splits or feature subsets — the comparison requires identical training data for fairness
- When the goal is evaluating the alignment tool itself (DeepRTAlign vs XCMS/MZmine/OpenMS) rather than comparing downstream classifiers
- When only training-set performance is available and no independent held-out test set exists — AUC ranking would not be an unbiased comparison

## Inputs

- Labeled feature-pair training data from DeepRTAlign-aligned LC-MS runs (m/z, charge, RT, intensity attributes; ~400,000 pairs from HCC-T)
- Train/held-out test split of the labeled pairs
- Reference classifiers: random forest, KNN, SVM, logistic regression; candidate DNN in PyTorch (3 hidden layers x 5000 neurons)

## Outputs

- Per-model test-set AUC values for DNN, RF, KNN, SVM and LR
- Ranked model-comparison table (as in Supplementary Table 3)

## How to apply

Assemble a labeled feature-pair dataset — in the paper, ~400,000 labeled pairs of MS features from the HCC-T LC-MS cohort aligned by DeepRTAlign, each described by m/z, charge, RT and intensity attributes — and split into training and held-out test sets. Train the candidate deep neural network in PyTorch (3 hidden layers of 5000 neurons each) and train the four reference classifiers (random forest, k-nearest neighbors, support vector machine, logistic regression) on the identical training split, using 10-fold cross-validation on the training data before final fitting. Evaluate all five trained models on the same held-out test set and compute the area under the ROC curve (AUC) for each. Compile the per-model test AUCs into a comparison table and rank the models; correctness is confirmed if the DNN attains the highest test AUC, reproducing the ranking in Supplementary Table 3.

## Related tools

- **PyTorch** (Framework used to construct and train the 3-hidden-layer (5000 neurons each) DNN classifier compared against RF, KNN, SVM and LR) — https://pytorch.org/
- **DeepRTAlign** (Produces the aligned MS features from the HCC-T LC-MS cohort from which the labeled feature-pair training/testing data are collected) — github.com/PHOENIXcenter/DeepRTAlign

## Evaluation signals

- All five models are trained on the identical training split of the ~400,000 labeled feature pairs
- Each model's AUC is computed on the same held-out test set, after 10-fold cross-validation on HCC-T
- The DNN attains the highest test AUC among DNN, RF, KNN, SVM and LR, matching the ranking in Supplementary Table 3
- Comparison table contains one AUC entry per model (five rows) rather than averaged-fold scores alone

## Limitations

- The reported ranking was validated on the HCC hepatocellular carcinoma cohort; AUC superiority of the DNN may not transfer to other feature-pair datasets
- Benchmark quality depends on DeepRTAlign's upstream alignment: the paper notes aligned MS features carry more effective information than peptides and proteins, so feature source affects results
- The DNN architecture (3 hidden layers, 5000 neurons each) and 10-fold CV setup define the specific comparison; other architectures or CV schemes may change the ranking

## Evidence

- [abstract] using the MS features aligned by DeepRTAlign, we trained and validated a robust classifier to predict the early recurrence of hepatocellular carcinoma: "using the MS features aligned by DeepRTAlign, we trained and validated a robust classifier to predict the early recurrence of hepatocellular carcinoma."
- [discussion] aligned MS features have more effective information compared with peptides and proteins: "The results showed that aligned MS features have more effective information compared with peptides and proteins."
- [readme] Install Pytorch CPU version, please refer to https://pytorch.org/.: "Install Pytorch CPU version, please refer to https://pytorch.org/."
