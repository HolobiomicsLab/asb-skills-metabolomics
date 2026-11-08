---
name: retention-order-accuracy-comparison
description: Use when evaluating whether molecular fingerprint representation (counting
  vs binary, Min-Max vs Tanimoto kernel) affects pairwise retention order accuracy
  in a single-system RankSVM setting.
when_to_use_negative:
- The goal is the multi-system training setting (Tables 4 or Figures 4) — use the
  all_on_one or *_perc scenarios instead, not baseline_single.
- Retention time prediction rather than retention order prediction is needed — use
  the SVR estimator rather than RankSVM.
- The binary/counting MACCS fingerprints have not yet been computed in data/processed;
  fingerprint calculation with rcdk/rcdkTools must be run first.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3360
tools:
- name: retention_order_prediction (evaluation_scenarios_main.py)
  role: Runs the RankSVM baseline_single scenario for both fingerprint configurations
    and stores per-system accuracies
  repo: https://version.aalto.fi/gitlab/bache1/retention_order_prediction.git
- name: rcdkTools
  role: Computes the MACCS counting fingerprints (maccsCount_f2dcf0b3) via a modified
    rcdk supporting CDK substructure fingerprints
  repo: https://github.com/bachi55/rcdkTools
- name: rcdk
  role: CDK-based fingerprint calculation used during data pre-processing
  repo: https://github.com/rajarshi/cdkr
- name: R (ECCB2018.Rmd helper scripts)
  role: Loads and visualizes the stored results for both fingerprint configurations
  repo: https://version.aalto.fi/gitlab/bache1/retention_order_prediction.git
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/bty590
    title: Liquid-chromatography retention order prediction for metabolite identification
  source_content_hash: sha256:24d1772ba4c4fe42d6aab06520fa9f744967ae3c5ad13e5c36afb5afd68fb5f8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Python and
  - R
  - R (ECCB2018.Rmd helper scripts)
  - json >=
  - rcdk
  - rcdkTools
  - retention_order_prediction (evaluation_scenarios_main.py)
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# retention-order-accuracy-comparison

## Summary

Reproduces the Table 3 (first two columns) comparison showing that counting MACCS fingerprints with the Min-Max kernel outperform binary MACCS fingerprints with the Tanimoto kernel for RankSVM retention order prediction when a single PredRet system is used for both training and testing.

## When to use

Use when evaluating whether molecular fingerprint representation (counting vs binary, Min-Max vs Tanimoto kernel) affects pairwise retention order accuracy in a single-system RankSVM setting.

## When NOT to use

- The goal is the multi-system training setting (Tables 4 or Figures 4) — use the all_on_one or *_perc scenarios instead, not baseline_single.
- Retention time prediction rather than retention order prediction is needed — use the SVR estimator rather than RankSVM.
- The binary/counting MACCS fingerprints have not yet been computed in data/processed; fingerprint calculation with rcdk/rcdkTools must be run first.

## Inputs

- PredRet retention time datasets for Eawag_XBridgeC18, FEM_long, RIKEN, UFZ_Phenomenex, LIFE_old
- results/raw/PredRet/v2/config.json experiment configuration (molecule_representation: kernel, predictor, feature_scaler, poly_feature_exp)
- Pre-computed MACCS binary fingerprints and MACCS counting fingerprints (maccsCount_f2dcf0b3) in data/processed

## Outputs

- Pairwise retention order prediction accuracy per target system for counting MACCS (minmax kernel), stored under results/PredRet/v2/final/ranksvm_slacktype=on_pairs/allow_overlap=True_d_lower=0_d_upper=16_ireverse=False_type=order_graph/difference/maccsCount_f2dcf0b3/minmax/baseline_single
- Corresponding accuracies for binary MACCS (tanimoto kernel) under .../maccs/tanimoto/baseline_single
- A two-column comparison table reproducing Table 3 (first two columns)

## How to apply

Run the baseline_single scenario with the counting-fingerprint configuration: 'python src/evaluation_scenarios_main.py ranksvm baseline_single 10 -1 results/raw/PredRet/v2/config.json 2 False', where SYSSET=10 selects the PredRet systems Eawag_XBridgeC18, FEM_long, RIKEN, UFZ_Phenomenex and LIFE_old, TSYSIDX=-1 runs all target systems in one job, and 2 jobs parallelize the hyper-parameter search. Ensure MACCS counting fingerprints were computed beforehand with the rcdkTools package (a modified rcdk supporting CDK substructure fingerprints is required for MACCS counting keys). Then edit the config.json molecule_representation block from kernel 'minmax' with predictor ['maccsCount_f2dcf0b3'] to kernel 'tanimoto' with predictor ['maccs'] and rerun the identical command. Load both result sets from results/PredRet/v2/final/ranksvm_slacktype=on_pairs/... (directories maccsCount_f2dcf0b3/minmax/baseline_single and maccs/tanimoto/baseline_single) using the R helper scripts in results/scripts, compute pairwise retention order prediction accuracy per target system, and compare against the Table 3 values (e.g. 0.844 counting vs 0.796 binary for Eawag_XBridgeC18).

## Related tools

- **retention_order_prediction (evaluation_scenarios_main.py)** (Runs the RankSVM baseline_single scenario for both fingerprint configurations and stores per-system accuracies) — https://version.aalto.fi/gitlab/bache1/retention_order_prediction.git
- **rcdkTools** (Computes the MACCS counting fingerprints (maccsCount_f2dcf0b3) via a modified rcdk supporting CDK substructure fingerprints) — https://github.com/bachi55/rcdkTools
- **rcdk** (CDK-based fingerprint calculation used during data pre-processing) — https://github.com/rajarshi/cdkr
- **R (ECCB2018.Rmd helper scripts)** (Loads and visualizes the stored results for both fingerprint configurations) — https://version.aalto.fi/gitlab/bache1/retention_order_prediction.git

## Examples

```
python src/evaluation_scenarios_main.py ranksvm baseline_single 10 -1 results/raw/PredRet/v2/config.json 2 False
```

## Evaluation signals

- Both result directories (.../maccsCount_f2dcf0b3/minmax/baseline_single and .../maccs/tanimoto/baseline_single) exist and contain accuracy values for all five target systems.
- Counting-MACCS (minmax) accuracy equals or exceeds binary-MACCS (tanimoto) accuracy for each target system, matching the reported finding.
- Absolute accuracies are consistent with Table 3 values (e.g. 0.844 vs 0.796 for Eawag_XBridgeC18).
- The hyper-parameter search ran with the requested number of parallel jobs (2) and results were written to the final (non-debug) directory.
- Only measurements added before July 2015, keeping the lowest RT and removing molecules with >5% RT spread, were used in pre-processing.

## Limitations

- The comparison is valid only for the single-system training setting (baseline_single); counting fingerprints may behave differently when data from heterogeneous LC systems are pooled (all_on_one).
- MACCS counting fingerprints require the modified rcdk build from bachi55/cdkr (branch support_cdk_substructure_fps) plus the countable MACCS SMARTS patterns; the original rcdk package does not support CDK substructure fingerprints.
- The R script for reproducing results depends on specific package versions (R 3.4, sklearn >= 0.19.0, etc.) and OpenBabel >= 2.3.2 only if pre-processing is repeated.

## Evidence

- [readme] rcdkTools purpose: [evidence span withheld — anchor status: external_doc]
