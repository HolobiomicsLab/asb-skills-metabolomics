---
name: spatial-graph-louvain-clustering
description: Use when you have a pixel- or spot-resolution spatial metabolomics dataset
  (e.
when_to_use_negative:
- The input is already a clustering result or region label image — no need to re-run
  FindNeighbors/FindClusters.
- Data are single-cell spatial transcriptomics without a metabolite modality; the
  metabolite-based PCA (RunMetabolicPCA) and DEM differential analysis are specific
  to m/z feature matrices.
- The data are high-resolution FT-ICR/Orbitrap SM intended for METASPACE/pySM-style
  annotation rather than graph-based region segmentation on TOF-style MALDI pixel
  data.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3520
tools:
- name: SpaMTP
  role: R package that wraps Cardinal and Seurat; supplies RunMetabolicPCA and FindAllDEMs
    (pseudo-bulking + EdgeR) used for the metabolite PCA embedding and differential
    peak calling
  repo: github.com/GenomicsMachineLearning/SpaMTP
- name: Seurat
  role: Provides the Seurat Class Object foundation, FindNeighbors and FindClusters
    (Louvain) used to build the graph and partition spatial pixels
  repo: https://satijalab.org/seurat/
- name: Cardinal
  role: Handles processing/import of metabolomics MALDI-imaging data (e.g. readImzML
    of the imzML file) before conversion to the SpaMTP Seurat object
- name: R
  role: Language in which the SpaMTP, Cardinal and Seurat clustering pipeline is executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2024.10.31.621429
    title: 'SpaMTP: Integrative Statistical Analysis and Visualisation of Spatial
      Metabolomics and Transcriptomics data'
  source_content_hash: sha256:3c8641b2e9f0e94ecbcbc6b332c853ff1cb5a018bb9c3dc5f4704957d581d6d5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Cardinal
  - R
  - Seurat
  - SpaMTP
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# spatial-graph-louvain-clustering

## Summary

Segments spatial metabolomics (MALDI/imzML) tissue sections into metabolically distinct regions by computing a metabolite-based PCA embedding (SpaMTP RunMetabolicPCA), building a shared-nearest-neighbour graph with Seurat FindNeighbors, and partitioning it with Louvain FindClusters, followed by FindAllDEMs to identify differentially expressed m/z peaks per cluster.

## When to use

Use when you have a pixel- or spot-resolution spatial metabolomics dataset (e.g. MALDI imzML converted to a SpaMTP/Seurat object) and need to segment tissue into regions such as urothelium, lamina propria or muscle using unsupervised clustering on metabolite features, then call region-defining m/z peaks.

## When NOT to use

- The input is already a clustering result or region label image — no need to re-run FindNeighbors/FindClusters.
- Data are single-cell spatial transcriptomics without a metabolite modality; the metabolite-based PCA (RunMetabolicPCA) and DEM differential analysis are specific to m/z feature matrices.
- The data are high-resolution FT-ICR/Orbitrap SM intended for METASPACE/pySM-style annotation rather than graph-based region segmentation on TOF-style MALDI pixel data.

## Inputs

- Pre-processed spatial metabolomics MALDI dataset in imzML format (e.g. murine urinary bladder MALDI data)
- SpaMTP Seurat object with metabolite m/z features
- Optional: known region-defining lipid m/z markers (e.g. PC(34:1) m/z 798.541, SM(34:1) m/z 741.5037, PE(38:1) m/z 812.5566)

## Outputs

- Cluster assignments for each spatial pixel/spot (tissue region labels such as urothelium, lamina propria, muscle)
- Metabolite PCA embeddings (RunMetabolicPCA output)
- Per-cluster differentially expressed m/z peaks (FindAllDEMs output with logFC and FDR statistics)
- Cluster annotation table mapping clusters to tissue regions with top co-localising masses

## How to apply

Load the pre-processed MALDI/imzML data and convert it into a SpaMTP Seurat object, then run SpaMTP's RunMetabolicPCA to compute metabolite-based PCA embeddings. Feed those embeddings into Seurat FindNeighbors and FindClusters (graph-based Louvain) to segment tissue regions; the graph must be built on the metabolite PCA space, not on raw m/z intensities, so that spatially coherent metabolically similar pixels cluster together. After clustering, run SpaMTP's FindAllDEMs, which pseudo-bulks each cluster and applies EdgeR differential expression with logFC and FDR thresholds, to obtain significantly differentially expressed m/z peaks per cluster. Annotate clusters by mapping known region-defining lipids — e.g. PC(34:1) at m/z 798.541 for urothelium, SM(34:1) at m/z 741.5037, and PE(38:1) at m/z 812.5566 — onto the cluster-specific DEM lists. In the murine urinary bladder case study this recovered anatomically consistent regions with each cluster characterised by a top co-localising mass (e.g. m/z 743.5467 for lamina propria) and ~70 significant differentially expressed peaks.

## Related tools

- **SpaMTP** (R package that wraps Cardinal and Seurat; supplies RunMetabolicPCA and FindAllDEMs (pseudo-bulking + EdgeR) used for the metabolite PCA embedding and differential peak calling) — github.com/GenomicsMachineLearning/SpaMTP
- **Seurat** (Provides the Seurat Class Object foundation, FindNeighbors and FindClusters (Louvain) used to build the graph and partition spatial pixels) — https://satijalab.org/seurat/
- **Cardinal** (Handles processing/import of metabolomics MALDI-imaging data (e.g. readImzML of the imzML file) before conversion to the SpaMTP Seurat object)
- **R** (Language in which the SpaMTP, Cardinal and Seurat clustering pipeline is executed)

## Examples

```
R: spamtp_obj <- RunMetabolicPCA(spamtp_obj); spamtp_obj <- FindNeighbors(spamtp_obj, dims = 1:30); spamtp_obj <- FindClusters(spamtp_obj); dems <- FindAllDEMs(spamtp_obj)
```

## Evaluation signals

- Cluster/region annotations recover expected tissue layers (urothelium, lamina propria, muscle/adventitia) with known region-defining lipids up-regulated in their respective clusters — e.g. PC(34:1) in urothelium, SM(34:1) in muscle layers.
- FindAllDEMs returns a non-empty list of significant m/z peaks consistent with the reported ~70 significantly differentially expressed peaks in the bladder case study.
- Each cluster is characterised by a top co-localising mass (e.g. m/z 743.5467 for lamina propria) that can be matched to a plausible lipid identity.
- The graph used by FindNeighbors was built on the RunMetabolicPCA embedding, not on raw m/z intensity matrices — check the assay/dim.reduction passed to FindNeighbors.
- EdgeR FDR threshold is applied to the pseudo-bulked DEM results; verify that reported peaks pass the stated logFC and FDR cutoffs.

## Limitations

- Clustering quality depends on prior pre-processing of the MALDI data; SpaMTP expects data already processed via Cardinal before conversion to the Seurat object.
- Annotation of clusters to tissue regions relies on prior knowledge of region-defining lipids; without such markers the clusters remain unsupervised and unannotated.
- The workflow is designed for TOF-style MALDI pixel data; high-resolution instruments (FT-ICR/Orbitrap) may be better served by METASPACE/pySM-based annotation pipelines.
- The article's discussion section provides no explicit benchmark metrics or version/dependency requirements for reproducing the clustering.

## Evidence

- [other] Running the documented SpaMTP pipeline on the murine bladder MALDI data produces cluster/region annotations (urothelium, lamina propria, muscle) with co-localising masses such as PC(34:1) m/z-798.541: [evidence span withheld — anchor status: not_found]
- [readme] If SpaMTP contributes to your work, please cite:: [evidence span withheld — anchor status: external_doc]
