---
name: msi-pixel-to-spot-binning
description: Use when pairing pixel-resolution MALDI imaging mass spectrometry data
  with spot-based ST data (e.g. Visium) from the same tissue section, and a shared
  spot-level object is needed for multi-omic integration.
when_to_use_negative:
- When the SM and ST data are already at matching resolution (no pixel-to-spot aggregation
  needed)
- When analysing spatial metabolomics alone without any paired spatial-omics modality
  — SpaMTP still supports SM-only workflows, but binning is only needed for integration
- When working with single-cell-resolution SM data exported from METASPACE, where
  spot binning to 55 µm Visium geometry does not apply
edam_topics:
- http://edamontology.org/topic_3334
- http://edamontology.org/topic_0203
tools:
- name: SpaMTP
  role: Provides AlignSpatialOmics and MapSpatialOmics to align modalities and bin
    MALDI pixels into Visium spots, building the joint Seurat object; ImageMZPlot
    validates binning fidelity
  repo: https://github.com/GenomicsMachineLearning/SpaMTP
- name: Cardinal
  role: Reads and processes the imzML MALDI pixel data before spatial alignment and
    binning
  repo: https://github.com/GenomicsMachineLearning/SpaMTP
- name: Seurat
  role: Provides the Seurat class object foundation on which the binned SPM/SPT assays
    and downstream WNN integration are implemented
  repo: https://github.com/GenomicsMachineLearning/SpaMTP
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.10.31.621429
    title: 'SpaMTP: Integrative Statistical Analysis and Visualisation of Spatial
      Metabolomics and Transcriptomics data'
  source_content_hash: sha256:da9a325d5f1395998f817365b80211601e3b278131049083a9c7d15a09a11f91
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/GenomicsMachineLearning/SpaMTP
  tools:
  - Cardinal
  - R
  - Seurat
  - SpaMTP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3334
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# MSI pixel-to-spot binning

## Summary

Aligns an MALDI-TOF spatial metabolomics image (pixel-level imzML) onto paired spatial transcriptomics capture spots (e.g. 55 µm 10X Visium) and bins MALDI pixels into colocalised spots, producing a single Seurat object holding both 'SPM' and 'SPT' assays for integrative analysis.

## When to use

Use when pairing pixel-resolution MALDI imaging mass spectrometry data with spot-based ST data (e.g. Visium) from the same tissue section, and a shared spot-level object is needed for multi-omic integration.

## When NOT to use

- When the SM and ST data are already at matching resolution (no pixel-to-spot aggregation needed)
- When analysing spatial metabolomics alone without any paired spatial-omics modality — SpaMTP still supports SM-only workflows, but binning is only needed for integration
- When working with single-cell-resolution SM data exported from METASPACE, where spot binning to 55 µm Visium geometry does not apply

## Inputs

- MALDI-TOF imaging MS data in imzML format (e.g. FMP-10 and DHB matrix runs of the Parkinson's mouse brain section)
- Paired 10X Visium spatial transcriptomics data from the same tissue section (55 µm spots)
- Optionally annotated metabolite m/z values (e.g. via AddCustomAnnotation with FMP-10 derivative rules)

## Outputs

- A single SpaMTP Seurat object with colocalised 'SPM' (spatial metabolomics) and 'SPT' (spatial transcriptomics) assays on shared spot barcodes
- Spot-level metabolite abundance matrix produced by pixel binning

## How to apply

Load the MALDI imzML via Cardinal::readImzML and the paired Visium data, then run SpaMTP's AlignSpatialOmics to align the modalities and MapSpatialOmics to bin MALDI pixels into the 55 µm Visium spots they colocalise with, assigning binned metabolite intensities to spot barcodes. The result is one SpaMTP Seurat object carrying an 'SPM' assay (metabolites) and an 'SPT' assay (transcriptomics) on the same spots, which is the prerequisite for MultiOmicIntegration (WNN with modality weighting metabolite/gene = 0.6/0.4 in the Parkinson's mouse brain case study). Before clustering, verify binning fidelity by comparing spot-binned versus raw-pixel expression of a spatially informative metabolite (e.g. dopamine, HMDB:HMDB0000073) with ImageMZPlot. The rationale is that ST packages cannot consume pixel-level SM data, so aggregation to the shared spot grid is what makes SM–ST integration computable at all.

## Related tools

- **SpaMTP** (Provides AlignSpatialOmics and MapSpatialOmics to align modalities and bin MALDI pixels into Visium spots, building the joint Seurat object; ImageMZPlot validates binning fidelity) — https://github.com/GenomicsMachineLearning/SpaMTP
- **Cardinal** (Reads and processes the imzML MALDI pixel data before spatial alignment and binning) — https://github.com/GenomicsMachineLearning/SpaMTP
- **Seurat** (Provides the Seurat class object foundation on which the binned SPM/SPT assays and downstream WNN integration are implemented) — https://github.com/GenomicsMachineLearning/SpaMTP

## Evaluation signals

- Spot-binned and raw-pixel expression profiles of a spatially validated metabolite (e.g. dopamine) agree when compared with ImageMZPlot
- Every MALDI pixel is assigned to exactly one colocalised 55 µm Visium spot, and the resulting object contains both 'SPM' and 'SPT' assays on the same spot barcodes
- Downstream, ST-only, SM-only, and WNN-integrated clusterings can be computed on the same spot set and tabulated per spot (wnn_cluster_comparison.csv)
- With the paper's 0.6/0.4 metabolite/gene weighting, integrated clustering recovers two opposing striatal spot populations (lesioned vs intact) that ST-only clustering misses

## Limitations

- No explicit SpaMTP version number is stated in the methods; pin the installed commit/version yourself
- Binning aggregates pixel-level SM detail to spot resolution, so sub-spot metabolic heterogeneity is lost and fidelity depends on spot-pixel colocalisation
- The paper demonstrates the weighting (0.6/0.4) on a Parkinson's mouse brain Visium/MALDI-TOF pair; other tissues or resolutions may need re-tuned alignment and weights

## Evidence

- [other] Align and map metabolic signals to the 55 µm Visium spots with AlignSpatialOmics and MapSpatialOmics, binning MALDI pixels into colocalised spots to build a single SpaMTP Seurat object with 'SPM' and 'SPT' assays.: [evidence span withheld — anchor status: not_found]
- [readme] SpaMTP builds on a Seurat class object combining Cardinal SM processing with Seurat cell-centric analyses.: [evidence span withheld — anchor status: external_doc]
- [methods] Verify spot-binning fidelity by comparing spot-binned versus raw pixel expression for dopamine: [evidence span withheld — anchor status: not_found]
