---
name: conventional-ann-classification
description: Use when benchmarking a meta-learning or transfer-learning SCMS classifier
  and you need a conventional deep-learning baseline trained on the same train/validation/test
  splits of pretreated, peak-aligned single-cell metabolomics data.
when_to_use_negative:
- The goal is high-accuracy phenotype prediction from very few labeled cells — the
  conventional ANN reaches only 29–66% accuracy on some WM115/WM266-4 tasks, so the
  meta-learning model is the appropriate choice instead
- Input data have not been pretreated (noise/background subtraction, TIC normalization,
  10 ppm peak alignment, missing-value removal); the ANN expects the aligned feature
  matrix
- The data are bulk (population-level) metabolomics rather than single-cell profiles,
  since the paper's motivation is heterogeneity at the single-cell level
edam_operation: http://edamontology.org/operation_2936
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_3474
tools:
- name: MetaPhenotype
  role: The reference meta-learning model whose training splits and evaluation metrics
    (accuracy, AUROC) the ANN baseline must replicate for fair comparison
  repo: github.com/songyuan93/MetaPhenotype
- name: Single-probe SCMS
  role: Instrumentation that generates the per-cell metabolomics profiles used as
    the ANN's input features
- name: Geena2
  role: Online tool used upstream for peak alignment (mass tolerance 10 ppm) to produce
    the feature matrix the ANN trains on
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c02038
    title: 'MetaPhenotype: A Transferable Meta-Learning Model for Single-Cell Mass
      Spectrometry-Based Cell Phenotype Prediction Using Limited Number of Cells'
  source_content_hash: sha256:8e95f60e7f12ec50212036c0df1b5001c22ff458c361544f1c9b88eed851c494
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Geena2
  - MetaPhenotype
  - Single-probe SCMS
  edam_operation: http://edamontology.org/operation_2936
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# conventional-ann-classification

## Summary

Train a fully connected artificial neural network (ANN) on pretreated single-cell mass spectrometry (SCMS) metabolomics feature matrices as a conventional baseline against MetaPhenotype, reporting accuracy and AUROC for each primary/metastatic melanoma cell-pair classification task.

## When to use

Use when benchmarking a meta-learning or transfer-learning SCMS classifier and you need a conventional deep-learning baseline trained on the same train/validation/test splits of pretreated, peak-aligned single-cell metabolomics data.

## When NOT to use

- The goal is high-accuracy phenotype prediction from very few labeled cells — the conventional ANN reaches only 29–66% accuracy on some WM115/WM266-4 tasks, so the meta-learning model is the appropriate choice instead
- Input data have not been pretreated (noise/background subtraction, TIC normalization, 10 ppm peak alignment, missing-value removal); the ANN expects the aligned feature matrix
- The data are bulk (population-level) metabolomics rather than single-cell profiles, since the paper's motivation is heterogeneity at the single-cell level

## Inputs

- Pretreated and peak-aligned single-cell mass spectrometry feature matrix (dataset.csv) for a primary/metastatic melanoma cell pair, with per-cell labels
- Train/validation/test split definitions matching those used for the meta-learning model

## Outputs

- Test-split classification accuracy for the ANN baseline per task (e.g. 92% treated IGR, 66% untreated WM, 29% treated WM)
- Test-split AUROC for the ANN baseline per task
- Comparison table of ANN vs MetaPhenotype (>90% accuracy) and other baselines

## How to apply

Load the pretreated and peak-aligned single-cell MS feature matrix (the dataset.csv style table produced by the established SCMS pretreatment protocol: noise subtraction, background subtraction, TIC normalization, peak alignment with Geena2 at 10 ppm, missing-value removal). Split the cells into train/validation/test sets identical to those used for the meta-learning model, for the three transfer tasks: treated IGR39/IGR37, untreated WM115/WM266-4, and treated WM115/WM266-4. Train a standard fully connected ANN on the training split and compute accuracy and AUROC on the test split after the same 50 epochs of training used for other baselines. Record the results side-by-side with SVM, random forest, and LeNet baselines to quantify how poorly a conventional ANN handles limited SCMS data without meta-learning.

## Related tools

- **MetaPhenotype** (The reference meta-learning model whose training splits and evaluation metrics (accuracy, AUROC) the ANN baseline must replicate for fair comparison) — github.com/songyuan93/MetaPhenotype
- **Single-probe SCMS** (Instrumentation that generates the per-cell metabolomics profiles used as the ANN's input features)
- **Geena2** (Online tool used upstream for peak alignment (mass tolerance 10 ppm) to produce the feature matrix the ANN trains on)

## Examples

```
python base.py  # trains baseline models on dataset.csv from the Input data folder; evaluate ANN accuracy and AUROC on the identical train/validation/test splits used for MetaPhenotype
```

## Evaluation signals

- ANN accuracy on treated IGR39/IGR37 should be near 92%; on untreated WM115/WM266-4 near 66%; on treated WM115/WM266-4 near 29%
- AUROC for weaker ANN tasks should fall in the 0.4–0.6 range reported for conventional baselines
- The same train/validation/test splits as MetaPhenotype must be used; any comparison with different splits is invalid
- Training should run the same number of epochs (50 total epochs) as the other models before computing metrics
- ANN performance should be reported alongside SVM, random forest, and LeNet baselines, all below MetaPhenotype's >90% accuracy

## Limitations

- When limited SCMS data are available for training, conventional machine learning models can encounter overfitting
- Ambient SCMS methods yield relatively small numbers of cells, so conventional ANN performance degrades badly on hard tasks (29% accuracy on treated WM cells)
- Without meta-learning, the ANN lacks the rapid adaptation and transferability demonstrated by MetaPhenotype

## Evidence

- [results] ANN baseline accuracy range: [evidence span withheld — anchor status: not_found]
- [results] MetaPhenotype outperforms conventional baselines: [evidence span withheld — anchor status: not_found]
- [readme] README input data instructions: [evidence span withheld — anchor status: external_doc]
- [readme] README running instructions: [evidence span withheld — anchor status: external_doc]
