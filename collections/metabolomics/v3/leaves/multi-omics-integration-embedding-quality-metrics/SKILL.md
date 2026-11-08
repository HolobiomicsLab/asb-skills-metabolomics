---
name: multi-omics-integration-embedding-quality-metrics
description: Use when comparing integration methods (e.g., SpatialMETA, totalVI, SpatialGLUE,
  MISO, Seurat v5 BNN, Stabmap, spaMultiVAE) on paired ST-SM samples and you need
  quantitative, ranked evidence of which method's joint embedding preserves spatial
  biology.
when_to_use_negative:
- When the task is horizontal (cross-sample) integration rather than vertical (cross-modal
  ST-SM) integration — the metric families and ranking here benchmark vertical integration
  performance
- When ground-truth labels are unavailable or the tissue cannot be manually annotated
  from H&E — the biological conservation metrics depend on the manually annotated
  label key, not an independent gold standard
- When data are at single-cell or subcellular resolution — SpatialMETA and this benchmark
  are designed for low-resolution ST and SM data
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3940
- http://edamontology.org/topic_3170
tools:
- name: SpatialMETA
  role: Method under benchmark; ConditionalVAESTSM embeddings are scored alongside
    baselines
  repo: https://github.com/WanluLiuLab/SpatialMETA
- name: totalVI
  role: Baseline integration method whose joint embedding is scored with the same
    metric families
- name: Seurat
  role: Baseline (v5 with BNN) integration method scored in the benchmark
- name: Stabmap
  role: Baseline integration method scored in the benchmark
- name: spaMultiVAE
  role: Baseline spatial multi-omics integration method scored in the benchmark
- name: MISO
  role: Baseline integration method run in 2-mode and 3-mode configurations and scored
    in the benchmark
- name: SpatialGLUE
  role: Baseline spatial multi-omics integration method scored in the benchmark
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-63915-z
    title: Integrating cross-sample and cross-modal data for spatial transcriptomics
      and metabolomics with SpatialMETA
  source_content_hash: sha256:48c8e6f51c42637af6b739bdf837238713f6814da2396244a7553096b23783ff
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
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3940
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# Multi-omics integration embedding quality metrics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark framework for scoring joint embeddings from vertical integration of spatial transcriptomics (ST) and spatial metabolomics (SM) data using four scIB-style metric families — reconstruction accuracy, continuity, marker, and biological conservation — aggregated into an overall score to rank integration methods.

## When to use

Use when comparing integration methods (e.g., SpatialMETA, totalVI, SpatialGLUE, MISO, Seurat v5 BNN, Stabmap, spaMultiVAE) on paired ST-SM samples and you need quantitative, ranked evidence of which method's joint embedding best preserves spatial biology.

## When NOT to use

- When the task is horizontal (cross-sample) integration rather than vertical (cross-modal ST-SM) integration — the metric families and ranking here benchmark vertical integration performance
- When ground-truth labels are unavailable or the tissue cannot be manually annotated from H&E — the biological conservation metrics depend on the manually annotated label key, not an independent gold standard
- When data are at single-cell or subcellular resolution — SpatialMETA and this benchmark are designed for low-resolution ST and SM data

## Inputs

- Joint latent embeddings from each integration method applied to the eleven vertically integrated ST-SM samples (human ccRCC, GBM, mouse brain)
- Manually H&E-annotated ground-truth label key for spatial clusters (supplied as the 'label key' parameter to scIB)
- Normalized expression outputs from SpatialMETA's get_normalized_expression for reconstruction-accuracy evaluation
- scIB-style metric implementations

## Outputs

- Per-method scores for four metric families: reconstruction accuracy, continuity, marker, and biological conservation
- Aggregated overall score per method with ranking (reproducing Fig. 2b ordering)
- benchmark_overall_scores.csv containing one row per evaluated method (SpatialMETA plus 7 baselines)

## How to apply

Run each integration method (e.g., SpatialMETA's ConditionalVAESTSM via fit + get_latent_embedding, or baselines totalVI, spaMultiVAE, MISO 2/3-mode, Seurat v5 BNN, Stabmap, SpatialGLUE) on the same eleven vertically integrated ST-SM samples from human ccRCC, GBM, and mouse brain. For each method's joint embedding, compute the four metric families (reconstruction accuracy, continuity, marker, biological conservation) using scIB-style metrics, supplying the manually H&E-annotated label key as the ground truth for the biological conservation scores. Aggregate the four metric families into a single overall score per method and rank methods, as summarized in Fig. 2b and Supplementary Fig. 2e-h. Write the ranked table (one row per method, including an overall-score column) to a file such as benchmark_overall_scores.csv. Note that detailed metric values are only available in the paper's source data files, so use those to validate reproduced numbers.

## Related tools

- **SpatialMETA** (Method under benchmark; ConditionalVAESTSM embeddings are scored alongside baselines) — https://github.com/WanluLiuLab/SpatialMETA
- **totalVI** (Baseline integration method whose joint embedding is scored with the same metric families)
- **Seurat** (Baseline (v5 with BNN) integration method scored in the benchmark)
- **Stabmap** (Baseline integration method scored in the benchmark)
- **spaMultiVAE** (Baseline spatial multi-omics integration method scored in the benchmark)
- **MISO** (Baseline integration method run in 2-mode and 3-mode configurations and scored in the benchmark)
- **SpatialGLUE** (Baseline spatial multi-omics integration method scored in the benchmark)

## Examples

```
from spatialmeta.model import ConditionalVAESTSM; model.fit(adata); emb = model.get_latent_embedding(adata)  # then score emb with scIB-style metrics vs. H&E label key
```

## Evaluation signals

- benchmark_overall_scores.csv contains one row per evaluated method — SpatialMETA plus 7 baselines (totalVI, spaMultiVAE, MISO x2, Seurat v5 BNN, Stabmap, SpatialGLUE) — with an overall-score column
- SpatialMETA ranks first in the aggregated overall score, matching the ordering reported in Fig. 2b and Supplementary Fig. 2e-h
- Each method receives scores in all four metric families (reconstruction accuracy, continuity, marker, biological conservation) computed with the same scIB-style setup and the same manually annotated label key
- Reproduced metric values match the detailed values provided in the paper's source data files
- All methods are evaluated on the identical eleven ccRCC, GBM, and mouse brain vertically integrated samples to ensure fair comparison

## Limitations

- Ground-truth labels for biological conservation were manually annotated from H&E histological features, not an independent gold standard, so scores depend on annotation quality
- Detailed benchmark metric values are only provided in source data files rather than in the main text
- Benchmark covers a limited set of samples (eleven from human ccRCC, GBM, and mouse brain in previously published studies), so conclusions may not generalize to other tissues or modalities

## Evidence

- [discussion] Ground-truth labels from H&E: [evidence span withheld — anchor status: not_found]
- [readme] README install command: [evidence span withheld — anchor status: external_doc]
