---
name: modality-weighted-spatial-clustering
description: Use when you have colocalised SM ('SPM' assay) and ST ('SPT' assay) data
  on shared spatial units — e.g. MALDI pixels binned into Visium spots — and need
  integrated spot-level clusters that a single modality alone cannot resolve.
when_to_use_negative:
- 'You hold only one modality (SM-only or ST-only): run standard Seurat FindNeighbors/FindClusters
  on the single assay instead — MultiOmicIntegration requires both ''SPM'' and ''SPT''
  assays on shared coordinates.'
- 'The spatial units of the two modalities do not colocalise and cannot be binned
  (e.g. unpaired tissue sections or incompatible resolutions): MapSpatialOmics spot-binning
  must be feasible first, otherwise integration is built on misaligned measurements.'
- 'Modality-specific quality checks fail (e.g. spot-binned metabolic signal diverges
  from raw-pixel signal for key metabolites like dopamine): fix the mapping before
  clustering, since WNN clusters will inherit binning artefacts.'
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3382
tools:
- name: SpaMTP
  role: R package supplying MultiOmicIntegration (Seurat-based WNN) plus AlignSpatialOmics/MapSpatialOmics
    for building the binned paired object; install via devtools::install_github("GenomicsMachineLearning/SpaMTP")
  repo: https://github.com/GenomicsMachineLearning/SpaMTP
- name: Seurat
  role: Foundation of SpaMTP's Seurat Class Object; FindNeighbors/FindClusters used
    for both single-modality baselines and the integrated embedding clustering
  repo: https://satijalab.org/seurat/
- name: Cardinal
  role: Upstream processing of MALDI imzML metabolomics data that feeds the 'SPM'
    assay before integration
- name: R
  role: Runtime language for the SpaMTP/Seurat integrative clustering workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.10.31.621429
    title: 'SpaMTP: Integrative Statistical Analysis and Visualisation of Spatial
      Metabolomics and Transcriptomics data'
  source_content_hash: sha256:f50004d6572fec8065855a68f2ff416516d38e5354a2e4ab6e6c25f76e744174
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
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# modality-weighted-spatial-clustering

## Summary

SpaMTP's MultiOmicIntegration performs Seurat-based weighted-nearest-neighbour (WNN) integration of spatial metabolomics and spatial transcriptomics assays so that per-spot clusters reflect combined metabolite and gene expression, recovering spot populations (e.g. opposing striatal metabolic activity) that single-modality clustering misses.

## When to use

Use when you have colocalised SM ('SPM' assay) and ST ('SPT' assay) data on shared spatial units — e.g. MALDI pixels binned into Visium spots — and need integrated spot-level clusters that a single modality alone cannot resolve.

## When NOT to use

- You hold only one modality (SM-only or ST-only): run standard Seurat FindNeighbors/FindClusters on the single assay instead — MultiOmicIntegration requires both 'SPM' and 'SPT' assays on shared coordinates.
- The spatial units of the two modalities do not colocalise and cannot be binned (e.g. unpaired tissue sections or incompatible resolutions): MapSpatialOmics spot-binning must be feasible first, otherwise integration is built on misaligned measurements.
- Modality-specific quality checks fail (e.g. spot-binned metabolic signal diverges from raw-pixel signal for key metabolites like dopamine): fix the mapping before clustering, since WNN clusters will inherit binning artefacts.

## Inputs

