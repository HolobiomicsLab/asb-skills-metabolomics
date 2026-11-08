---
name: pathway-hyperlink-classification-training
description: Use when labeled pathway sets (positives from TomatoCyc, negatives from
  MetaCyc plus random sampling) must be classified from metabolite correlation network
  waveforms, or when reproducing NetAurHPD-style AUC benchmarks.
when_to_use_negative:
- Input is raw metabolomics intensity data rather than already-auralized pathway signals
  — imputation (probabilistic PCA via pcaMethods in Rstudio), Metabolite CN construction
  (Pearson |r| >= 0.3, p <= 0.01), and auralization must happen first
- The network is a generic centrality-estimation task rather than hyperlink/pathway
  classification — use the centrality-learning auralization pipelines instead of the
  M5 classifier
- Only two compounds per pathway are observed in the Metabolite CN, since the dataset
  keeps only pathways with more than two compounds present
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3941
tools:
- name: NetAurHPD
  role: Framework whose M5 classifier and Youden-threshold optimization perform the
    pathway classification; source code on GitHub
  repo: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- name: M5
  role: Very deep convolutional neural network trained as the binary classification
    model on pathway waveform signals
  repo: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- name: Adam optimizer
  role: Optimization algorithm for M5 training with learning rate 0.0001
- name: PyTorch
  role: Implementation backend for network auralization and the M5 architecture
  repo: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:ea853e8e1ceda3ee95a2945e93ca8ba10a5b4bf67e276bd6680950f94dd1ba79
  source_papers:
  - doi: 10.48550/arxiv.2410.22030
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
  tools:
  - Adam optimizer
  - M5
  - NetAurHPD
  - PyTorch
  - Rstudio
  - pcaMethods
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3941
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# pathway-hyperlink-classification-training

## Summary

Trains the M5 deep convolutional classifier on auralized pathway signals (averaged metabolite waveforms from Metabolite CNs) to predict presence/absence of metabolic pathways, then optimizes the decision threshold via Youden's J statistic and computes micro-average AUC under 10-fold cross-validation.

## When to use

Use when labeled pathway sets (positives from TomatoCyc, negatives from MetaCyc plus random sampling) must be classified from metabolite correlation network waveforms, or when reproducing NetAurHPD-style AUC benchmarks.

## When NOT to use

- Input is raw metabolomics intensity data rather than already-auralized pathway signals — imputation (probabilistic PCA via pcaMethods in Rstudio), Metabolite CN construction (Pearson |r| >= 0.3, p <= 0.01), and auralization must happen first
- The network is a generic centrality-estimation task rather than hyperlink/pathway classification — use the centrality-learning auralization pipelines instead of the M5 classifier
- Only two compounds per pathway are observed in the Metabolite CN, since the dataset keeps only pathways with more than two compounds present

## Inputs

- Per-metabolite waveform representations obtained from network auralization of tomato pericarp Metabolite CNs (2001, 2003, 2004 seasons)
- Labeled pathway dict of the form ID: {'label': 'positive'|'negative', 'nodes': [...]} built from TomatoCyc (169 positives) and MetaCyc + 85 random negatives
- Hyperparameters from config: stride=8, n_channel=32, lr, epochs

## Outputs

- Micro-average AUC (target values: 0.87 for 2001, 0.857 for 2003, 0.884 for 2004) from pooled validation-fold predictions
- Accuracy, recall, FPR, precision, and F1 at the Youden-optimal threshold, matching NetAurHPD rows of Table 1(a)-(c)
- Trained M5 classifier and per-pathway class predictions

## How to apply

Construct the labeled pathway dataset from 169 TomatoCyc pathways as positives and MetaCyc non-plant pathways plus 85 randomly sampled negative pathways over the 52 common metabolites, keeping only pathways with more than two compounds appearing in the Metabolite CN. Run network auralization (Algorithm 1, PyTorch) on each Metabolite CN to get per-metabolite waveforms, then average each pathway's participants' waveforms by index into a single pathway signal. Train the M5 convolutional network (four convolution blocks, Adam optimizer, learning rate 0.0001, stride 8) with 10-fold cross-validation, downsampling negatives so each session is balanced positive-negative. After validation-fold predictions are pooled, learn the optimal classification threshold from the Youden index (maximizing TPR - FPR) and compute micro-average AUC, accuracy, recall, FPR, precision, and F1. The README's result_optimization_and_analysis module implements the threshold optimization and metric calculation, and the NetAurHPD_M5 module handles training and evaluation.

## Related tools

- **NetAurHPD** (Framework whose M5 classifier and Youden-threshold optimization perform the pathway classification; source code on GitHub) — https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- **M5** (Very deep convolutional neural network trained as the binary classification model on pathway waveform signals) — https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- **Adam optimizer** (Optimization algorithm for M5 training with learning rate 0.0001)
- **PyTorch** (Implementation backend for network auralization and the M5 architecture) — https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method

## Examples

```
instrument.networkx_auralization(train_positive_hyperlink_dict, train_hyperlink_dict, test_hyperlink_dict, nodes, how_graph=True); NetAurHPD_DL_architecture.predict(train_hyperlinks_waveforms, y_train, test_hyperlinks_waveforms, y_test, lr=config.lr, total_iters=config.epochs)
```

## Evaluation signals

- Micro-average AUC pooled across validation folds approximates 0.87 (2001), 0.857 (2003), and 0.884 (2004)
- Assembled per-year metrics match the NetAurHPD rows in Table 1(a)-(c)
- Each training session is balanced positive-negative after downsampling negatives
- Every pathway in the dataset appears exactly once in a validation set across the 10-fold cross-validation
- Threshold chosen by Youden index maximizes TPR - FPR on validation predictions

## Limitations

- NetAurHPD has a low level of explainability and cannot provide biologists with a full understanding of its results
- Models do not consistently improve with more data; in some sessions the AUCs of all models decreased
- Sparsity of labeled data is the most important challenge in metabolic pathway prediction

## Evidence

- [methods] we will train M5 as classification model (Supplementary Fig 7.g) to predict the class of each pathway: [evidence span withheld — anchor status: not_found]
- [methods] In 2001 the micro-average AUC was 0.87 ... For 2003 the micro-average AUC was 0.857 ... In 2004 the micro-average AUC was 0.884: [evidence span withheld — anchor status: not_found]
