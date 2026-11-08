---
name: zinb-gaussian-decoder-modeling
description: Use when integrating matched spot-level ST count matrices and continuous
  SM metabolite intensity matrices from one or more samples, and you need modality-faithful
  reconstruction plus batch correction in a shared latent embedding.
when_to_use_negative:
- Input data are at single-cell or subcellular resolution — SpatialMETA is designed
  for low-resolution ST and SM data
- More than two modalities need to be integrated — SpatialMETA currently only supports
  integration of two modalities
- ST data are not raw/processed count data and SM data are not continuous intensities
  (e.g., already-embedded or batch-corrected features), since the ZINB and Gaussian
  decoders assume these distributions
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SpatialMETA
  role: Provides the ConditionalVAESTSM CVAE model with ZINB (ST) and Gaussian (SM)
    decoders, batch-label decoding, MMD loss, and get_latent_embedding / get_modality_contribution
    accessors
  repo: github.com/WanluLiuLab/SpatialMETA
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-63915-z
    title: Integrating cross-sample and cross-modal data for spatial transcriptomics
      and metabolomics with SpatialMETA
  source_content_hash: sha256:dd56edd5b6ac7e0823ddbfd1509ebcde1f20767ad6f558a0eaf2677238b2ba7c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/WanluLiuLab/SpatialMETA
  tools:
  - SpatialMETA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# zinb-gaussian-decoder-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train SpatialMETA's ConditionalVAESTSM so that ST count data is reconstructed with a zero-inflated negative binomial (ZINB) likelihood and SM mass-spectrometry intensities with a Gaussian likelihood, with batch labels fed to the decoders and an MMD loss across samples for batch correction. This modality-specific loss design is what enables simultaneous vertical and horizontal integration of ST and SM within one CVAE framework.

## When to use

Use when integrating matched spot-level ST count matrices and continuous SM metabolite intensity matrices from one or more samples, and you need modality-faithful reconstruction plus batch correction in a shared latent embedding.

## When NOT to use

- Input data are at single-cell or subcellular resolution — SpatialMETA is designed for low-resolution ST and SM data
- More than two modalities need to be integrated — SpatialMETA currently only supports integration of two modalities
- ST data are not raw/processed count data and SM data are not continuous intensities (e.g., already-embedded or batch-corrected features), since the ZINB and Gaussian decoders assume these distributions

## Inputs

- Preprocessed joint ST/SM AnnData object (e.g., human ccRCC sample loaded via spatialmeta.data.load_adata) with a sample/batch condition key in obs
- Batch/sample condition labels for each spot
- Optionally, filtered feature sets (batch-biased SVGs/SVMs removed) and SM data reassigned to ST spots via KNN

## Outputs

- Trained ConditionalVAESTSM model checkpoint
- Per-spot shared latent embedding from get_latent_embedding()
- Per-spot modality contribution vector (angular similarity between ST and SM embeddings), written to modality_contribution.csv

## How to apply

Instantiate spatialmeta.model.ConditionalVAESTSM with batch-invariant ST and SM encoders whose posteriors are concatenated into a joint latent space. Train via ConditionalVAESTSM.fit(), optimizing a ZINB reconstruction loss for ST count data and a Gaussian reconstruction loss for the continuous SM intensity data, while incorporating batch labels during decoding and an MMD loss across samples to remove batch effects and preserve biology. The rationale is that ST is sequencing count data (suited to ZINB) whereas SM is continuous mass-spectrometry intensity (suited to Gaussian), so a single likelihood would mis-model one modality. After fitting, extract the per-spot shared latent embedding with ConditionalVAESTSM.get_latent_embedding(), save the checkpoint, and optionally compute per-spot modality contribution as the angular similarity between the ST- and SM-derived embeddings. Prior filtering (e.g., removal of batch-biased SVGs/SVMs with fold change > min_logfc=3) and SM-to-ST alignment/reassignment should precede model fitting.

## Related tools

- **SpatialMETA** (Provides the ConditionalVAESTSM CVAE model with ZINB (ST) and Gaussian (SM) decoders, batch-label decoding, MMD loss, and get_latent_embedding / get_modality_contribution accessors) — github.com/WanluLiuLab/SpatialMETA

## Examples

```
from spatialmeta.model import ConditionalVAESTSM; model = ConditionalVAESTSM(adata, batch_key='sample'); model.fit(); emb = model.get_latent_embedding(); contrib = model.get_modality_contribution()
```

## Evaluation signals

- The trained checkpoint reloads without error from the saved model file
- get_latent_embedding() returns a latent vector for every spot in the joint AnnData
- Modality contributions (angular similarities) are complete and finite for all spots
- The model demonstrates superior reconstruction capability and fused modality representation, accurately capturing ST and SM feature distributions relative to benchmark tools
- Batch-corrected latent space preserves biology (e.g., manual H&E-based spatial cluster annotations are recoverable via scIB evaluation)

## Limitations

- Horizontal integration was validated only on a limited number of samples from a single study
- Ground-truth labels for biology conservation evaluation were manually annotated from H&E histological features, not an independent gold standard
- Batch-biased features must be removed beforehand (fold change > min_logfc, default 3, expression in >= min_frac default 0.8 of spots, shared across min_samples default 2) or integration quality degrades

## Evidence

- [methods] ZINB likelihood for ST counts, Gaussian for SM intensities: [evidence span withheld — anchor status: not_found]
