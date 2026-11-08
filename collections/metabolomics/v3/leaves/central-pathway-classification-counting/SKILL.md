---
name: central-pathway-classification-counting
description: Use when comparing hyperlink prediction models (e.g., NetAurHPD vs CHESHIRE)
  on metabolic pathway classification and you must report survival-critical central-pathway
  recovery, not just AUC.
when_to_use_negative:
- When the evaluation goal is plain ranking performance (AUC/accuracy) on a balanced
  test set with no designated survival-critical pathway subset — the central-pathway
  count adds nothing without the 13 TomatoCyc critical pathways.
- When pathway labels are abundant and no tough realistic condition (holdout of pathways
  containing low-involvement metabolites) has been constructed; the skill is designed
  for the limited-known-pathways regime.
- When inputs are not metabolite correlation networks annotated against TomatoCyc/MetaCyc
  pathways — the central-pathway set is defined entirely in TomatoCyc terms.
tools:
- name: CHESHIRE
  role: State-of-the-art hyperlink prediction baseline trained on the reduced pathway
    training sets and scored with threshold 0.5 to obtain the AUC and central-pathway
    counts being compared
- name: NetAurHPD
  role: Proposed method whose published Figure 5 results (AUC curves and central-pathway
    counts) serve as the reference the reproduced CHESHIRE numbers are tabulated against
  repo: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- name: PyTorch
  role: Framework in which the CHESHIRE baseline is trained (10 sessions per year,
    75 epochs) before evaluation on each year's test set
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:b47e9a7f79702c2ab3adb7e32366996bd70e786e81f2b597a024298d6e58df16
  source_papers:
  - doi: 10.48550/arxiv.2410.22030
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
  tools:
  - CHESHIRE
  - NetAurHPD
  - PyTorch
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# central-pathway-classification-counting

## Summary

Evaluation protocol that, alongside AUC, counts how many of the 13 TomatoCyc central pathways (identified as critical for organism survival) a hyperlink prediction classifier correctly classifies on held-out tomato pericarp test sets, exposing failures that average AUC hides.

## When to use

Use when comparing hyperlink prediction models (e.g., NetAurHPD vs CHESHIRE) on metabolic pathway classification and you must report survival-critical central-pathway recovery, not just AUC.

## When NOT to use

- When the evaluation goal is plain ranking performance (AUC/accuracy) on a balanced test set with no designated survival-critical pathway subset — the central-pathway count adds nothing without the 13 TomatoCyc critical pathways.
- When pathway labels are abundant and no tough realistic condition (holdout of pathways containing low-involvement metabolites) has been constructed; the skill is designed for the limited-known-pathways regime.
- When inputs are not metabolite correlation networks annotated against TomatoCyc/MetaCyc pathways — the central-pathway set is defined entirely in TomatoCyc terms.

## Inputs

- Per-year test set of held-out pathway labels (40 TomatoCyc positive, 27 MetaCyc negative, 13 random negative pathways for 2001/2003/2004) from the tough realistic condition construction
- List of the 13 TomatoCyc central pathways identified as critical for organism survival
- Trained model predictions or classification scores (CHESHIRE threshold 0.5; NetAurHPD with Youden-index threshold) for each test pathway per year and per training fraction (10%–100%)
- Reported NetAurHPD and NetAurHPD-with-one-Augmentation results from Figure 5 for comparison

## Outputs

- Per-year table of CHESHIRE micro-average AUC and count of correctly classified central pathways (0–13) across the 10 training fractions
- Side-by-side comparison against NetAurHPD and NetAurHPD-with-one-Augmentation curves matching the left (AUC) and right (central-pathway count) panels of Figure 5

## How to apply