- SpaMTP Seurat object containing paired spatial modalities: 'SPM' assay (binned spatial metabolomics m/z intensities) and 'SPT' assay (spatial transcriptomics gene counts) on the same spot coordinates
- MALDI-TOF imzML pixel-level data (e.g. Parkinson's mouse brain FMP-10/DHB matrix run)
- Spatial transcriptomics spot-level data (e.g. 10X Visium, 55 µm spots) from the same tissue section
- Optimised modality weight vector (e.g. metabolite 0.6, gene 0.4)

## Outputs

- Integrated WNN embedding for all spots across both modalities
- Per-spot integrated cluster assignments comparable against SM-only and ST-only cluster tables
- Identification of spatially localised spot populations (e.g. lesioned vs intact striatal spots with opposing metabolic activity) absent from single-modality clustering

## How to apply

Build one SpaMTP Seurat object holding both modalities: map MALDI-TOF metabolic signals onto transcriptomic spots with AlignSpatialOmics and MapSpatialOmics (e.g. 55 µm Visium spots receiving binned MALDI pixels), producing 'SPM' and 'SPT' assays on identical coordinates. Validate spot-binning fidelity first (e.g. compare spot-binned vs raw-pixel dopamine signal with ImageMZPlot) before clustering, since integration quality depends on faithful mapping. Run modality-only baselines — Seurat FindNeighbors/FindClusters separately on 'SPT' and 'SPM' assays — to establish what each modality resolves alone. Then run SpaMTP's MultiOmicIntegration, a Seurat-based weighted-nearest-neighbours approach with optimised modality weighting (metabolite:gene = 0.6:0.4, as reported for the Parkinson's mouse brain case), and cluster the resulting integrated embedding. The rationale for weighting rather than equal fusion is that metabolic information carried the signal discriminating lesioned-vs-intact striatal populations that ST-only clustering missed.

## Related tools

- **SpaMTP** (R package supplying MultiOmicIntegration (Seurat-based WNN) plus AlignSpatialOmics/MapSpatialOmics for building the binned paired object; install via devtools::install_github("GenomicsMachineLearning/SpaMTP")) — https://github.com/GenomicsMachineLearning/SpaMTP
- **Seurat** (Foundation of SpaMTP's Seurat Class Object; FindNeighbors/FindClusters used for both single-modality baselines and the integrated embedding clustering) — https://satijalab.org/seurat/
- **Cardinal** (Upstream processing of MALDI imzML metabolomics data that feeds the 'SPM' assay before integration)
- **R** (Runtime language for the SpaMTP/Seurat integrative clustering workflow)

## Examples

```
spamtp_obj <- MultiOmicIntegration(spamtp_obj, modality_weights = c(SPM = 0.6, SPT = 0.4)); spamtp_obj <- FindNeighbors(spamtp_obj, reduction = "wnn"); spamtp_obj <- FindClusters(spamtp_obj)
```

## Evaluation signals

- Both 'SPM' and 'SPT' assays exist in the same Seurat object and share identical spot identifiers/coordinates before MultiOmicIntegration is invoked.
- Spot-binning fidelity check passes: spot-binned and raw-pixel expression agree for target metabolites (e.g. dopamine) when plotted with ImageMZPlot.
- Modality weighting matches the optimised ratio (0.6 metabolite / 0.4 gene) rather than arbitrary 0.5/0.5.
- Integrated clusters are compared tab-side against SM-only and ST-only cluster assignments (e.g. wnn_cluster_comparison.csv) to show clusters the single modalities missed.
- Integrated clustering recovers biologically expected spatial structure — e.g. two striatal spot populations with opposing metabolic activity (lesioned vs intact) — that ST-only clustering does not resolve.

## Limitations

- Integration quality is bounded by spot-binning fidelity; misaligned or over-binned MALDI pixels propagate artefacts into WNN clusters.
- The optimised 0.6/0.4 metabolite-to-gene weighting was reported for the Parkinson's mouse brain case; other tissues or platforms may require re-optimisation.
- The biorxiv version does not report explicit quantitative benchmarks, metrics, or evaluation results for the integration step — validation rests on biological recovery (lesioned vs intact striata) rather than numeric accuracy figures.

## Evidence

- [abstract] SpaMTP is a versatile software that implements an end-to-end integrative analysis of spatial metabolomics and transcriptomics data: [evidence span withheld — anchor status: not_found]
- [methods] MultiOmicIntegration is Seurat-based WNN with optimised 0.6/0.4 weighting: [evidence span withheld — anchor status: not_found]
- [methods] binning MALDI pixels into colocalised spots via AlignSpatialOmics and MapSpatialOmics: [evidence span withheld — anchor status: not_found]
