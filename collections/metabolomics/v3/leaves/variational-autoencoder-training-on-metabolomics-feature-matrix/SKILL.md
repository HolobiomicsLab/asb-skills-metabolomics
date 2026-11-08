---
name: variational-autoencoder-training-on-metabolomics-feature-matrix
description: Use when you have a standardized LC/MS1 feature table of blood-sample
  metabolite intensities (e.g. [n_samples, 2736] floats) and want unsupervised latent
  embeddings that organize samples by disease or age without supervision.
when_to_use_negative:
- 'Your goal is raw LC/MS signal detection: use mzLearn to generate the feature matrix
  first — it works from mzML files and its outputs are ion MS1 level only, without
  de-isotoping or MS/MS identification'
- 'Your data are not HILIC positive-mode metabolomics feature vectors: the pre-trained
  model was developed specifically for HILIC positive mode, and multi-modal integration
  of other LC types/modes is not yet developed'
- You already have a fine-tuned task model and need supervised predictions (binary/multiclass/survival)
  — use the fine-tuning workflow with pretrained_dir rather than pretraining from
  scratch
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3474
tools:
- name: mzLearn
  role: Produces the upstream MS1 ion-feature matrix (2,736 robust metabolite signals
    across 20,548 blood samples) that the VAE is pre-trained on
  repo: http://mzlearn.com/
- name: VAE
  role: Unsupervised generative model trained with reconstruction + KL-divergence
    losses on the standardized feature matrix; its encoder yields latent representations
    for embedding and fine-tuning
  repo: https://github.com/ReviveMed/mzEmbed
- name: mzEmbed (mz_embed)
  role: Python package implementing Optuna-driven VAE pretraining, encoder export,
    and latent-embedding extraction
  repo: https://github.com/ReviveMed/mzEmbed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2025.01.26.634927
    title: mzLearn, a data-driven LC/MS signal detection algorithm, enables pre-trained
      generative models for enhanced patient stratification
  source_content_hash: sha256:97568fb7b9336333c78419c5d18319c292fc63ee247f1a3f2634740f24200612
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
  - mzEmbed (mz_embed)
  - mzLearn
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# variational-autoencoder-training-on-metabolomics-feature-matrix

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train an unsupervised variational autoencoder (VAE) on the mzLearn MS1 ion-feature matrix (2,736 robust metabolite signals across 20,548 blood samples from 22 public studies) to learn latent metabolite representations that capture demographic and disease structure, yielding a reusable pre-trained checkpoint (vae_pretrained.pt / best_model.pt) for downstream fine-tuning.

## When to use

Use when you have a standardized LC/MS1 feature table of blood-sample metabolite intensities (e.g. [n_samples, 2736] floats) and want unsupervised latent embeddings that organize samples by disease or age without supervision.

## When NOT to use

- Your goal is raw LC/MS signal detection: use mzLearn to generate the feature matrix first — it works from mzML files and its outputs are ion MS1 level only, without de-isotoping or MS/MS identification
- Your data are not HILIC positive-mode metabolomics feature vectors: the pre-trained model was developed specifically for HILIC positive mode, and multi-modal integration of other LC types/modes is not yet developed
- You already have a fine-tuned task model and need supervised predictions (binary/multiclass/survival) — use the fine-tuning workflow with pretrained_dir rather than pretraining from scratch

## Inputs

- mzLearn MS1 ion-feature matrix: CSV of floats, shape [n_samples, 2736] (first column is row index), z-scored
- Split files X_Pretrain_Discovery_Train.csv, X_Pretrain_Discovery_Val.csv, X_Pretrain_Test.csv (optional y_* labels for disease/age)
- Hyperparameters: latent_size, n_layers {2,3,4}, kl_weight, dropout, learning_rate, trials, max_epochs, seed

## Outputs

- Pre-trained VAE checkpoint (vae_pretrained.pt / best_model.pt) with config.json, study.html, and logs
- Exported encoder (encoder.pt) via get_pretrain_encoder
- Latent embeddings Z_Pretrain_{All,Discovery_Train,Discovery_Val,Test}.csv of shape [n, latent_size]
- Latent plots (e.g. UMAP colored by disease and age labels) in output_dir

## How to apply

Load the mzLearn MS1 ion-feature matrix and z-score (standardize) the signal intensities before training. Split the data into X_Pretrain_Discovery_Train, X_Pretrain_Discovery_Val, and X_Pretrain_Test CSVs, then run Optuna-driven unsupervised VAE pretraining (hyperparameters include latent_size ~460, n_layers in {2,3,4}, KL annealed from 0 to kl_weight, dropout 0–0.5, learning_rate, up to max_epochs with early stopping on validation loss). Mini-batch SGD optimizes reconstruction plus KL-divergence loss; save the best checkpoint (best_model.pt, i.e. vae_pretrained.pt). Encode all samples through the encoder and export latent embeddings Z_Pretrain_*.csv for UMAP visualization and downstream fine-tuning.

## Related tools

- **mzLearn** (Produces the upstream MS1 ion-feature matrix (2,736 robust metabolite signals across 20,548 blood samples) that the VAE is pre-trained on) — http://mzlearn.com/
- **VAE** (Unsupervised generative model trained with reconstruction + KL-divergence losses on the standardized feature matrix; its encoder yields latent representations for embedding and fine-tuning) — https://github.com/ReviveMed/mzEmbed
- **mzEmbed (mz_embed)** (Python package implementing Optuna-driven VAE pretraining, encoder export, and latent-embedding extraction) — https://github.com/ReviveMed/mzEmbed

## Examples

```
python -m mz_embed.pretrain.run_pretrain_VAE_main --data_location /path/to/pretrain_csvs --latent_size 460 --n_layers 3 --trials 40 --max_epochs 200 --output_dir /path/to/out_pretrain
```

## Evaluation signals

- vae_pretrained.pt / best_model.pt loads successfully and matches the saved config.json (latent_size 460, n_layers within {2,3,4})
- Latent UMAP colored by disease and age labels shows visibly clearer separation than the same UMAP computed on raw standard-scaled input intensities (which show mixing), consistent with Fig. 4 / Supplementary Figs 7-8
- Latent arrays Z_Pretrain_*.csv have shape [n_split, latent_size] matching each input split's row count
- Early stopping monitored on validation loss during the Optuna study (study.html) completed without divergent KL/reconstruction losses

## Limitations

- The current pre-trained model is specific to HILIC positive mode; other LC modes and modalities are not covered, and multi-modal learning for integrating various LC types and modes is not yet developed
- Iterative mzLearn processing upstream can take up to 2.5 days on a single machine; parallel processing on multiple compute nodes is not yet implemented
- Batch covariates or domain-invariant regularization are not yet incorporated into large-scale deployments
- mzLearn outputs are limited to the ion MS1 level; no de-isotoping or MS/MS integration into pretraining is implemented

## Evidence

- [results] Without supervision, the pre-trained VAE learned latent representations that cluster samples with similar disease and age characteristics: [evidence span withheld — anchor status: not_found]
- [readme] Run Optuna-driven unsupervised pretraining of the VAE on the standardized 2,736-feature matrix.: [evidence span withheld — anchor status: external_doc]
- [readme] Feature matrices: CSV of floats, shape [n_samples, 2736] (the robust mzLearn peak set).: [evidence span withheld — anchor status: not_found]
