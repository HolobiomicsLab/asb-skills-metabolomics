---
name: retention-order-pairwise-accuracy-evaluation
description: Use when you must quantify how accurately a retention-order model ranks
  pairs of molecules for specific chromatographic systems (e.g. Eawag_XBridgeC18,
  FEM_long, RIKEN), and compare single-system versus pooled multi-system training
  regimes.
when_to_use_negative:
- When the goal is metabolite identification performance (MS/MS + retention-order
  integration with IOKR and dynamic programming, scenario met_ident_perf_GS_BS) rather
  than pairwise-order accuracy on PredRet systems
- When predicting absolute retention times is the end goal — SVR is the wrong tool
  for cross-system pooling because 'The SVR retention time baseline does not benefit
  from retention time data that comes from different chromatographic systems'
- When the input is already aggregated per-system results; this skill runs the scenario
  estimation itself, not re-analysis of finished accuracy tables
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: evaluation_scenarios_main.py
  role: Entry-point script that runs the ranksvm/svr estimators under baseline_single,
    all_on_one and percentage-swept scenarios and stores scenario results
  repo: https://version.aalto.fi/gitlab/bache1/retention_order_prediction.git
- name: scikit-learn
  role: Provides the SVR retention-time baseline whose predictions are mapped to pairwise
    orders, and hyper-parameter search infrastructure for RankSVM
- name: joblib
  role: Parallelizes the hyper-parameter search over NJOBS cpus
- name: pandas
  role: Aggregates pairwise accuracy per target system into the Table 4 / Figure 4
    result tables
- name: rcdkTools
  role: Computes the MACCS counting fingerprints used as molecule representation for
    the pairwise MinMax/Tanimoto kernels
  repo: https://github.com/bachi55/rcdkTools
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/bty590
    title: Liquid-chromatography retention order prediction for metabolite identification
  source_content_hash: sha256:8c5806289fce0a60489addb59077caeeb8ec38fd51554c3009b6fdb73c0ac575
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IOKR
  - Python
  - Python and
  - R
  - evaluation_scenarios_main.py
  - joblib
  - numpy
  - pandas
  - rcdkTools
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 21
---

# retention-order-pairwise-accuracy-evaluation

## Summary

Evaluates retention-order predictors (RankSVM, or SVR mapped to pairwise orders) by pairwise accuracy over molecule pairs across evaluation scenarios (baseline_single, all_on_one with LTSO True/False) on the PredRet dataset, enabling comparison against the paper's Table 3/4 and Figure 4.

## When to use

Use when you must quantify how accurately a retention-order model ranks pairs of molecules for specific chromatographic systems (e.g. Eawag_XBridgeC18, FEM_long, RIKEN), and compare single-system versus pooled multi-system training regimes.

## When NOT to use

- When the goal is metabolite identification performance (MS/MS + retention-order integration with IOKR and dynamic programming, scenario met_ident_perf_GS_BS) rather than pairwise-order accuracy on PredRet systems
- When predicting absolute retention times is the end goal — SVR is the wrong tool for cross-system pooling because 'The SVR retention time baseline does not benefit from retention time data that comes from different chromatographic systems'
- When the input is already aggregated per-system results; this skill runs the scenario estimation itself, not re-analysis of finished accuracy tables

## Inputs

- PredRet retention time measurements configured via results/raw/PredRet/v2/config.json (systems Eawag_XBridgeC18, FEM_long, RIKEN, UFZ_Phenomenex, LIFE_old)
- Molecule representations: MACCS counting fingerprints with MinMax kernel and binary MACCS fingerprints with Tanimoto kernel (molecule_representation block of config.json)
- CLI arguments: ESTIMATOR {ranksvm, svr}, SCENARIO {baseline_single, baseline_single_perc, all_on_one, all_on_one_perc}, SYSSET {10, imp, 10_imp}, TSYSIDX, config path, NJOBS, DEBUG

## Outputs

