---
name: boxplot-distribution-analysis-of-model-variability
description: Use when a model has been retrained many times (e.g., 20 sessions of
  10-fold cross-validation) under several configurations of a numeric factor (e.g.,
  number of data Augmentations), and you must compare central tendency AND variability
  of AUC distributions rather than single-point scores.
when_to_use_negative:
- You have only a single train/test split or one cross-validation run, so there is
  no distribution of AUC values to summarize as a box plot
- The comparison factor is a single pairwise contrast (e.g., baseline vs CHESHIRE
  only) rather than an ordered multi-level ablation such as 1/3/5/7 Augmentations
- The output metric is deterministic (no session-to-session variance), making whisker-range
  analysis of variability meaningless
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: NetAurHPD
  role: Model whose repeated-session AUC distributions are aggregated into the per-Augmentation-count
    box plots
  repo: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- name: PyTorch
  role: Framework used to train the M5 classifier (Adam optimizer) whose per-session
    AUC variability is analyzed
  repo: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:8d57b904d6d259449e8c9a8e295168afeaad9812ce2e8cd476d544d7e921d172
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# boxplot-distribution-analysis-of-model-variability

## Summary

Aggregates micro-average AUC results from repeated cross-validation sessions (e.g., 20 sessions of 10-fold CV) into per-configuration box plots (median, mean, maximum AUC, interquartile range, and whisker range) to quantify both central performance and run-to-run variability of a model, as done for NetAurHPD across 0/1/3/5/7 data Augmentations.

## When to use

Use when a model has been retrained many times (e.g., 20 sessions of 10-fold cross-validation) under several configurations of a numeric factor (e.g., number of data Augmentations), and you must compare central tendency AND variability of AUC distributions rather than single-point scores.

## When NOT to use

- You have only a single train/test split or one cross-validation run, so there is no distribution of AUC values to summarize as a box plot
- The comparison factor is a single pairwise contrast (e.g., baseline vs CHESHIRE only) rather than an ordered multi-level ablation such as 1/3/5/7 Augmentations
- The output metric is deterministic (no session-to-session variance), making whisker-range analysis of variability meaningless

## Inputs

- Micro-average AUC values per session, per model configuration (baseline vs 1/3/5/7 Augmentations), per year (2001, 2003, 2004), from 20 sessions of 10-fold cross-validation
- Configuration labels indicating the number of Augmentations used for training (0, 1, 3, 5, 7)

## Outputs

- Box plots per number of Augmentations reporting median, mean, maximum AUC, interquartile range, and whisker range (paper Fig 4 format)
- Average whisker-range statistic per configuration quantifying variability reduction (baseline ~0.082 vs augmented ~0.07)

## How to apply

For each configuration of the ablation factor (baseline without augmentation, plus 1, 3, or 5, or 7 Augmentations randomly drawn from the 10 pre-computed Augmentations), collect the micro-average AUC from every cross-validation session across the 2001, 2003, and 2004 tomato pericarp harvesting seasons, where each pathway appeared once in a validation set. Group the AUC values by configuration and, following the paper's Fig 4 format, summarize each group as a box plot reporting the median, mean, maximum AUC, interquartile range, and whisker range. Decision point: select the configuration whose median/mean/maximum AUC is highest while whisker range is smallest — here one Augmentation, which reduced the average whisker range from 0.082 (NetAurHPD alone) to 0.07. Do not conclude superiority from means alone; the variability reduction is the co-primary criterion, and remember the paper's caveat that models do not consistently improve with more data, so per-session decreases are expected and must be visible in the box spread rather than averaged away.

## Related tools

- **NetAurHPD** (Model whose repeated-session AUC distributions are aggregated into the per-Augmentation-count box plots) — https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- **PyTorch** (Framework used to train the M5 classifier (Adam optimizer) whose per-session AUC variability is analyzed) — https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method

## Evaluation signals

- Average whisker range for augmented models decreases from about 0.082 (baseline, NetAurHPD alone) to about 0.07
- NetAurHPD with one Augmentation shows the best median, mean, AND maximum AUC among 0/1/3/5/7 configurations
- Per-year best micro-average AUC values reproduce the reported 0.87 (2001), 0.857 (2003), and 0.884 (2004)
- Every pathway appears exactly once in a validation set across the 20 sessions of 10-fold cross-validation
- Box plot components (median, mean, max, IQR, whisker range) are all reported per configuration, matching the paper's Fig 4 format

## Limitations

- Models do not consistently improve with more data; in some sessions the AUCs of all models decreased, so box plots must show per-session regressions rather than hide them
- Aggregate box statistics can mask configuration-specific bad sessions; the paper reports some sessions where all models' AUCs dropped
- Comparison is grounded in the M5 classifier with the paper's Adam-optimizer training regime; transferring the variability conclusion to other architectures is not validated by this evidence

## Evidence

- [methods] Training with one Augmentation improved NetAurHPD stability and reduced variability: "the range decreases from an average of 0.082 for NetAurHPD alone to an average of 0.07"
- [methods] Repeated cross-validation protocol underlying the AUC distribution: "We evaluated the proposed solution with 10-fold cross-validation, where each pathway in the dataset appeared once in the validation set"
- [readme] Metric source for the box plots (AUC produced by the pipeline's optimization step): "Optimizes the threshold using Youden's J Statistic and calculates several evaluation metrics: AUC, accuracy, recall, precision, F1 score And fpr."
- [discussion] Caveat that per-session AUC can decrease with more data: "the models do not consistently improve with more data. In some sessions, the AUCs of all models decreased"
