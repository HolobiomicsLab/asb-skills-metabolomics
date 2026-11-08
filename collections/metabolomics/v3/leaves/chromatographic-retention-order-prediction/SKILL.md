---
name: chromatographic-retention-order-prediction
description: Use when the goal is to predict elution order of metabolites directly
  from molecular structure (InChI) across LC systems, especially when retention times
  are not comparable across instruments/configurations.
when_to_use_negative:
- When you need an absolute retention time value rather than an ordering — use the
  SVR baseline (src/evaluation_scenarios_main.py estimator 'svr') instead, since the
  paper notes SVR was limited in using data from other chromatographic systems.
- When the molecules of interest have very small retention times (not interacting
  with the chromatographic system) — the workflow removes such molecules for each
  system before training.
- When a single homogeneous dataset with directly comparable retention times is available
  and no cross-system transfer is needed — simpler regression may suffice.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_0091
tools:
- name: rcdk
  role: Calculates MACCS molecular fingerprints (counting and binary) from molecular
    structures inside R via the CDK library
  repo: https://github.com/rajarshi/cdkr
- name: rcdkTools
  role: Wrapper around rcdk that simplifies fingerprint calculation for larger sets
    of molecules and enables MACCS counting fingerprints via a modified rcdk supporting
    CDK substructure fingerprints
  repo: https://github.com/bachi55/rcdkTools
- name: retention_order_prediction
  role: Python implementation of the RankSVM order predictor, evaluation scenarios
    (e.g. baseline_single) and result storage
  repo: https://version.aalto.fi/gitlab/bache1/retention_order_prediction.git
- name: scikit-learn (sklearn)
  role: Python ML dependency used by the order predictor code (>= 0.19.0)
- name: joblib
  role: Parallelizes the hyper-parameter (C) grid search for the RankSVM
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/bty590
    title: Liquid-chromatography retention order prediction for metabolite identification
  source_content_hash: sha256:6878b4c23c5b20b8eb4eb3c6e92a0ad5fbf839f4b84e79facf085c4792a38581
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rajarshi/cdkr
  tools:
  - PyTorch
  - Python
  - joblib
  - numpy
  - pandas
  - rcdk
  - rcdkTools
  - retention_order_prediction
  - scikit-learn (sklearn)
  - scipy
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# chromatographic-retention-order-prediction

## Summary

Trains a ranking support vector machine (RankSVM) on pairwise retention-order preferences extracted from heterogeneous LC retention-time datasets (e.g. PredRet), using MACCS counting or binary molecular fingerprints as representations, to predict the order in which molecules elute from an LC column for any target chromatographic system.

## When to use

Use when the goal is to predict elution order of metabolites directly from molecular structure (InChI) across LC systems, especially when retention times are not comparable across instruments/configurations.

## When NOT to use

- When you need an absolute retention time value rather than an ordering — use the SVR baseline (src/evaluation_scenarios_main.py estimator 'svr') instead, since the paper notes SVR was limited in using data from other chromatographic systems.
- When the molecules of interest have very small retention times (not interacting with the chromatographic system) — the workflow removes such molecules for each system before training.
- When a single homogeneous dataset with directly comparable retention times is available and no cross-system transfer is needed — simpler regression may suffice.

## Inputs

- Retention-time measurements from multiple LC systems (e.g. PredRet datasets: Eawag_XBridgeC18, FEM_long, RIKEN, UFZ_Phenomenex, LIFE_old), restricted to measurements added before July 2015, with lowest RT kept and molecules whose lowest and largest RT differ by more than 5% removed
- Molecular structures as InChI strings for all training and target molecules
- Experiment configuration JSON (e.g. results/raw/PredRet/v2/config.json) specifying molecular representation, kernel, and predictor (e.g. 'maccsCount_f2dcf0b3' with 'minmax' kernel or 'maccs' with 'tanimoto' kernel)

## Outputs

- Trained RankSVM model with learned weight vector w
- Pairwise retention-order predictions for the target LC system stored in retention_order_predictions.csv
- Pairwise order prediction accuracy on the held-out target system, comparable to baseline_single results in Table 3/4 (first two columns)

## How to apply

