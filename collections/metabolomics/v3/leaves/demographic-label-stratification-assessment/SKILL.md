---
name: demographic-label-stratification-assessment
description: Use when you have a pre-trained VAE over a metabolomics feature matrix
  plus per-sample demographic labels (disease category, age group) and need to visually
  assess whether the learned latent space organizes samples by those labels better
  than the raw features.
when_to_use_negative:
- You already have a supervised classifier or want label-driven separation — this
  skill only assesses unsupervised stratification; the VAE is trained without demographic
  supervision
- The feature matrix is raw/unprocessed or not the 2,736-feature robust mzLearn peak
  set — the pre-trained encoder expects exactly that standardized input shape
- The goal is de-isotoping, annotation, or MS/MS compound identification — mzLearn
  outputs are ion MS1 level and downstream workflows handle those steps separately
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: mzLearn
  role: Produces the 2,736-feature MS1 ion-feature matrix from 22 public studies /
    20,548 blood samples that is embedded here
- name: VAE
  role: Pre-trained generative encoder whose latent representations are projected
    with UMAP and colored by demographic labels
- name: mzEmbed
  role: Codebase exposing run_pretrain_VAE_main and get_pretrain_encoder to train/load
    the VAE and export latent embeddings
  repo: github.com/ReviveMed/mzEmbed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2025.01.26.634927
    title: mzLearn, a data-driven LC/MS signal detection algorithm, enables pre-trained
      generative models for enhanced patient stratification
  source_content_hash: sha256:cf52ac3c453b8832135875322171292d63af417fc1619f5bcd7dc62a7c87299a
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# demographic-label-stratification-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encodes all samples through a pre-trained VAE trained on the mzLearn MS1 ion-feature matrix (2,736 robust metabolite signals across 20,548 blood samples) and compares UMAP visualizations of the latent space versus the raw, standard-scaled input data, colored by disease and age labels, to demonstrate unsupervised demographic stratification.

## When to use

Use when you have a pre-trained VAE over a metabolomics feature matrix plus per-sample demographic labels (disease category, age group) and need to visually assess whether the learned latent space organizes samples by those labels better than the raw features.

## When NOT to use

- You already have a supervised classifier or want label-driven separation — this skill only assesses unsupervised stratification; the VAE is trained without demographic supervision
- The feature matrix is raw/unprocessed or not the 2,736-feature robust mzLearn peak set — the pre-trained encoder expects exactly that standardized input shape
- The goal is de-isotoping, annotation, or MS/MS compound identification — mzLearn outputs are ion MS1 level and downstream workflows handle those steps separately

## Inputs

- mzLearn MS1 ion-feature matrix: float CSV of shape [n_samples, 2736] (e.g. X_Pretrain_All.csv, z-scored intensities)
- Per-sample demographic label table (disease category, age group) aligned to the feature matrix rows
- Pre-trained VAE checkpoint (best_model.pt / vae_pretrained.pt) from mzEmbed pretraining

## Outputs

- Latent embeddings per split (Z_Pretrain_*.csv) produced by get_pretrain_encoder
- UMAP projection of VAE latent vectors colored by disease and age labels
- Comparison UMAP projection of the raw standard-scaled 2,736-feature intensities colored by the same labels

## How to apply

Load the mzLearn MS1 ion-feature matrix (2,736 features, 20,548 blood samples from 22 public studies) together with the sample disease and age demographic labels, and standardize the signal intensities (the README expects z-scored float CSVs of shape [n, 2736]). Encode every sample through the pre-trained VAE encoder — exported via get_pretrain_encoder, which saves encoder.pt and Z_Pretrain_*.csv per split — and embed the resulting latent vectors with UMAP. Produce a parallel UMAP on the raw, standard-scaled input intensities so both projections share the same sample set. Color each UMAP by disease and age labels: the latent UMAP should show visibly clearer separation of samples by these labels than the raw-data UMAP, mirroring Fig. 4 and Supplementary Figs 7–8, where the raw intensities show mixing and the latent space clusters samples with similar disease and age characteristics without any supervised label information. Judge success by expert visual comparison of the two figures, confirming vae_pretrained.pt (or best_model.pt) loads and that the latent embeddings retain the cohort's demographic structure.

## Related tools

- **mzLearn** (Produces the 2,736-feature MS1 ion-feature matrix from 22 public studies / 20,548 blood samples that is embedded here)
- **VAE** (Pre-trained generative encoder whose latent representations are projected with UMAP and colored by demographic labels)
- **mzEmbed** (Codebase exposing run_pretrain_VAE_main and get_pretrain_encoder to train/load the VAE and export latent embeddings) — github.com/ReviveMed/mzEmbed

## Examples

```
from mz_embed.pretrain.get_pretrain_encoder import get_pretrain_encoder; enc, Z, Y = get_pretrain_encoder('/path/to/out_pretrain', '/path/to/pretrain_csvs', '/path/to/encoder_out')
```

## Evaluation signals

- vae_pretrained.pt (best_model.pt) loads successfully via get_pretrain_encoder and the latent array shape is [n_samples, latent_size] (e.g. 460)
- Latent UMAP shows visibly clearer separation by disease and age labels than the raw standard-scaled feature UMAP, consistent with Fig. 4 / Supplementary Figs 7–8
- Encoder and embeddings export correctly: output_dir contains encoder.pt and Z_Pretrain_{All,Discovery_Train,Discovery_Val,Test}.csv
- Latent embeddings reproduce the cohort's demographic structure without using labels during VAE training (unsupervised check)
- Both UMAPs are computed on the identical sample set so the raw-vs-latent comparison is paired

## Limitations

- The current pre-trained model is specific to HILIC positive mode; other LC modes and modalities are not covered
- Assessment is visual/expert review; no quantitative metric is prescribed for stratification quality
- mzLearn outputs are limited to the ion MS1 level; no de-isotoping, annotation, or MS/MS identification is performed here

## Evidence

- [results] Unsupervised VAE latent UMAP separates disease and age better than raw data: [evidence span withheld — anchor status: not_found]
- [readme] Encoder export API signature and outputs: [evidence span withheld — anchor status: external_doc]
- [readme] Expected input shape and z-scoring for pretrain matrices: [evidence span withheld — anchor status: not_found]
