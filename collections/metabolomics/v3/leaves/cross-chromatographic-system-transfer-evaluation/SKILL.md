---
name: cross-chromatographic-system-transfer-evaluation
description: Use when you have PredRet-style retention time data from several chromatographic
  systems and need to quantify whether multi-system training (all_on_one / all_on_one_perc
  scenarios) outperforms single-system training on a target system.
when_to_use_negative:
- When only a single chromatographic system is available with no multi-system training
  data — use the baseline_single scenario instead
- When the goal is joint metabolite identification from MS/MS spectra (met_ident_perf_GS_BS
  scenario with IOKR scores and dynamic programming), not transfer-accuracy benchmarking
- When retention times must be predicted on a common absolute scale across systems
  rather than compared as within-system orderings
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: retention_order_prediction
  role: Main Python evaluation framework implementing the all_on_one and all_on_one_perc
    transfer scenarios in evaluation_scenarios_main.py
  repo: https://version.aalto.fi/gitlab/bache1/retention_order_prediction.git
- name: rcdkTools
  role: R wrapper for computing MACCS counting fingerprints via the CDK
  repo: https://github.com/bachi55/rcdkTools
- name: rcdk
  role: R package used for fingerprint calculation in the data pre-processing step
  repo: https://github.com/rajarshi/cdkr
- name: scikit-learn
  role: Provides the SVR estimator and hyper-parameter search infrastructure used
    by the scenarios
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/bty590
    title: Liquid-chromatography retention order prediction for metabolite identification
  source_content_hash: sha256:f4cbb7bbf1b4de1583ee470089656992d328c3d7951dd1ccf0d19f6212cb0e47
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
  - rcdk
  - rcdkTools
  - retention_order_prediction
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 18
---

# cross-chromatographic-system-transfer-evaluation

## Summary

Evaluates whether retention order predictors (RankSVM or SVR) trained on retention time measurements from multiple heterogeneous LC systems generalize to a single held-out or partially-observed target system, the key transfer experiment of the Bach et al. 2018 retention order paper.

## When to use

Use when you have PredRet-style retention time data from several chromatographic systems and need to quantify whether multi-system training (all_on_one / all_on_one_perc scenarios) outperforms single-system training on a target system.

## When NOT to use

- When only a single chromatographic system is available with no multi-system training data — use the baseline_single scenario instead
- When the goal is joint metabolite identification from MS/MS spectra (met_ident_perf_GS_BS scenario with IOKR scores and dynamic programming), not transfer-accuracy benchmarking
- When retention times must be predicted on a common absolute scale across systems rather than compared as within-system orderings

## Inputs

- PredRet retention time measurements configured via results/raw/PredRet/v2/config.json
- System set '10': Eawag_XBridgeC18, FEM_long, RIKEN, UFZ_Phenomenex, LIFE_old
- MACCS counting fingerprints (maccsCount_f2dcf0b3) with MinMax kernel, or binary maccs with tanimoto kernel
- ESTIMATOR choice: ranksvm or svr

## Outputs

- Pairwise retention-order prediction accuracy per target system and training regime, stored under results/PredRet/v2/final
- Accuracy-versus-percentage curves (Figure 4): solid lines for all-systems training, stroked lines for single-system training
- Table 4 columns comparing LTSO=True vs LTSO=False transfer settings

## How to apply

Run the evaluation scenarios in src/evaluation_scenarios_main.py from the aalto-ics-kepaco/retention_order_prediction repository, passing an ESTIMATOR ('ranksvm' or 'svr'), the SCENARIO ('all_on_one' for a fixed split with LTSO=True and LTSO=False, or 'all_on_one_perc' to sweep the percentage of target-system data included in training), the SYSSET (e.g. '10' covering Eawag_XBridgeC18, FEM_long, RIKEN, UFZ_Phenomenex, LIFE_old), a TSYSIDX, the config file results/raw/PredRet/v2/config.json, and NJOBS for parallel hyper-parameter search via joblib. The RankSVM predicts retention order directly, while the SVR regresses absolute retention times and maps predictions to order by comparing predicted times; molecules are represented with MACCS counting fingerprints under a MinMax kernel (switch the config to maccs/tanimoto for the binary-fingerprint baseline). Results are written under results/PredRet/v2/final and judged by pairwise prediction accuracy, with solid lines in Figure 4 showing all-systems training and stroked lines single-system training.

## Related tools

- **retention_order_prediction** (Main Python evaluation framework implementing the all_on_one and all_on_one_perc transfer scenarios in evaluation_scenarios_main.py) — https://version.aalto.fi/gitlab/bache1/retention_order_prediction.git
- **rcdkTools** (R wrapper for computing MACCS counting fingerprints via the CDK) — https://github.com/bachi55/rcdkTools
- **rcdk** (R package used for fingerprint calculation in the data pre-processing step) — https://github.com/rajarshi/cdkr
- **scikit-learn** (Provides the SVR estimator and hyper-parameter search infrastructure used by the scenarios)

## Examples

```
python src/evaluation_scenarios_main.py ranksvm all_on_one_perc 10 0 results/raw/PredRet/v2/config.json 2 False
```

## Evaluation signals

- All-systems (all_on_one) RankSVM pairwise accuracy should exceed the single-system baseline_single accuracy, especially at low percentages of target-system training data
- Counting fingerprints (maccsCount with minmax kernel) should outperform binary fingerprints (maccs with tanimoto kernel) under identical scenario settings
- The LTSO=False variant (target system excluded from training) isolates pure transfer; LTSO=True (target in training) should score higher, matching Table 4 column ordering
- Accuracy-versus-percentage curves should be monotone increasing in the percentage of target-system data, qualitatively reproducing Figure 4
- Results directory structure must appear under results/PredRet/v2/final/<estimator>/.../difference/<fingerprint>/<kernel>/<scenario>

## Limitations

- SVR-based retention time prediction is limited in its capacity to use data from other chromatographic systems, unlike RankSVM
- The target system may or may not be included in training (LTSO True vs False), which changes the interpretation of transfer performance
- Results are compared qualitatively to Figure 4 trends rather than to exact numeric values; the paper does not report quantitative accuracy values in some sections

## Evidence

- [intro] We demonstrate that the retention order framework is able to benefit from retention time measurements arising from heterogeneous LC systems and configurations: [evidence span withheld — anchor status: not_found]
- [results] Increasing the percentage of target-system retention time data used for training improves pairwise prediction accuracy of the retention order models: [evidence span withheld — anchor status: not_found]
