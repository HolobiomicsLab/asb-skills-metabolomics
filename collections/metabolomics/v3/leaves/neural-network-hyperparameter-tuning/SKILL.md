---
name: neural-network-hyperparameter-tuning
description: Use when preparing a MiMeNet run on paired microbial/metabolite count
  CSVs and you must fix the MLP architecture (number of layers, layer size, L2 penalty,
  dropout rate) via a JSON net_params file for shared or per-partition tuning.
when_to_use_negative:
- The metabolite prediction task is handled by regularized linear regression (MelonnPan-style),
  which tunes a weight matrix per metabolite rather than MLP hyper-parameters.
- The input data is not paired microbe/metabolite count matrices in the comma-delimited
  CSV format MiMeNet_train.py expects.
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0621
tools:
- name: MiMeNet
  role: Neural network framework whose MLP hyper-parameters (layers, size, L2 penalty,
    dropout) are specified in a JSON net_params file and tuned before cross-validated
    SCC evaluation
  repo: https://github.com/YDaiLab/MiMeNet
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1009021
    title: 'MiMeNet: Exploring microbiome-metabolome relationships using neural networks'
  source_content_hash: sha256:ae87b966b473b16eff0c09006eb7d282f5a6c246798fc5a5a351748ef0e8c218
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YDaiLab/MiMeNet
  tools:
  - MiMeNet
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# neural-network-hyperparameter-tuning

## Summary

Tune and share MLP hyper-parameters for MiMeNet metabolite prediction via a JSON net_params file (layers, layer size, L2 penalty, dropout), either once from the first CV partition or separately per partition, before cross-validated SCC evaluation.

## When to use

Use when preparing a MiMeNet run on paired microbial/metabolite count CSVs and you must fix the MLP architecture (number of layers, layer size, L2 penalty, dropout rate) via a JSON net_params file for shared or per-partition tuning.

## When NOT to use

- The metabolite prediction task is handled by regularized linear regression (MelonnPan-style), which tunes a weight matrix per metabolite rather than MLP hyper-parameters.
- The input data is not paired microbe/metabolite count matrices in the comma-delimited CSV format MiMeNet_train.py expects.

## Inputs

- JSON net_params file containing neural network number of layers, layer size, L2 penalty, and dropout rate
- Comma-delimited CSV of samples by microbial features (micro)
- Comma-delimited CSV of samples by metabolomic features (metab)

## Outputs

- Tuned MLP hyper-parameters (layers, layer size, L2 penalty, dropout rate) applied across cross-validated training runs
- Per-metabolite Spearman correlation coefficients (SCC) from the cross-validated evaluation that uses the tuned parameters

## How to apply

Specify the multi-layer perceptron's number of layers, layer size, L2 penalty, and dropout rate in a JSON net_params file and pass it to MiMeNet_train.py with -net_params. For shared tuning, tune once from the first partition of the cross-validation and reuse those parameters across all partitions; for per-partition tuning, tune separately within each partition. Tune hyper-parameters before the cross-validated evaluation that produces per-metabolite Spearman correlation coefficients (SCC). Robustness experiments vary num_cv (10, 5, 3, 2) and num_run_cv (10 iterations), and per-partition tuning was tested to confirm it does not cause overfitting — sharing tuned parameters across partitions is acceptable when sample size is limited.

## Related tools

- **MiMeNet** (Neural network framework whose MLP hyper-parameters (layers, size, L2 penalty, dropout) are specified in a JSON net_params file and tuned before cross-validated SCC evaluation) — https://github.com/YDaiLab/MiMeNet

## Examples

```
python MiMeNet_train.py -micro data/IBD/microbiome_PRISM.csv -metab data/IBD/metabolome_PRISM.csv -micro_norm None -metab_norm CLR -net_params results/IBD/network_parameters.txt -num_run_cv 10 -output IBD
```

## Evaluation signals

- The JSON net_params file parses and supplies all four fields: number of layers, layer size, L2 penalty, and dropout rate
- MiMeNet_train.py runs to completion with -net_params pointing at the file and num_run_cv set to 10 iterations of cross-validation
- Shared hyper-parameters tuned from the first partition do not degrade mean SCC relative to per-partition tuning (no overfitting expected)
- Resulting per-metabolite SCC values fall in the ranges reported for MiMeNet (e.g. mean SCC ~0.457 for cystic fibrosis, ~0.309 for IBD PRISM)

## Limitations

- Hyper-parameter values were tuned from the first partition in the robustness analysis; separate tuning per partition adds cost and was checked for overfitting rather than guaranteed improvement.
- MiMeNet, like all methods in the paper, is still limited in providing biological plausibility and mechanistic insights; tuning optimizes predictive SCC, not mechanistic fidelity.
- Only architecture-level hyper-parameters (layers, size, L2 penalty, dropout) are exposed via net_params; other training settings are controlled through separate CLI flags.

## Evidence

- [readme] net_params: "net_params| JSON file containing neural network number of layers, layer size, L2 penalty, and dropout rate"
- [methods] Hyper-parameter tuning workflow: "Then models are evaluated in a cross-validated fashion resulting in Spearman correlation coefficients (SCC) for each metabolite"
- [supplementary] Shared hyper-parameters robustness finding: "Using 10-iterations of 10-fold cross-validation, evaluations using shared hyper-parameters tuned from the first partition"
- [results] Set the multi-layer perceptron hyper-parameters: [evidence span withheld — anchor status: not_found]
