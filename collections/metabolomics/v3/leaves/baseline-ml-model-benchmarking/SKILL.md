---
name: baseline-ml-model-benchmarking
description: Use when a new deep learning model on aligned LC-MS features (or a comparable
  tabular classification task) must be compared against RF, KNN, SVM, and LR baselines
  to prove it adds predictive value.
when_to_use_negative:
- The classification target is peptide/protein-level rather than aligned MS feature
  pairs — the paper found aligned MS features carry more effective information, so
  swapping input representations changes the benchmark
- Data are raw, unaligned feature lists — run RT alignment (e.g., DeepRTAlign with
  QC on score and fdr cutoff 0.01) before building classifier training pairs
- Only a single model is of interest and no baseline comparison is required
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0121
tools:
- name: DeepRTAlign
  role: Produces the aligned MS features whose feature-feature pairs serve as the
    training/testing instances for the classifier benchmark
  repo: https://github.com/PHOENIXcenter/DeepRTAlign
- name: PyTorch
  role: Framework used to construct and train the deep neural network classifier (3
    hidden layers × 5000 neurons)
  repo: https://pytorch.org/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-43909-5
    title: 'DeepRTAlign: toward accurate retention time alignment for large cohort
      mass spectrometry data analysis'
  source_content_hash: sha256:39da1f1068b889e8e8941d675cac20c7ae52fbb81e9cefafe3dc888fe35b5fdf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PHOENIXcenter/DeepRTAlign
  tools:
  - DeepRTAlign
  - PyTorch
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# baseline-ml-model-benchmarking

## Summary

Benchmark a deep neural network against four classical classifiers (random forest, k-nearest neighbors, SVM, logistic regression) on identical training splits of DeepRTAlign-aligned LC-MS feature pairs, ranking models by ROC AUC on held-out test sets.

## When to use

Use when a new deep learning model on aligned LC-MS features (or a comparable tabular classification task) must be compared against RF, KNN, SVM, and LR baselines to prove it adds predictive value.

## When NOT to use

- The classification target is peptide/protein-level rather than aligned MS feature pairs — the paper found aligned MS features carry more effective information, so swapping input representations changes the benchmark
- Data are raw, unaligned feature lists — run RT alignment (e.g., DeepRTAlign with QC on score and fdr cutoff 0.01) before building classifier training pairs
- Only a single model is of interest and no baseline comparison is required

## Inputs

- Labeled feature-feature pairs from DeepRTAlign-aligned LC-MS runs, with m/z, charge, RT, and intensity attributes (~400,000 pairs in the HCC-T cohort)
- Training/held-out test split assignment for each feature pair

## Outputs

- Per-model test-set ROC AUC values for DNN, RF, KNN, SVM, and LR
- Ranked model comparison table (cf. Supplementary Table 3 of the paper)

## How to apply

Collect labeled feature-feature pairs from the aligned LC-MS cohort (attributes: m/z, charge, RT, intensity) — in the DeepRTAlign HCC-T study, ~400,000 pairs from a hepatocellular carcinoma cohort — and split into training and held-out test sets. Construct the deep neural network in PyTorch with 3 hidden layers of 5000 neurons each, and train reference classifiers (random forest, k-nearest neighbors, SVM, logistic regression) on the identical training split to ensure a fair comparison. Perform 10-fold cross-validation on the HCC-T training set, then evaluate all five trained models on the independent test set, computing the area under the ROC curve (AUC) for each. Compile per-model test AUCs into a comparison table and rank the models, verifying that the DNN attains the highest AUC as reported in Supplementary Table 3 of the DeepRTAlign paper.

## Related tools

- **DeepRTAlign** (Produces the aligned MS features whose feature-feature pairs serve as the training/testing instances for the classifier benchmark) — https://github.com/PHOENIXcenter/DeepRTAlign
- **PyTorch** (Framework used to construct and train the deep neural network classifier (3 hidden layers × 5000 neurons)) — https://pytorch.org/

## Evaluation signals

- All five models (DNN, RF, KNN, SVM, LR) are trained on the identical training split — any divergence in split invalidates the comparison
- 10-fold cross-validation is performed on the HCC-T training set before held-out test evaluation
- The DNN attains the highest AUC among the five models on the independent test set, reproducing the Supplementary Table 3 ranking
- All reported AUC values fall in [0, 1] and are computed on held-out data, not the training set

## Limitations

- Generalizability was bounded in the paper; the determined applicability boundary was specific to the proteomic/metabolomic datasets tested, so AUC superiority of the DNN may not transfer to other cohorts
- The benchmark covers only five classifier families; other deep or ensemble architectures were not compared
- DeepRTAlign's default QC workflow (score-based, fdr 0.01) can report very few results when samples differ greatly or many features share nearly identical m/z — such degenerate alignments would degrade the classifier inputs

## Evidence

- [abstract] A robust classifier was trained and validated using MS features aligned by DeepRTAlign to predict early recurrence of hepatocellular carcinoma: [evidence span withheld — anchor status: not_found]
- [discussion] aligned features outperform peptide features: "The results showed that aligned MS features have more effective information compared with peptides and proteins."
- [readme] PyTorch dependency: "Install Pytorch CPU version, please refer to https://pytorch.org/."
- [readme] DeepRTAlign feature list support: "DeepRTAlign supports Dinosaur, OpenMS, MaxQuant, XICFinder and any other text list containing m/z, charge, RT and Intensity information"
- [discussion] generalizability boundary: "and determined its generalizability boundary on multiple proteomic and metabolomic datasets."