- Pairwise prediction accuracy per target system stored under results/PredRet/v2/final/<estimator>/<scenario> directories keyed by fingerprint and kernel
- Six-column Table 4 comparison (RankSVM/SVR x baseline_single, LTSO True, LTSO False), e.g. exported as table4_ranksvm_vs_svr.csv
- Accuracy-versus-percentage curves for Figure 4 (solid lines: all-systems training; stroked lines: single-system training)

## How to apply

Run src/evaluation_scenarios_main.py with the chosen ESTIMATOR ('ranksvm' or 'svr'), SCENARIO ('baseline_single' for a single system used as training and target; 'all_on_one' for all systems used for training with a single target, both LTSO=True and LTSO=False variants), SYSSET 10 (Eawag_XBridgeC18, FEM_long, RIKEN, UFZ_Phenomenex, LIFE_old), TSYSIDX=-1 to evaluate all target systems in one job, and the PredRet config (results/raw/PredRet/v2/config.json) selecting MACCS counting fingerprints with the MinMax kernel (and the binary MACCS/Tanimoto variant for comparison). RankSVM hyper-parameters are tuned by a parallel joblib search (NJOBS, e.g. 2) with slack on pairs and the pairwise MinMax/Tanimoto kernel; SVR regresses absolute retention times and its predictions are mapped to pairwise orders by comparing predicted times. Aggregate pairwise prediction accuracy per target system with pandas/R and assemble the six-column Table 4 layout (RankSVM/SVR x baseline_single, LTSO True, LTSO False); for Figure 4, sweep percentages of target-system data via baseline_single_perc and all_on_one_perc and plot accuracy-versus-percentage curves (solid lines = all-systems training, stroked = single-system).

## Related tools

- **evaluation_scenarios_main.py** (Entry-point script that runs the ranksvm/svr estimators under baseline_single, all_on_one and percentage-swept scenarios and stores scenario results) — https://version.aalto.fi/gitlab/bache1/retention_order_prediction.git
- **scikit-learn** (Provides the SVR retention-time baseline whose predictions are mapped to pairwise orders, and hyper-parameter search infrastructure for RankSVM)
- **joblib** (Parallelizes the hyper-parameter search over NJOBS cpus)
- **pandas** (Aggregates pairwise accuracy per target system into the Table 4 / Figure 4 result tables)
- **rcdkTools** (Computes the MACCS counting fingerprints used as molecule representation for the pairwise MinMax/Tanimoto kernels) — https://github.com/bachi55/rcdkTools

## Examples

```
python src/evaluation_scenarios_main.py ranksvm all_on_one 10 -1 results/raw/PredRet/v2/config.json 2 False
```

## Evaluation signals

- Reproduced pairwise accuracies agree with the values reported in Table 4 for RankSVM vs SVR (first two columns from baseline_single, columns 3-4 from LTSO=True, columns 5-6 from LTSO=False)
- RankSVM retains its advantage over SVR in the pooled multi-system training settings (all_on_one), and RankSVM gains accuracy from multi-system training while SVR does not
- Results directory structure matches the documented hierarchy, e.g. results/PredRet/v2/final/ranksvm_slacktype=on_pairs/allow_overlap=True_d_lower=0_d_upper=16_ireverse=False_type=order_graph/difference/maccsCount_f2dcf0b3/minmax/baseline_single
- For Figure 4, all-systems RankSVM accuracy exceeds the single-system baseline especially at low target-system percentages, and counting fingerprints outperform binary fingerprints
- DEBUG=False runs write to the final directory rather than a smoke-test location

## Limitations

- The SVR retention-time baseline is limited in using data from other chromatographic systems, unlike RankSVM, so multi-system SVR comparisons cannot show pooled-training gains
- Percentage scenarios (baseline_single_perc, all_on_one_perc) support only qualitative validation against Figure 4 trends rather than exact numeric reproduction
- Switching between counting and binary fingerprints requires manually editing the config.json molecule_representation block (kernel minmax/predictor maccsCount vs kernel tanimoto/predictor maccs)
- Code tested only with Python 3.5/3.6 with pinned package versions (scipy >= 0.19.1, sklearn >= 0.19.0, etc.)

## Evidence
