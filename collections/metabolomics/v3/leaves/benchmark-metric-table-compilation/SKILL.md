---
name: benchmark-metric-table-compilation
description: Use when several integration methods (e.g. SpatialMETA, Seurat CCA/RPCA,
  scVI, scANVI, scPoli, spaVAE, PCA) have been run on the same inputs and their embedding-level
  scores must be collected into one comparable metric table.
when_to_use_negative:
- The task is single-sample embedding or visualization rather than cross-method comparison
  — no metric table across methods can be compiled from one method alone
- Ground-truth labels are absent, since the biology-conservation (marker) metric requires
  H&E-annotated labels passed via the scIB 'label key' parameter
- Data are single-cell or subcellular-resolution ST/SM; SpatialMETA and this benchmark
  are designed for low-resolution ST and SM data only
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: SpatialMETA
  role: Provides the ConditionalVAESM embeddings and normalized expression being benchmarked
    against baselines
  repo: github.com/WanluLiuLab/SpatialMETA
- name: Seurat
  role: Baseline horizontal-integration method (CCA/RPCA) whose embeddings are scored
    in the metric table
- name: scVI
  role: Baseline horizontal-integration method scored alongside SpatialMETA
- name: scANVI
  role: Baseline horizontal-integration method scored alongside SpatialMETA
- name: scPoli
  role: Top-performing baseline (with SpatialMETA and Seurat CCA/RPCA) in SM-only
    ccRCC integration benchmark
- name: spaVAE
  role: Baseline spatial multi-omics integration method scored in the comparison
- name: scIB
  role: Metric framework computing continuity, marker (biology conservation via 'label
    key'), and batch-correction scores for each embedding
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-63915-z
    title: Integrating cross-sample and cross-modal data for spatial transcriptomics
      and metabolomics with SpatialMETA
  source_content_hash: sha256:f1913c2b731d6e87c066052c89d54c9cb49ce0213fdb23ce03f78537c86ced71
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/WanluLiuLab/SpatialMETA
  tools:
  - Seurat
  - SpatialMETA
  - scANVI
  - scIB
  - scPoli
  - scVI
  - spaVAE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# benchmark-metric-table-compilation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assembles per-method benchmark metric values (continuity, marker/biology-conservation, and batch-correction scores from scIB-style evaluation) into a single CSV table, e.g. mirroring Supplementary Fig. 11 of the SpatialMETA paper, so horizontal-integration methods can be ranked side by side.

## When to use

Use when several integration methods (e.g. SpatialMETA, Seurat CCA/RPCA, scVI, scANVI, scPoli, spaVAE, PCA) have been run on the same inputs and their embedding-level scores must be collected into one comparable metric table.

## When NOT to use

- The task is single-sample embedding or visualization rather than cross-method comparison — no metric table across methods can be compiled from one method alone
- Ground-truth labels are absent, since the biology-conservation (marker) metric requires H&E-annotated labels passed via the scIB 'label key' parameter
- Data are single-cell or subcellular-resolution ST/SM; SpatialMETA and this benchmark are designed for low-resolution ST and SM data only

## Inputs

- Latent embeddings from SpatialMETA ConditionalVAESM.get_latent_embedding for SM-only ccRCC integration
- Latent embeddings from Seurat CCA/RPCA, scVI, scANVI, scPoli, spaVAE, and PCA run on the same filtered metabolite matrices
- H&E-annotated ground-truth spatial cluster labels (label key passed to scIB)
- Filtered AnnDataSM metabolite intensity matrices with batch-biased SVMs removed (fold change > min_logfc default 3, expression in >= min_frac default 0.8 of spots)

## Outputs

- sm_horizontal_integration_benchmark_metrics.csv with per-method rows for continuity, marker (biology conservation), and batch-correction metric categories
- A table mirroring Supplementary Fig. 11 ranking all seven integration methods

## How to apply

After each integration method produces a shared latent embedding over the same filtered metabolite matrices, score every embedding with scIB-style continuity, marker (biology conservation against H&E-annotated ground-truth labels), and batch-correction metrics. Before scoring, remove batch-biased SVMs defined as features with fold change greater than min_logfc (default 3) expressed in >= min_frac (default 0.8) of spots. Assemble the per-method values into a table with metric rows for all seven methods (SpatialMETA plus six baselines) covering all three metric categories, and save it as sm_horizontal_integration_benchmark_metrics.csv. Validate that SpatialMETA ranks at or near the top across categories, consistent with its reported superior overall performance; exact metric values in the paper live only in source data files, not the main text.

## Related tools

- **SpatialMETA** (Provides the ConditionalVAESM embeddings and normalized expression being benchmarked against baselines) — github.com/WanluLiuLab/SpatialMETA
- **Seurat** (Baseline horizontal-integration method (CCA/RPCA) whose embeddings are scored in the metric table)
- **scVI** (Baseline horizontal-integration method scored alongside SpatialMETA)
- **scANVI** (Baseline horizontal-integration method scored alongside SpatialMETA)
- **scPoli** (Top-performing baseline (with SpatialMETA and Seurat CCA/RPCA) in SM-only ccRCC integration benchmark)
- **spaVAE** (Baseline spatial multi-omics integration method scored in the comparison)
- **scIB** (Metric framework computing continuity, marker (biology conservation via 'label key'), and batch-correction scores for each embedding)

## Examples

```
import pandas as pd; pd.DataFrame(metrics_rows).to_csv('sm_horizontal_integration_benchmark_metrics.csv', index=False)
```

## Evaluation signals

- The CSV contains metric rows for all seven methods: SpatialMETA plus Seurat CCA/RPCA, scVI, scANVI, scPoli, spaVAE, and PCA
- The table covers all three metric categories: continuity, marker (biology conservation), and batch correction
- SpatialMETA ranks at or near the top across the categories, consistent with the reported superior overall performance (with scPoli and Seurat CCA/RPCA as the other top performers)
- Batch-biased SVMs (fold change > min_logfc default 3, expressed in >= min_frac default 0.8 of spots) were removed before scoring all methods identically
- All methods are scored on the same filtered metabolite matrices and identical scIB parameters, so metric differences reflect embedding quality rather than preprocessing differences

## Limitations

- Ground-truth labels for biology conservation are manually annotated from H&E histological features, not an independent gold standard, so marker-score values carry annotator bias
- Detailed benchmark metric values are only provided in source data files rather than in the main text, so the compiled table is the primary record
- Horizontal integration was validated on a limited number of samples from a single study, so the table's ranking may not generalize to other datasets

## Evidence

- [discussion] Benchmark analyses demonstrate that, compared to previously published non-spatial single-cell or spatial multi-omics tools, SpatialMETA achieved superior overall performance: "Benchmark analyses demonstrate that, compared to previously published non-spatial single-cell or spatial multi-omics tools, SpatialMETA achieved superior overall performance"
- [discussion] scIB ground-truth labels annotated from H&E: [evidence span withheld — anchor status: not_found]
- [discussion] batch-biased SVM definition with min_logfc default 3: [evidence span withheld — anchor status: not_found]
- [discussion] metric values only in source data files: "Detailed benchmark metric values are provided in source data files"
- [readme] README change log naming SpatialMETA model modules: "Update `spatialmeta.model.ConditionalVAESTSM`"
