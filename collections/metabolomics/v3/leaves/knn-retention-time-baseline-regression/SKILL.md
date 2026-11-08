---
name: knn-retention-time-baseline-regression
description: Use when benchmarking an ML retention-time predictor (e.g., a Keras regression
  model on ECFP fingerprints) against a structure-free nearest-neighbor baseline over
  METLIN SMRT-like RT data stratified by Tanimoto similarity thresholds.
when_to_use_negative:
- Validation molecules have no training-set counterpart above 50% Tanimoto similarity
  — the nearest-neighbor RT estimate is meaningless at near-zero structural similarity
- The task is RT prediction itself rather than benchmarking — use the trained DLM,
  since at most thresholds the k-NN median is only a baseline that the DLM significantly
  outperforms
- Retention times were measured under a different chromatographic method than the
  training set, as the paper's projection-across-CMs analysis shows models may not
  transfer directly
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Keras
  role: Trains the deep-learning regression model (4 hidden layers of 1000/500/200/100
    relu units, L2 0.0001, adam lr 0.01, MSE loss, 20 epochs, batch size 35) whose
    RT predictions are benchmarked against the k-NN baseline
- name: k-nearest neighbors regression
  role: 'Implements the naive baseline: retrieve the 3 most Tanimoto-similar training
    molecules and take the median of their experimental RT'
- name: Tanimoto similarity
  role: Defines neighbor retrieval and the 95/90/80/70/50% threshold groups over ECFP
    fingerprints
- name: Wilcoxon rank test
  role: Paired test comparing naive vs. DLM relative RT errors within each similarity-threshold
    group
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-13680-7
    title: The METLIN small molecule dataset for machine learning-based retention
      time prediction
  source_content_hash: sha256:dbf65f9087d9393488f22685a92fce6c2fc039d2624d4d2d75e08a5a787f2660
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Keras
  - Tanimoto similarity
  - Wilcoxon rank test
  - k-nearest neighbors regression
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# knn-retention-time-baseline-regression

## Summary

A naive k-NN baseline (k=3) that predicts LC-MS retention time as the median experimental RT of the three most Tanimoto-similar training molecules, used to benchmark deep-learning RT predictors (e.g., a Keras DLM on ECFP fingerprints) via threshold-stratified paired Wilcoxon tests.

## When to use

Use when benchmarking an ML retention-time predictor (e.g., a Keras regression model on ECFP fingerprints) against a structure-free nearest-neighbor baseline over METLIN SMRT-like RT data stratified by Tanimoto similarity thresholds.

## When NOT to use

- Validation molecules have no training-set counterpart above 50% Tanimoto similarity — the nearest-neighbor RT estimate is meaningless at near-zero structural similarity
- The task is RT prediction itself rather than benchmarking — use the trained DLM, since at most thresholds the k-NN median is only a baseline that the DLM significantly outperforms
- Retention times were measured under a different chromatographic method than the training set, as the paper's projection-across-CMs analysis shows models may not transfer directly

## Inputs

- METLIN SMRT dataset: molecules with experimentally acquired reverse-phase chromatography RT (seconds) and ECFP fingerprints
- 75/25 train/validation split of the molecules
- DLM (Keras) RT predictions for the validation set

## Outputs

- Per-molecule relative RT errors for the k=3 median-RT baseline and the DLM (knn_dlm_error_comparison.tsv)
- Per-threshold-group summaries (median, IQR of errors) at 95/90/80/70/50% Tanimoto cutoffs
- Paired Wilcoxon rank-test P values comparing baseline vs. DLM per threshold group

## How to apply

Split the METLIN SMRT molecules (RT in seconds plus ECFP fingerprints) 75% training / 25% validation. For each validation molecule, compute Tanimoto similarity to all training molecules, retrieve the 3 most similar training molecules, and take the median of their experimental RT as the naive prediction. Assign validation molecules to similarity-threshold groups (95, 90, 80, 70, 50%) where at least one training counterpart exceeds the threshold, then compute relative RT errors for both the k-NN baseline and the DLM predictions. Within each group, run a paired Wilcoxon rank test comparing naive vs. DLM relative errors; in the source study the DLM was significantly better (P < 0.0001) at 90/80/70/50% but NOT at 95%, where very similar neighbors make the naive median competitive.

## Related tools

- **Keras** (Trains the deep-learning regression model (4 hidden layers of 1000/500/200/100 relu units, L2 0.0001, adam lr 0.01, MSE loss, 20 epochs, batch size 35) whose RT predictions are benchmarked against the k-NN baseline)
- **k-nearest neighbors regression** (Implements the naive baseline: retrieve the 3 most Tanimoto-similar training molecules and take the median of their experimental RT)
- **Tanimoto similarity** (Defines neighbor retrieval and the 95/90/80/70/50% threshold groups over ECFP fingerprints)
- **Wilcoxon rank test** (Paired test comparing naive vs. DLM relative RT errors within each similarity-threshold group)

## Examples

```
from scipy.spatial.distance import pdist; from scipy.stats import wilcoxon; D = 1 - pdist(X_val, X_train, metric='jaccard'); stat, p = wilcoxon(err_knn, err_dlm)
```

## Evaluation signals

- Significance pattern reproduced: paired Wilcoxon P < 0.0001 in favor of the DLM at 90/80/70/50% Tanimoto thresholds
- No statistically significant mean-error difference at the 95% Tanimoto threshold
- Group membership check: each threshold group contains only validation molecules with at least one training counterpart above the cutoff
- Baseline prediction equals the median experimental RT of exactly the 3 most similar training molecules for every grouped validation molecule
- Output file knn_dlm_error_comparison.tsv contains per-molecule errors, group median/IQR summaries, and Wilcoxon P values

## Limitations

- At 95% Tanimoto similarity the naive k-NN baseline is statistically indistinguishable from the DLM, so the baseline is only weak in that regime
- The baseline requires structurally similar training molecules; performance degrades when neighbors are dissimilar
- The comparison is tied to the specific 75/25 split and the Keras DLM architecture; other splits/models may shift the threshold at which significance is lost

## Evidence

- [abstract] SMRT dataset scale: "an experimentally acquired reverse-phase chromatography retention time dataset covering up to 80,038 small molecules"
- [intro] Motivation for ML RT baselines: "the lack of experimental RT datasets has hampered the design of ML methods for large-scale RT prediction"
- [other] Threshold-stratified significance pattern: [evidence span withheld — anchor status: not_found]
- [other] Baseline procedure: [evidence span withheld — anchor status: not_found]
