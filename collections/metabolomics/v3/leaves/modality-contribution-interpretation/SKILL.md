---
name: modality-contribution-interpretation
description: Use when you have a trained SpatialMETA ConditionalVAESTSM on joint ST-SM
  spots and need to quantify which modality (gene expression or metabolite intensity)
  dominates each spot's shared latent embedding.
when_to_use_negative:
- Data are single-cell or subcellular resolution ST/SM — SpatialMETA is designed for
  low-resolution ST and SM data only
- More than two modalities are involved — SpatialMETA currently supports integration
  of only two modalities
- No trained ConditionalVAESTSM model or joint latent embedding exists yet — first
  perform CVAE training with ZINB/Gaussian losses and MMD correction
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SpatialMETA
  role: Framework whose ConditionalVAESTSM.get_modality_contribution() computes the
    angular similarity between ST- and SM-derived embeddings; get_latent_embedding()
    supplies the shared latent vectors
  repo: https://github.com/WanluLiuLab/SpatialMETA
provenance:
  source_task_ids:
  - task_006
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-63915-z
    title: Integrating cross-sample and cross-modal data for spatial transcriptomics
      and metabolomics with SpatialMETA
  source_content_hash: sha256:5dd9ef67cffebb8ec02a13ed18e3d7814c0200458b352c02dcc4f3c14a155697
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
  - ConditionalVAESTSM.get_modality_contribution
  - DestVI
  - SpatialMETA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# modality-contribution-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes and interprets per-spot modality contribution from a trained SpatialMETA ConditionalVAESTSM by measuring the angular similarity between the ST-derived and SM-derived latent embeddings, quantifying how much spatial transcriptomics versus spatial metabolomics drives each spot's joint representation.

## When to use

Use when you have a trained SpatialMETA ConditionalVAESTSM on joint ST-SM spots and need to quantify which modality (gene expression or metabolite intensity) dominates each spot's shared latent embedding.

## When NOT to use

- Data are single-cell or subcellular resolution ST/SM — SpatialMETA is designed for low-resolution ST and SM data only
- More than two modalities are involved — SpatialMETA currently supports integration of only two modalities
- No trained ConditionalVAESTSM model or joint latent embedding exists yet — first perform CVAE training with ZINB/Gaussian losses and MMD correction

## Inputs

- Trained SpatialMETA ConditionalVAESTSM model checkpoint (reloadable)
- Joint ST-SM AnnData produced by spatialmeta.pp.joint_adata_sm_st with sample/batch condition key in obs

## Outputs

- Per-spot modality contribution vector (angular similarity between ST- and SM-derived embeddings)
- modality_contribution.csv with one finite value per spot
- Spatial plots of modality contribution (e.g., PNG via spatialmeta.pl.plot_spot_sm_st)

## How to apply

After training ConditionalVAESTSM with ZINB reconstruction loss for ST counts and Gaussian reconstruction loss for SM mass-spectrometry intensities plus MMD batch correction across samples, extract the shared latent embedding via get_latent_embedding(). Call ConditionalVAESTSM.get_modality_contribution() to compute, for every spot, the angular similarity between the ST-encoder and SM-encoder posteriors in the joint latent space — this is interpretable because both posteriors are concatenated into one embedding. Validate that a contribution value is returned for every spot and that all values are finite. Write the per-spot vector to modality_contribution.csv and optionally overlay it spatially with spatialmeta.pl.plot_spot_sm_st to identify regions where metabolomics (e.g., lipid-metabolite hotspots in ccRCC) or transcriptomics dominates.

## Related tools

- **SpatialMETA** (Framework whose ConditionalVAESTSM.get_modality_contribution() computes the angular similarity between ST- and SM-derived embeddings; get_latent_embedding() supplies the shared latent vectors) — https://github.com/WanluLiuLab/SpatialMETA

## Examples

```
import spatialmeta; vae = spatialmeta.model.ConditionalVAESTSM.load('checkpoint.pt'); contrib = vae.get_modality_contribution(); contrib.to_csv('modality_contribution.csv')
```

## Evaluation signals

- get_latent_embedding() returns one latent vector for every spot in the joint AnnData
- The modality contribution vector has one finite (non-NaN, non-inf) value per spot
- The saved model checkpoint reloads without error before calling get_modality_contribution()
- modality_contribution.csv values lie in the valid range of an angular-similarity measure
- Spatially plotted contributions co-localize with expected biology (e.g., higher SM contribution in lipid-metabolite-rich Imm_4 regions of ccRCC Y7_T)

## Limitations

- Interpretation assumes only two modalities (ST and SM); more than two modalities is not supported
- Contributions reflect angular similarity of encoder posteriors, not direct experimental perturbation evidence; biological significance requires further validation
- Applied only to low-resolution spot-level ST/SM data after SM-to-ST alignment and KNN reassignment; misalignment propagates into contributions

## Evidence

- [results] Tool function for modality contribution: [evidence span withheld — anchor status: not_found]
- [readme] README confirming the relevant model class: [evidence span withheld — anchor status: external_doc]
