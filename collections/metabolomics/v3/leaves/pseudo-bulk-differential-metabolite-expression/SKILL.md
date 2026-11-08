---
name: pseudo-bulk-differential-metabolite-expression
description: Use when a MALDI/imaging-MS spatial metabolomics dataset (e.g. imzML
  converted to a SpaMTP Seurat object) has been segmented into clusters (e.g. urothelium,
  lamina propria, muscle) and you need significantly differentially expressed m/z
  peaks between those clusters.
when_to_use_negative:
- Input is already an aggregated/bulk feature table without spatial pixels — pseudo-bulking
  and spatial clustering are unnecessary
- The goal is m/z-to-metabolite annotation (use SpaMTP's AnnotateBigData/CalculateAnnotationStatistics
  pipeline instead of DEM testing)
- Clustering quality is unverified — DEM calls inherit any segmentation errors from
  the Louvain clustering step
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: SpaMTP
  role: Provides FindAllDEMs (pseudo-bulking + EdgeR) and RunMetabolicPCA; converts
    Cardinal imaging data into a Seurat-class object for downstream differential metabolite
    testing
  repo: https://github.com/GenomicsMachineLearning/SpaMTP
- name: EdgeR
  role: Negative-binomial differential expression engine called by FindAllDEMs on
    pseudo-bulked per-cluster m/z counts
- name: Seurat
  role: FindNeighbors/FindClusters Louvain clustering on metabolite PCA embeddings
    to define the clusters compared by FindAllDEMs
  repo: https://satijalab.org/seurat/
- name: Cardinal
  role: Reads and pre-processes the MALDI imzML data before conversion into the SpaMTP
    Seurat object
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2024.10.31.621429
    title: 'SpaMTP: Integrative Statistical Analysis and Visualisation of Spatial
      Metabolomics and Transcriptomics data'
  source_content_hash: sha256:ba4249a8fa0b5a2b9f1abd9cfc4187fdcbf5ad22a163e3a68da6b3d0531bd662
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/GenomicsMachineLearning/SpaMTP
  tools:
  - Cardinal
  - EdgeR
  - R
  - Seurat
  - SpaMTP
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# pseudo-bulk-differential-metabolite-expression

## Summary

Identifies significantly differentially expressed metabolites (DEMs) between spatial metabolomics clusters by pseudo-bulking pixels per cluster and testing with EdgeR via SpaMTP's FindAllDEMs, yielding annotated m/z peaks that define tissue regions.

## When to use

Use when a MALDI/imaging-MS spatial metabolomics dataset (e.g. imzML converted to a SpaMTP Seurat object) has been segmented into clusters (e.g. urothelium, lamina propria, muscle) and you need significantly differentially expressed m/z peaks between those clusters.

## When NOT to use

- Input is already an aggregated/bulk feature table without spatial pixels — pseudo-bulking and spatial clustering are unnecessary
- The goal is m/z-to-metabolite annotation (use SpaMTP's AnnotateBigData/CalculateAnnotationStatistics pipeline instead of DEM testing)
- Clustering quality is unverified — DEM calls inherit any segmentation errors from the Louvain clustering step

## Inputs

- Pre-processed spatial metabolomics data in imzML/MALDI format (e.g. murine urinary bladder dataset)
- SpaMTP Seurat object converted from the Cardinal-processed imaging data
- Cluster assignments from Louvain clustering on RunMetabolicPCA embeddings
- logFC and FDR thresholds for DEM calling

## Outputs

- Per-cluster table of significantly differentially expressed m/z peaks (DEMs) with logFC and FDR values
- Cluster-to-region annotation (e.g. urothelium, lamina propria, muscle/adventitia)
- Top co-localising mass per cluster (e.g. m/z-743.5467 for lamina propria)

## How to apply

Convert the Cardinal-processed MALDI data (imzML) into a SpaMTP Seurat object, then compute metabolite-based embeddings with RunMetabolicPCA and segment tissue using Seurat's FindNeighbors/FindClusters (graph-based Louvain clustering). For each cluster, SpaMTP's FindAllDEMs pseudo-bulks the pixel-level m/z intensities and runs EdgeR (negative-binomial differential testing) between clusters, applying the logFC and FDR thresholds to call significant DEMs. Map the resulting peaks to tissue regions using known region-defining lipids such as PC(34:1) m/z-798.541 (urothelium), SM(34:1) m/z-741.5037 (muscle layers) and PE(38:1) m/z-812.5566, and record the top co-localising mass per cluster (e.g. m/z-743.5467 for lamina propria). The rationale is that pseudo-bulking before EdgeR respects pixel replication within a region while avoiding treating every pixel as an independent replicate.

## Related tools

- **SpaMTP** (Provides FindAllDEMs (pseudo-bulking + EdgeR) and RunMetabolicPCA; converts Cardinal imaging data into a Seurat-class object for downstream differential metabolite testing) — https://github.com/GenomicsMachineLearning/SpaMTP
- **EdgeR** (Negative-binomial differential expression engine called by FindAllDEMs on pseudo-bulked per-cluster m/z counts)
- **Seurat** (FindNeighbors/FindClusters Louvain clustering on metabolite PCA embeddings to define the clusters compared by FindAllDEMs) — https://satijalab.org/seurat/
- **Cardinal** (Reads and pre-processes the MALDI imzML data before conversion into the SpaMTP Seurat object)

## Examples

```
devtools::install_github("GenomicsMachineLearning/SpaMTP"); dems <- SpaMTP::FindAllDEMs(spamtp_bladder_obj)
```

## Evaluation signals

- DEM analysis returns a non-empty significant peak list consistent with the reported ~70 significantly differentially expressed m/z peaks for the murine bladder dataset
- Region-defining lipids are recovered as up-regulated in their expected clusters: PC(34:1) m/z-798.541 in urothelium, SM(34:1) m/z-741.5037 in muscle layers, PE(38:1) m/z-812.5566 co-localised
- Cluster-to-region mapping is biologically coherent (urothelium, lamina propria, muscle/adventitia) and the top co-localising mass per cluster is recorded (e.g. m/z-743.5467 for lamina propria)
- All reported DEMs satisfy the applied logFC and FDR thresholds from FindAllDEMs
- Pseudo-bulked per-cluster input matrices to EdgeR contain one aggregated profile per cluster rather than per-pixel rows

## Limitations

- No explicit SpaMTP version number is documented in the article; reproducibility depends on recording the installed version
- DE calls depend on the preceding RunMetabolicPCA + Louvain clustering; changing clustering resolution changes both region assignments and the DEM list
- Downstream differential-expression analyses are incompatible with raw Cardinal objects — data must first be converted into the SpaMTP/Seurat object structure
- SpaMTP's metabolic annotation and testing must respect matrix/adduct rules (e.g. MALDIMatrixRules) or peak identities may be mis-assigned

## Evidence

- [readme] SpaMTP scope: integrative SM/ST analysis: [evidence span withheld — anchor status: external_doc]
- [other] Workflow grounds clustering in RunMetabolicPCA + Seurat graph-based clustering: [evidence span withheld — anchor status: not_found]
