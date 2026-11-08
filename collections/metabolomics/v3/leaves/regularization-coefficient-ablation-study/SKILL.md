---
name: regularization-coefficient-ablation-study
description: Use when you need to attribute normalization performance to individual
  components of a multi-term adversarial loss (e.g. RALPS's d_lambda, g_lambda, v_lambda)
  via scenario-based ablation with fixed shared hyperparameters.
when_to_use_negative:
- The goal is to compare RALPS against other batch-correction methods rather than
  dissect its internal loss terms
- The dataset is single-batch, so no interbatch harmonization or reference-sample-based
  ablation is meaningful
- You only need a final normalized table and cannot afford the 4x randomized grid
  search training cost (~5.82 minutes per 30-epoch run per scenario)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: RALPS
  role: Core normalization pipeline run per ablation scenario via config file
  repo: https://github.com/zamboni-lab/RALPS
- name: PyTorch
  role: Training backend (device selection) for the adversarial autoencoder and classifier
- name: pandas
  role: Aggregation of per-scenario evaluation metrics
- name: scikit-learn
  role: Clustering-based evaluation of normalized benchmark samples
- name: hdbscan
  role: Clustering of reference samples for the clustering criterion
- name: matplotlib
  role: Plotting the scenario comparison mirroring Figure 4a
- name: seaborn
  role: Statistical plotting of the ablation results
- name: numpy
  role: Numerical handling of masked/replaced intensities and metrics
- name: scipy
  role: Cross-correlation computation for the replicate criterion
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad096
    title: Regularized adversarial learning for normalization of multi-batch untargeted
      metabolomics data
  source_content_hash: sha256:517d42b2a3bb237d49d1ea1bd927f2c5e74598d8a5d3a890e3abc7168f101f95
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# regularization-coefficient-ablation-study

## Summary

Reproduces the RALPS loss-term ablation (Figure 4a) by running the normalization pipeline in four configurations that vary d_lambda, g_lambda and v_lambda from all-zero to all-active, then comparing clustering, replicate cross-correlation and batch VC criteria to show the three terms act synergistically.

## When to use

Use when you need to attribute normalization performance to individual components of a multi-term adversarial loss (e.g. RALPS's d_lambda, g_lambda, v_lambda) via scenario-based ablation with fixed shared hyperparameters.

## When NOT to use

- The goal is to compare RALPS against other batch-correction methods rather than dissect its internal loss terms
- The dataset is single-batch, so no interbatch harmonization or reference-sample-based ablation is meaningful
- You only need a final normalized table and cannot afford the 4x randomized grid search training cost (~5.82 minutes per 30-epoch run per scenario)

## Inputs

- CSV metabolomics feature table (data_path) with ~3000 samples and ~150 metabolites
- CSV batch info file (info_path) with sample batch and replicate group labels
- Four RALPS config CSV files differing only in d_lambda, g_lambda, v_lambda

## Outputs

- Per-scenario normalized data matrices from RALPS
- Per-scenario clustering, replicate cross-correlation and batch VC criterion scores
- Scenario-comparison plot mirroring Figure 4a (matplotlib/seaborn)

## How to apply

Create four RALPS config CSVs that differ only in the regularization coefficients d_lambda (classifier loss coef), g_lambda (autoencoder regularization term coef) and v_lambda (variation loss coef), spanning scenario 0 (all zero) to scenario 3 (all three non-zero, each sampled in the 0.-10. interval). Keep shared settings fixed: latent_dim=-1 so the bottleneck dimension is automatically derived from PCA at variance_ratio 0.9,0.95,0.99, epochs=30, skip_epochs=3, and min_relevant_intensity=1000. For each scenario, run 'python ralps.py -n path/to/config.csv' from the src directory so the randomized grid search uniformly samples d_lr and g_lr from 0.00005-0.005 and batch_size from 32,64,128 at train_ratio=0.9 with PyTorch. Missing values are replaced with min_relevant_intensity=1000 before training, and values below this threshold are masked with zeros after normalization. Evaluate each scenario's normalized benchmark output on clustering (hdbscan/scikit-learn), replicate cross-correlation and batch VC criteria using replicate group information from the batch info file, aggregate with pandas, and plot the scenario comparison with matplotlib/seaborn to mirror Figure 4a. Validation requires that the all-three-terms scenario attains top performance on all three criteria; introducing lambda_g should improve clustering of reference samples while lambda_d contributes only subtle additional effects.

## Related tools

- **RALPS** (Core normalization pipeline run per ablation scenario via config file) — https://github.com/zamboni-lab/RALPS
- **PyTorch** (Training backend (device selection) for the adversarial autoencoder and classifier)
- **pandas** (Aggregation of per-scenario evaluation metrics)
- **scikit-learn** (Clustering-based evaluation of normalized benchmark samples)
- **hdbscan** (Clustering of reference samples for the clustering criterion)
- **matplotlib** (Plotting the scenario comparison mirroring Figure 4a)
- **seaborn** (Statistical plotting of the ablation results)
- **numpy** (Numerical handling of masked/replaced intensities and metrics)
- **scipy** (Cross-correlation computation for the replicate criterion)

## Examples

```
python ralps.py -n path/to/config.csv
```

## Evaluation signals

- Scenario 3 (non-zero d_lambda, g_lambda, v_lambda) ranks top on all three criteria: clustering, replicate cross-correlation and batch VC
- Introducing lambda_g measurably improves clustering of reference samples relative to scenario 0-1
- lambda_d alone produces only subtle additional effects over prior scenarios
- All scenarios use identical latent_dim=-1 (PCA-derived at variance_ratio 0.9,0.95,0.99), epochs=30, skip_epochs=3, and min_relevant_intensity=1000, differing only in the three lambda coefficients
- Values below min_relevant_intensity=1000 are masked with zeros in each scenario's normalized output

## Limitations

- The randomized grid search samples d_lr/g_lr in 0.00005-0.005 and batch_size in 32,64,128 per run, so scenario differences can include stochastic hyperparameter variation; multiple grid_size runs may be needed for stability
- Training cost scales with scenarios: ~5.82 minutes per 30-epoch run for a ~3000-sample/~150-metabolite dataset
- ablation conclusions were demonstrated on two large multi-batch metabolomics datasets with reference samples present in every batch; other experimental designs may need re-validation

## Evidence

- [abstract] RALPS builds on deep adversarial learning with a three-term loss function that mitigates batch effects while preserving biological identity, spectral properties and coefficients of variation.: "RALPS builds on deep adversarial learning with a three-term loss function that mitigates batch effects while preserving biological identity, spectral properties and coefficients of variation."
- [methods] Run the following command from the `src` directory to normalize data with RALPS: `python ralps.py -n path/to/config.csv`: "Run the following command from the `src` directory to normalize data with RALPS: `python ralps.py -n path/to/config.csv`"
- [readme] values will be uniformly sampled in the randomized grid search using defined options or intervals: "values will be uniformly sampled in the randomized grid search using defined options or intervals"
- [readme] missing values before normalization are replaced with this;<br/>values below this after normalization are masked with zeros: "missing values before normalization are replaced with this;<br/>values below this after normalization are masked with zeros"
- [methods] # of epochs to skip for model selection: "# of epochs to skip for model selection"
