---
name: spatial-omics-benchmark-evaluation
description: Use when a joint spatial transcriptomics-metabolomics embedding must
  be quantitatively compared against competing integration tools, requiring metric
  computation and a ranked overall-score table.
when_to_use_negative:
- Input data is at single-cell or subcellular resolution — SpatialMETA is designed
  for low-resolution ST and SM data
- Integration involves more than two modalities — SpatialMETA currently only supports
  integration of two modalities
- No ground-truth labels are available for biological conservation scoring, since
  evaluation depends on manually annotated H&E histological labels
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3517
tools:
- name: SpatialMETA
  role: CVAE-based framework whose ConditionalVAESTSM embeddings are benchmarked against
    baselines
  repo: https://github.com/WanluLiuLab/SpatialMETA
- name: totalVI
  role: Vertical-integration baseline run on the same ST-SM samples for comparison
- name: Seurat
  role: Vertical-integration baseline (Seurat v5 with BNN) run on the same samples
- name: Stabmap
  role: Vertical-integration baseline run on the same samples
- name: spaMultiVAE
  role: Spatial multi-omics baseline for the embedding comparison
- name: MISO
  role: Spatial multi-omics baseline run in both 2-mode and 3-mode configurations
- name: SpatialGLUE
  role: Spatial multi-omics baseline for the embedding comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-63915-z
    title: Integrating cross-sample and cross-modal data for spatial transcriptomics
      and metabolomics with SpatialMETA
  source_content_hash: sha256:92ff73b7df787de947bbfc8dadf47a21fd728c9ee53143867f6e44db89301f7f
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
  - MISO
  - Seurat
  - SpatialGLUE
  - SpatialMETA
  - Stabmap
  - spaMultiVAE
  - totalVI
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# spatial-omics-benchmark-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a joint ST-SM integration embedding (e.g. SpatialMETA) against vertical-integration baselines by computing four scIB-style metric families — reconstruction accuracy, continuity, marker, and biological conservation — on the same eleven ccRCC, GBM, and mouse brain samples, then ranking methods by an aggregated overall score.

## When to use

Use when a joint spatial transcriptomics-metabolomics embedding must be quantitatively compared against competing integration tools, requiring metric computation and a ranked overall-score table.

## When NOT to use

- Input data is at single-cell or subcellular resolution — SpatialMETA is designed for low-resolution ST and SM data
- Integration involves more than two modalities — SpatialMETA currently only supports integration of two modalities
- No ground-truth labels are available for biological conservation scoring, since evaluation depends on manually annotated H&E histological labels

## Inputs

- Eleven vertically integrated ST-SM samples derived from human ccRCC, GBM, and mouse brain datasets
- Joint latent embeddings from SpatialMETA and each baseline (totalVI, spaMultiVAE, MISO 2-mode/3-mode, Seurat v5 BNN, Stabmap, SpatialGLUE)
- Manually H&E-annotated histological labels used as the scIB 'label key' ground truth

## Outputs

- benchmark_overall_scores.csv with one row per evaluated method (SpatialMETA plus 7 baselines) and an overall-score column
- Per-method values for the four metric families: reconstruction accuracy, continuity, marker, and biological conservation

## How to apply

Load the eleven vertically integrated ST-SM samples (human ccRCC, GBM, mouse brain) with spatialmeta.data.load_adata. Train SpatialMETA's ConditionalVAESTSM via spatialmeta.model.ConditionalVAESTSM.fit and extract get_latent_embedding and get_normalized_expression, then run the same data through the baselines: totalVI, spaMultiVAE, MISO (2-mode and 3-mode), Seurat v5 with BNN, Stabmap, and SpatialGLUE. For each method's joint embedding, compute the four metric families (reconstruction accuracy, continuity, marker, biological conservation) using scIB-style metrics, supplying the manually H&E-annotated labels as the scIB 'label key'. Aggregate the four metric families into a single overall score per method, rank the methods, and write the table to benchmark_overall_scores.csv, reproducing the ordering of Fig. 2b and Supplementary Fig. 2e-h.

## Related tools

- **SpatialMETA** (CVAE-based framework whose ConditionalVAESTSM embeddings are benchmarked against baselines) — https://github.com/WanluLiuLab/SpatialMETA
- **totalVI** (Vertical-integration baseline run on the same ST-SM samples for comparison)
- **Seurat** (Vertical-integration baseline (Seurat v5 with BNN) run on the same samples)
- **Stabmap** (Vertical-integration baseline run on the same samples)
- **spaMultiVAE** (Spatial multi-omics baseline for the embedding comparison)
- **MISO** (Spatial multi-omics baseline run in both 2-mode and 3-mode configurations)
- **SpatialGLUE** (Spatial multi-omics baseline for the embedding comparison)

## Examples

```
from spatialmeta.model import ConditionalVAESTSM; model = ConditionalVAESTSM.fit(adata); emb = model.get_latent_embedding(adata)
```

## Evaluation signals

- benchmark_overall_scores.csv contains exactly one row per method: SpatialMETA plus 7 baselines
- SpatialMETA is ranked first by overall score, consistent with the Fig. 2b summary
- All four metric families (reconstruction accuracy, continuity, marker, biological conservation) are computed for every method's joint embedding
- scIB is invoked with the manually H&E-annotated label key rather than an independent gold standard
- The ranking reproduces the ordering reported in Supplementary Fig. 2e-h

## Limitations

- Ground-truth labels for biology conservation were manually annotated from H&E histological features, not an independent gold standard
- Detailed benchmark metric values are provided only in source data files rather than in the main text
- Baseline reconstruction quality is sensitive to the alignment and KNN reassignment of SM data to ST spots prior to integration

## Evidence

- [discussion] The ground-truth labels used for evaluation (specified via the "label key" parameter in scIB) were manually annotated based on histological features identified through H&E staining: [evidence span withheld — anchor status: not_found]
