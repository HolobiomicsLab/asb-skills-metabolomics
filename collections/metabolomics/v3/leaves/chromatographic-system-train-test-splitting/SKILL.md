---
name: chromatographic-system-train-test-splitting
description: Use when evaluating RankSVM or SVR retention order/time prediction across
  multiple chromatographic systems (e.g. Eawag_XBridgeC18, FEM_long, RIKEN, UFZ_Phenomenex,
  LIFE_old) and you must decide which systems serve as training vs. target data.
when_to_use_negative:
- When the task is predicting absolute retention times for a single LC system with
  SVR and no cross-system generalization is intended (SVR cannot exploit data from
  other systems in the same way).
- When the input data are already combined retention-order + MS/MS identification
  scores from the dynamic programming integration step, not raw retention time measurements.
- When the user only wants to preprocess fingerprints or compute counting MACCS keys;
  that belongs to the fingerprint calculation skill, not to train/test splitting across
  systems.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3315
tools:
- name: evaluation_scenarios_main.py
  role: Entry-point script that instantiates the chosen train/test scenario (baseline_single,
    all_on_one, etc.) for RankSVM or SVR retention order/time prediction.
  repo: https://version.aalto.fi/gitlab/bache1/retention_order_prediction.git
- name: rcdkTools
  role: Computes the counting fingerprints (e.g. MACCS counting) that feed the RankSVM
    models compared across the split scenarios.
  repo: https://github.com/bachi55/rcdkTools
- name: rcdk (cdkr)
  role: Used for fingerprint calculation on PredRet molecules before the train/test
    split scenarios are executed.
  repo: https://github.com/rajarshi/cdkr
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/bty590
    title: Liquid-chromatography retention order prediction for metabolite identification
  source_content_hash: sha256:0b7c5a16424327fe136c38b7b182519108b069696b8332cdb1fcd735dd8a1543
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Python and
  - R
  - evaluation_scenarios_main.py
  - json >=
  - rcdk (cdkr)
  - rcdkTools
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# chromatographic-system-train-test-splitting

## Summary

Defines and runs evaluation scenarios in which RankSVM retention order predictors are trained and tested on different combinations of LC chromatographic systems (single-system baseline, all-systems training with a held-out target), using the evaluation_scenarios_main.py script from the retention_order_prediction repository.

## When to use

Use when evaluating RankSVM or SVR retention order/time prediction across multiple chromatographic systems (e.g. Eawag_XBridgeC18, FEM_long, RIKEN, UFZ_Phenomenex, LIFE_old) and you must decide which systems serve as training vs. target data.

## When NOT to use

- When the task is predicting absolute retention times for a single LC system with SVR and no cross-system generalization is intended (SVR cannot exploit data from other systems in the same way).
- When the input data are already combined retention-order + MS/MS identification scores from the dynamic programming integration step, not raw retention time measurements.
- When the user only wants to preprocess fingerprints or compute counting MACCS keys; that belongs to the fingerprint calculation skill, not to train/test splitting across systems.

## Inputs

- PredRet dataset (LC-MS retention time measurements from systems Eawag_XBridgeC18, FEM_long, RIKEN, UFZ_Phenomenex, LIFE_old)
- Configuration JSON specifying molecular representation (kernel and predictor), e.g. results/raw/PredRet/v2/config.json
- Preprocessed fingerprints (e.g. MACCS counting or binary) computed with rcdkTools/rcdk
- Estimator choice ('ranksvm' or 'svr'), scenario name, system set identifier, TSYSIDX, NJOBS and DEBUG flags

## Outputs

- Retention order prediction accuracy per target system (pairwise prediction accuracy for RankSVM)
- Serialized results directory tree, e.g. results/PredRet/v2/final/ranksvm_slacktype=on_pairs/allow_overlap=True_d_lower=0_d_upper=16_ireverse=False_type=order_graph/difference/<predictor>/<kernel>/<scenario>
- Comparison tables/figures consistent with Table 3, Table 4 or Figure 4 of the paper

