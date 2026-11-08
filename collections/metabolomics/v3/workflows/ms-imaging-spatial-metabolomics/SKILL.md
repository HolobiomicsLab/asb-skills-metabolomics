---
name: ms-imaging-spatial-metabolomics-workflow
description: 'Use when you have mass-spectrometry imaging data (imzML, e.g. MALDI/DESI)
  and want spatially-resolved metabolite annotations — pixel preprocessing and m/z
  alignment, FDR-controlled spatial annotation, spatial segmentation, and region-wise
  comparison.

  '
license: CC-BY-4.0
metadata:
  kind: composite-workflow
  techniques: []
  stage_count: 4
  member_skills:
  - imzml-spectral-data-processing
  - pixel-wise-peak-picking-of-imaging-spectra
  - mass-spectrometry-imaging-signal-preprocessing
  - imzml-maldimsi-data-preprocessing
  - maldi-metabolite-annotation-via-metaspace
  - mass-signature-database-annotation
  - molecular-identifier-mapping-between-imaging-and-lcmsms
  - exact-mass-metabolite-annotation
  - spatial-shrunken-centroids-pixel-clustering
  - tissue-region-annotation-from-lipid-co-localisation
  - mz-peak-spatial-colocalisation-analysis
  - mass-spectrometry-image-segmentation
  - spatial-cluster-differential-metabolite-analysis
  - pseudo-bulk-differential-metabolite-expression
  - pseudo-bulk-pooling-of-spatial-metabolite-intensities
  - differential-metabolite-abundance-testing--8274b5843fb2ecb1
  member_tools:
  - R
  - imzMLConverter
  - massPix
  - CellProfiler
  - Fiji
  - METASPACE
  - Python
  - SpaceM
  - Cardinal
  - SMAnalyst
  - Seurat
  - SpaMTP
  - edgeR
  coverage_gaps: []
  derived_from_workflows: []
  bound_by: perspicacite-semantic
  collection: https://w3id.org/holobiomicslab/asb-skill/collection/metabolomics/v3
schema_version: 0.3.0
attribution:
  generator: AgenticScienceBuilder
  promoter: Louis-Félix Nothias
  sponsor: CNRS & Université Côte d'Azur
---

# Mass Spectrometry Imaging — Spatial Metabolomics

## Summary

End-to-end MSI spatial metabolomics: from imzML to FDR-controlled ion-image annotations and anatomically-segmented, region-compared metabolite maps.


## When to use

Use when you have mass-spectrometry imaging data (imzML, e.g. MALDI/DESI) and want spatially-resolved metabolite annotations — pixel preprocessing and m/z alignment, FDR-controlled spatial annotation, spatial segmentation, and region-wise comparison.


## When NOT to use

- The data is not this platform.
- You need a single atomic step, not the full pipeline (use the leaf skill directly via the router).

## Stages

### Stage 1 — preprocess_imaging

**Goal:** load imzML, peak pick, align m/z across pixels

**EDAM operation:** http://edamontology.org/operation_3215

**Inputs:** imzML · **Outputs:** feature-table

**Candidate leaf skills:** `imzml-spectral-data-processing` (primary), `pixel-wise-peak-picking-of-imaging-spectra`, `mass-spectrometry-imaging-signal-preprocessing`, `imzml-maldimsi-data-preprocessing`

**Tools (primary):** R, imzMLConverter, massPix

**Other candidate tools:** MALDI-2, Snakemake, msiFlow, MSIght, Python, SciPy, Docker, imzy

**Grounding:** 3 KB(s); DOIs: 10.1007/s11306-017-1252-5, 10.1021/acs.jproteome.4c01140, 10.1038/s41467-024-55306-7

### Stage 2 — spatial_annotation

**Goal:** annotate ion images to metabolites with FDR control

**EDAM operation:** http://edamontology.org/operation_3803

**Inputs:** feature-table · **Outputs:** tsv

**Candidate leaf skills:** `maldi-metabolite-annotation-via-metaspace` (primary), `mass-signature-database-annotation`, `molecular-identifier-mapping-between-imaging-and-lcmsms`, `exact-mass-metabolite-annotation`

**Tools (primary):** CellProfiler, Fiji, METASPACE, Python, SpaceM

**Other candidate tools:** DESI-MSI, DIMPLE, LOTUS, MSiReader v1.03, Metlin Classic, METASPACE-ML, Cardinal, HMDB, LIPIDMAPS, R, SpaMTP

**Grounding:** 4 KB(s); DOIs: 10.1038/s41467-024-52213-9, 10.1038/s41592-021-01198-0, 10.1101/2024.10.31.621429, 10.1101/2025.09.22.677919

### Stage 3 — segmentation

**Goal:** spatial segmentation / clustering into regions

**EDAM operation:** http://edamontology.org/operation_3432

**Inputs:** feature-table · **Outputs:** tsv

**Candidate leaf skills:** `spatial-shrunken-centroids-pixel-clustering` (primary), `tissue-region-annotation-from-lipid-co-localisation`, `mz-peak-spatial-colocalisation-analysis`, `mass-spectrometry-image-segmentation`

**Tools (primary):** Cardinal, R, SMAnalyst

**Other candidate tools:** Seurat, SpaMTP, EdgeR, BiocParallel, matter

**Grounding:** 3 KB(s); DOIs: 10.1093/bioinformatics/btv146, 10.1101/2024.10.31.621429, 10.3390/biom15111562

### Stage 4 — region_statistics

**Goal:** compare metabolite intensities across spatial regions

**EDAM operation:** http://edamontology.org/operation_3659

**Inputs:** tsv, tsv · **Outputs:** tsv

**Candidate leaf skills:** `spatial-cluster-differential-metabolite-analysis` (primary), `pseudo-bulk-differential-metabolite-expression`, `pseudo-bulk-pooling-of-spatial-metabolite-intensities`, `differential-metabolite-abundance-testing--8274b5843fb2ecb1`

**Tools (primary):** Cardinal, R, Seurat, SpaMTP, edgeR

**Other candidate tools:** R (>= v4.1.0), SMAnalyst

**Grounding:** 2 KB(s); DOIs: 10.1101/2024.10.31.621429, 10.3390/biom15111562

## Grounding

Each stage carries the `kb_slugs`/`dois` of the leaves it draws on. Ground any stage against its source paper with the collection's `/ground` command or `bin/perspicacite_kb_bind.py` (Perspicacité KB; serverless local-clone fallback).

## Verification contract

`workflow.yaml` declares the stage graph and its typed outputs; the final stage emits the master deliverable. Automatic grading of that graph (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages as an outline — the structure is validated, the execution is not.

## Provenance

Generated by `compose_workflows.py` (semantic binding + EDAM-aware primary selection). `derived_from_workflows` lists the ASB per-paper workflows whose structure corroborated this pipeline; it is a provenance record, and no ablation experiment consuming it is released. Validated structurally by `validate_workflows.py` through `release_gate.py`: the collection is the hard-gated artefact and this workflows layer is additive.
