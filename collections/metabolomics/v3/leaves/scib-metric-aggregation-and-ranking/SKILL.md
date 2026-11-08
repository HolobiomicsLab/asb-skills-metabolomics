---
name: scib-metric-aggregation-and-ranking
description: Use when you have joint embeddings from multiple integration methods
  (e.g. SpatialMETA, totalVI, MISO, SpatialGLUE) on the same ST-SM samples and need
  to quantitatively rank their integration quality.
when_to_use_negative:
- You need raw numeric metric values for the main text — the paper only provides detailed
  benchmark metric values in source data files, not the article body
- Your integration task is horizontal (cross-sample) only with a different sample
  cohort, or involves more than two modalities — SpatialMETA's benchmark scope here
  is vertical ST-SM integration of two modalities
- 'An independent gold-standard label set is required: the ground-truth labels here
  were manually annotated from H&E staining, not an independent gold standard'
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: SpatialMETA
  role: Method under test; provides ConditionalVAESTSM embeddings via get_latent_embedding
    and get_normalized_expression for metric computation
  repo: https://github.com/WanluLiuLab/SpatialMETA
- name: totalVI
  role: Baseline integration method whose joint embedding is scored and ranked
- name: Seurat
  role: Baseline (Seurat v5 with BNN) whose joint embedding is scored and ranked
- name: Stabmap
  role: Baseline integration method whose joint embedding is scored and ranked
- name: spaMultiVAE
  role: Baseline integration method whose joint embedding is scored and ranked
- name: MISO
  role: Baseline run in both 2-mode and 3-mode configurations, scored and ranked
- name: SpatialGLUE
  role: Baseline integration method whose joint embedding is scored and ranked
- name: scIB
  role: Metric framework used to compute the four metric families with the manually
    annotated label key
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-63915-z
    title: Integrating cross-sample and cross-modal data for spatial transcriptomics
      and metabolomics with SpatialMETA
  source_content_hash: sha256:b1f3ac9071b5538bbc70e66f1b44e6ebd675b537df3c4c2bce22c07513a777a4
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# scib-metric-aggregation-and-ranking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark integration methods by computing four scIB-style metric families (reconstruction accuracy, continuity, marker, and biological conservation scores) on each method's joint embedding of ST-SM data, then aggregating them into a single overall score per method and ranking methods to reproduce the paper's Fig. 2b comparison.

## When to use

Use when you have joint embeddings from multiple integration methods (e.g. SpatialMETA, totalVI, MISO, SpatialGLUE) on the same ST-SM samples and need to quantitatively rank their integration quality.

## When NOT to use

- You need raw numeric metric values for the main text — the paper only provides detailed benchmark metric values in source data files, not the article body
- Your integration task is horizontal (cross-sample) only with a different sample cohort, or involves more than two modalities — SpatialMETA's benchmark scope here is vertical ST-SM integration of two modalities
- An independent gold-standard label set is required: the ground-truth labels here were manually annotated from H&E staining, not an independent gold standard

## Inputs

- Joint embeddings (latent representations) from each evaluated integration method, computed on the same eleven vertically integrated ST-SM samples
- Manually annotated ground-truth spatial cluster labels derived from H&E histological features (scIB 'label key')
- Reconstructed expression outputs (e.g. from SpatialMETA's get_normalized_expression) for reconstruction-accuracy metrics

## Outputs

- benchmark_overall_scores.csv containing one row per evaluated method with an overall-score column, ranked so the best method is first
- Per-method values for the four metric families: reconstruction accuracy, continuity, marker, and biological conservation scores

## How to apply

For each method's joint embedding computed on the same eleven vertically integrated ST-SM samples (human ccRCC, GBM, and mouse brain), compute the four metric families using scIB-style metrics, supplying the manually H&E-annotated label key as the 'label key' parameter for biology conservation evaluation. Aggregate the four metric families into a single overall score per method, then sort methods by overall score. Write the ranked table (one row per method: SpatialMETA plus 7 baselines — totalVI, spaMultiVAE, MISO 2-mode and 3-mode, Seurat v5 with BNN, Stabmap, SpatialGLUE) to benchmark_overall_scores.csv. Rationale: the paper reports overall ranking rather than raw metric values in the main text, so aggregation into an overall score is the basis of the comparison in Fig. 2b and Supplementary Fig. 2e-h.

## Related tools

- **SpatialMETA** (Method under test; provides ConditionalVAESTSM embeddings via get_latent_embedding and get_normalized_expression for metric computation) — https://github.com/WanluLiuLab/SpatialMETA
- **totalVI** (Baseline integration method whose joint embedding is scored and ranked)
- **Seurat** (Baseline (Seurat v5 with BNN) whose joint embedding is scored and ranked)
- **Stabmap** (Baseline integration method whose joint embedding is scored and ranked)
- **spaMultiVAE** (Baseline integration method whose joint embedding is scored and ranked)
- **MISO** (Baseline run in both 2-mode and 3-mode configurations, scored and ranked)
- **SpatialGLUE** (Baseline integration method whose joint embedding is scored and ranked)
- **scIB** (Metric framework used to compute the four metric families with the manually annotated label key)

## Examples

```
python run_benchmark.py --samples 11 --methods spatialmeta totalVI spaMultiVAE MISO2 MISO3 seurat_bnn stabmap spatialglue --label_key handmade_annotation --out benchmark_overall_scores.csv
```

## Evaluation signals

- benchmark_overall_scores.csv contains exactly one row per evaluated method: SpatialMETA plus the 7 baselines (totalVI, spaMultiVAE, MISO 2-mode, MISO 3-mode, Seurat v5 BNN, Stabmap, SpatialGLUE)
- The table has an overall-score column and SpatialMETA is ranked first, matching Fig. 2b and the reported finding of superior overall performance
- Each method's score is computed from the same four metric families (reconstruction accuracy, continuity, marker, biological conservation) applied to embeddings on the identical eleven samples
- The scIB label key used for biology conservation corresponds to manually H&E-annotated histological labels, not automated clusters
- The ranking ordering is consistent with the Supplementary Fig. 2e-h per-sample benchmark results

## Limitations

- Ground-truth labels for biology conservation were manually annotated from H&E histological features, not an independent gold standard
- Detailed benchmark metric values are only provided in source data files rather than in the main text
- SpatialMETA is designed for low-resolution ST and SM data and does not support single-cell or subcellular resolution benchmarking

## Evidence

- [discussion] The ground-truth labels used for evaluation were manually annotated from H&E histological features: [evidence span withheld — anchor status: not_found]
