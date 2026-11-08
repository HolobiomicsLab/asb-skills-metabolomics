---
name: multi-omic-wnn-integration
description: Use when a paired SM (e.g. MALDI-TOF imzML) and ST (e.g. 10X Visium)
  dataset has been mapped onto common spatial units (spots/pixels) and unimodal clustering
  of one assay misses metabolically defined populations.
when_to_use_negative:
- Only one modality is present (SM-only or ST-only) — run Seurat FindNeighbors/FindClusters
  on the single assay instead; WNN requires co-localised 'SPM' and 'SPT' assays.
- The SM and ST measurements come from different, unaligned tissue sections — AlignSpatialOmics/MapSpatialOmics
  binning cannot produce valid colocalised spots, so integration weights would be
  meaningless.
- SM pixels have not yet been binned to the transcriptomic spot grid and the question
  concerns pixel-level (sub-spot) metabolic structure — spot binning deliberately
  sacrifices pixel resolution.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SpaMTP
  role: Provides AlignSpatialOmics/MapSpatialOmics for spot binning, AddCustomAnnotation
    for metabolite m/z annotation, and MultiOmicIntegration for the weighted-nearest-neighbour
    SM+ST integration
  repo: https://github.com/GenomicsMachineLearning/SpaMTP
- name: Seurat
  role: Underlying cell-centric framework whose weighted nearest-neighbours method
    and FindNeighbors/FindClusters are used for baseline ST-only clustering and the
    integrated embedding
- name: Cardinal
  role: Processes the MALDI-TOF metabolomics data (e.g. readImzML of the mouse liver
    MALDI dataset) upstream of integration
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.10.31.621429
    title: 'SpaMTP: Integrative Statistical Analysis and Visualisation of Spatial
      Metabolomics and Transcriptomics data'
  source_content_hash: sha256:07f6907dd2d35e5d02bbf8f0df4d8f8876881615d0b8ec52b9f6842fdb76fc91
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# multi-omic-wnn-integration

## Summary

Weighted-nearest-neighbour (WNN) integration of spatial metabolomics (SM) and spatial transcriptomics (ST) data into a single clustering solution, as implemented by SpaMTP's MultiOmicIntegration on a Seurat-class object holding co-registered 'SPM' and 'SPT' assays.

## When to use

Use when a paired SM (e.g. MALDI-TOF imzML) and ST (e.g. 10X Visium) dataset has been mapped onto common spatial units (spots/pixels) and unimodal clustering of one assay misses metabolically defined populations.

## When NOT to use

- Only one modality is present (SM-only or ST-only) — run Seurat FindNeighbors/FindClusters on the single assay instead; WNN requires co-localised 'SPM' and 'SPT' assays.
- The SM and ST measurements come from different, unaligned tissue sections — AlignSpatialOmics/MapSpatialOmics binning cannot produce valid colocalised spots, so integration weights would be meaningless.
- SM pixels have not yet been binned to the transcriptomic spot grid and the question concerns pixel-level (sub-spot) metabolic structure — spot binning deliberately sacrifices pixel resolution.

## Inputs

- Paired MALDI-TOF SM data (imzML, e.g. FMP-10 and DHB matrix runs) from the same tissue section as the ST data
- Paired spatial transcriptomics data (10X Visium, 55 µm spot grid)
- Metabolite annotations for m/z values (e.g. FMP-10 derivative annotations added via SpaMTP's AddCustomAnnotation)
- A SpaMTP Seurat-class object with co-registered 'SPM' (metabolite) and 'SPT' (transcript) assays

## Outputs

- Integrated WNN embedding of spots with cluster assignments
- Per-spot cluster-assignment table for SM-only, ST-only, and WNN-integrated clustering (wnn_cluster_comparison.csv)
- Annotation of integrated clusters by anatomical region (e.g. striatum: lesioned vs intact)

## How to apply

First bin the MALDI pixels into the ST spot grid (55 µm Visium spots in the Parkinson's mouse brain case) with SpaMTP's AlignSpatialOmics and MapSpatialOmics, yielding one SpaMTP Seurat object with 'SPM' and 'SPT' assays, and verify spot-binning fidelity by comparing spot-binned versus raw pixel signal for a landmark metabolite (e.g. dopamine via ImageMZPlot). Run ST-only (Seurat FindNeighbors/FindClusters on 'SPT') and SM-only (on 'SPM') clustering as baselines. Then apply SpaMTP's MultiOmicIntegration, a Seurat-based weighted nearest-neighbours approach, using optimised modality weighting of metabolite/gene = 0.6/0.4 (reflecting the greater informativeness of metabolic signal in this system), and cluster the resulting integrated embedding. Assemble a per-spot table of SM-only, ST-only and WNN-integrated cluster assignments to judge whether integration recovers populations invisible to either modality alone. The rationale is that no existing package supports integrative SM + other spatial-omics analysis; SpaMTP bridges Cardinal's SM processing with Seurat's cell-centric statistics to make the integration end-to-end.

## Related tools

- **SpaMTP** (Provides AlignSpatialOmics/MapSpatialOmics for spot binning, AddCustomAnnotation for metabolite m/z annotation, and MultiOmicIntegration for the weighted-nearest-neighbour SM+ST integration) — https://github.com/GenomicsMachineLearning/SpaMTP
- **Seurat** (Underlying cell-centric framework whose weighted nearest-neighbours method and FindNeighbors/FindClusters are used for baseline ST-only clustering and the integrated embedding)
- **Cardinal** (Processes the MALDI-TOF metabolomics data (e.g. readImzML of the mouse liver MALDI dataset) upstream of integration)

## Examples

```
devtools::install_github("GenomicsMachineLearning/SpaMTP"); integrated <- MultiOmicIntegration(spa_obj, modality.weights = c(SPM = 0.6, SPT = 0.4))
```

## Evaluation signals

- Integrated (WNN) clustering recovers two striatal spot populations with opposing metabolic activity (lesioned vs intact) that ST-only clustering misses.
- The optimised modality weighting metabolite/gene = 0.6/0.4 is used, matching the value reported in the paper.
- Spot-binning fidelity is confirmed by comparing spot-binned versus raw pixel expression for dopamine (e.g. with ImageMZPlot) before integration.
- wnn_cluster_comparison.csv contains per-spot assignments for all three solutions (SM-only, ST-only, WNN) on the same spot identifiers.
- Integrated clusters localise to expected anatomical regions (striatum, lesioned vs intact hemisphere) when visualised spatially.

## Limitations

- The 0.6/0.4 metabolite/gene modality weighting was optimised for the Parkinson's mouse brain case study; other tissues may require re-optimisation.
- No explicit SpaMTP version number, DOI, or Zenodo/GitHub release identifier is stated in the article's methods/discussion, complicating exact reproducibility.
- METASPACE and pySM annotation routes are optimised for high-resolution instruments (FT-ICR, Orbitrap) and do not apply to most TOF data, so annotation quality for MALDI-TOF inputs depends on SpaMTP's own annotation functions.
- No comparison benchmarks, metrics, or quantitative evaluation results are reported for the integration itself in the article's discussion section.

## Evidence

- [readme] SpaMTP is an R package designed for the integrative analysis of spatial metabolomics and spatial transcriptomics data.: [evidence span withheld — anchor status: external_doc]
