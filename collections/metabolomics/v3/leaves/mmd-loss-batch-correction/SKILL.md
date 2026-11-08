---
name: mmd-loss-batch-correction
description: Use when integrating joint ST/SM spot-level AnnData across multiple samples
  (e.g. human ccRCC, GBM, mouse brain) and batch effects from different samples must
  be corrected within a CVAE latent space rather than by feature-level scaling.
when_to_use_negative:
- Data are single-sample with no cross-sample batch structure — there is no across-sample
  distribution to align with MMD
- ST and SM data are high-resolution (single-cell or subcellular); SpatialMETA is
  designed for low-resolution ST and SM data
- More than two modalities must be integrated; SpatialMETA currently only supports
  integration of two modalities
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SpatialMETA
  role: Provides ConditionalVAESTSM whose fit() combines ZINB (ST), Gaussian (SM)
    reconstruction losses with an across-sample MMD loss for batch correction
  repo: https://github.com/WanluLiuLab/SpatialMETA
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-63915-z
    title: Integrating cross-sample and cross-modal data for spatial transcriptomics
      and metabolomics with SpatialMETA
  source_content_hash: sha256:483cf726e70a1c12d46c9af85ff3d66adc5d1c7e25124af14d43f316c664837e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/WanluLiuLab/SpatialMETA
  repo_url: https://github.com/WanluLiuLab/SpatialMETA
  tools:
  - SpatialMETA
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# mmd-loss-batch-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train the SpatialMETA ConditionalVAESTSM with a maximum mean discrepancy (MMD) loss across samples, alongside ZINB (ST) and Gaussian (SM) reconstruction losses, so the shared latent embedding corrects cross-sample batch effects while preserving biological structure.

## When to use

Use when integrating joint ST/SM spot-level AnnData across multiple samples (e.g. human ccRCC, GBM, mouse brain) and batch effects from different samples must be corrected within a CVAE latent space rather than by feature-level scaling.

## When NOT to use

- Data are single-sample with no cross-sample batch structure — there is no across-sample distribution to align with MMD
- ST and SM data are high-resolution (single-cell or subcellular); SpatialMETA is designed for low-resolution ST and SM data
- More than two modalities must be integrated; SpatialMETA currently only supports integration of two modalities

## Inputs

- Preprocessed joint ST/SM AnnData for multiple samples (e.g. human ccRCC) with a sample/batch condition key in obs
- Feature-filtered ST count matrix and SM intensity matrix (batch-biased SVGs/SVMs removed)
- Batch/sample label key per spot

## Outputs

- Trained ConditionalVAESTSM model checkpoint
- Per-spot shared latent embedding from get_latent_embedding()
- Batch-corrected joint latent space usable for downstream clustering and modality-contribution analysis

## How to apply

Instantiate spatialmeta.model.ConditionalVAESTSM with batch-invariant ST and SM encoders whose posteriors are concatenated into a joint latent embedding, conditioning decoding on the sample/batch key stored in obs. Before training, remove batch-biased SVGs/SVMs — features with fold change greater than min_logfc (default 3), expressed in at least min_frac (default 0.8) of spots, shared across min_samples (default 2) — since batch-biased SVGs and SVMs are removed to facilitate better vertical integration. Call ConditionalVAESTSM.fit(), which optimizes a ZINB reconstruction loss for ST count data and a Gaussian reconstruction loss for continuous SM intensity data together with an MMD loss across samples, so the model simultaneously integrates batch information and the joint embedding. The MMD term aligns per-sample latent distributions without needing matched features across batches, which is critical because SM data are continuous mass spectrometry intensities rather than sequencing counts. After training, extract per-spot embeddings with get_latent_embedding() and reload the saved checkpoint to confirm the correction is stable.

## Related tools

- **SpatialMETA** (Provides ConditionalVAESTSM whose fit() combines ZINB (ST), Gaussian (SM) reconstruction losses with an across-sample MMD loss for batch correction) — https://github.com/WanluLiuLab/SpatialMETA

## Examples

```
from spatialmeta.model import ConditionalVAESTSM; model = ConditionalVAESTSM(adata); model.fit(adata); emb = model.get_latent_embedding()
```

## Evaluation signals

- Checkpoint reloads and get_latent_embedding() returns a vector for every spot
- Batch-biased SVGs/SVMs (fold change > min_logfc=3, present in >= 0.8 of spots across >= 2 samples) were removed before training
- Training minimizes ZINB loss on ST counts and Gaussian loss on SM intensities jointly with the MMD term across samples
- Downstream spatial clusters are unified across samples (consistent interactions), indicating batch effects are corrected while biology is conserved
- Per-spot modality contributions (angular similarity between ST- and SM-derived embeddings) are complete and finite

## Limitations

- Horizontal integration was demonstrated on a limited number of samples from a single study, so MMD correction is not broadly validated across studies
- Ground-truth labels for biology conservation evaluation were manually annotated from H&E histological features, not an independent gold standard
- Only two modalities are supported; the batch-correction framework does not extend to >2 modality integration in the current implementation

## Evidence

- [methods] MMD loss optimizes batch correction while preserving biology: [evidence span withheld — anchor status: not_found]
- [discussion] Remove batch-biased features before integration: [evidence span withheld — anchor status: not_found]
