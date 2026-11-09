---
name: replicate-group-clustering-regularization
description: Use when normalizing multi-batch untargeted metabolomics data (e.g. flow-injection
  MS) where reference samples such as pooled study samples or NIST 1950 SRM are present
  in every batch and their biological similarity must be preserved during batch-effect
  removal.
when_to_use_negative:
- Data has not been normalized per batch first — RALPS is designed as a second step
  after intrabatch correction of temporal LC-MS drifts
- No replicate/reference samples are present across batches, so there is no basis
  for the clustering regularization term
- A single-batch dataset with no interbatch differences to harmonize
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: RALPS
  role: Framework implementing the three-term adversarial training loop with the replicate-clustering
    regularization term
  repo: https://github.com/zamboni-lab/RALPS
- name: PyTorch
  role: Builds and trains the autoencoder and batch classifier jointly
- name: hdbscan
  role: Clusters replicate groups in the embedded space for the regularization term
- name: umap-learn
  role: Projects the embedded space used for HDBSCAN clustering of replicates
- name: scikit-learn
  role: Derives latent_dim from PCA at variance_ratio 0.9/0.95/0.99
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad096
    title: Regularized adversarial learning for normalization of multi-batch untargeted
      metabolomics data
  source_content_hash: sha256:cd86a164746379df850acddf57dba3125255df6eda05c78ad9cd670d3a5b235b
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
  - seaborn
  - umap-learn
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
  techniques:
  - direct-infusion-MS
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# Replicate-group clustering regularization

## Summary

A regularization term (g_lambda) added to the RALPS autoencoder loss that rewards tight clustering of reference/replicate samples in the embedded space, preserving biological identity while the adversarial classifier removes batch effects.

## When to use

Use when normalizing multi-batch untargeted metabolomics data (e.g. flow-injection MS) where reference samples such as pooled study samples or NIST 1950 SRM are present in every batch and their biological similarity must be preserved during batch-effect removal.

## When NOT to use

- Data has not been normalized per batch first — RALPS is designed as a second step after intrabatch correction of temporal LC-MS drifts
- No replicate/reference samples are present across batches, so there is no basis for the clustering regularization term
- A single-batch dataset with no interbatch differences to harmonize

## Inputs

- CSV data matrix of metabolomics intensities (data_path)
- CSV batch information file annotating samples, batches, and replicate/reference groups (info_path)
- Config CSV with g_lambda, d_lambda, v_lambda, learning rates, batch_size, and min_relevant_intensity

## Outputs

- Normalized data table with batch effects removed and replicate groups clustered (values below min_relevant_intensity masked with zeros)
- Best autoencoder checkpoint (or all checkpoints when keep_checkpoints=True)
- Evaluation plots (PNG by default) showing replicate clustering and coefficients of variation

## How to apply

Build the autoencoder and batch classifier in PyTorch and train them jointly for 30 epochs, optimizing the classifier loss (d_lambda), the autoencoder regularization term (g_lambda), and the variation loss (v_lambda) with learning rates sampled uniformly from 0.00005–0.005 in a randomized grid search. The g_lambda term penalizes embeddings where replicate groups of reference samples (mean n_replicates=3) do not fall into the same cluster after UMAP embedding plus HDBSCAN (hdbscan==0.8.27) clustering, so reference samples stay tightly clustered — preserving spectral similarity — while the adversarial classifier loss drives interbatch differences out of the latent space. A variation loss (v_lambda) additionally penalizes per-sample coefficient-of-variation increases beyond allowed_vc_increase=0.05, preventing variance expansion during reconstruction. Skip the first skip_epochs=3 epochs for model selection and keep only the best autoencoder checkpoint. Missing values are replaced with min_relevant_intensity=1000 before training, and values below that threshold are masked with zeros in the normalized output.

## Related tools

- **RALPS** (Framework implementing the three-term adversarial training loop with the replicate-clustering regularization term) — https://github.com/zamboni-lab/RALPS
- **PyTorch** (Builds and trains the autoencoder and batch classifier jointly)
- **hdbscan** (Clusters replicate groups in the embedded space for the regularization term)
- **umap-learn** (Projects the embedded space used for HDBSCAN clustering of replicates)
- **scikit-learn** (Derives latent_dim from PCA at variance_ratio 0.9/0.95/0.99)

## Examples

```
python ralps.py -n path/to/config.csv
```

## Evaluation signals

- Reference samples' spectral similarity is preserved after normalization (similarity of reference samples in the embedded space)
- Benchmark-sample coefficients of variation decrease relative to input, with per-sample VC increase not exceeding allowed_vc_increase=0.05
- Replicate groups of reference samples fall into the same UMAP+HDBSCAN cluster in the embedded space
- Output directories and files follow the default RALPS layout so downstream checkpoint evaluation and outlier-removal commands work
- Values below min_relevant_intensity=1000 appear as zeros in the normalized output table

## Limitations

- Each batch should first be normalized individually to suppress intrabatch problems such as temporal LC-MS drifts; RALPS only harmonizes between batches
- Hyperparameter search is randomized over d_lr/g_lr (0.00005–0.005) and batch_size {32,64,128}; a grid_size of 1 (default) may miss good parameter sets
- Training requires ~5.82 minutes per run (30 epochs) for ~3000 samples and ~150 metabolites, so large grid searches are costly

## Evidence

- [abstract] RALPS builds on deep adversarial learning with a three-term loss function that mitigates batch effects while preserving biological identity, spectral properties and coefficients of variation.: "RALPS builds on deep adversarial learning with a three-term loss function that mitigates batch effects while preserving biological identity, spectral properties and coefficients of variation."
- [readme] RALPS tries to reconstruct the original data while (i) removing interbatch differences on the basis of replicate measurements across batches and (ii) avoiding an expansion of the overall variance: "RALPS tries to reconstruct the original data while (i) removing interbatch differences on the basis of replicate measurements across batches and (ii) avoiding an expansion of the overall variance"
- [intro] each batch should be first normalized individually to suppress intrabatch problems, e.g. temporal trends associated to drifts in LC-MS.: "each batch should be first normalized individually to suppress intrabatch problems, e.g. temporal trends associated to drifts in LC-MS. RALPS is used in a second step to harmonize multiple batches"
- [methods] missing values before normalization are replaced with this; values below this after normalization are masked with zeros (min_relevant_intensity=1000): "missing values before normalization are replaced with this;<br/>values below this after normalization are masked with zeros"
- [readme] fraction of sample's VC increase allowed (not contributing to the variation loss) — allowed_vc_increase default 0.05: "fraction of sample's VC increase allowed (not contributing to the variation loss)"
