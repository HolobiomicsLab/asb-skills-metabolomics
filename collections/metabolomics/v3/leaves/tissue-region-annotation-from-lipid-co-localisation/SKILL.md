---
name: tissue-region-annotation-from-lipid-co-localisation
description: Use when a processed MALDI/imzML spatial metabolomics dataset (e.g. murine
  urinary bladder) must be segmented into anatomical regions and each cluster annotated
  using known region-defining lipid m/z markers plus differentially expressed m/z
  peaks.
when_to_use_negative:
- Input is a feature/annotation table or the goal is m/z metabolite annotation refinement
  rather than spatial region annotation — use SpaMTP's annotation functions (AnnotateBigData,
  RefineLipids, CalculateAnnotationStatistics) instead
- The data has not yet been processed into imzML/SpaMTP Seurat object form, so no
  per-pixel m/z matrix exists to cluster
- High-resolution instrument data suited to METASPACE/pySM annotation workflows, which
  the article notes are 'optimised for SM data generated via high resolution instruments'
  and 'do not yet apply to most data obtained with time-of-flight (TOF) analysers'
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SpaMTP
  role: Provides RunMetabolicPCA and FindAllDEMs (pseudo-bulking + EdgeR) for metabolite-PCA
    embeddings and differential m/z testing on a Seurat-class object
  repo: github.com/GenomicsMachineLearning/SpaMTP
- name: Seurat
  role: FindNeighbors and FindClusters (graph-based Louvain clustering) on the metabolite
    PCA embeddings to segment tissue regions
  repo: https://github.com/satijalab/seurat
- name: Cardinal
  role: Reads the MALDI imzML data (e.g. readImzML) that SpaMTP bridges into a Seurat-class
    object
  repo: https://github.com/kbemis/Cardinal
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2024.10.31.621429
    title: 'SpaMTP: Integrative Statistical Analysis and Visualisation of Spatial
      Metabolomics and Transcriptomics data'
  source_content_hash: sha256:e29688315abcfc852fef80cbb52e6020279cc41d26b0e56ce1c57ae10544ed96
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# tissue-region-annotation-from-lipid-co-localisation

## Summary

Annotates spatial-metabolomics tissue regions by combining metabolite-PCA-based Louvain clustering (SpaMTP/Seurat) with known region-defining lipid masses (e.g. PC(34:1) m/z-798.541, SM(34:1) m/z-741.5037, PE(38:1) m/z-812.5566) and EdgeR-based differential m/z testing via FindAllDEMs.

## When to use

Use when a processed MALDI/imzML spatial metabolomics dataset (e.g. murine urinary bladder) must be segmented into anatomical regions and each cluster annotated using known region-defining lipid m/z markers plus differentially expressed m/z peaks.

## When NOT to use

- Input is a feature/annotation table or the goal is m/z metabolite annotation refinement rather than spatial region annotation — use SpaMTP's annotation functions (AnnotateBigData, RefineLipids, CalculateAnnotationStatistics) instead
- The data has not yet been processed into imzML/SpaMTP Seurat object form, so no per-pixel m/z matrix exists to cluster
- High-resolution instrument data suited to METASPACE/pySM annotation workflows, which the article notes are 'optimised for SM data generated via high resolution instruments' and 'do not yet apply to most data obtained with time-of-flight (TOF) analysers'

## Inputs

- Pre-processed MALDI imaging dataset in imzML format (e.g. murine urinary bladder MALDI dataset)
- SpaMTP Seurat object converted from the imzML data
- List of known region-defining lipid m/z values (e.g. PC(34:1) m/z-798.541, SM(34:1) m/z-741.5037, PE(38:1) m/z-812.5566)
- logFC and FDR significance thresholds for FindAllDEMs

## Outputs

- Cluster/region annotations assigning spatial clusters to urothelium, lamina propria, and muscle/adventitia
- Top co-localising m/z mass per cluster (e.g. m/z-743.5467 for lamina propria)
- Non-empty list of significantly differentially expressed m/z peaks (DEMs) consistent with ~70 reported significant peaks

## How to apply

Load the pre-processed MALDI imzML dataset and convert it into a SpaMTP Seurat object (Cardinal handles the metabolomics data import). Run SpaMTP's RunMetabolicPCA to compute metabolite-based PCA embeddings, then apply Seurat FindNeighbors and FindClusters (graph-based Louvain clustering) to segment tissue regions. Run FindAllDEMs, which pseudo-bulks pixels per cluster and calls EdgeR, to identify significantly differentially expressed m/z peaks between tissue-specific clusters using logFC and FDR thresholds. Finally, map clusters to regions (urothelium, lamina propria, muscle/adventitia) using region-defining lipids — PC(34:1) m/z-798.541 for urothelium, SM(34:1) m/z-741.5037 for muscle layers, PE(38:1) m/z-812.5566 — and record the top co-localising mass per cluster (e.g. m/z-743.5467 for lamina propria).

## Related tools

- **SpaMTP** (Provides RunMetabolicPCA and FindAllDEMs (pseudo-bulking + EdgeR) for metabolite-PCA embeddings and differential m/z testing on a Seurat-class object) — github.com/GenomicsMachineLearning/SpaMTP
- **Seurat** (FindNeighbors and FindClusters (graph-based Louvain clustering) on the metabolite PCA embeddings to segment tissue regions) — https://github.com/satijalab/seurat
- **Cardinal** (Reads the MALDI imzML data (e.g. readImzML) that SpaMTP bridges into a Seurat-class object) — https://github.com/kbemis/Cardinal

## Examples

```
if (!require("SpaMTP")) devtools::install_github("GenomicsMachineLearning/SpaMTP")
```

## Evaluation signals

- PC(34:1) m/z-798.541 is confirmed up-regulated in the cluster annotated as urothelium
- SM(34:1) m/z-741.5037 is confirmed up-regulated in the cluster annotated as the muscle layers
- FindAllDEMs returns a non-empty list of significant m/z peaks consistent with the reported 70 significantly differentially expressed peaks
- Clusters partition the tissue into the expected three regions (urothelium, lamina propria, muscle/adventitia), each with a recorded top co-localising mass (e.g. m/z-743.5467 for lamina propria)

## Limitations

- No explicit SpaMTP version number is given in the article, so reproducibility depends on pinning the GitHub commit or release (a prebuilt Apptainer image spamtp-1.1.sif is available)
- The logFC and FDR threshold values for FindAllDEMs are referenced but not numerically specified in the provided context
- Region assignment depends on the availability of validated region-defining lipid markers; tissues lacking such known masses cannot be annotated this way
- Clustering quality depends on the RunMetabolicPCA embedding and Louvain resolution choices, which are not parameterised in the provided context

## Evidence

- [other] Running the documented SpaMTP pipeline on the murine bladder MALDI data produces cluster/region annotations (urothelium, lamina propria, muscle) with co-localising masses: [evidence span withheld — anchor status: not_found]
- [methods] Run SpaMTP's FindAllDEMs (pseudo-bulking + EdgeR) to identify significantly differentially expressed m/z peaks between the tissue-specific clusters, applying the logFC and FDR thresholds: [evidence span withheld — anchor status: not_found]
- [readme] SpaMTP inherits functionalities from two well established R packages (Cardinal and Seurat) to present a user-friendly platform for integrative spatial-omics analysis.: [evidence span withheld — anchor status: external_doc]
