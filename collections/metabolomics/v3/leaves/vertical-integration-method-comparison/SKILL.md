---
name: vertical-integration-method-comparison
description: Use when comparing integration methods (e.g., SpatialMETA vs. totalVI,
  spaMultiVAE, MISO, Seurat v5, Stabmap, SpatialGLUE) on paired ST-SM samples using
  the four metric families and an overall score.
when_to_use_negative:
- Evaluating horizontal (cross-sample) integration of ST-SM data rather than vertical
  integration of paired modalities on the same section
- Input ST/SM data are at single-cell or subcellular resolution — SpatialMETA and
  this benchmark are designed for low-resolution ST and SM data
- No manually annotated ground-truth labels are available, since the scIB biological
  conservation evaluation requires the H&E-derived label key
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3922
tools:
- name: SpatialMETA
  role: CVAE-based method under evaluation (ConditionalVAESTSM) whose joint ST-SM
    embedding is benchmarked against baselines
  repo: https://github.com/WanluLiuLab/SpatialMETA
- name: totalVI
  role: Baseline vertical integration method run on the same ST-SM samples for comparison
- name: spaMultiVAE
  role: Baseline vertical integration method run on the same ST-SM samples for comparison
- name: MISO
  role: Baseline run in both 2-mode and 3-mode configurations on the same samples
- name: Seurat
  role: Baseline vertical integration using Seurat v5 with BNN on the same samples
- name: Stabmap
  role: Baseline vertical integration method run on the same ST-SM samples
- name: SpatialGLUE
  role: Baseline spatial multi-omics integration method run on the same samples
- name: scIB
  role: Metric framework for computing reconstruction accuracy, continuity, marker,
    and biological conservation scores from each joint embedding
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-63915-z
    title: Integrating cross-sample and cross-modal data for spatial transcriptomics
      and metabolomics with SpatialMETA
  source_content_hash: sha256:98d155f5fd00b4c113d230167d981cb76a1a76adf2fb3b240880421a411e8183
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
  - scIB
  - spaMultiVAE
  - totalVI
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3922
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# vertical-integration-method-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark vertical integration of paired spatial transcriptomics (ST) and spatial metabolomics (SM) samples by computing scIB-style reconstruction accuracy, continuity, marker, and biological conservation metrics for SpatialMETA and seven baseline methods, then aggregating them into an overall score ranking.

## When to use

Use when comparing integration methods (e.g., SpatialMETA vs. totalVI, spaMultiVAE, MISO, Seurat v5, Stabmap, SpatialGLUE) on paired ST-SM samples using the four metric families and an overall score.

## When NOT to use

- Evaluating horizontal (cross-sample) integration of ST-SM data rather than vertical integration of paired modalities on the same section
- Input ST/SM data are at single-cell or subcellular resolution — SpatialMETA and this benchmark are designed for low-resolution ST and SM data
- No manually annotated ground-truth labels are available, since the scIB biological conservation evaluation requires the H&E-derived label key

## Inputs

- Eleven vertically integrated ST-SM samples from human ccRCC, GBM, and mouse brain datasets (loadable via spatialmeta.data.load_adata)
- Manually H&E-annotated ground-truth label key for biological conservation evaluation
- Joint embeddings from SpatialMETA ConditionalVAESTSM and 7 baselines (totalVI, spaMultiVAE, MISO 2/3-mode, Seurat v5 BNN, Stabmap, SpatialGLUE)

## Outputs

- Per-method scores for four metric families (reconstruction accuracy, continuity, marker, biological conservation)
- benchmark_overall_scores.csv: one row per method with an overall-score column and rank ordering matching Fig. 2b / Supplementary Fig. 2e-h

## How to apply

Load the eleven vertically integrated ST-SM samples (human ccRCC, GBM, and mouse brain) with spatialmeta.data.load_adata, then train SpatialMETA's ConditionalVAESTSM via spatialmeta.model.ConditionalVAESTSM.fit and extract get_latent_embedding and get_normalized_expression. Run each baseline (totalVI, spaMultiVAE, MISO 2-mode and 3-mode, Seurat v5 with BNN, Stabmap, SpatialGLUE) on the same data to obtain a joint embedding. For every embedding, compute the four metric families — reconstruction accuracy, continuity, marker, and biological conservation — using scIB-style metrics with the manually H&E-annotated label key supplied to scIB. Aggregate the metric families into a single overall score per method and rank, writing benchmark_overall_scores.csv that should reproduce the ordering reported in Fig. 2b and Supplementary Fig. 2e-h, with SpatialMETA ranked first.

## Related tools

- **SpatialMETA** (CVAE-based method under evaluation (ConditionalVAESTSM) whose joint ST-SM embedding is benchmarked against baselines) — https://github.com/WanluLiuLab/SpatialMETA
- **totalVI** (Baseline vertical integration method run on the same ST-SM samples for comparison)
- **spaMultiVAE** (Baseline vertical integration method run on the same ST-SM samples for comparison)
- **MISO** (Baseline run in both 2-mode and 3-mode configurations on the same samples)
- **Seurat** (Baseline vertical integration using Seurat v5 with BNN on the same samples)
- **Stabmap** (Baseline vertical integration method run on the same ST-SM samples)
- **SpatialGLUE** (Baseline spatial multi-omics integration method run on the same samples)
- **scIB** (Metric framework for computing reconstruction accuracy, continuity, marker, and biological conservation scores from each joint embedding)

## Examples

```
import spatialmeta; adata = spatialmeta.data.load_adata(...); model = spatialmeta.model.ConditionalVAESTSM(adata); model.fit(); emb = model.get_latent_embedding(adata); expr = model.get_normalized_expression(adata)
```

## Evaluation signals

- benchmark_overall_scores.csv contains exactly one row per method: SpatialMETA plus the 7 baselines (8 rows total)
- The table includes an overall-score column aggregating the four metric families
- SpatialMETA is ranked first in overall score, consistent with the reported finding
- The ranking order matches Fig. 2b and Supplementary Fig. 2e-h
- Ground-truth labels for biological conservation were supplied via the label key parameter in scIB and derived from manual H&E histological annotation

## Limitations

- Ground-truth labels for biology conservation were manually annotated from H&E histological features, not an independent gold standard
- Detailed benchmark metric values are only provided in source data files rather than in the main text
- The comparison uses previously published non-spatial single-cell or spatial multi-omics tools as baselines, which were not designed for continuous SM intensity data

## Evidence

- [discussion] The ground-truth labels used for evaluation (specified via the "label key" parameter in scIB) were manually annotated based on histological features identified through H&E staining: [evidence span withheld — anchor status: not_found]
