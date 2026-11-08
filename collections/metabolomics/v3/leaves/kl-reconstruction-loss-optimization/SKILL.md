---
name: kl-reconstruction-loss-optimization
description: Use when training a VAE on a standardized LC/MS feature matrix (e.g.
  z-scored float CSVs of shape [n_samples, 2736] from mzLearn) and the goal is an
  unsupervised latent embedding that clusters samples by disease or age without label
  supervision.
when_to_use_negative:
- Input is a supervised task (e.g. renal cell carcinoma survival prediction) rather
  than unsupervised pretraining on the full public cohort — use the fine-tuning entry
  points instead
- The LC/MS data is not an mzLearn-derived MS1 feature matrix (e.g. raw mzML or MS2
  spectra) — mzLearn outputs are limited to the ion MS1 level and no MS/MS integration
  into pretraining is implemented
- The data are not in HILIC positive mode or represent other LC types/modes — the
  current pre-trained model was developed specifically for HILIC positive mode and
  multi-modal learning is not yet developed
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3172
tools:
- name: mzLearn
  role: Produces the upstream MS1 ion-feature matrix (2,736 robust metabolite signals)
    that is standardized and fed to the VAE for KL/reconstruction optimization
- name: VAE
  role: Unsupervised variational autoencoder whose reconstruction and KL-divergence
    losses are optimized over the standardized mzLearn feature matrix
- name: mzEmbed
  role: Python package containing run_pretrain_VAE_main (Optuna-driven pretraining)
    and get_pretrain_encoder (encoder/latent export) that implement this skill
  repo: github.com/ReviveMed/mzEmbed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2025.01.26.634927
    title: mzLearn, a data-driven LC/MS signal detection algorithm, enables pre-trained
      generative models for enhanced patient stratification
  source_content_hash: sha256:0bc6d5be3581cdcaf1a1be7f52faa554a69c4465ab90b2bd9a6a8fa5ea7b02c5
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ReviveMed/mzEmbed
  tools:
  - VAE
  - mzEmbed
  - mzLearn
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# KL-reconstruction loss optimization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Optimizes the joint KL-divergence and reconstruction loss of a variational autoencoder (VAE) over the mzLearn MS1 ion-feature matrix (2,736 robust metabolite signals across 20,548 blood samples) to learn unsupervised latent representations of untargeted metabolomics samples.

## When to use

Use when training a VAE on a standardized LC/MS feature matrix (e.g. z-scored float CSVs of shape [n_samples, 2736] from mzLearn) and the goal is an unsupervised latent embedding that clusters samples by disease or age without label supervision.

## When NOT to use

- Input is a supervised task (e.g. renal cell carcinoma survival prediction) rather than unsupervised pretraining on the full public cohort — use the fine-tuning entry points instead
- The LC/MS data is not an mzLearn-derived MS1 feature matrix (e.g. raw mzML or MS2 spectra) — mzLearn outputs are limited to the ion MS1 level and no MS/MS integration into pretraining is implemented
- The data are not in HILIC positive mode or represent other LC types/modes — the current pre-trained model was developed specifically for HILIC positive mode and multi-modal learning is not yet developed

## Inputs

- mzLearn MS1 ion-feature matrix as CSVs of floats with shape [n_samples, 2736], z-scored, indexed by first column (X_Pretrain_Discovery_Train.csv, X_Pretrain_Discovery_Val.csv, X_Pretrain_Test.csv)
- Optional y_Pretrain_* label CSVs (disease, age) for downstream latent visualization
- Saved VAE checkpoint best_model.pt from the pretraining output directory

## Outputs

- best_model.pt — the pre-trained VAE checkpoint saved in the pretraining output directory
- study.html (Optuna trial report), config.json, and training logs
- encoder.pt — the exported VAE encoder module
- Z_Pretrain_*.csv — latent embeddings per split of shape [n_samples, latent_size]
- Latent UMAP figures colored by disease and age labels for visual inspection

## How to apply

Assemble the mzLearn MS1 ion-feature matrix (2,736 robust metabolite signals detected across 22 public studies and 20,548 blood samples) as CSVs of floats and z-score (standardize) the intensities. Split into X_Pretrain_Discovery_Train / X_Pretrain_Discovery_Val / X_Pretrain_Test files; feed the standardized matrices into `mz_embed.pretrain.run_pretrain_VAE_main`, which performs Optuna-driven optimization of the joint reconstruction and KL-divergence loss, with the KL term annealing from 0 to a target `kl_weight` (latent_size ~460, 3 encoder/decoder layers, 40 trials, early stopping on validation loss after up to 200 epochs). After the best checkpoint (`best_model.pt`) is selected on validation loss, export the encoder and latent embeddings via `get_pretrain_encoder` for downstream UMAP visualization. Rationale: the pre-trained VAE trained this way learned latent representations that cluster samples with similar disease and age characteristics, whereas the raw (unembedded) input intensities showed mixing of these labels.

## Related tools

- **mzLearn** (Produces the upstream MS1 ion-feature matrix (2,736 robust metabolite signals) that is standardized and fed to the VAE for KL/reconstruction optimization)
- **VAE** (Unsupervised variational autoencoder whose reconstruction and KL-divergence losses are optimized over the standardized mzLearn feature matrix)
- **mzEmbed** (Python package containing run_pretrain_VAE_main (Optuna-driven pretraining) and get_pretrain_encoder (encoder/latent export) that implement this skill) — github.com/ReviveMed/mzEmbed

## Examples

```
python -m mz_embed.pretrain.run_pretrain_VAE_main --data_location /path/to/pretrain_csvs --latent_size 460 --n_layers 3 --trials 40 --max_epochs 200 --output_dir /path/to/out_pretrain
```

## Evaluation signals

- vae_pretrained.pt / best_model.pt loads without error via get_pretrain_encoder and encodes all splits
- Latent UMAP colored by disease and age labels shows visibly clearer separation than the same UMAP computed on the raw (unembedded) standardized features, consistent with Fig. 4 / Supplementary Figs 7-8 (expert visual review)
- Validation loss (monitored for early stopping) decreases relative to a randomly initialized baseline over up to 200 max_epochs with the KL weight annealed from 0 to the target kl_weight
- Latent arrays Z_Pretrain_*.csv have shape [n_samples, latent_size] matching the exported encoder's latent_size setting (e.g. 460)

## Limitations

- KL annealing is limited to a single kl_weight target; no batch covariates or domain-invariant regularization are incorporated into large-scale deployments
- The pre-trained model is specific to HILIC positive mode; other LC modes and modalities are not covered and multi-modal learning is not yet developed
- Iterative learning upstream of the feature matrix is computationally expensive (up to 2.5 days on a single machine) and parallel processing on multiple compute nodes is not yet implemented
- mzLearn outputs are limited to the ion MS1 level; no de-isotoping or MS/MS identification is performed upstream

## Evidence

- [results] Unsupervised VAE latent clustering by disease and age: [evidence span withheld — anchor status: not_found]
- [readme] Pretraining objective combines reconstruction and KL-divergence: [evidence span withheld — anchor status: external_doc]
- [readme] Standardized 2,736-feature pretraining input: [evidence span withheld — anchor status: external_doc]
- [abstract] Feature matrix dimensions from mzLearn: [evidence span withheld — anchor status: not_found]
