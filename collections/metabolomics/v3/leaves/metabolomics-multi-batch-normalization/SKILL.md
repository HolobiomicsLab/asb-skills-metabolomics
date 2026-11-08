---
name: metabolomics-multi-batch-normalization
description: Use when a feature-count matrix (e.g. 170 metabolites x 2856 samples)
  spans multiple LC-MS batches with reference samples (pooled/NIST 1950 SRM) present
  in each batch and interbatch differences must be removed before analysis.
when_to_use_negative:
- Data contains only a single batch — RALPS is designed to harmonize multi-batch measurements,
  not intrabatch drift correction (normalize each batch individually first)
- No reference/replicate samples (e.g. pooled QC or NIST 1950 SRM) are present in
  the batches to define group/batch info for training
- The dataset has been pre-normalized so interbatch differences are already removed
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: RALPS
  role: core adversarial autoencoder normalization tool run via `python ralps.py -n
    path/to/config.csv` from the src directory
  repo: https://github.com/zamboni-lab/RALPS
- name: PyTorch
  role: framework for training the adversarial autoencoder and batch classifier (torch==1.8.1)
- name: pandas
  role: loading the data and batch info CSV files and aggregating per-scenario metrics
- name: numpy
  role: computing replicate cross-correlations and variance metrics on normalized
    data
- name: scikit-learn
  role: PCA-based automatic derivation of latent_dim at variance_ratio 0.9/0.95/0.99
- name: hdbscan
  role: clustering of reference samples in the embedded space used by the regularization
    term
- name: matplotlib
  role: rendering evaluation plots (plots_extension=png)
- name: seaborn
  role: plot styling for evaluation and ablation figures
- name: umap-learn
  role: embedding used with HDBSCAN to encourage replicate groups into the same cluster
provenance:
  source_task_ids:
  - task_005
  - task_008
  - task_002
  - task_004
  - task_006
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad096
    title: Regularized adversarial learning for normalization of multi-batch untargeted
      metabolomics data
  source_content_hash: sha256:c3ed9460b42940fb3f7380dd020196f4965d9b4b83f7b28a72f806dfe65b97ea
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
  - umap-learn
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.47058823529411764
n_claims: 17
---

# metabolomics-multi-batch-normalization

## Summary

Normalize multi-batch untargeted metabolomics data with RALPS (Regularized Adversarial Learning Preserving Similarity), a PyTorch adversarial autoencoder with a three-term loss that removes interbatch differences while preserving biological identity, spectral properties and coefficients of variation.

## When to use

Use when a feature-count matrix (e.g. 170 metabolites x 2856 samples) spans multiple LC-MS batches with reference samples (pooled/NIST 1950 SRM) present in each batch and interbatch differences must be removed before analysis.

## When NOT to use

- Data contains only a single batch — RALPS is designed to harmonize multi-batch measurements, not intrabatch drift correction (normalize each batch individually first)
- No reference/replicate samples (e.g. pooled QC or NIST 1950 SRM) are present in the batches to define group/batch info for training
- The dataset has been pre-normalized so interbatch differences are already removed

## Inputs

- csv feature count matrix (metabolites x samples, e.g. 170 features x 2856 samples from seven batches)
- csv batch info file with batch, group and benchmark columns identifying reference replicates (e.g. P2_S_0001 NIST1950 and P2_S_PP_0001 purine/pyrimidine-spiked serum)
- config.csv specifying data_path, info_path, out_path and hyperparameters

## Outputs

- normalized data matrix (normalized/data.csv) with post-normalization values below min_relevant_intensity masked with zeros
- post-normalization metrics (replicate cross-correlations, batch-wise coefficients of variation)
- evaluation plots (png) rendered with matplotlib/seaborn
- best model checkpoint (or all checkpoints in a checkpoints directory when keep_checkpoints=True)

## How to apply

