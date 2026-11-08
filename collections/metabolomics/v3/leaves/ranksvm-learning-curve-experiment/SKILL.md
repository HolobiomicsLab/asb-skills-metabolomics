---
name: ranksvm-learning-curve-experiment
description: Use when quantifying how much labeled retention time data from the target
  chromatographic system is needed, or when comparing single-system training against
  training on all heterogeneous LC systems at varying target-data fractions.
when_to_use_negative:
- When the target question is absolute retention time accuracy on a single system
  without varying training-set size — use the baseline_single scenario (Tables 3/4)
  instead
- When the goal is to evaluate multi-system benefit at full training size with the
  target system left single-out — the all_on_one scenario (LTSO=False columns of Table
  4) is the appropriate fixed-size variant
- When the analysis concerns metabolite identification performance combined with MS/MS
  scores (met_ident_perf_GS_BS scenario / IOKR integration), not learning curves
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3172
tools:
- name: retention_order_prediction (evaluation_scenarios_main.py)
  role: Hosts the baseline_single_perc and all_on_one_perc scenarios that sweep target-system
    training percentages for the RankSVM/SVR estimators
  repo: https://version.aalto.fi/gitlab/bache1/retention_order_prediction.git
- name: scikit-learn
  role: Provides the RankSVM and SVR estimators and hyper-parameter search used in
    the percentage sweep
- name: joblib
  role: Parallelizes the hyper-parameter search across NJOBS during the learning-curve
    runs
- name: rcdkTools / rcdk
  role: Computes MACCS counting fingerprints used as molecule representation for the
    RankSVM/SVR models
  repo: https://github.com/bachi55/rcdkTools
- name: pandas
  role: Aggregates accuracy-versus-percentage results before plotting
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/bty590
    title: Liquid-chromatography retention order prediction for metabolite identification
  source_content_hash: sha256:b0ccb9cc36c9faaa95288209eb5579fd565a13964a81a01507f60df3acf68909
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IOKR
  - Python and
  - R
  - joblib
  - numpy
  - pandas
  - rcdkTools / rcdk
  - retention_order_prediction (evaluation_scenarios_main.py)
  - scikit-learn
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5882352941176471
n_claims: 17
---

# RankSVM learning-curve experiment (accuracy vs. percentage of target-system training data)

## Summary

Runs retention-order learning-curve experiments in which the percentage of target-system retention time measurements used for training is varied, comparing RankSVM (and SVR) pairwise accuracy under single-system (baseline_single_perc) versus all-systems (all_on_one_perc) training to reproduce Figure 4 trends.

## When to use

Use when quantifying how much labeled retention time data from the target chromatographic system is needed, or when comparing single-system training against training on all heterogeneous LC systems at varying target-data fractions.

## When NOT to use

- When the target question is absolute retention time accuracy on a single system without varying training-set size — use the baseline_single scenario (Tables 3/4) instead
- When the goal is to evaluate multi-system benefit at full training size with the target system left single-out — the all_on_one scenario (LTSO=False columns of Table 4) is the appropriate fixed-size variant
- When the analysis concerns metabolite identification performance combined with MS/MS scores (met_ident_perf_GS_BS scenario / IOKR integration), not learning curves

## Inputs

- PredRet retention time measurements configured by results/raw/PredRet/v2/config.json (systems: Eawag_XBridgeC18, FEM_long, RIKEN, UFZ_Phenomenex, LIFE_old via SYSSET=10)
- MACCS counting fingerprint representation (maccsCount_f2dcf0b3) with minmax kernel, per config.json molecule_representation block
- Estimator choice: ranksvm or svr
- Target system index TSYSIDX (-1 runs all target systems in a single job)
- NJOBS for parallel hyper-parameter search (e.g. 2)

## Outputs

- Pairwise prediction accuracy values for each estimator, target-system percentage, and training regime, stored under results/PredRet/v2/final/.../baseline_single_perc and .../all_on_one_perc
- Accuracy-versus-percentage learning-curve plots (Figure 4 style: solid lines = all-systems training, stroked lines = single-system training)

## How to apply

Load the PredRet retention time measurements via the configuration file results/raw/PredRet/v2/config.json in Python (pandas/numpy), representing molecules with MACCS counting fingerprints under the MinMax kernel. Run the baseline_single_perc scenario in src/evaluation_scenarios_main.py with the ranksvm estimator: a single chromatographic system used as both training and target, sweeping different percentages of data used for training, with the hyper-parameter search parallelized over NJOBS via joblib. Then run all_on_one_perc: all systems used for training with a single target system, varying the percentage of target-system data included in training, for both LTSO=True (target system left in training) and LTSO=False (left single-out). Repeat both scenarios with the svr estimator, which regresses absolute retention times and maps predictions to order by comparing predicted times; note the paper found SVR is limited in its capacity to use data from other chromatographic systems unlike RankSVM. Evaluate pairwise prediction accuracy for each estimator, percentage, and regime, store results under results/PredRet/v2/final, aggregate with pandas, and plot accuracy-versus-percentage curves in matplotlib (solid lines for all-systems training, stroked lines for single-system training).

## Related tools

- **retention_order_prediction (evaluation_scenarios_main.py)** (Hosts the baseline_single_perc and all_on_one_perc scenarios that sweep target-system training percentages for the RankSVM/SVR estimators) — https://version.aalto.fi/gitlab/bache1/retention_order_prediction.git
- **scikit-learn** (Provides the RankSVM and SVR estimators and hyper-parameter search used in the percentage sweep)
- **joblib** (Parallelizes the hyper-parameter search across NJOBS during the learning-curve runs)
- **rcdkTools / rcdk** (Computes MACCS counting fingerprints used as molecule representation for the RankSVM/SVR models) — https://github.com/bachi55/rcdkTools
- **pandas** (Aggregates accuracy-versus-percentage results before plotting)

## Examples

```
python src/evaluation_scenarios_main.py ranksvm baseline_single_perc 10 -1 results/raw/PredRet/v2/config.json 2 False
```

## Evaluation signals

- All-systems (all_on_one_perc) RankSVM accuracy exceeds the single-system baseline (baseline_single_perc), especially at low target-system percentages
- MACCS counting fingerprints (minmax kernel) outperform binary MACCS fingerprints (tanimoto kernel) at equivalent percentages
- Accuracy increases monotonically (qualitatively) as the percentage of target-system training data increases for both estimators and regimes
- Result directories follow the expected nesting, e.g. results/PredRet/v2/final/ranksvm_slacktype=on_pairs/allow_overlap=True_d_lower=0_d_upper=16_ireverse=False_type=order_graph/difference/maccsCount_f2dcf0b3/minmax/<scenario>
- SVR-based curves show limited benefit from other-system data compared to RankSVM, consistent with the paper's discussion

## Limitations

- SVR-based retention time prediction is limited in its capacity to use data from other chromatographic systems, unlike RankSVM
- Results are validated by qualitative agreement with Figure 4 trends, not by exact numeric reproduction
- Reproducing the plots requires the optional matplotlib >= 2.1 package, and visualization/loading helpers live in results/scripts (R)

## Evidence
