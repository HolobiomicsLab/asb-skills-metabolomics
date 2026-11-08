---
name: roc-threshold-optimization
description: Use when candidate identities from accurate mass search (e.g. 10 ppm
  HMDB search) must be filtered or ranked by predicted-vs-experimental RT error, and
  you need an objective error-threshold choice.
when_to_use_negative:
- When candidate filtering is by m/z accuracy alone rather than predicted-vs-experimental
  RT error (e.g. no experimental RT data exist for the chromatographic method)
- When a single fixed RT tolerance window is already established and no threshold
  optimization against TP/TN/FP/FN trade-offs is required
- When evaluating a classifier on continuous scores unrelated to RT error — the 2.5%
  relative-RT-error discretization is specific to RT-based candidate filtering
tools:
- name: R
  role: Environment for computing TP/TN/FP/FN per RT-error bin, building ROC curves,
    and compiling results tables
- name: HMDB
  role: Source of the molecule list (6,823 molecules with KEGG numbers) from which
    putative candidates are retrieved via 10 ppm accurate mass search before RT-based
    ranking
- name: PredRet database
  role: Provides the experimental RT data for ECMs 1-4 (FEM_long, LIFE_old, RIKEN,
    FEM_orbitrap_plasma) used as ground truth for TP/TN/FP/FN assignment
- name: Keras for R
  role: Trains the deep learning model producing the predicted RTs whose errors against
    experimental RTs are thresholded
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-13680-7
    title: The METLIN small molecule dataset for machine learning-based retention
      time prediction
  source_content_hash: sha256:7485b2a56a682e8403ae842b8fb045af8309cd910d53d7d67095a49c206d2a88
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HMDB
  - Keras for R
  - PredRet database
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# roc-threshold-optimization

## Summary

Discretizes the relative retention-time (RT) error between projected predicted RTs and experimental RTs into 2.5% bins, computes TP/TN/FP/FN per bin to build ROC curves, and selects the filtering threshold that maximizes TPR while minimizing FPR for candidate identity ranking.

## When to use

Use when candidate identities from accurate mass search (e.g. 10 ppm HMDB search) must be filtered or ranked by predicted-vs-experimental RT error, and you need an objective error-threshold choice.

## When NOT to use

- When candidate filtering is by m/z accuracy alone rather than predicted-vs-experimental RT error (e.g. no experimental RT data exist for the chromatographic method)
- When a single fixed RT tolerance window is already established and no threshold optimization against TP/TN/FP/FN trade-offs is required
- When evaluating a classifier on continuous scores unrelated to RT error — the 2.5% relative-RT-error discretization is specific to RT-based candidate filtering

## Inputs

- Ranked candidate list per compound: putative identities from 10 ppm accurate mass search of protonated/deprotonated ion m/z against the HMDB molecule list
- RT error values per candidate (projected DLM-predicted RT minus experimental RT from PredRet ECM 1-4 data)

## Outputs

- ROC curve per experimental chromatographic method (ECM 1-4) with TP/TN/FP/FN per 2.5% RT-error bin
- Optimal RT-error filtering threshold per ECM (maximizing TPR, minimizing FPR)
- ROC AUC values (expected range 0.65-0.69)
- Top-3 candidate ranking accuracy percentages compiled with projection errors and thresholds into a results table

## How to apply

First rank putative candidates (retrieved via 10 ppm accurate mass search against 6,823 HMDB molecules with KEGG numbers) by the absolute RT error between the projected deep-learning-predicted RT and the experimental RT of the chromatographic method (ECM). Discretize the relative RT error from 0 to 100% in 2.5% intervals; at each interval compute TP, TN, FP, FN to generate an ROC curve per experimental chromatographic method (ECM 1-4: FEM_long, LIFE_old, RIKEN, FEM_orbitrap_plasma). Select the optimal filtering threshold as the point maximizing TPR while minimizing FPR, and report the AUC (0.65-0.69 across ECMs). Complement ROC analysis with top-3 ranking accuracy, where the correct identity ranked within the top three candidates serves as the practical utility metric (~70% across ECMs).

## Related tools

- **R** (Environment for computing TP/TN/FP/FN per RT-error bin, building ROC curves, and compiling results tables)
- **HMDB** (Source of the molecule list (6,823 molecules with KEGG numbers) from which putative candidates are retrieved via 10 ppm accurate mass search before RT-based ranking)
- **PredRet database** (Provides the experimental RT data for ECMs 1-4 (FEM_long, LIFE_old, RIKEN, FEM_orbitrap_plasma) used as ground truth for TP/TN/FP/FN assignment)
- **Keras for R** (Trains the deep learning model producing the predicted RTs whose errors against experimental RTs are thresholded)

## Examples

```
Rscript roc_threshold.R --pred-rt dl_predicted_rt.csv --exp-rt predret_ECM1.csv --mass-tolerance 10ppm --bins 2.5 --fit poly(rt_exp ~ poly(rt_pred, 4))
```

## Evaluation signals

- ROC AUC values per ECM fall within the reported 0.65-0.69 range (Fig. 5d, i, n, s)
- Top-3 ranking accuracy is approximately 70% for each of ECMs 1-4 (Fig. 5e, j, o, t)
- TP/TN/FP/FN counts are computed at every 2.5% interval of relative RT error from 0 to 100%
- The selected threshold corresponds to the ROC point maximizing TPR while minimizing FPR, not an arbitrary cutoff
- Per-candidate RT errors are computed after projection of predicted RTs onto each ECM via quartic (poly) regression fitted on 50 identified molecules

## Limitations

- ROC AUC is modest (0.65-0.69), so RT-error filtering alone does not uniquely identify compounds — it must be combined with accurate mass search and top-3 ranking
- Optimal thresholds are ECM-specific: each projection function is fitted on only 50 randomly selected identified molecules per chromatographic method, so thresholds may not transfer to other methods
- Candidate ranking is limited by the HMDB list with KEGG numbers and the 10 ppm mass search window; the correct identity may be absent from candidates

## Evidence

- [abstract] the correct molecular identity was ranked among the top 3 candidates based on their predicted retention time: "in 70% of the cases, the correct molecular identity was ranked among the top 3 candidates based on their predicted retention time"
- [methods] discretizing the relative RT error from 0 to 100% in 2.5% intervals, computing TP/TN/FP/FN and determining the optimal filtering threshold that maximizes TPR while minimizing FPR: [evidence span withheld — anchor status: not_found]
- [intro] Further, we focused on the ability to rank and filter putative candidates in real metabolomics experiments based on accurate mass search and predicted RT: "Further, we focused on the ability to rank and filter putative candidates in real metabolomics experiments based on accurate mass search and predicted RT"
