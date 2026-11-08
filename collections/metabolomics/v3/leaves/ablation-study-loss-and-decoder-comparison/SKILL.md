---
name: ablation-study-loss-and-decoder-comparison
description: Use when benchmarking a CVAE-based ST-SM integration model and you must
  justify design choices (loss function, normalization, decoder sharing) with quantitative
  ablation evidence.
when_to_use_negative:
- Input ST/SM data are at single-cell or subcellular resolution — SpatialMETA is designed
  for low-resolution ST and SM data only
- You need to integrate more than two modalities — SpatialMETA currently supports
  only two modalities (ST and SM)
- The goal is horizontal (cross-sample) integration benchmarking rather than within-sample
  vertical integration design ablation
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3912
tools:
- name: SpatialMETA
  role: Provides ConditionalVAESTSM with fit, get_normalized_expression, get_latent_embedding,
    and get_modality_contribution functions used to train and score each ablation
    configuration
  repo: https://github.com/WanluLiuLab/SpatialMETA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-63915-z
    title: Integrating cross-sample and cross-modal data for spatial transcriptomics
      and metabolomics with SpatialMETA
  source_content_hash: sha256:364f14399389a34a84016330bba0789ccb26b27a50186dd017f7ab4107c0f9ca
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3912
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# ablation-study-loss-and-decoder-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Systematically ablates SpatialMETA's ConditionalVAESTSM design components — SM loss function (Gaussian vs MSE), SM normalization, and shared vs independent decoders — to quantify how each affects vertical integration of ST and SM data in the ccRCC case.

## When to use

Use when benchmarking a CVAE-based ST-SM integration model and you must justify design choices (loss function, normalization, decoder sharing) with quantitative ablation evidence.

## When NOT to use

- Input ST/SM data are at single-cell or subcellular resolution — SpatialMETA is designed for low-resolution ST and SM data only
- You need to integrate more than two modalities — SpatialMETA currently supports only two modalities (ST and SM)
- The goal is horizontal (cross-sample) integration benchmarking rather than within-sample vertical integration design ablation

## Inputs

- Preprocessed joint ST-SM AnnData object built with spatialmeta.pp.joint_adata_sm_st (ccRCC vertical-integration case)
- Manually annotated spatial cluster labels (ground truth for biological conservation scoring)

## Outputs

- ablation_metrics.csv — configuration-by-metric table for the loss/normalization/decoder grid
- Per-configuration reconstructed ST and SM expression/intensity profiles from get_normalized_expression
- Per-configuration fused latent embeddings from get_latent_embedding
- Per-configuration modality contribution values from get_modality_contribution

## How to apply

Build the preprocessed joint ST-SM AnnData with spatialmeta.pp.joint_adata_sm_st for the ccRCC vertical-integration case, then train ConditionalVAESTSM via ConditionalVAESTSM.fit for every cell of the ablation grid: Gaussian vs MSE reconstruction loss on the SM modality, SM normalization on vs off, and shared vs independent decoders. For each configuration, reconstruct expression/intensity profiles with ConditionalVAESTSM.get_normalized_expression and extract the fused embedding with ConditionalVAESTSM.get_latent_embedding. Score each configuration on reconstruction error of ST and SM feature distributions, spatial continuity of the latent embedding, marker feature recovery, and biological conservation against the manually annotated spatial cluster labels (ground truth from H&E-based annotation). Compute modality contribution per configuration via angular similarity between embeddings using ConditionalVAESTSM.get_modality_contribution. The expected conclusion is that Gaussian loss and SM normalization are critical for effective vertical integration and the shared decoder improves modality fusion; write all metric values into ablation_metrics.csv.

## Related tools

- **SpatialMETA** (Provides ConditionalVAESTSM with fit, get_normalized_expression, get_latent_embedding, and get_modality_contribution functions used to train and score each ablation configuration) — https://github.com/WanluLiuLab/SpatialMETA

## Examples

```
model = ConditionalVAESTSM(adata, loss='gaussian', normalize_sm=True, shared_decoder=True); ConditionalVAESTSM.fit(model); emb = ConditionalVAESTSM.get_latent_embedding(model)
```

## Evaluation signals

- Gaussian-loss and SM-normalization configurations should outperform MSE-loss and unnormalized configurations on reconstruction error of ST and SM feature distributions
- Shared-decoder configurations should show higher modality fusion than independent-decoder ones, per the article's finding
- Latent embedding should exhibit spatial continuity consistent with tissue structure in the ccRCC section
- Biological conservation score against manually annotated spatial cluster labels should not degrade when a component is added
- ablation_metrics.csv should contain one complete row (all metrics) for each of the 8 grid configurations

## Limitations

- Ground-truth labels for biological conservation evaluation were manually annotated from H&E histological features, not an independent gold standard
- Detailed benchmark metric values are only provided in source data files rather than in the main text
- SpatialMETA is designed for low-resolution ST and SM data and does not support single-cell or subcellular resolution

## Evidence

- [results] The ablation benchmarking showed that the Gaussian loss function and SM data normalization are critical for effective vertical integration, and the shared decoder architecture improved modality fusion over independent decoders.: [evidence span withheld — anchor status: not_found]
- [abstract] SpatialMETA employs tailored decoders and loss functions.: "SpatialMETA employs tailored decoders and loss functions to enhance modality fusion, batch effect correction and biological conservation"
- [discussion] Modality contribution is interpretable via angular similarity.: "the model design enables the calculation of the angular similarity between embeddings, improving the interpretability of modality contributions"
- [readme] README confirms ConditionalVAESTSM updates.: "Update `spatialmeta.model.ConditionalVAESTSM`"
