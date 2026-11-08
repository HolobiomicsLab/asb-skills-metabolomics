---
name: auroc-accuracy-classification-evaluation
description: Use when a trained or adapted classifier on SCMS metabolomics feature
  data must be scored on held-out cells and its AUROC, F-1, and accuracy compared
  against published baselines or adaptation thresholds.
when_to_use_negative:
- The model has not been trained or adapted on any task — evaluation requires a fitted
  classifier on a defined test split
- The data are raw, unpretreated SCMS spectra rather than the pretreated, peak-aligned
  feature matrix (10 ppm Geena2 alignment, TIC normalization, missing-value removal
  already applied)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_0121
tools:
- name: MetaPhenotype
  role: Meta-learning model being evaluated; its AUROC/F-1/accuracy are computed after
    1- or 5-shot MAML adaptation
  repo: github.com/songyuan93/MetaPhenotype
- name: Single-probe SCMS
  role: Source of the single-cell metabolomics data on which classification metrics
    are computed
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c02038
    title: 'MetaPhenotype: A Transferable Meta-Learning Model for Single-Cell Mass
      Spectrometry-Based Cell Phenotype Prediction Using Limited Number of Cells'
  source_content_hash: sha256:9118900dcd90f63c2ae02ab9537637163404ae1779863949612ff2e70e8cfe56
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MetaPhenotype
  - Single-probe SCMS
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# auroc-accuracy-classification-evaluation

## Summary

Evaluates binary cell-phenotype classifiers (e.g., MetaPhenotype vs. SVM/RF/LeNet/ANN baselines) on single-cell mass spectrometry (SCMS) data using AUROC, F-1 score, and prediction accuracy after 50 total epochs of training, benchmarking against reported thresholds (>90% for MetaPhenotype, <60% for SVM/RF/LeNet).

## When to use

Use when a trained or adapted classifier on SCMS metabolomics feature data must be scored on held-out cells and its AUROC, F-1, and accuracy compared against published baselines or adaptation thresholds.

## When NOT to use

- The model has not been trained or adapted on any task — evaluation requires a fitted classifier on a defined test split
- The data are raw, unpretreated SCMS spectra rather than the pretreated, peak-aligned feature matrix (10 ppm Geena2 alignment, TIC normalization, missing-value removal already applied)

## Inputs

- Trained/adapted MetaPhenotype or baseline classifier model checkpoint
- Held-out test split of pretreated, peak-aligned single-cell SCMS feature matrix (dataset.csv) for each melanoma cell group

## Outputs

- Per-method AUROC, F-1 score, and prediction accuracy for each cell group and shot setting
- Comparison table of classifier metrics across methods and transfer tasks (matching Figure 3 / Table S6)

## How to apply

After training or MAML inner-loop adaptation of the LeNet 1D-CNN backbone (or baseline classifiers like scikit-learn SVC/RandomForest), evaluate on the held-out test split of cells from each transfer task (treated IGR39/IGR37, untreated WM115/WM266-4, treated WM115/WM266-4). Compute AUROC, F-1 score, and prediction accuracy after 50 total epochs of training, as specified in the article's protocol. Aggregate per-method metrics across the three cell groups and both shot settings (1-shot, 5-shot) into a comparison table matching Figure 3 and Table S6. Judge success against paper benchmarks: MetaPhenotype should exceed 90% accuracy with 5 adaptation steps (>95% on IGR39/IGR37), while SVM/RF/LeNet should fall below 60% accuracy with AUROC of 0.4-0.6, and ANN should reach roughly 92% (treated IGR), 66% (untreated WM), and 29% (treated WM).

## Related tools

- **MetaPhenotype** (Meta-learning model being evaluated; its AUROC/F-1/accuracy are computed after 1- or 5-shot MAML adaptation) — github.com/songyuan93/MetaPhenotype
- **Single-probe SCMS** (Source of the single-cell metabolomics data on which classification metrics are computed)

## Examples

```
python base.py  # trains/evaluates MetaPhenotype on dataset.csv (dataloader.py reads dataset.csv; meta.py holds the model)
```

## Evaluation signals

- MetaPhenotype accuracy on unseen cell groups (treated IGR39/IGR37, WM115/WM266-4) exceeds 90% after 5 adaptation steps
- SVM, RF, and LeNet baselines fall below 60% accuracy with AUROC values in the 0.4-0.6 range
- ANN baseline yields approximately 92%, 66%, and 29% accuracy on treated IGR39/IGR37, untreated WM115/WM266-4, and treated WM115/WM266-4 respectively
- Metrics are computed after 50 total epochs of training, per the article's stated protocol
- Aggregated table values are consistent across all three cell groups and both 1-shot and 5-shot settings

## Limitations

- AUROC values as low as 0.4-0.6 for conventional models (SVM, RF, LeNet) indicate they can perform near or below chance on limited SCMS data
- Conventional machine learning models can encounter overfitting when limited training data are available
- MAML training is memory- and computationally intensive due to inner-loop gradient steps and is sensitive to hyperparameters

## Evidence

- [intro] Metrics computed after fixed epochs: "we computed the area under the receiver operating characteristic curve (AUROC), F-1 score, and prediction accuracy after 50 total epochs of training"
- [abstract] Rapid adaptation threshold: [evidence span withheld — anchor status: external_doc]
- [results] Baseline AUROC failure range: [evidence span withheld — anchor status: not_found]
- [results] Five-step adaptation benchmark: [evidence span withheld — anchor status: not_found]
- [intro] Overfitting caveat: "when limited data are available for training, conventional machine learning models can encounter overfitting"
