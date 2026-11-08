---
name: cross-chromatographic-system-generalization-testing
description: Use when you need to quantify whether retention order or retention time
  models generalize across chromatographic systems, e.g. reproducing the Table 4 RankSVM-vs-SVR
  comparison on PredRet data.
when_to_use_negative:
- You only have a single chromatographic system's measurements and cannot vary the
  training system set — the all_on_one scenario requires heterogeneous LC systems
- Your target is predicting raw retention time values rather than testing cross-system
  transfer of pairwise retention order
- You need percentage-of-data curves (varying training fractions) instead of the LTSO
  True/False comparison — use baseline_single_perc or all_on_one_perc instead
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: retention_order_prediction
  role: Hosts evaluation_scenarios_main.py, which implements the baseline_single and
    all_on_one evaluation scenarios for ranksvm/svr estimators
  repo: https://version.aalto.fi/gitlab/bache1/retention_order_prediction.git
- name: scikit-learn (sklearn)
  role: Provides the SVR retention time baseline whose predicted retention times are
    mapped to pairwise orders for accuracy computation
- name: joblib
  role: Parallelizes the hyper-parameter search over regularization parameter C (slack
    on pairs, pairwise MinMax/Tanimoto kernel)
- name: pandas
  role: Aggregates pairwise prediction accuracy per target system to assemble the
    six-column Table 4 comparison
- name: rcdkTools
  role: Computes MACCS counting fingerprints (and binary variants) used as molecular
    representations in both training settings
  repo: https://github.com/bachi55/rcdkTools
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/bty590
    title: Liquid-chromatography retention order prediction for metabolite identification
  source_content_hash: sha256:45f647846019fcf420320929cfa35087564f1cf2e5d5b690e4bc6df362149a5d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Python
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
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 16
---

# cross-chromatographic-system-generalization-testing

## Summary

Tests whether a predictor (RankSVM on retention order vs SVR on retention time) benefits from pooling RT measurements across heterogeneous LC systems by comparing single-system training (baseline_single) against all-systems training with a single target system (all_on_one, with and without leaving the target system out of training, LTSO True/False).

## When to use

Use when you need to quantify whether retention order or retention time models generalize across chromatographic systems, e.g. reproducing the Table 4 RankSVM-vs-SVR comparison on PredRet data.

## When NOT to use

- You only have a single chromatographic system's measurements and cannot vary the training system set — the all_on_one scenario requires heterogeneous LC systems
- Your target is predicting raw retention time values rather than testing cross-system transfer of pairwise retention order
- You need percentage-of-data curves (varying training fractions) instead of the LTSO True/False comparison — use baseline_single_perc or all_on_one_perc instead

## Inputs

- results/raw/PredRet/v2/config.json (PredRet experiment configuration with molecule_representation kernel and predictor settings)
- PredRet retention time measurements for systems Eawag_XBridgeC18, FEM_long, RIKEN, UFZ_Phenomenex, LIFE_old
- MACCS counting (maccsCount) and binary (maccs) molecular fingerprints
- Estimator choice: {'ranksvm', 'svr'}; scenario choice: {'baseline_single', 'all_on_one'}; SYSSET '10'; TSYSIDX (-1 for all targets)

## Outputs

- Pairwise prediction accuracy aggregated per target system for each scenario and estimator
- Six-column comparison table mirroring Table 4 (RankSVM/SVR x baseline_single, LTSO True, LTSO False), e.g. saved as table4_ranksvm_vs_svr.csv
- Results stored under results/PredRet/v2/final/<estimator>_<slacktype>/<params>/<fingerprint>/<kernel>/<scenario>

## How to apply

Load the experiment configuration results/raw/PredRet/v2/config.json with SYSSET '10' (Eawag_XBridgeC18, FEM_long, RIKEN, UFZ_Phenomenex, LIFE_old), the maccsCount fingerprint with MinMax kernel plus a binary MACCS/Tanimoto variant for comparison. Run src/evaluation_scenarios_main.py under the baseline_single scenario (single system as training and target) for both ranksvm and svr estimators, with regularization parameter C tuned by a parallel joblib hyper-parameter search (slack on pairs, pairwise MinMax/Tanimoto kernel). Repeat both estimators under the all_on_one scenario (all systems used for training, single target system) once with LTSO True and once with LTSO False. Convert SVR predicted retention times to pairwise orders, aggregate pairwise prediction accuracy per target system with pandas, and assemble a six-column table mirroring Table 4 (RankSVM/SVR x baseline_single, LTSO True, LTSO False). The expected outcome is that RankSVM gains accuracy from multi-system training while SVR does not.

## Related tools

- **retention_order_prediction** (Hosts evaluation_scenarios_main.py, which implements the baseline_single and all_on_one evaluation scenarios for ranksvm/svr estimators) — https://version.aalto.fi/gitlab/bache1/retention_order_prediction.git
- **scikit-learn (sklearn)** (Provides the SVR retention time baseline whose predicted retention times are mapped to pairwise orders for accuracy computation)
- **joblib** (Parallelizes the hyper-parameter search over regularization parameter C (slack on pairs, pairwise MinMax/Tanimoto kernel))
- **pandas** (Aggregates pairwise prediction accuracy per target system to assemble the six-column Table 4 comparison)
- **rcdkTools** (Computes MACCS counting fingerprints (and binary variants) used as molecular representations in both training settings) — https://github.com/bachi55/rcdkTools

## Examples

```
python src/evaluation_scenarios_main.py ranksvm all_on_one 10 -1 results/raw/PredRet/v2/config.json 2 False
```

## Evaluation signals

- Reproduced accuracies agree with the RankSVM/SVR values reported in Table 4 for baseline_single, LTSO True, and LTSO False columns
- RankSVM retains its advantage over SVR in the pooled multi-system training settings (all_on_one LTSO True and False)
- SVR pairwise accuracy under LTSO True does not improve over its baseline_single accuracy (no cross-system transfer for retention time regression)
- Results directory structure matches results/PredRet/v2/final/.../all_on_one with the expected fingerprint/kernel subfolders (maccsCount_f2dcf0b3/minmax and maccs/tanimoto)
- Table saved as table4_ranksvm_vs_svr.csv contains six columns covering both estimators across the three training settings

## Limitations

- SVR-based retention time prediction is limited in its capacity to use data from other chromatographic systems, unlike RankSVM
- Retention order gains apply to pairwise ordering; molecules with very small retention times (non-interacting with the chromatographic system) and molecules with >5% retention time spread are removed during preprocessing, so conclusions do not cover them
- Only measurements added before July 2015 were considered, so replication on updated PredRet data may differ

## Evidence

- [intro] We demonstrate that the retention order framework is able to benefit from retention time measurements arising from heterogeneous LC systems and configurations: [evidence span withheld — anchor status: not_found]
