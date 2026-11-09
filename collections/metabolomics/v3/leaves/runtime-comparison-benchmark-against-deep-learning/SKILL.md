---
name: runtime-comparison-benchmark-against-deep-learning
description: Use when a deep learning classifier (e.g. massNet on 85,062-variable
  mass spectra) must be benchmarked against a classical baseline like SVM on identical
  train/test splits and metrics.
when_to_use_negative:
- The analysis goal is multi-class tissue classification — the source model is demonstrated
  only for binary normal-vs-tumor classification, so a binary SVM baseline comparison
  would not be informative.
- The input is already a low-dimensional feature table (e.g. a short list of hand-picked
  m/z peaks) where the 174-fold speed advantage over SVM, driven by high-dimensional
  spectra, would not apply.
edam_operation: http://edamontology.org/operation_2938
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3474
tools:
- name: massNet
  role: Deep learning classifier whose accuracy and runtime serve as the reference
    values in the side-by-side benchmark table
  repo: https://github.com/wabdelmoula/massNet
- name: Python
  role: Implementation environment (3.6.12) used to train the SVM and assemble the
    comparison table
  repo: https://github.com/wabdelmoula/massNet
- name: support vector machine (SVM)
  role: Classical machine learning baseline trained on identical pixel-wise normal/tumor
    labels and evaluated on the same withheld test set
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btac032
    title: 'massNet: integrated processing and classification of spatially resolved
      mass spectrometry data using deep learning for rapid tumor delineation'
  source_content_hash: sha256:c603f3ea105aa033e11ee042f797776376e2bcbd745632c592c053a41272c0bd
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/wabdelmoula/massNet
  repo_url: https://github.com/wabdelmoula/massNet
  tools:
  - Python
  - massNet
  - support vector machine (SVM)
  edam_operation: http://edamontology.org/operation_2938
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# runtime-comparison-benchmark-against-deep-learning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks a classical machine learning baseline (SVM) against a deep learning classifier (massNet) on the same withheld MALDI FT-ICR MSI test set, assembling accuracy, precision, recall, F1-score, ROC, and runtime into a side-by-side table to quantify the speed/accuracy trade-off.

## When to use

Use when a deep learning classifier (e.g. massNet on 85,062-variable mass spectra) must be benchmarked against a classical baseline like SVM on identical train/test splits and metrics.

## When NOT to use

- The analysis goal is multi-class tissue classification — the source model is demonstrated only for binary normal-vs-tumor classification, so a binary SVM baseline comparison would not be informative.
- The input is already a low-dimensional feature table (e.g. a short list of hand-picked m/z peaks) where the 174-fold speed advantage over SVM, driven by high-dimensional spectra, would not apply.

## Inputs

- Withheld MALDI FT-ICR MSI test set (9,798 normal and 2,861 tumor spectra; 85,062 m/z variables) in HDF5 format
- MALDI FT-ICR MSI training set with pixel-wise normal/tumor labels (10,024 normal and 3,644 tumor spectra; 85,062 m/z variables)
- massNet test-set predictions and reported classification runtime for the speed-ratio computation

## Outputs

- Side-by-side benchmark table (Table 1) with SVM and massNet accuracy, precision, recall, F1-score, and runtime (e.g. svm_benchmark_table.csv)
- SVM-to-massNet runtime speed ratio (expected: ~174-fold gain in speed)
- SVM ROC curve on the withheld test set

## How to apply

Train an SVM classifier (sklearn 0.23.2 in the massNet environment) on the pixel-wise normal/tumor labels of the training set (10,024 normal and 3,644 tumor spectra with 85,062 m/z variables), then apply it to the withheld test set (9,798 normal and 2,861 tumor spectra) loaded from the converted HDF5 MSI file. Evaluate the predictions with the same established metrics used for massNet: accuracy, precision, recall, F1-score, and ROC, ensuring both models see identical data splits so the comparison is fair. Measure the classification runtime of the trained SVM on the test set and compute the speed ratio against the massNet runtime to reproduce the reported 174-fold gain. Assemble both models' metrics and runtimes into a single side-by-side table (Table 1) and save it, e.g. as svm_benchmark_table.csv, verifying the computed speed ratio is consistent with the published figure.

## Related tools

- **massNet** (Deep learning classifier whose accuracy and runtime serve as the reference values in the side-by-side benchmark table) — https://github.com/wabdelmoula/massNet
- **Python** (Implementation environment (3.6.12) used to train the SVM and assemble the comparison table) — https://github.com/wabdelmoula/massNet
- **support vector machine (SVM)** (Classical machine learning baseline trained on identical pixel-wise normal/tumor labels and evaluated on the same withheld test set)

## Examples

```
from sklearn.svm import SVC; svm = SVC(); svm.fit(X_train, y_train); preds = svm.predict(X_test); import time; t = time.time() - t0
```

## Evaluation signals

- The reproduced table includes SVM accuracy, precision, recall, F1-score, and runtime alongside the massNet values
- The computed SVM-to-massNet speed ratio is consistent with the reported 174-fold gain
- SVM test-set accuracy falls at or slightly below the massNet value (93.45% vs. 95%) on the identical withheld test set
- Both classifiers are evaluated on the same split: training set of 10,024 normal and 3,644 tumor spectra; test set of 9,798 normal and 2,861 tumor spectra with 85,062 m/z variables
- The comparison uses the same metric set the paper reports: accuracy, precision, recall, F1-score, and ROC

## Limitations

- The benchmark is limited to binary normal-vs-tumor classification; multi-class extension is left for future development, so the comparison does not cover multi-class tasks.
- Ground truth labels derive solely from pathologist-annotated H&E histology transferred to the MSI data, an approximation that may be suboptimal for intra-tumor heterogeneity.
- No software version numbers or hyperparameter values for the trained massNet model are reported, which limits exact reproducibility of the runtime comparison.

## Evidence
