---
name: mz-peak-spatial-colocalisation-analysis
description: Use when a spatial metabolomics image (e.g. imzML MALDI dataset) must
  be segmented into anatomical regions and m/z peaks assigned to those regions — e.g.
  urothelium/lamina propria/muscle in the murine bladder — with significant DEMs reported.
when_to_use_negative:
- The task is m/z-to-metabolite annotation of high-resolution instrument data (FT-ICR/Orbitrap),
  where METASPACE/pySM-style annotation rather than spatial segmentation is the right
  step
- Input is already a feature table with no pixel-level spatial coordinates, so tissue-region
  clustering and co-localisation are undefined
- The goal is LC-MS feature annotation or MS/MS spectral matching (e.g. ms1_id), not
  spatial colocalisation of imaging peaks
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: SpaMTP
  role: Provides RunMetabolicPCA and FindAllDEMs (pseudo-bulking + EdgeR) used to
    compute PCA embeddings and differential m/z peaks on the spatial metabolomics
    object
  repo: https://github.com/GenomicsMachineLearning/SpaMTP
- name: Seurat
  role: Provides FindNeighbors and FindClusters (graph-based Louvain clustering) run
    on the metabolite PCA embeddings to segment tissue regions
  repo: https://github.com/GenomicsMachineLearning/SpaMTP
- name: Cardinal
  role: Reads the MALDI imzML data (readImzML) whose processed peaks feed the SpaMTP
    Seurat object
  repo: https://github.com/GenomicsMachineLearning/SpaMTP
- name: EdgeR
  role: Statistical engine behind FindAllDEMs' pseudo-bulk differential metabolite
    testing with logFC and FDR thresholds
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2024.10.31.621429
    title: 'SpaMTP: Integrative Statistical Analysis and Visualisation of Spatial
      Metabolomics and Transcriptomics data'
  source_content_hash: sha256:1b311075cf719cbc3e54a4f13032d4e2560799f39204b93edf86c2e46488217b
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# m/z peak spatial colocalisation analysis

## Summary

Segments a spatial metabolomics (MALDI-IMS) image into tissue regions via metabolite-based PCA and Louvain clustering, then identifies differentially expressed m/z peaks and co-localising lipid masses per cluster using SpaMTP.

## When to use

Use when a spatial metabolomics image (e.g. imzML MALDI dataset) must be segmented into anatomical regions and m/z peaks assigned to those regions — e.g. urothelium/lamina propria/muscle in the murine bladder — with significant DEMs reported.

## When NOT to use

- The task is m/z-to-metabolite annotation of high-resolution instrument data (FT-ICR/Orbitrap), where METASPACE/pySM-style annotation rather than spatial segmentation is the right step
- Input is already a feature table with no pixel-level spatial coordinates, so tissue-region clustering and co-localisation are undefined
- The goal is LC-MS feature annotation or MS/MS spectral matching (e.g. ms1_id), not spatial colocalisation of imaging peaks

## Inputs

- Pre-processed MALDI-IMS metabolomics dataset (e.g. murine urinary bladder imzML) as a SpaMTP Seurat object
- Metabolite-based PCA embeddings from RunMetabolicPCA
- Region-defining lipid m/z list (PC(34:1) m/z-798.541, SM(34:1) m/z-741.5037, PE(38:1) m/z-812.5566)

## Outputs

- Cluster/region annotations (urothelium, lamina propria, muscle/adventitia) per spatial pixel
- Top co-localising m/z mass per cluster (e.g. m/z-743.5467 for lamina propria)
- Table of significantly differentially expressed m/z peaks (FindAllDEMs, pseudo-bulk + EdgeR with logFC/FDR thresholds)

## How to apply

Load the pre-processed MALDI dataset (imzML) and convert it into a SpaMTP Seurat object, then compute metabolite-based PCA embeddings with RunMetabolicPCA. Run Seurat's FindNeighbors and FindClusters (graph-based Louvain clustering) on those embeddings to segment tissue regions. Use FindAllDEMs, which applies pseudo-bulking plus EdgeR, to find significantly differentially expressed m/z peaks between clusters under user-set logFC and FDR thresholds. Annotate clusters by mapping known region-defining lipids: PC(34:1) m/z-798.541 for urothelium, SM(34:1) m/z-741.5037 for muscle layers, and PE(38:1) m/z-812.5566; record the top co-localising mass per cluster (e.g. m/z-743.5467 for lamina propria). Rationale: SpaMTP bridges Cardinal's metabolomics processing with Seurat-style cell-centric analyses, so clustering and differential expression operate directly on the spatial m/z peak matrix.

## Related tools

- **SpaMTP** (Provides RunMetabolicPCA and FindAllDEMs (pseudo-bulking + EdgeR) used to compute PCA embeddings and differential m/z peaks on the spatial metabolomics object) — https://github.com/GenomicsMachineLearning/SpaMTP
- **Seurat** (Provides FindNeighbors and FindClusters (graph-based Louvain clustering) run on the metabolite PCA embeddings to segment tissue regions) — https://github.com/GenomicsMachineLearning/SpaMTP
- **Cardinal** (Reads the MALDI imzML data (readImzML) whose processed peaks feed the SpaMTP Seurat object) — https://github.com/GenomicsMachineLearning/SpaMTP
- **EdgeR** (Statistical engine behind FindAllDEMs' pseudo-bulk differential metabolite testing with logFC and FDR thresholds)

## Examples

```
Rscript -e 'library(SpaMTP); spa <- RunMetabolicPCA(spa); spa <- FindNeighbors(spa); spa <- FindClusters(spa); dems <- FindAllDEMs(spa)'
```

## Evaluation signals

- Region-defining metabolites are recovered as up-regulated in their clusters: PC(34:1) m/z-798.541 in urothelium and SM(34:1) m/z-741.5037 in muscle layers
- FindAllDEMs returns a non-empty list of significant m/z peaks consistent with the reported ~70 significantly differentially expressed peaks
- Cluster partition separates urothelium, lamina propria and muscle/adventitia regions as spatially coherent pixel sets
- Each cluster has a recorded top co-localising mass (e.g. m/z-743.5467 for lamina propria)

## Limitations

- No specific SpaMTP version is stated in the paper; install via devtools::install_github and record the commit/version yourself
- The paper does not state the exact logFC/FDR cutoff values for FindAllDEMs, so thresholds must be chosen and reported by the analyst
- Pseudo-bulk + EdgeR differential testing requires adequate pixels/spots per cluster; sparse or tiny clusters will not yield reliable DEMs

## Evidence

- [readme] SpaMTP package purpose: [evidence span withheld — anchor status: external_doc]
