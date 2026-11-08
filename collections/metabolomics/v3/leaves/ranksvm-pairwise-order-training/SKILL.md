---
name: ranksvm-pairwise-order-training
description: Use when you have retention time measurements from one or more LC systems
  and need to predict elution order from molecular structure, especially when retention
  times are on incomparable scales across systems so pairwise preferences within a
  system are the only consistent supervision.
when_to_use_negative:
- The task is predicting absolute retention times on a common scale — use the SVR
  baseline instead; the paper notes SVR was 'limited in this capacity' to exploit
  data from other chromatographic systems, which is exactly the setting where RankSVM
  on pairwise order should be preferred.
- The training and target molecules come from different chromatographic systems but
  you intend to train on cross-system time comparisons — preferences must be built
  only within a system (s_i = s_j); cross-system pairs are not valid supervision.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: Python (scikit-learn, numpy, scipy, pandas, joblib)
  role: Implements the RankSVM dual solver, pairwise preference extraction, kernel
    construction, and parallel hyper-parameter grid search
  repo: https://version.aalto.fi/gitlab/bache1/retention_order_prediction.git
- name: rcdk
  role: Calculates MACCS molecular fingerprints from InChI structures via the CDK
  repo: https://github.com/rajarshi/cdkr
- name: rcdkTools
  role: Wraps rcdk to compute counting MACCS fingerprints (requires the modified CDK
    substructure fingerprint support branch)
  repo: https://github.com/bachi55/rcdkTools
- name: R (3.4)
  role: Data pre-processing scripts and loading/visualizing stored evaluation results
  repo: https://version.aalto.fi/gitlab/bache1/retention_order_prediction.git
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/bty590
    title: Liquid-chromatography retention order prediction for metabolite identification
  source_content_hash: sha256:0e7b11f792f2890c11f8d8c34cf43511bb7d707f7a3904c09c5b514a3ec809b1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Python
  - Python (scikit-learn, numpy, scipy, pandas, joblib)
  - Python and
  - R
  - R (3.4)
  - joblib
  - json >=
  - numpy
  - pandas
  - rcdk
  - rcdkTools
  - scikit-learn (sklearn)
  - scipy
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# ranksvm-pairwise-order-training

## Summary

Trains a ranking support vector machine (RankSVM) to predict the pairwise order in which molecules elute from an LC column, using molecular fingerprint kernels (MinMax on counting MACCS, Tanimoto on binary MACCS) and the dual Q = A K_m A^T decomposition so the full pairwise kernel matrix never has to be materialized.

## When to use

Use when you have retention time measurements from one or more LC systems and need to predict elution order from molecular structure, especially when retention times are on incomparable scales across systems so pairwise preferences within a system are the only consistent supervision.

## When NOT to use

- The task is predicting absolute retention times on a common scale — use the SVR baseline instead; the paper notes SVR was 'limited in this capacity' to exploit data from other chromatographic systems, which is exactly the setting where RankSVM on pairwise order should be preferred.
- The training and target molecules come from different chromatographic systems but you intend to train on cross-system time comparisons — preferences must be built only within a system (s_i = s_j); cross-system pairs are not valid supervision.

## Inputs

- Retention time data for PredRet LC systems (Eawag_XBridgeC18, FEM_long, RIKEN, UFZ_Phenomenex, LIFE_old) with experiment config JSON (e.g. results/raw/PredRet/v2/config.json)
- InChI representations of the molecular structures
- MACCS molecular fingerprints (counting variant 'maccsCount_f2dcf0b3' with 'minmax' kernel, or binary 'maccs' with 'tanimoto' kernel) computed with rcdk/rcdkTools

## Outputs

- Trained RankSVM model (learned w over pairwise fingerprint differences)
- Pairwise retention order predictions for the target system, stored as retention_order_predictions.csv
- Pairwise order prediction accuracy on the held-out target system

## How to apply

