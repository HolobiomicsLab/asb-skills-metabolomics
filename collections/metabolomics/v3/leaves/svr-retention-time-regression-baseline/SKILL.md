---
name: svr-retention-time-regression-baseline
description: Use when reproducing or benchmarking retention-time-based prediction
  on PredRet chromatographic systems (e.g. baseline_single_perc or all_on_one_perc
  scenarios), or when quantifying how SVR accuracy scales with the percentage of target-system
  training data.
when_to_use_negative:
- When the goal is multi-system transfer learning of retention order — the SVR on
  absolute retention times was shown to be limited in this capacity, so prefer the
  RankSVM order predictor.
- When the input is already MS/MS spectral candidate scores (e.g. IOKR output) — this
  skill addresses retention-time/Order prediction from molecular structure only, not
  metabolite identification scoring.
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: scikit-learn (sklearn)
  role: Provides the SVR estimator regressing absolute retention times from molecular
    fingerprints
- name: retention_order_prediction
  role: Hosts evaluation_scenarios_main.py, which runs the SVR scenarios (baseline_single_perc,
    all_on_one_perc) and stores results under results/PredRet/v2/final
  repo: https://version.aalto.fi/gitlab/bache1/retention_order_prediction.git
- name: rcdkTools
  role: Computes the MACCS counting fingerprints (via CDK) used as SVR input features
  repo: https://github.com/bachi55/rcdkTools
- name: joblib
  role: Parallelizes the SVR hyper-parameter search over NJOBS
- name: pandas
  role: Aggregates scenario results for plotting accuracy-versus-percentage curves
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/bty590
    title: Liquid-chromatography retention order prediction for metabolite identification
  source_content_hash: sha256:33d8cdf0b1aaee658f7a8c8c67a76cb9a8f29289eebfe6f44c89d0c768ab8344
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
  - rcdkTools
  - retention_order_prediction
  - scikit-learn
  - scikit-learn (sklearn)
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 18
---

# svr-retention-time-regression-baseline

## Summary

Baseline that regresses absolute LC retention times with a support vector regression (SVR) on MACCS counting fingerprints (MinMax kernel) and maps predicted times back to retention orders by pairwise comparison. Used in the PredRet evaluation scenarios to benchmark against the RankSVM order predictor, especially in the Figure 4 accuracy-vs-percentage sweeps.

## When to use

Use when reproducing or benchmarking retention-time-based prediction on PredRet chromatographic systems (e.g. baseline_single_perc or all_on_one_perc scenarios), or when quantifying how SVR accuracy scales with the percentage of target-system training data.

## When NOT to use

- When the goal is multi-system transfer learning of retention order — the SVR on absolute retention times was shown to be limited in this capacity, so prefer the RankSVM order predictor.
- When the input is already MS/MS spectral candidate scores (e.g. IOKR output) — this skill addresses retention-time/Order prediction from molecular structure only, not metabolite identification scoring.

## Inputs

- PredRet retention time measurements (configured via results/raw/PredRet/v2/config.json)
- MACCS counting fingerprints (maccsCount_f2dcf0b3) with MinMax kernel as molecule representation
- Evaluation parameters: ESTIMATOR ('svr'), SCENARIO ('baseline_single_perc', 'all_on_one_perc'), SYSSET ('10', 'imp', '10_imp'), TSYSIDX, NJOBS, DEBUG

## Outputs

- Scenario result files stored under results/PredRet/v2/final for the svr estimator
- Pairwise retention-order prediction accuracy per estimator, target system, and training percentage
- Accuracy-versus-percentage curves (Figure 4 style, solid vs stroked lines) via matplotlib/pandas aggregation

## How to apply

Load the PredRet retention time measurements with the experiment configuration file (results/raw/PredRet/v2/config.json) in Python using pandas and numpy, representing molecules with MACCS counting fingerprints under the MinMax kernel. Run the estimator through src/evaluation_scenarios_main.py with ESTIMATOR='svr', sweeping scenarios such as baseline_single_perc (single system as training and target) and all_on_one_perc (all systems for training, single target, varying percentage of target-system data). Tune hyper-parameters with the built-in parallel search (NJOBS via joblib), and keep DEBUG=False so results are written to results/PredRet/v2/final rather than the smoke-test directory. To obtain order accuracy, compare predicted retention times pairwise and evaluate the fraction of correctly predicted retention orders, then aggregate with pandas and plot accuracy-versus-percentage curves (stroked lines for single-system, solid lines for all-systems training). Note the article's key caveat: unlike the RankSVM, the SVR on absolute retention times is limited in its ability to exploit data from other chromatographic systems.

## Related tools

- **scikit-learn (sklearn)** (Provides the SVR estimator regressing absolute retention times from molecular fingerprints)
- **retention_order_prediction** (Hosts evaluation_scenarios_main.py, which runs the SVR scenarios (baseline_single_perc, all_on_one_perc) and stores results under results/PredRet/v2/final) — https://version.aalto.fi/gitlab/bache1/retention_order_prediction.git
- **rcdkTools** (Computes the MACCS counting fingerprints (via CDK) used as SVR input features) — https://github.com/bachi55/rcdkTools
- **joblib** (Parallelizes the SVR hyper-parameter search over NJOBS)
- **pandas** (Aggregates scenario results for plotting accuracy-versus-percentage curves)

## Examples

```
python src/evaluation_scenarios_main.py svr baseline_single_perc 10 -1 results/raw/PredRet/v2/config.json 2 False
```

## Evaluation signals

- Result files appear under results/PredRet/v2/final (not the debug directory) when DEBUG=False
- Pairwise order accuracy derived from SVR-predicted times must be computed by comparing predicted times, not by regressing orders directly
- Accuracy should increase monotonically (qualitatively) as the percentage of target-system training data grows, matching Figure 4 trends
- In multi-system settings (all_on_one_perc, solid lines), SVR accuracy should be notably below the analogous RankSVM accuracy, consistent with the paper's finding that SVR is limited at using other systems' data
- Curves generated for MACCS counting fingerprints with MinMax kernel should outperform the binary MACCS / Tanimoto kernel variant

## Limitations

- The SVR regresses absolute retention times, which are not directly comparable across instruments; its ability to exploit heterogeneous LC systems' training data is limited compared to the RankSVM retention-order approach
- Retention time measurements must be pre-processed: keep the lowest retention time per molecule and remove molecules whose lowest and largest retention times differ by more than 5%
- Results are validated only by qualitative agreement with Figure 4 trends, not against released numeric tables

## Evidence
