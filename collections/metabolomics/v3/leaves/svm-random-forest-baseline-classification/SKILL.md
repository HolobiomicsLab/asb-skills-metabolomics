---
name: svm-random-forest-baseline-classification
description: Use when benchmarking a new SCMS cell-phenotype classifier against conventional
  ML baselines on the same pretreated single-cell metabolomic feature matrix and identical
  train/validation/test splits.
when_to_use_negative:
- When the goal is cell-phenotype prediction from limited SCMS data with few adaptation
  samples — that is the MetaPhenotype meta-learning workflow, not a conventional baseline.
- When the input is raw, unprocessed single-cell spectra — conventional baselines
  here assume the data have already undergone noise subtraction, background subtraction,
  TIC normalization, and peak alignment.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3524
tools:
- name: MetaPhenotype
  role: Provides the pretreated/peak-aligned input feature matrix and reference splits;
    the same LeNet 1D-CNN backbone is retrained alone as a baseline, and its reported
    accuracy (>95%) is the comparison target.
  repo: github.com/songyuan93/MetaPhenotype
- name: Single-probe SCMS
  role: Source of the single-cell mass spectrometry metabolomic profiles that form
    the feature matrix the baselines are trained and evaluated on.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c02038
    title: 'MetaPhenotype: A Transferable Meta-Learning Model for Single-Cell Mass
      Spectrometry-Based Cell Phenotype Prediction Using Limited Number of Cells'
  source_content_hash: sha256:adfbffd9da5e8b47a2a034c2f76420f950571ac586f9e341fa7e881eb51ec647
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MetaPhenotype
  - Single-probe SCMS
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# svm-random-forest-baseline-classification

## Summary

Trains conventional machine learning classifiers (SVM, random forest, LeNet-style 1D-CNN backbone alone, and fully connected ANN) on the same pretreated, peak-aligned single-cell mass spectrometry feature matrix and identical train/validation/test splits used for MetaPhenotype, to establish baseline accuracy and AUROC for comparison.

## When to use

Use when benchmarking a new SCMS cell-phenotype classifier against conventional ML baselines on the same pretreated single-cell metabolomic feature matrix and identical train/validation/test splits.

## When NOT to use

- When the goal is cell-phenotype prediction from limited SCMS data with few adaptation samples — that is the MetaPhenotype meta-learning workflow, not a conventional baseline.
- When the input is raw, unprocessed single-cell spectra — conventional baselines here assume the data have already undergone noise subtraction, background subtraction, TIC normalization, and peak alignment.

## Inputs

- Pretreated and peak-aligned single-cell mass spectrometry feature matrix (dataset.csv-style), e.g. melanoma cell-line pairs IGR39/IGR37 and WM115/WM266-4, with/without vemurafenib treatment
- Train/validation/test cell splits identical to those used for the MetaPhenotype evaluation

## Outputs

- Results table of accuracy and AUROC for each baseline model (SVM, random forest, LeNet backbone, ANN) per transfer task
- Reported baseline values for comparison: <60% accuracy for SVM/RF/LeNet, ANN accuracies of 92%/66%/29%, and AUROC 0.4–0.6

## How to apply

Load the pretreated and peak-aligned SCMS feature matrix (the dataset.csv-style input produced by noise subtraction, background subtraction, TIC normalization, and Geena2 peak alignment with 10 ppm mass tolerance) for the melanoma cell pairs being studied, and split cells into train/validation/test sets identical to those used for MetaPhenotype. Train an SVC (scikit-learn) and a RandomForestClassifier on the training split, then evaluate accuracy and AUROC on the test split. Train the LeNet-style 1D-CNN backbone alone — the same backbone MetaPhenotype uses, but without the MAML meta-learning outer loop — on the same splits. Finally, train a conventional fully connected ANN on the same splits for each transfer task (drug-treated cells, untreated cells, and a different cell-line pair), recording accuracy and AUROC for every model. The rationale is that MetaPhenotype's gains over >90% accuracy must be contextualized against baselines that the paper reports at <60% accuracy (SVM, RF, LeNet), AUROC in the 0.4–0.6 range, and ANN accuracies of 92%, 66%, and 29% on treated IGR, untreated WM, and treated WM cells respectively.

## Related tools

- **MetaPhenotype** (Provides the pretreated/peak-aligned input feature matrix and reference splits; the same LeNet 1D-CNN backbone is retrained alone as a baseline, and its reported accuracy (>95%) is the comparison target.) — github.com/songyuan93/MetaPhenotype
- **Single-probe SCMS** (Source of the single-cell mass spectrometry metabolomic profiles that form the feature matrix the baselines are trained and evaluated on.)

## Examples

```
python base.py  # trains the LeNet/MetaPhenotype backbone on dataset.csv; for scikit-learn baselines: python -c "from sklearn.svm import SVC; from sklearn.ensemble import RandomForestClassifier; svc=SVC(); rf=RandomForestClassifier()"
```

## Evaluation signals

- SVM, RF, and LeNet accuracies should fall below 60%, consistent with the article's reported baseline values.
- AUROC for SVM/RF/LeNet should be in the 0.4–0.6 range.
- ANN accuracies should land near the reported values of 92% (treated IGR39/IGR37), 66% (untreated WM115/WM266-4), and 29% (treated WM115/WM266-4).
- All baselines must be trained and tested on exactly the same train/validation/test splits used for MetaPhenotype, otherwise the comparison is invalid.

## Limitations

- Conventional deep learning models can overfit when limited SCMS training data are available, which the article cites as the motivation for meta-learning.
- PCA and similar approaches cannot effectively discriminate phenotypically similar cells such as IGR39/IGR37, so univariate or dimensionality-reduction baselines may be inadequate.
- Ambient SCMS methods yield relatively small numbers of cells, so baseline training data may be insufficient for strong conventional-model performance.

## Evidence

- [results] Baseline models underperform on SCMS splits: [evidence span withheld — anchor status: not_found]
