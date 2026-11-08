---
name: striatal-cluster-map-comparison
description: Use when you have co-registered spatial metabolomics (MALDI-TOF/imzML)
  and spatial transcriptomics (e.g. 10X Visium) data on the same tissue section and
  need to test whether multi-omic integration recovers spatial populations that unimodal
  clustering misses.
when_to_use_negative:
- Only one modality is available (SM or ST alone) — integration requires paired, spatially
  co-registered data on the same section
- The MALDI and transcriptomics data have not been aligned to a shared spot grid (e.g.
  MALDI pixels not binned into 55 µm Visium spots) — run AlignSpatialOmics/MapSpatialOmics
  first
- The goal is metabolite m/z annotation rather than clustering — use SpaMTP's annotation
  pipeline (AnnotateBigData, BuildMZAnnotationIndex) instead
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3308
tools:
- name: SpaMTP
  role: Provides MultiOmicIntegration (Seurat-based WNN with optimised metabolite/gene
    weighting), AlignSpatialOmics/MapSpatialOmics for spot binning, and ImageMZPlot
    for validation
  repo: https://github.com/GenomicsMachineLearning/SpaMTP
- name: Seurat
  role: 'Underlying framework: FindNeighbors/FindClusters for ST-only and SM-only
    baseline clustering; Seurat Class Object is SpaMTP''s foundation'
  repo: https://satijalab.org/seurat/
- name: Cardinal
  role: Reads the MALDI-TOF imzML data that feed the SPM assay prior to integration
- name: R
  role: Language in which the entire clustering and integration workflow is executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.10.31.621429
    title: 'SpaMTP: Integrative Statistical Analysis and Visualisation of Spatial
      Metabolomics and Transcriptomics data'
  source_content_hash: sha256:f1647cb20678cd7177e42934bc932c8a8bfe4ec21b10b4df5f7a512d87984c9e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/GenomicsMachineLearning/SpaMTP
  tools:
  - Cardinal
  - R
  - Seurat
  - SpaMTP
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# Striatal cluster map comparison (SM-only vs ST-only vs WNN-integrated)

## Summary

Compares per-spot cluster assignments from SM-only, ST-only, and WNN-integrated clustering of paired spatial metabolomics/transcriptomics data using SpaMTP's MultiOmicIntegration (Seurat-based weighted nearest neighbours), to reveal striatal spot populations that single-modality clustering misses.

## When to use

Use when you have co-registered spatial metabolomics (MALDI-TOF/imzML) and spatial transcriptomics (e.g. 10X Visium) data on the same tissue section and need to test whether multi-omic integration recovers spatial populations that unimodal clustering misses.

## When NOT to use

- Only one modality is available (SM or ST alone) — integration requires paired, spatially co-registered data on the same section
- The MALDI and transcriptomics data have not been aligned to a shared spot grid (e.g. MALDI pixels not binned into 55 µm Visium spots) — run AlignSpatialOmics/MapSpatialOmics first
- The goal is metabolite m/z annotation rather than clustering — use SpaMTP's annotation pipeline (AnnotateBigData, BuildMZAnnotationIndex) instead

## Inputs

- SpaMTP Seurat object with co-registered 'SPM' (spatial metabolomics) and 'SPT' (spatial transcriptomics) assays from paired MALDI-TOF (imzML) and 10X Visium data
- Spot-binned MALDI data (55 µm Visium spot grid) with custom m/z annotations (e.g. FMP-10 matrix derivatives via AddCustomAnnotation)
- Tissue-region metadata distinguishing striatal subregions (lesioned vs intact hemisphere)

## Outputs

- Per-spot cluster assignment table for SM-only, ST-only, and WNN-integrated clustering (e.g. wnn_cluster_comparison.csv)
- Integrated WNN embedding with cluster labels
- Annotated map of integrated clusters localised to striatal subpopulations (lesioned vs intact) with opposing metabolic activity

## How to apply

First align the MALDI pixels to the 55 µm Visium spots with AlignSpatialOmics/MapSpatialOmics, building a single SpaMTP Seurat object holding 'SPM' (metabolite) and 'SPT' (transcript) assays, and check spot-binning fidelity by comparing spot-binned vs raw pixel signal for a marker such as dopamine (HMDB:HMDB0000073) with ImageMZPlot. Run baselines: Seurat FindNeighbors/FindClusters on the 'SPT' assay for ST-only clustering and on the 'SPM' assay for SM-only clustering. Then integrate both modalities with SpaMTP's MultiOmicIntegration — a Seurat-based weighted nearest-neighbours approach — using the optimised metabolite/gene modality weighting of 0.6/0.4 reported for the Parkinson's mouse brain case study, and cluster the integrated embedding. Assemble a table of per-spot cluster assignments for SM-only, ST-only, and WNN-integrated clustering, and annotate which integrated clusters localise to the striatum (lesioned vs intact sides). The key decision point is comparing maps side-by-side: integration is judged valuable when it resolves biologically distinct spot populations that the transcriptomics-only baseline collapses.

## Related tools

- **SpaMTP** (Provides MultiOmicIntegration (Seurat-based WNN with optimised metabolite/gene weighting), AlignSpatialOmics/MapSpatialOmics for spot binning, and ImageMZPlot for validation) — https://github.com/GenomicsMachineLearning/SpaMTP
- **Seurat** (Underlying framework: FindNeighbors/FindClusters for ST-only and SM-only baseline clustering; Seurat Class Object is SpaMTP's foundation) — https://satijalab.org/seurat/
- **Cardinal** (Reads the MALDI-TOF imzML data that feed the SPM assay prior to integration)
- **R** (Language in which the entire clustering and integration workflow is executed)

## Examples

```
devtools::install_github("GenomicsMachineLearning/SpaMTP")
```

## Evaluation signals

- Integrated (WNN) clustering recovers two striatal spot populations with opposing metabolic activity (lesioned vs intact) that ST-only clustering misses
- Spot-binned dopamine expression matches raw MALDI pixel expression when plotted with ImageMZPlot, confirming spot-binning fidelity before clustering
- wnn_cluster_comparison.csv contains a cluster assignment for every Visium spot across all three methods (SM-only, ST-only, integrated)
- Modality weighting in MultiOmicIntegration is set to the optimised metabolite/gene ratio 0.6/0.4 rather than an arbitrary equal split
- Integrated clusters labelled striatal localise anatomically to the striatum and agree with lesioned vs intact hemisphere metadata

## Limitations

- No specific SpaMTP version, DOI for the vignette, or hardware/environment requirements are stated in the article; pin the GitHub commit or use the provided Apptainer image (spamtp-1.1.sif) for reproducibility
- METASPACE and pySM annotation tools are optimised for high-resolution instruments (FT-ICR/Orbitrap) and do not apply to most time-of-flight (TOF) MALDI data like the FMP-10/DHB runs used here
- The paired targeted metabolomics comparison section of the annotation vignette is a placeholder ('To come!'), so cross-validation against targeted assays is not yet documented

## Evidence

- [readme] SpaMTP's R foundation and scope: [evidence span withheld — anchor status: external_doc]
