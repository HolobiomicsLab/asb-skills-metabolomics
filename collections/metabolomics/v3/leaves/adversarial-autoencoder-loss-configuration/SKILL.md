---
name: adversarial-autoencoder-loss-configuration
description: Use when running or ablating RALPS's composite objective on multi-batch
  metabolomics intensity tables to test whether the classifier, autoencoder-regularization
  and variation loss terms each contribute measurably to normalization quality.
when_to_use_negative:
- Data are single-batch with intrabatch drift only — RALPS targets multi-batch harmonization
  after per-batch normalization (e.g. temporal LC-MS drifts handled separately)
- The goal is comparing RALPS against other batch-correction methods rather than ablating
  its own loss terms
- No replicate/reference samples present in every batch, since interbatch differences
  are assessed on the basis of replicate measurements across batches
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: RALPS
  role: Adversarial autoencoder normalization method whose three-term loss (d_lambda,
    g_lambda, v_lambda) is configured and ablated via config CSV and ralps.py
  repo: https://github.com/zamboni-lab/RALPS
- name: PyTorch
  role: Backend used to train the adversarial autoencoder with the randomized grid
    search over d_lr, g_lr and batch_size on the chosen device
- name: hdbscan
  role: Clustering of reference/benchmark samples for the clustering evaluation criterion
- name: scikit-learn
  role: Clustering and evaluation utilities for the benchmark criteria
- name: pandas
  role: Aggregation of per-scenario metric values
- name: matplotlib
  role: Plotting the scenario comparison (Figure 4a)
- name: seaborn
  role: Plot styling for the scenario comparison plot
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad096
    title: Regularized adversarial learning for normalization of multi-batch untargeted
      metabolomics data
  source_content_hash: sha256:cd23ff46283a19644d176321d6d3cf97ea48db4be3dddcd5ce336ba34093e9b0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zamboni-lab/RALPS
  tools:
  - PyTorch
  - RALPS
  - hdbscan
  - matplotlib
  - numpy
  - pandas
  - scikit-learn
  - scipy
  - seaborn
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# adversarial-autoencoder-loss-configuration

## Summary

Configure and ablate the three-term composite loss (d_lambda, g_lambda, v_lambda) of RALPS, a regularized adversarial learning method for normalizing multi-batch untargeted metabolomics data, to determine which regularization terms drive clustering, replicate cross-correlation and batch VC performance.

## When to use

Use when running or ablating RALPS's composite objective on multi-batch metabolomics intensity tables to test whether the classifier, autoencoder-regularization and variation loss terms each contribute measurably to normalization quality.

## When NOT to use

- Data are single-batch with intrabatch drift only — RALPS targets multi-batch harmonization after per-batch normalization (e.g. temporal LC-MS drifts handled separately)
- The goal is comparing RALPS against other batch-correction methods rather than ablating its own loss terms
- No replicate/reference samples present in every batch, since interbatch differences are assessed on the basis of replicate measurements across batches

## Inputs

- CSV data file of multi-batch untargeted metabolomics intensities (data_path)
- CSV batch information file with replicate/sample group labels (info_path)
- RALPS config CSV files per ablation scenario, setting d_lambda, g_lambda, v_lambda, latent_dim=-1, variance_ratio 0.9,0.95,0.99, epochs=30, skip_epochs=3, min_relevant_intensity=1000

## Outputs

- Normalized data matrices per ablation scenario
- Per-scenario evaluation metrics on clustering, replicate cross-correlation and batch VC criteria
- Scenario-comparison plot mirroring Figure 4a (matplotlib/seaborn)
- Aggregated per-scenario metric tables (pandas)

## How to apply

Create RALPS config CSV files that differ only in the regularization coefficients d_lambda (classifier loss coef), g_lambda (autoencoder regularization term coef) and v_lambda (variation loss coef), spanning scenario 0 (all zero) to scenario 3 (all three non-zero, each sampled in the 0.-10. interval), with shared settings latent_dim=-1 (automatically derived from PCA at variance_ratio 0.9,0.95,0.99), epochs=30, skip_epochs=3 and min_relevant_intensity=1000. For each scenario run 'python ralps.py -n path/to/config.csv' from the src directory so the randomized grid search uniformly samples d_lr and g_lr from 0.00005-0.005 and batch_size from 32,64,128 with train_ratio=0.9, using PyTorch on the chosen device. Missing values are replaced with min_relevant_intensity=1000 before training, and values below this threshold are masked with zeros after normalization. Evaluate each scenario's normalized output on benchmark samples using clustering (hdbscan/scikit-learn), replicate cross-correlation and batch VC criteria informed by the batch info file, then aggregate with pandas and plot with matplotlib/seaborn to mirror Figure 4a. The rationale is that the three loss terms act synergistically: λg improves clustering of reference samples, λd has subtle additional effects, and top performance requires λv > 0.

## Related tools

- **RALPS** (Adversarial autoencoder normalization method whose three-term loss (d_lambda, g_lambda, v_lambda) is configured and ablated via config CSV and ralps.py) — https://github.com/zamboni-lab/RALPS
- **PyTorch** (Backend used to train the adversarial autoencoder with the randomized grid search over d_lr, g_lr and batch_size on the chosen device)
- **hdbscan** (Clustering of reference/benchmark samples for the clustering evaluation criterion)
- **scikit-learn** (Clustering and evaluation utilities for the benchmark criteria)
- **pandas** (Aggregation of per-scenario metric values)
- **matplotlib** (Plotting the scenario comparison (Figure 4a))
- **seaborn** (Plot styling for the scenario comparison plot)

## Examples

```
python ralps.py -n path/to/config.csv
```

## Evaluation signals

- Scenario 3 (non-zero d_lambda, g_lambda and v_lambda) attains top performance on all three evaluation criteria — clustering, replicate cross-correlation and batch VC — reproducing the paper's ablation result
- Introducing λg improves clustering of reference samples relative to the all-zero scenario 0
- Each run replaces missing values with min_relevant_intensity=1000 before training and masks values below 1000 with zeros after normalization
- Hyperparameters are sampled within the defined intervals: d_lr/g_lr in 0.00005-0.005, batch_size in {32,64,128}, d_lambda/g_lambda/v_lambda in 0.-10.
- Model selection skips the first skip_epochs=3 of the 30 training epochs, as set in the config

## Limitations

- The ablation is stochastic because d_lr, g_lr, batch_size and lambda coefficients are sampled in a randomized grid search, so per-scenario results vary between runs
- The λd term shows only subtle additional effects on its own, so single-term contributions may be hard to detect without the full three-criteria evaluation
- Training cost is non-trivial: ~5.82 minutes per run on average (30 epochs) for a dataset with ~3000 samples and ~150 metabolites

## Evidence

- [abstract] three-term loss function: "RALPS builds on deep adversarial learning with a three-term loss function that mitigates batch effects while preserving biological identity, spectral properties and coefficients of variation."
- [results] ablation finding: [evidence span withheld — anchor status: not_found]
- [methods] run command: "Run the following command from the `src` directory to normalize data with RALPS: `python ralps.py -n path/to/config.csv`"
- [readme] lambda defaults and sampling: "d_lambda | classifier loss coef | 0.-10."
- [readme] min_relevant_intensity handling: "missing values before normalization are replaced with this;<br/>values below this after normalization are masked with zeros |                1000"
- [readme] latent_dim derivation: "latent_dim | dimension of the bottleneck linear layer | -1 (automatically derived from PCA)"