From the PredRet data (config JSON, e.g. PredRet/v2/config.json) extract the pairwise preference set P(s) = {(i,j) | s_i = s_j, t_i < t_j} within each chromatographic system, so the earlier-eluting molecule is preferred. Compute MACCS fingerprints with the CDK via rcdk/rcdkTools — counting fingerprints for the MinMax kernel, binary fingerprints for the Tanimoto kernel — and build the molecular kernel matrix [K_m]_ij = phi(m_i)^T phi(m_j). Train the RankSVM by solving the dual max_alpha -1/2 alpha^T Q alpha + 1^T alpha with 0 <= alpha_ij <= C, decomposing Q = A K_m A^T to avoid materializing the full pairwise kernel matrix; select C by a parallel hyper-parameter grid search (joblib). Apply the learned w to the target system, predicting pairwise order via f(m_i, m_j) = sign(w^T (phi(m_j) - phi(m_i))), and store the model and predictions (e.g. retention_order_predictions.csv).

## Related tools

- **Python (scikit-learn, numpy, scipy, pandas, joblib)** (Implements the RankSVM dual solver, pairwise preference extraction, kernel construction, and parallel hyper-parameter grid search) — https://version.aalto.fi/gitlab/bache1/retention_order_prediction.git
- **rcdk** (Calculates MACCS molecular fingerprints from InChI structures via the CDK) — https://github.com/rajarshi/cdkr
- **rcdkTools** (Wraps rcdk to compute counting MACCS fingerprints (requires the modified CDK substructure fingerprint support branch)) — https://github.com/bachi55/rcdkTools
- **R (3.4)** (Data pre-processing scripts and loading/visualizing stored evaluation results) — https://version.aalto.fi/gitlab/bache1/retention_order_prediction.git

## Examples

```
python src/evaluation_scenarios_main.py ranksvm baseline_single 10 -1 results/raw/PredRet/v2/config.json 2 False
```

## Evaluation signals

- Pairwise order prediction accuracy on the held-out target system should reproduce or be consistent with the Table 3 / Table 4 (first two columns) baseline_single values, e.g. 0.844 vs 0.796 for Eawag_XBridgeC18 (counting vs binary MACCS).
- The counting-MACCS/MinMax configuration should score at or above the binary-MACCS/Tanimoto configuration per target system.
- Results directory structure must exist: results/PredRet/v2/final/ranksvm_slacktype=on_pairs/allow_overlap=True_d_lower=0_d_upper=16_ireverse=False_type=order_graph/difference/<predictor>/<kernel>/baseline_single.
- The pairwise preference set must contain only within-system pairs (s_i = s_j with t_i < t_j); validate no cross-system pairs enter Q.
- The decomposition Q = A K_m A^T must keep Q's dimensions at the number of pairs, with the molecular kernel K_m of size n_molecules x n_molecules.

## Limitations

- Counting fingerprints require a modified rcdk/CDK build (the bachi55 fork with branch support_cdk_substructure_fps and the countable MACCS SMARTS patterns); the original rcdk only supports binary MACCS.
- Binary MACCS with the Tanimoto kernel systematically underperforms counting MACCS with MinMax in the single-system setting, so binary representations are a weaker baseline.
- Retention time measurements must be preprocessed: keep only the lowest retention time per molecule and remove molecules whose lowest and largest RT differ by more than 5%; additionally remove molecules with very small retention times that do not interact with the chromatographic system.
- The regularization parameter C must be tuned per run via grid search; results are stored under hyper-parameter-labeled directory names and must be matched to the correct configuration.

## Evidence

- [methods] pairwise preference set extraction: [evidence span withheld — anchor status: not_found]
- [methods] dual Q decomposition: [evidence span withheld — anchor status: not_found]
- [methods] prediction rule: [evidence span withheld — anchor status: not_found]
- [readme] kernel choice: [evidence span withheld — anchor status: not_found]
- [results] counting beats binary: [evidence span withheld — anchor status: not_found]
