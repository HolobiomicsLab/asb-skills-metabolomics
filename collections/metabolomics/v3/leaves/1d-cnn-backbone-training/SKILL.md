---
name: 1d-cnn-backbone-training
description: Reference for benchmarking a LeNet-style 1D-CNN (without meta-learning)
  as a baseline classifier on single-cell mass spectrometry metabolomic feature tables,
  e.g. primary vs metastatic melanoma cells, to quantify the overfitting risk of conventional
  deep learning on limited SCMS data.
when_to_use_negative:
- Do not use when the goal is the actual high-accuracy phenotype prediction — the
  backbone alone underperforms (<60% accuracy) and MetaPhenotype is the appropriate
  model.
- Do not use when only very few cells are available for adaptation to a new task (e.g.
  1- or 5-shot) — without the meta-learning outer loop the model cannot rapidly adapt,
  which is the stated limitation motivating MetaPhenotype.
- Do not use on bulk metabolomics profiles — the skill is specific to per-cell SCMS
  feature vectors.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0154
tools:
- name: MetaPhenotype
  role: Provides the LeNet-style 1D-CNN backbone architecture and the meta.py code
    from which the baseline (backbone-only) variant is derived for benchmarking
  repo: github.com/songyuan93/MetaPhenotype
- name: Single-probe SCMS
  role: Generates the single-cell metabolomic profiles used as training and evaluation
    input for the 1D-CNN backbone
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c02038
    title: 'MetaPhenotype: A Transferable Meta-Learning Model for Single-Cell Mass
      Spectrometry-Based Cell Phenotype Prediction Using Limited Number of Cells'
  source_content_hash: sha256:64588aeba10db0ba5dd278b6c6581b11a0ced7691a15e0095c75a1ff337853a5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MetaPhenotype
  - Single-probe SCMS
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# 1d-cnn-backbone-training

## Summary

Trains a 1D-CNN backbone (LeNet-style) alone on single-cell mass spectrometry (SCMS) feature data to benchmark cell phenotype classification against the full MetaPhenotype meta-learning model, establishing how far a conventional deep model goes when trained data are limited.

## When to use

Reference for benchmarking a LeNet-style 1D-CNN (without meta-learning) as a baseline classifier on single-cell mass spectrometry metabolomic feature tables, e.g. primary vs metastatic melanoma cells, to quantify the overfitting risk of conventional deep learning on limited SCMS data.

## When NOT to use

- Do not use when the goal is the actual high-accuracy phenotype prediction — the backbone alone underperforms (<60% accuracy) and MetaPhenotype is the appropriate model.
- Do not use when only very few cells are available for adaptation to a new task (e.g. 1- or 5-shot) — without the meta-learning outer loop the model cannot rapidly adapt, which is the stated limitation motivating MetaPhenotype.
- Do not use on bulk metabolomics profiles — the skill is specific to per-cell SCMS feature vectors.

## Inputs

- dataset.csv — pretreated SCMS single-cell feature matrix (TIC-normalized, Geena2-aligned at 10 ppm) from the MetaPhenotype repo Input data folder
- Cell phenotype labels (primary vs metastatic melanoma, e.g. IGR39/IGR37, WM115/WM266-4, with/without vemurafenib treatment)
- Identical train/validation/test cell splits used for the MetaPhenotype runs

## Outputs

- Test-set prediction accuracy, AUROC, and F1 score for the LeNet-style 1D-CNN baseline
- Comparison table of backbone scores versus MetaPhenotype and SVM/RF/ANN baselines (expected backbone accuracy <60%, AUROC 0.4–0.6)

## How to apply

Load the pretreated, peak-aligned SCMS metabolomic feature matrix (peaks exported from Thermo Xcalibur Qual Browser 3.0, noise/background subtracted, TIC-normalized, aligned with Geena2 at 10 ppm mass tolerance, missing values removed) and use the same train/validation/test splits as MetaPhenotype. Build a LeNet-style 1D-CNN backbone — chosen because 1D-CNNs have less computational complexity and are easier to train on 1D spectra when limited training data are available — and train it on the cell phenotype labels without the MAML meta-learning outer loop. Evaluate after 50 total epochs of training using AUROC, F1 score, and prediction accuracy. Record results alongside conventional ML baselines (SVM, random forest, ANN) and the MetaPhenotype model: published baselines show SVM/RF/LeNet fall below 60% accuracy with AUROC of only 0.4–0.6, whereas the full meta-learner exceeds 90–95%, so the backbone's low scores are the expected, informative outcome of this benchmark.

## Related tools

- **MetaPhenotype** (Provides the LeNet-style 1D-CNN backbone architecture and the meta.py code from which the baseline (backbone-only) variant is derived for benchmarking) — github.com/songyuan93/MetaPhenotype
- **Single-probe SCMS** (Generates the single-cell metabolomic profiles used as training and evaluation input for the 1D-CNN backbone)

## Examples

```
python base.py  # trains the MetaPhenotype model from the repo on dataset.csv; the LeNet 1D-CNN backbone lives in meta.py
```

## Evaluation signals

- Backbone test accuracy should land below 60% when trained on the same limited SCMS splits — a value near or above 90% would indicate leakage of meta-learning components or data leakage into training.
- AUROC for the backbone-only model should fall in the 0.4–0.6 range on the reported melanoma transfer tasks.
- Metrics (accuracy, AUROC, F1) must be computed after 50 total epochs of training, matching the MetaPhenotype evaluation protocol.
- Train/val/test splits must be identical to those used for MetaPhenotype so the comparison isolates the model architecture, not data partitioning.
- Feature table must pass the pretreatment checks: TIC normalization applied, peaks aligned at 10 ppm mass tolerance via Geena2, and missing-value rows removed before training.

## Limitations

- When limited SCMS data are available, conventional models like the backbone-only 1D-CNN can encounter overfitting, which is why the paper introduces the meta-learning wrapper.
- PCA cannot effectively discriminate phenotypically similar cells such as IGR39/IGR37, so feature-space diagnostics alone do not predict whether the backbone will separate the classes.
- Baseline comparison is only meaningful on the paper's specific tasks (treated IGR, untreated/treated WM115/WM266-4); transfer performance on other cell pairs is not established by this benchmark.

## Evidence

- [intro] 1D-CNN, which is easier to train due to its less computational complexity, would be clearly preferable for analyzing 1D signals, particularly when limited training data are available: [evidence span withheld — anchor status: not_found]
- [intro] when limited data are available for training, conventional machine learning models can encounter overfitting: "when limited data are available for training, conventional machine learning models can encounter overfitting"
- [methods] we computed the area under the receiver operating characteristic curve (AUROC), F-1 score, and prediction accuracy after 50 total epochs of training: "we computed the area under the receiver operating characteristic curve (AUROC), F-1 score, and prediction accuracy after 50 total epochs of training"
- [readme] The code for MetaPhenotype model were included in meta.py file.: [evidence span withheld — anchor status: external_doc]
- [results] SVM, RF, and LeNet displayed <60% accuracy and AUROC values of only 0.4-0.6: [evidence span withheld — anchor status: not_found]
