---
name: cross-task-transfer-benchmark-comparison
description: Use when you need to justify a new meta-learning/transfer model by comparing
  it against SVM, random forest, LeNet, and ANN baselines trained on identical single-cell
  mass-spectrometry splits, especially when training cell counts are small.
when_to_use_negative:
- The goal is to evaluate MetaPhenotype itself or tune its meta-learning hyperparameters
  (adaptation shots, epochs) — this skill only reproduces the conventional-classifier
  baselines reported in the paper, not the meta-learning pipeline.
- Input data have not gone through the SCMS pretreatment pipeline (noise/background
  subtraction, TIC normalization, Geena2 peak alignment at 10 ppm mass tolerance,
  missing-value removal); training classifiers on unaligned raw peak lists is out
  of scope.
- The dataset is large and conventional deep learning is appropriate — the paper's
  premise is that ambient SCMS yields relatively small cell numbers, so when data
  are plentiful this benchmark adds little beyond what standard supervised pipelines
  already provide.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: MetaPhenotype
  role: Source of the 1D-CNN (LeNet) backbone architecture and the reference splits/protocol
    that the baselines are compared against; repository code (base.py, dataloade.py,
    meta.py) defines the data loader and model structure.
  repo: https://github.com/songyuan93/MetaPhenotype
- name: Single-probe SCMS
  role: Experimental technique that generated the single-cell mass-spectrometry feature
    matrix (metabolomic profiles of melanoma cell pairs) used as the benchmark input
    data.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c02038
    title: 'MetaPhenotype: A Transferable Meta-Learning Model for Single-Cell Mass
      Spectrometry-Based Cell Phenotype Prediction Using Limited Number of Cells'
  source_content_hash: sha256:52eb2d36adccb448c8e37d6be4b8e9503d458202b9a2976192b3435eaa65248d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/songyuan93/MetaPhenotype
  tools:
  - MetaPhenotype
  - Single-probe SCMS
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# cross-task-transfer-benchmark-comparison

## Summary

Benchmarks conventional ML classifiers (SVM, random forest, LeNet-style 1D-CNN, and a fully connected ANN) against a meta-learning model on the same pretreated single-cell mass-spectrometry feature splits, quantifying how much phenotype-classification performance is attributable to the meta-learning approach rather than any classifier. It grounds claims of transferability by exposing how baselines degrade (<60% accuracy, AUROC 0.4–0.6) when SCMS training data are limited.

## When to use

Use when you need to justify a new meta-learning/transfer model by comparing it against SVM, random forest, LeNet, and ANN baselines trained on identical single-cell mass-spectrometry splits, especially when training cell counts are small.

## When NOT to use

- The goal is to evaluate MetaPhenotype itself or tune its meta-learning hyperparameters (adaptation shots, epochs) — this skill only reproduces the conventional-classifier baselines reported in the paper, not the meta-learning pipeline.
- Input data have not gone through the SCMS pretreatment pipeline (noise/background subtraction, TIC normalization, Geena2 peak alignment at 10 ppm mass tolerance, missing-value removal); training classifiers on unaligned raw peak lists is out of scope.
- The dataset is large and conventional deep learning is appropriate — the paper's premise is that ambient SCMS yields relatively small cell numbers, so when data are plentiful this benchmark adds little beyond what standard supervised pipelines already provide.

## Inputs

- dataset.csv — pretreated, peak-aligned SCMS metabolomics feature matrix (Single-probe data from primary/metastatic melanoma cell pairs, with and without vemurafenib)
- Predefined train/validation/test cell-level splits matching those used for MetaPhenotype
- Task labels distinguishing transfer tasks: treated IGR39/IGR37, untreated WM115/WM266-4, treated WM115/WM266-4

## Outputs

- Results table of accuracy and AUROC for each baseline (SVM, RF, LeNet, ANN) on each transfer task
- Comparison against MetaPhenotype's reported >90–95% accuracy and >0.9 AUROC on identical splits