## How to apply

Choose a scenario from evaluation_scenarios_main.py that matches the experimental question: 'baseline_single' trains and tests on a single system (reproducing Table 3/4 first two columns); 'all_on_one' trains on all systems and targets a single system with LTSO True or False (Table 4 columns 3-6); 'baseline_single_perc' and 'all_on_one_perc' vary the percentage of training data for Figures 4. Pass the estimator ('ranksvm' or 'svr'), the scenario, the system set (e.g. '10' for the five PredRet systems), TSYSIDX (-1 runs all target systems in one job), a config JSON path (e.g. results/raw/PredRet/v2/config.json), the number of jobs for hyper-parameter search, and a debug flag. Results are written under results/PredRet/v2/final/ with subdirectories encoding the scenario (e.g. ranksvm_slacktype=on_pairs/.../baseline_single). The key decision point is whether the target system is included in training (LTSO) since the paper shows that heterogeneous LC systems and configurations can still improve retention order prediction compared with training on one system alone.

## Related tools

- **evaluation_scenarios_main.py** (Entry-point script that instantiates the chosen train/test scenario (baseline_single, all_on_one, etc.) for RankSVM or SVR retention order/time prediction.) — https://version.aalto.fi/gitlab/bache1/retention_order_prediction.git
- **rcdkTools** (Computes the counting fingerprints (e.g. MACCS counting) that feed the RankSVM models compared across the split scenarios.) — https://github.com/bachi55/rcdkTools
- **rcdk (cdkr)** (Used for fingerprint calculation on PredRet molecules before the train/test split scenarios are executed.) — https://github.com/rajarshi/cdkr

## Examples

```
python src/evaluation_scenarios_main.py ranksvm baseline_single 10 -1 results/raw/PredRet/v2/config.json 2 False
```

## Evaluation signals

- The results directory path must contain the expected scenario folder name (e.g. baseline_single) and the predictor/kernel folders (e.g. maccsCount_f2dcf0b3/minmax or maccs/tanimoto).
- Pairwise prediction accuracies from the counting-MACCS + Min-Max configuration should match or exceed the binary-MACCS + Tanimoto configuration for the same split (Table 3 behaviour).
- Running with TSYSIDX = -1 must produce results for all five PredRet target systems in a single job; setting TSYSIDX to a specific index restricts evaluation to that system.
- Changing the scenario from baseline_single to all_on_one should change the results folder and, per the paper, show that training on heterogeneous systems improves accuracy over the single-system baseline.
- Running with DEBUG=True must not write results into the final/ directory.

## Limitations

- SVR retention time prediction is limited in its capacity to use data from other chromatographic systems, so cross-system splits are mainly meaningful for RankSVM order prediction.
- The PredRet measurements used were restricted to those added before July 2015, and molecules with very small retention times or >5% retention time spread were removed; the splits therefore assume this preprocessing has been applied.
- The hyper-parameter search parallelism (NJOBS) affects only runtime, not the split definition; changing it does not alter the train/test assignment.

## Evidence

- [readme] baseline_single: Single system used as training and target | Table 3, Table 4 (first two columns): "[```baseline_single```](src/evaluation_scenarios_main.py#L708) | Single system used as training and target | Table 3, Table 4 (first two columns)"
- [readme] all_on_one: All systems used for training. Single system used as target.: "[```all_on_one```](src/evaluation_scenarios_main.py#L615) | All systems used for training. Single system used as target. Target system in training (LTSO): True & False"
- [methods] Python evaluation entry point and required packages: "The [order predictor, e.g. RankSVM, and evaluation scripts are implemented in Python](src/). The code has been tested with Python 3.5 and 3.6. The following packages are required:"
- [abstract] Retention order better conserved between instruments than retention time: "We show that retention order is much better conserved between instruments than retention time."