First designate the central-pathway subset: the 13 TomatoCyc pathways identified as critical for organism survival within each year's held-out test set (40 TomatoCyc, 27 MetaCyc, 13 random pathways per year for 2001/2003/2004). Train and evaluate each model — CHESHIRE at a fixed classification threshold of 0.5 (10 sessions per year, training fraction grown from 10% to 100%, 75 epochs, source-recommended parameters) and NetAurHPD/NetAurHPD-with-one-Augmentation with its Youden-index-optimized threshold. For every year and training fraction, record micro-average AUC AND the count of correctly classified central pathways, tabulating the counts side by side so that a model with similar AUC but poor central-pathway recovery in a given year is exposed. Rationale: the paper's claim that NetAurHPD outperforms state-of-the-art under limited known pathways rests on both panels of Figure 5 — general AUC across all three years and central-pathway correctness, where CHESHIRE collapses in 2001 and 2003 while remaining competitive in 2004. Validate the reproduction by confirming both AUC curves and central-pathway counts match the left and right panels of Figure 5.

## Related tools

- **CHESHIRE** (State-of-the-art hyperlink prediction baseline trained on the reduced pathway training sets and scored with threshold 0.5 to obtain the AUC and central-pathway counts being compared)
- **NetAurHPD** (Proposed method whose published Figure 5 results (AUC curves and central-pathway counts) serve as the reference the reproduced CHESHIRE numbers are tabulated against) — https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- **PyTorch** (Framework in which the CHESHIRE baseline is trained (10 sessions per year, 75 epochs) before evaluation on each year's test set)

## Examples

```
optimize_results_and_calc_metrics(y_test, y_pred)  # compute AUC, accuracy, recall, precision, F1, fpr after threshold optimization
```

## Evaluation signals

- Central-pathway counts must fall in the range 0–13, since exactly 13 TomatoCyc central pathways are in each year's test set.
- The reproduced CHESHIRE AUC curves and central-pathway counts should match the left and right panels of Figure 5 for 2001, 2003, and 2004.
- Directional check: NetAurHPD should outperform CHESHIRE in general AUC across all three years and correctly predict more central pathways in 2001 and 2003.
- Sanity check that CHESHIRE yields poor central-pathway predictions in 2001 and 2003 but competitive results in 2004; discrepancies in 2004 reproduction warrant closer scrutiny rather than blanket failure.
- Metrics should be computed consistently with the repository's result-optimization module (Youden's J threshold optimization and AUC/accuracy/recall/precision/F1 for NetAurHPD, fixed 0.5 threshold for CHESHIRE).

## Limitations

- CHESHIRE struggles to reach high AUC and yields poor central-pathway predictions in 2001 and 2003; only the 2004 year is competitive, so conclusions are year-dependent.
- Models do not consistently improve with more data; in some sessions the AUCs of all models decreased, so monotonic improvement in training fraction should not be assumed.
- NetAurHPD has a low level of explainability and cannot provide biologists with a full understanding of its results, so central-pathway counts alone do not explain why pathways are recovered or missed.
- The count metric is defined only against the specific 13-pathway survival-critical TomatoCyc set of this study and does not transfer to organisms or pathway databases without an equivalent critical subset.

## Evidence

- [methods] In 2001 the micro-average AUC was 0.87 ... For 2003 the micro-average AUC was 0.857 ... In 2004 the micro-average AUC was 0.884: [evidence span withheld — anchor status: not_found]
- [other] recording AUC and the count of correctly classified central pathways (13 TomatoCyc pathways identified as critical for organism survival): [evidence span withheld — anchor status: not_found]
- [methods] We compare the performance of state-of-the-art CHESHIRE model and NetAurHPD in two versions: "We compare the performance of state-of-the-art CHESHIRE model and NetAurHPD in two versions"
- [readme] Optimizes the threshold using Youden's J Statistic and calculates several evaluation metrics: AUC, accuracy, recall, precision, F1 score And fpr.: "Optimizes the threshold using Youden's J Statistic and calculates several evaluation metrics: AUC, accuracy, recall, precision, F1 score And fpr."