First normalize each batch individually to suppress intrabatch problems such as temporal LC-MS drifts; RALPS is applied in a second step to harmonize multiple batches. Build a config.csv pointing to the data matrix and batch info file (batch, group, benchmark columns), replace missing values with min_relevant_intensity=1000 before training, and run the randomized grid search (`python ralps.py -n path/to/config.csv`) sampling d_lr and g_lr uniformly from 0.00005-0.005, d_lambda/g_lambda/v_lambda from 0.-10., and batch_size from {32,64,128} over grid_size runs of 30 epochs (skip_epochs=3 for model selection, train_ratio=0.9). The latent_dim is left at -1 so it is automatically derived from PCA at variance_ratio 0.9,0.95,0.99; the three-term loss (classifier loss, autoencoder regularization rewarding tight UMAP+HDBSCAN clustering of reference replicates, and variation loss penalizing per-sample VC increases beyond allowed_vc_increase=0.05) acts synergistically — ablations show top performance only when all three terms are active. After normalization, mask values below min_relevant_intensity with zeros, save normalized/data.csv, optionally remove outliers with `python ralps.py -r`, and evaluate via replicate cross-correlations and batch-wise coefficients of variation.

## Related tools

- **RALPS** (core adversarial autoencoder normalization tool run via `python ralps.py -n path/to/config.csv` from the src directory) — https://github.com/zamboni-lab/RALPS
- **PyTorch** (framework for training the adversarial autoencoder and batch classifier (torch==1.8.1))
- **pandas** (loading the data and batch info CSV files and aggregating per-scenario metrics)
- **numpy** (computing replicate cross-correlations and variance metrics on normalized data)
- **scikit-learn** (PCA-based automatic derivation of latent_dim at variance_ratio 0.9/0.95/0.99)
- **hdbscan** (clustering of reference samples in the embedded space used by the regularization term)
- **matplotlib** (rendering evaluation plots (plots_extension=png))
- **seaborn** (plot styling for evaluation and ablation figures)
- **umap-learn** (embedding used with HDBSCAN to encourage replicate groups into the same cluster)

## Examples

```
python ralps.py -n path/to/config.csv
```

## Evaluation signals

- Post-normalization replicate cross-correlations increase relative to pre-normalization (mean cross-correlation of replicates stays above 0.98 across PCA variance_ratio settings 0.99-0.7)
- Batch-wise coefficients of variation of the normalized data are reduced and batches no longer appear as isolated clusters in ordinations
- All normalized output values are strictly positive after masking (ComBat/WaveICA baselines produced ~1% negative values on the Cherkaoui dataset, RALPS did not)
- The scenario with all three loss terms non-zero (d_lambda, g_lambda, v_lambda) attains top performance on clustering, cross-correlation and batch VC criteria
- Output directories/files follow the default RALPS layout so downstream checkpoint evaluation (`-e`) and outlier removal (`-r`) commands work

## Limitations

- Each batch should be normalized individually first; RALPS is only the second harmonization step, not a substitute for intrabatch drift correction
- Normalization-induced variance tangibly worsens when the number of metabolites drops to approximately 17 (batch-wise VC stays stable down to 10% of initial features)
- Requires reference samples present across batches; training a ~3000-sample, ~150-metabolite dataset takes ~5.82 minutes per run (30 epochs)
- Randomized grid search (grid_size runs) means results depend on sampled hyperparameters; only the best checkpoint is kept unless keep_checkpoints=True

## Evidence

- [abstract] RALPS builds on deep adversarial learning with a three-term loss function that mitigates batch effects while preserving biological identity, spectral properties and coefficients of variation.: "RALPS builds on deep adversarial learning with a three-term loss function that mitigates batch effects while preserving biological identity, spectral properties and coefficients of variation."
- [intro] each batch should be first normalized individually to suppress intrabatch problems, e.g. temporal trends associated to drifts in LC-MS. RALPS is used in a second step to harmonize multiple batches.: "each batch should be first normalized individually to suppress intrabatch problems, e.g. temporal trends associated to drifts in LC-MS. RALPS is used in a second step to harmonize multiple batches."
- [readme] missing values before normalization are replaced with this; values below this after normalization are masked with zeros: [evidence span withheld — anchor status: not_found]
- [methods] Run the following command from the `src` directory to normalize data with RALPS: `python ralps.py -n path/to/config.csv`: "Run the following command from the `src` directory to normalize data with RALPS: `python ralps.py -n path/to/config.csv`"
- [readme] Training time required to normalize a dataset with ~3000 samples and ~150 metabolites was 5.82 minutes per run on average (30 epochs).: "Training time required to normalize a dataset with ~3000 samples and ~150 metabolites was 5.82 minutes per run on average (30 epochs)."
