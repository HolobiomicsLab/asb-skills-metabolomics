---
name: multi-omics-integration-baseline-comparison
description: Use when a new multi-modal integration embedding (e.g., fused spatial
  transcriptomics + spatial metabolomics latent space) must be fairly ranked against
  published integration methods on the same AnnData inputs.
when_to_use_negative:
- Data are single-cell or subcellular resolution ST/SM — SpatialMETA is designed for
  low-resolution ST and SM data
- More than two modalities must be integrated — SpatialMETA currently only supports
  integration of two modalities
- An independent gold-standard label set is required — the article's labels were manually
  annotated from H&E histological features, not an external ground truth
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3934
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3170
tools:
- name: SpatialMETA
  role: CVAE-based method under evaluation; ConditionalVAESTSM learns a shared latent
    space with tailored decoders and loss functions for modality fusion, batch effect
    correction and biological conservation
  repo: https://github.com/WanluLiuLab/SpatialMETA
- name: Seurat
  role: Vertical integration baseline using RPCA and CCA on identical joint ST-SM
    inputs
- name: totalVI
  role: Vertical integration baseline run with default recommended settings
- name: Stabmap
  role: Vertical integration baseline run with default recommended settings
- name: scVI
  role: Horizontal integration baseline across the five ccRCC samples
- name: scANVI
  role: Horizontal integration baseline across the five ccRCC samples
- name: scPoli
  role: Horizontal integration baseline across the five ccRCC samples
- name: spaVAE
  role: Spatial horizontal integration baseline across the five ccRCC samples
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-63915-z
    title: Integrating cross-sample and cross-modal data for spatial transcriptomics
      and metabolomics with SpatialMETA
  source_content_hash: sha256:80b7e8519e516d35778843357f23d93cb881b33f301134abe33a0b66b19d1c15
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
  - Seurat
  - SpatialMETA
  - Stabmap
  - scANVI
  - scPoli
  - scVI
  - spaVAE
  - totalVI
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3934
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# multi-omics-integration-baseline-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a novel joint ST-SM integration model (SpatialMETA's ConditionalVAESTSM) against vertical (Seurat RPCA/CCA, totalVI, Stabmap) and horizontal (scVI, scANVI, scPoli, spaVAE, PCA) baselines on identical inputs, scoring batch correction vs biological conservation with scIB.

## When to use

Use when a new multi-modal integration embedding (e.g., fused spatial transcriptomics + spatial metabolomics latent space) must be fairly ranked against published integration methods on the same AnnData inputs.

## When NOT to use

- Data are single-cell or subcellular resolution ST/SM — SpatialMETA is designed for low-resolution ST and SM data
- More than two modalities must be integrated — SpatialMETA currently only supports integration of two modalities
- An independent gold-standard label set is required — the article's labels were manually annotated from H&E histological features, not an external ground truth

## Inputs

- Processed joint ST-SM AnnData objects for multiple samples (batch-biased features removed)
- Latent embeddings from SpatialMETA ConditionalVAESTSM (get_latent_embedding)
- Baseline embeddings from Seurat RPCA/CCA, totalVI, Stabmap, scVI, scANVI, scPoli, spaVAE, PCA on the same inputs
- Manually annotated histology-derived label key (H&E-based) for scIB evaluation

## Outputs

- scIB metric values per method (Batch ASW, iLISI, PCR batch)
- SpatialMETA cross-modality, cross-sample, and overall integration scores
- Modality contribution via angular similarity between embeddings
- Ranked method comparison table (e.g., fig4d_integration_benchmark.csv)

## How to apply

Load the processed joint ST-SM AnnData objects (e.g., five ccRCC IM2 samples) produced by alignment and KNN reassignment, with batch-biased SVGs/SVMs removed (fold change > min_logfc default 3, expression >= min_frac default 0.8 across >= 2 samples). Train SpatialMETA's ConditionalVAESTSM to learn a shared latent space, extract embeddings via get_latent_embedding, then run each baseline (Seurat RPCA/CCA, totalVI, Stabmap for vertical; scVI, scANVI, scPoli, spaVAE, PCA for horizontal) with default recommended settings on identical inputs. Evaluate every embedding with scIB using manually annotated histology-derived labels as the label key, computing Batch ASW, iLISI and PCR batch, plus cross-modality, cross-sample and overall integration scores and modality contribution via angular similarity between embeddings. Aggregate metrics, rank methods by overall performance, and write the comparison table (e.g., fig4d_integration_benchmark.csv); the new method should rank at or near the top overall.

## Related tools

- **SpatialMETA** (CVAE-based method under evaluation; ConditionalVAESTSM learns a shared latent space with tailored decoders and loss functions for modality fusion, batch effect correction and biological conservation) — https://github.com/WanluLiuLab/SpatialMETA
- **Seurat** (Vertical integration baseline using RPCA and CCA on identical joint ST-SM inputs)
- **totalVI** (Vertical integration baseline run with default recommended settings)
- **Stabmap** (Vertical integration baseline run with default recommended settings)
- **scVI** (Horizontal integration baseline across the five ccRCC samples)
- **scANVI** (Horizontal integration baseline across the five ccRCC samples)
- **scPoli** (Horizontal integration baseline across the five ccRCC samples)
- **spaVAE** (Spatial horizontal integration baseline across the five ccRCC samples)

## Examples

```
git clone git@github.com:WanluLiuLab/SpatialMETA.git; python -c "import spatialmeta; model=spatialmeta.model.ConditionalVAESTSM; model.fit(adata); emb=model.get_latent_embedding(adata)"
```

## Evaluation signals

- The comparison CSV contains metric values for every method (all baselines plus SpatialMETA)
- SpatialMETA ranks at or near the top on overall integration score, consistent with the reported finding
- Batch correction metrics (Batch ASW, iLISI, PCR batch) are computed for every embedding against the same manually annotated histology-derived label key
- Cross-modality and cross-sample integration scores and angular-similarity modality contributions are reported for SpatialMETA
- All methods were run on identical preprocessed inputs (same batch-biased SVG/SVM filtering defaults: min_logfc 3, min_frac 0.8, min_samples 2)

## Limitations

- Horizontal integration was validated only on a limited number of samples from a single study
- Ground-truth labels for biology conservation evaluation were manually annotated from H&E histological features, not an independent gold standard
- Detailed benchmark metric values are only provided in source data files rather than in the main text

## Evidence

- [discussion] Genes or metabolites with a fold change greater than the threshold ("min_logfc", default is 3) were defined as batch-biased SVGs and SVMs: [evidence span withheld — anchor status: not_found]
- [discussion] The ground-truth labels used for evaluation (specified via the "label key" parameter in scIB) were manually annotated based on histological features: [evidence span withheld — anchor status: not_found]