## How to apply

Load the peak-aligned, TIC-normalized single-probe SCMS feature matrix (dataset.csv in the MetaPhenotype repo) and split cells into train/validation/test sets identical to those used for the meta-learning model. Train scikit-learn SVC and RandomForestClassifier on the training split, and evaluate accuracy plus AUROC (alongside F1) on the held-out test split. Retrain the LeNet-style 1D-CNN backbone alone — i.e. MetaPhenotype's LeNet backbone without the MAML meta-learning loop — on the same splits, since the authors chose 1D-CNN for its lower computational complexity on 1D SCMS signals. Also train a conventional fully connected ANN on the three transfer tasks: treated IGR39/IGR37 cells, untreated WM115/WM266-4 cells, and treated WM115/WM266-4 cells, which the paper reported at 92%, 66%, and 29% accuracy respectively. Compile accuracies and AUROC values into a results table and interpret degradation (baselines <60% accuracy, AUROC ~0.4–0.6) as evidence that the meta-learning component, not the backbone, drives the reported >90–95% accuracy.

## Related tools

- **MetaPhenotype** (Source of the 1D-CNN (LeNet) backbone architecture and the reference splits/protocol that the baselines are compared against; repository code (base.py, dataloade.py, meta.py) defines the data loader and model structure.) — https://github.com/songyuan93/MetaPhenotype
- **Single-probe SCMS** (Experimental technique that generated the single-cell mass-spectrometry feature matrix (metabolomic profiles of melanoma cell pairs) used as the benchmark input data.)

## Examples

```
python base.py  # trains/evaluates on dataset.csv; adapt backbone by removing the MAML loop in meta.py, then fit sklearn SVC/RandomForestClassifier on the same splits
```

## Evaluation signals

- SVM, RF, and LeNet baselines should land below 60% test accuracy and show AUROC in roughly the 0.4–0.6 range on the SCMS splits.
- ANN accuracies should reproduce the reported task-dependent pattern: ~92% on treated IGR39/IGR37 cells but only ~66% on untreated and ~29% on treated WM115/WM266-4 cells.
- All baselines must be trained and evaluated on exactly the same train/validation/test splits as MetaPhenotype; any split mismatch invalidates the comparison.
- MetaPhenotype's own results should exceed 90% accuracy (paper reports >95%) on the same splits, confirming the performance gap is attributable to meta-learning.
- The feature matrix must be post-pretreatment (noise subtraction, background subtraction, TIC normalization, 10 ppm Geena2 peak alignment, missing values removed) before any classifier is fit.

## Limitations

- Baseline degradation is specific to limited SCMS sample sizes; with large training sets conventional models may behave differently, so conclusions do not transfer to data-rich settings.
- LeNet here means the 1D-CNN backbone alone, not the full MAML loop; comparing against the full MetaPhenotype is a separate skill.
- The comparison is anchored to the paper's specific cell pairs and drug-treatment tasks (IGR39/IGR37 and WM115/WM266-4, with/without vemurafenib); results may not generalize to other cell types or instruments.
- Task-dependent ANN behavior (e.g. only 29% accuracy on treated WM115/WM266-4) shows that baseline performance is highly sensitive to which transfer task is used, so a single-task comparison can be misleading.

## Evidence

- [results] MetaPhenotype achieved over 95% prediction accuracy, whereas SVM, RF, and LeNet displayed <60% accuracy and AUROC values of only 0.4-0.6: [evidence span withheld — anchor status: not_found]
- [results] ANN reached 92% accuracy on treated IGR39/IGR37 cells but only 66% and 29% on untreated and treated WM115/WM266-4 cells: [evidence span withheld — anchor status: not_found]
- [intro] 1D-CNN, which is easier to train due to its less computational complexity, would be clearly preferable for analyzing 1D signals: [evidence span withheld — anchor status: not_found]
- [readme] The input data were included in the Input data folder which named dataset.csv: [evidence span withheld — anchor status: external_doc]
