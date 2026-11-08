---
name: learning-curve-visualization
description: Use when quantifying how much target-system training data a RankSVM or
  SVR retention-order predictor needs, by sweeping target-system training percentages
  and plotting accuracy curves for single-system vs multi-system regimes.
when_to_use_negative:
- When the goal is a single accuracy number per system rather than accuracy as a function
  of training-set size — use the baseline_single or all_on_one scenarios (Tables 3
  and 4) instead.
- When the input representation is MACCS binary fingerprints with the Tanimoto kernel
  rather than counting fingerprints with MinMax — the Figure 4 comparison targets
  counting vs binary fingerprint differences, so mixing representations invalidates
  the curve comparison.
- When retention times have not been pre-processed (lowest RT kept, molecules with
  >5% RT spread removed); unprocessed PredRet measurements will distort the pairwise
  accuracy estimates.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: scikit-learn (sklearn)
  role: Provides the RankSVM and SVR estimators whose pairwise accuracy is swept over
    training percentages
- name: pandas
  role: Aggregates per-percentage scenario results into curves for plotting
- name: numpy
  role: Numerical handling of fingerprint/kernel data when loading the PredRet measurements
- name: joblib
  role: Parallelizes the hyper-parameter search over NJOBS for each percentage sweep
- name: matplotlib
  role: Plots the accuracy-versus-percentage learning curves (solid vs stroked lines)
- name: Python 3.5 and 3.6
  role: Runtime for evaluation_scenarios_main.py and the result aggregation/plotting
    scripts
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/bty590
    title: Liquid-chromatography retention order prediction for metabolite identification
  source_content_hash: sha256:0bff0dc3751cc6cd734ca19e17e868340a14036e13eb0f75fb29e66d116e48ef
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IOKR
  - Python 3.5 and 3.6
  - Python and
  - R
  - joblib
  - matplotlib
  - numpy
  - pandas
  - scikit-learn
  - scikit-learn (sklearn)
  - scipy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# learning-curve-visualization

## Summary

Plots pairwise retention-order prediction accuracy as a function of the percentage of target-system retention time data used for training, contrasting single-system (baseline_single_perc, stroked lines) with all-systems training (all_on_one_perc, solid lines) to reproduce Figure 4.

## When to use

Use when quantifying how much target-system training data a RankSVM or SVR retention-order predictor needs, by sweeping target-system training percentages and plotting accuracy curves for single-system vs multi-system regimes.

## When NOT to use

- When the goal is a single accuracy number per system rather than accuracy as a function of training-set size — use the baseline_single or all_on_one scenarios (Tables 3 and 4) instead.
- When the input representation is MACCS binary fingerprints with the Tanimoto kernel rather than counting fingerprints with MinMax — the Figure 4 comparison targets counting vs binary fingerprint differences, so mixing representations invalidates the curve comparison.
- When retention times have not been pre-processed (lowest RT kept, molecules with >5% RT spread removed); unprocessed PredRet measurements will distort the pairwise accuracy estimates.

## Inputs

- PredRet retention time measurements configured via results/raw/PredRet/v2/config.json
- MACCS counting fingerprints (maccsCount_f2dcf0b3) with MinMax kernel as molecule representation
- Estimator choice: RankSVM or SVR
- Scenario choice: baseline_single_perc (single training system) or all_on_one_perc (all training systems, single target)
- LTSO flag (True/False) for whether the target system appears in training
- NJOBS value controlling parallel hyper-parameter search via joblib

## Outputs

- Scenario results stored under results/PredRet/v2/final (e.g. ranksvm_slacktype=on_pairs/allow_overlap=True_d_lower=0_d_upper=16_ireverse=False_type=order_graph/difference/maccsCount_f2dcf0b3/minmax/<scenario>)
- Pairwise prediction accuracy values for each estimator, training percentage, and training regime
- Accuracy-versus-percentage learning curves (Figure 4) with solid lines (all-systems training) and stroked lines (single-system training)

## How to apply

Load the PredRet retention time measurements via the experiment configuration file (results/raw/PredRet/v2/config.json) in Python with pandas and numpy, using MACCS counting fingerprints with the MinMax kernel as the molecule representation. Run the baseline_single_perc scenario of src/evaluation_scenarios_main.py with the RankSVM estimator — a single chromatographic system used for both training and target, sweeping different percentages of training data — with the hyper-parameter search parallelized over NJOBS via joblib. Then run all_on_one_perc with RankSVM, where all systems are used for training and a single system is the target, varying the percentage of target-system data included in training for both LTSO=True (target system in training) and LTSO=False. Repeat both scenarios with the SVR estimator, which regresses absolute retention times and maps predictions to order by comparing predicted times. Evaluate pairwise prediction accuracy for each estimator, percentage, and training regime, store scenario results under results/PredRet/v2/final, then aggregate with pandas and plot accuracy-versus-percentage curves in matplotlib, using solid lines for all-systems training and stroked lines for single-system training.

## Related tools

- **scikit-learn (sklearn)** (Provides the RankSVM and SVR estimators whose pairwise accuracy is swept over training percentages)
- **pandas** (Aggregates per-percentage scenario results into curves for plotting)
- **numpy** (Numerical handling of fingerprint/kernel data when loading the PredRet measurements)
- **joblib** (Parallelizes the hyper-parameter search over NJOBS for each percentage sweep)
- **matplotlib** (Plots the accuracy-versus-percentage learning curves (solid vs stroked lines))
- **Python 3.5 and 3.6** (Runtime for evaluation_scenarios_main.py and the result aggregation/plotting scripts)

## Examples

```
python src/evaluation_scenarios_main.py ranksvm all_on_one_perc 10 -1 results/raw/PredRet/v2/config.json 2 False
```

## Evaluation signals

- All-systems RankSVM accuracy should exceed the single-system baseline, especially at low target-system training percentages (qualitative agreement with Figure 4).
- Counting fingerprints (maccsCount with MinMax kernel) should outperform binary fingerprints (maccs with Tanimoto kernel) across the percentage sweep.
- Results are written to the expected directory tree under results/PredRet/v2/final with the estimator/scenario path structure described in the README.
- Curves should be monotonically non-decreasing as the percentage of target-system training data increases for both RankSVM and SVR regimes.
- The solid-line (all_on_one_perc) and stroked-line (baseline_single_perc) families must be plotted per the README's Figure 4 line-style convention.

## Limitations

- SVR-based retention time prediction is limited in its capacity to use data from other chromatographic systems, unlike RankSVM, so its multi-system curves benefit less from heterogeneous training data.
- Validation is qualitative — curves are judged by agreement with the paper's Figure 4 trends, not by exact numeric reproduction.
- The PredRet measurements used are restricted to those added before July 2015, and molecules with very small retention times or >5% RT spread are removed during pre-processing.

## Evidence

- [results] Source card workflow: [evidence span withheld — anchor status: not_found]