From the retention-time data (e.g. PredRet configured via a JSON config such as results/raw/PredRet/v2/config.json), build the pairwise preference set P(s) = {(i,j) | s_i = s_j, t_i < t_j} within each chromatographic system, so that the earlier-eluting molecule is the preferred one. Compute MACCS fingerprints for all molecules from their InChI representations using CDK via rcdk/rcdkTools — counting fingerprints for the Min Max kernel, binary fingerprints for the Tanimoto kernel. Train RankSVM by solving the dual objective max_alpha -1/2 alpha^T Q alpha + 1^T alpha with 0 <= alpha_ij <= C, decomposing Q = A K_m A^T so the full pairwise kernel matrix never needs to be materialized, and select C by a parallel grid search (joblib). Apply the learned w to the target system, predicting pairwise order via f(m_i, m_j) = sign(w^T (phi(m_j) - phi(m_i))), and store predictions in retention_order_predictions.csv. To switch from counting to binary fingerprints, edit the molecule_representation block in the config JSON (kernel 'minmax' → 'tanimoto', predictor 'maccsCount_f2dcf0b3' → 'maccs').

## Related tools

- **rcdk** (Calculates MACCS molecular fingerprints (counting and binary) from molecular structures inside R via the CDK library) — https://github.com/rajarshi/cdkr
- **rcdkTools** (Wrapper around rcdk that simplifies fingerprint calculation for larger sets of molecules and enables MACCS counting fingerprints via a modified rcdk supporting CDK substructure fingerprints) — https://github.com/bachi55/rcdkTools
- **retention_order_prediction** (Python implementation of the RankSVM order predictor, evaluation scenarios (e.g. baseline_single) and result storage) — https://version.aalto.fi/gitlab/bache1/retention_order_prediction.git
- **scikit-learn (sklearn)** (Python ML dependency used by the order predictor code (>= 0.19.0))
- **joblib** (Parallelizes the hyper-parameter (C) grid search for the RankSVM)

## Examples

```
python src/evaluation_scenarios_main.py ranksvm baseline_single 10 -1 results/raw/PredRet/v2/config.json 2 False
```

## Evaluation signals

- Pairwise order prediction accuracy on the held-out target LC system reproduces the baseline_single numbers in Table 3 / Table 4 (first two columns) of the paper.
- Results directory tree matches the expected layout: results/PredRet/v2/final/ranksvm_slacktype=on_pairs/allow_overlap=True_d_lower=0_d_upper=16_ireverse=False_type=order_graph/difference/<predictor>/<kernel>/baseline_single.
- The config JSON molecule_representation block matches the intended fingerprint/kernel pair (e.g. 'minmax' kernel with 'maccsCount_f2dcf0b3' predictor, or 'tanimoto' kernel with 'maccs' predictor).
- Counting-fingerprint (MACCS counting) models should outperform the corresponding binary (Tanimoto) models, consistent with the paper's finding.
- Results land in the 'final' directory (not 'debug') when DEBUG is set to False.

## Limitations

- RankSVM (order-based) was shown to leverage retention-time measurements from heterogeneous LC systems and configurations, whereas SVR was limited in this capacity — so SVR should not be substituted when cross-system transfer is the goal.
- Only measurements added before July 2015 were considered in the paper's evaluation; molecules whose lowest and largest retention times differ by more than 5% are removed, and per-system removal of very small retention times is applied.
- MACCS counting fingerprints require the modified rcdk branch ('support_cdk_substructure_fps') that supports CDK substructure fingerprints; the original rcdk does not support them natively.
- Repeating full data pre-processing additionally requires OpenBabel >= 2.3.2 (obabel command-line tool).
- Code was tested with Python 3.5/3.6 and R 3.4; other versions are untested.

## Evidence

- [readme] Molecular structures are represented using MACCS fingerprints, calculated from InChI via rcdk/rcdkTools: "The [rcdkTools package](https://github.com/bachi55/rcdkTools) allows the
computation of several counting fingerprints through the
[Chemical Development Kit (CDK)]"
- [intro] Retention order predictions made for any chromatographic system without mapping retention times: "these predictions can be made for any chromatographic system without the need of first mapping the retention time to"
- [intro] Counting fingerprints outperform binary fingerprints for retention order prediction: "We show that counting fingerprints have superior performance in retention order prediction over standard binary fingerprints"
- [abstract] Retention order is better conserved between instruments than retention time: "We show that retention order is much better conserved between instruments than retention time."
- [abstract] Method can be trained on retention-time measurements from different LC systems without tedious pre-processing: "our method can be trained using retention time measurements from different LC systems and configurations without tedious pre-processing"
- [results] Pre-processing rule: keep lowest RT, remove molecules whose lowest and largest RT differ by more than 5%: "we keep only the lowest retention time and remove the molecule completely when the lowest and the largest retention time differs by more than 5%"
- [discussion] SVR was limited in using data from other chromatographic systems, unlike RankSVM: "The support vector regression (SVR) method for predicting retention times, on the other hand, was lim- ited in this capacity"
- [results] Per-system removal of very small retention times to avoid non-interacting molecules: [evidence span withheld — anchor status: not_found]
