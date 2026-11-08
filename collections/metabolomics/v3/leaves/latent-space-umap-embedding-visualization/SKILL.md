---
name: latent-space-umap-embedding-visualization
description: Use when you have a pre-trained metabolomics VAE checkpoint and an mzLearn
  MS1 feature matrix, and need to visually verify that learned latent representations
  stratify samples by disease or age labels versus raw intensities.
when_to_use_negative:
- The cohort was acquired in a mode other than HILIC positive LC — the released pre-trained
  encoder is specific to HILIC positive mode, so latents are not transferable.
- The goal is supervised prediction (classification or survival, e.g. IMDC AUC/C-index)
  rather than unsupervised structure inspection — use the fine-tuning and test-evaluation
  workflow instead.
- The feature matrix is not the mzLearn 2,736-feature peak set (e.g. raw mzML signals
  or an XCMS/ASARI feature table), since encoder input dimensionality and feature
  semantics will not match.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3315
tools:
- name: VAE
  role: Pre-trained variational autoencoder whose encoder outputs the latent vectors
    that are UMAP-embedded and label-colored
  repo: github.com/ReviveMed/mzEmbed
- name: mzLearn
  role: Produces the MS1 ion-feature matrix (2,736 robust signals across 20,548 blood
    samples) that is encoded into latent space
- name: UMAP
  role: Nonlinear embedding of VAE latent vectors (and, for contrast, the raw feature
    matrix) for 2-D label-colored visualization
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2025.01.26.634927
    title: mzLearn, a data-driven LC/MS signal detection algorithm, enables pre-trained
      generative models for enhanced patient stratification
  source_content_hash: sha256:873265c98575f3e036dfc07618b11b262b87fa52624b71de5c6770741f661563
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ReviveMed/mzEmbed
  tools:
  - UMAP
  - VAE
  - mzLearn
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# latent-space-umap-embedding-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encodes samples through the pre-trained metabolomics VAE encoder, embeds the latent vectors with UMAP, and colors the embedding by demographic labels (disease, age) to visually assess whether unsupervised representations organize samples better than the raw 2,736-feature mzLearn matrix.

## When to use

Use when you have a pre-trained metabolomics VAE checkpoint and an mzLearn MS1 feature matrix, and need to visually verify that learned latent representations stratify samples by disease or age labels versus raw intensities.

## When NOT to use

- The cohort was acquired in a mode other than HILIC positive LC — the released pre-trained encoder is specific to HILIC positive mode, so latents are not transferable.
- The goal is supervised prediction (classification or survival, e.g. IMDC AUC/C-index) rather than unsupervised structure inspection — use the fine-tuning and test-evaluation workflow instead.
- The feature matrix is not the mzLearn 2,736-feature peak set (e.g. raw mzML signals or an XCMS/ASARI feature table), since encoder input dimensionality and feature semantics will not match.

## Inputs

- Pre-trained VAE checkpoint (vae_pretrained.pt / best_model.pt with encoder.pt exported)
- mzLearn MS1 ion-feature matrix as CSV of floats, shape [n_samples, 2736], z-scored (X_Pretrain_*.csv with index_col=0)
- Sample demographic label table (disease category, age group) indexed to match samples

## Outputs

- Latent embedding arrays Z_Pretrain_*.csv of shape [n, latent_size] (e.g. latent_size=460)
- UMAP scatter plots of latent space colored by disease and age labels
- Comparison UMAP of the raw standard-scaled 2,736-feature input data for contrast

## How to apply

Load the pre-trained VAE (e.g. vae_pretrained.pt / best_model.pt, latent_size=460, n_layers=3) and encode all samples from the standardized 2,736-feature matrix (shape [n_samples, 2736], z-scored floats). Export the encoder and latent arrays Z via get_pretrain_encoder, then embed the latent vectors with UMAP and color the scatter by disease and age labels. Compute the identical UMAP on the raw (unembedded) standard-scaled feature data and compare: the latent embedding should show visibly clearer separation by disease/age while the raw-data UMAP shows mixed samples, matching Fig. 4 / Supplementary Figs 7-8. The rationale is that the VAE's reconstruction-plus-KL objective on the 20,548-sample blood cohort yields representations that reflect demographic structure without supervision.

## Related tools

- **VAE** (Pre-trained variational autoencoder whose encoder outputs the latent vectors that are UMAP-embedded and label-colored) — github.com/ReviveMed/mzEmbed
- **mzLearn** (Produces the MS1 ion-feature matrix (2,736 robust signals across 20,548 blood samples) that is encoded into latent space)
- **UMAP** (Nonlinear embedding of VAE latent vectors (and, for contrast, the raw feature matrix) for 2-D label-colored visualization)

## Examples

```
from mz_embed.pretrain.get_pretrain_encoder import get_pretrain_encoder; enc, Z, Y = get_pretrain_encoder("/path/to/out_pretrain", "/path/to/pretrain_csvs", "/path/to/encoder_out")
```

## Evaluation signals

- vae_pretrained.pt / best_model.pt loads and the encoder exports without error.
- Latent arrays Z_Pretrain_* have shape [n_samples, 460] consistent with the configured latent_size.
- Expert visual review: latent UMAP shows visibly clearer separation by disease/age labels than the UMAP of raw standard-scaled intensities, consistent with Fig. 4 / Supplementary Figs 7-8.
- get_pretrain_encoder writes encoder.pt and Z_Pretrain_*.csv to the output directory for each split (All, Discovery_Train, Discovery_Val, Test).
- Latent plots appear among the artifacts written to output_dir during pretraining.

## Limitations

- Assessment of label stratification in the UMAP is expert visual review, not a quantitative clustering metric.
- The pre-trained model was developed specifically for HILIC positive mode; other LC modes and modalities are not covered.
- mzLearn outputs are limited to the ion MS1 level — no de-isotoping or MS/MS identification is integrated into the embeddings.
- Batch covariates or domain-invariant regularization are not yet incorporated, which can affect cross-study structure in the embedding.

## Evidence

- [readme] get_pretrain_encoder exports encoder and latents: [evidence span withheld — anchor status: external_doc]
- [readme] README on VAE embeddings capturing demographics: [evidence span withheld — anchor status: external_doc]
