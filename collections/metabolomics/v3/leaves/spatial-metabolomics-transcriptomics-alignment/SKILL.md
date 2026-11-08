---
name: spatial-metabolomics-transcriptomics-alignment
description: Use when you hold paired SM (imzML, e.g. MALDI-TOF) and ST (e.g. 10X
  Visium) measurements from the same or adjacent tissue section and need metabolite
  signals mapped onto ST spots for joint clustering or WNN integration.
when_to_use_negative:
- Input is a single modality only (SM or ST alone) — use Cardinal/Seurat modality-specific
  workflows instead of cross-omic alignment
- SM data are single-cell resolution and you want sub-spot analysis — SpaMTP bins
  pixels into capture spots (e.g. 55 µm Visium), so pixel-level spatial detail is
  lost
- Data were generated on high-resolution instruments where METASPACE/pySM-style annotation
  is expected; SpaMTP's spot-mapping pipeline targets TOF-style SM data that those
  tools do not cover
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: SpaMTP
  role: Provides AlignSpatialOmics and MapSpatialOmics to co-register SM pixels to
    ST spots and build the combined Seurat object; MultiOmicIntegration for downstream
    WNN clustering
  repo: https://github.com/GenomicsMachineLearning/SpaMTP
- name: Cardinal
  role: Reads and preprocesses the MALDI-TOF imzML data before spot binning (SpaMTP
    bridges Cardinal processing into Seurat-style analysis)
  repo: https://github.com/GenomicsMachineLearning/SpaMTP
- name: Seurat
  role: Underlying object model and clustering (FindNeighbors/FindClusters) on the
    spot-mapped assays; WNN basis for MultiOmicIntegration
  repo: https://github.com/GenomicsMachineLearning/SpaMTP
- name: ms1_id
  role: Optional upstream pseudo-MS/MS annotation of imaging MS data (imzML) against
    indexed GNPS libraries before mapping metabolite identities
  repo: https://github.com/Philipbear/ms1_id
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.10.31.621429
    title: 'SpaMTP: Integrative Statistical Analysis and Visualisation of Spatial
      Metabolomics and Transcriptomics data'
  source_content_hash: sha256:aca19fe9a4b8c966ba1ec7ecce358cab44bbf03e88ea5845444469ada4bfb53f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/GenomicsMachineLearning/SpaMTP
  tools:
  - Cardinal
  - R
  - Seurat
  - SpaMTP
  - ms1_id
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# spatial-metabolomics-transcriptomics-alignment

## Summary

Spatial co-registration and spot-binning of MALDI-TOF spatial metabolomics (SM) pixels onto paired spatial transcriptomics (ST) capture areas — e.g. 55 µm 10X Visium spots — using SpaMTP to build a single Seurat object holding both 'SPM' and 'SPT' assays for downstream multi-omic integration.

## When to use

Use when you hold paired SM (imzML, e.g. MALDI-TOF) and ST (e.g. 10X Visium) measurements from the same or adjacent tissue section and need metabolite signals mapped onto ST spots for joint clustering or WNN integration.

## When NOT to use

- Input is a single modality only (SM or ST alone) — use Cardinal/Seurat modality-specific workflows instead of cross-omic alignment
- SM data are single-cell resolution and you want sub-spot analysis — SpaMTP bins pixels into capture spots (e.g. 55 µm Visium), so pixel-level spatial detail is lost
- Data were generated on high-resolution instruments where METASPACE/pySM-style annotation is expected; SpaMTP's spot-mapping pipeline targets TOF-style SM data that those tools do not cover

## Inputs

- MALDI-TOF imaging MS data in imzML/ibd format (e.g. FMP-10 or DHB matrix runs) read via Cardinal
- Paired spatial transcriptomics data (e.g. 10X Visium spot-level counts) from the same tissue section
- Optional metabolite annotation index (e.g. FMP-10 derivative annotations via AddCustomAnnotation)
- Tissue-region metadata (e.g. lesioned vs intact striatum) for downstream interpretation

## Outputs

- A single SpaMTP Seurat object containing colocalised 'SPM' (metabolite) and 'SPT' (transcript) assays at spot resolution
- Spot-binned metabolite intensity matrices aligned to ST spot coordinates
- Per-spot cluster assignments from SM-only, ST-only, and WNN-integrated clustering (e.g. wnn_cluster_comparison.csv)

## How to apply

Load the SM run (e.g. FMP-10 or DHB matrix MALDI-TOF imzML, processed with Cardinal) and the paired ST data into SpaMTP, then use AlignSpatialOmics to co-register the two coordinate systems and MapSpatialOmics to bin MALDI pixels into colocalised ST spots (55 µm for Visium), producing one SpaMTP Seurat object with an 'SPM' (metabolite) and 'SPT' (transcript) assay. Optionally annotate m/z values first with the FMP-10 derivative rules (MALDIMatrixRules(maldi_matrix = "FMP-10", polarity = "positive")) so mapped features carry metabolite names. Verify spot-binning fidelity by comparing spot-binned versus raw-pixel expression of a known spatially patterned metabolite such as dopamine (HMDB:HMDB0000073) with ImageMZPlot before trusting downstream maps. The output object feeds modality-specific baseline clustering (Seurat FindNeighbors/FindClusters per assay) or SpaMTP's MultiOmicIntegration WNN integration with optimised modality weighting (metabolite/gene = 0.6/0.4 in the Parkinson's mouse-brain case study).

## Related tools

- **SpaMTP** (Provides AlignSpatialOmics and MapSpatialOmics to co-register SM pixels to ST spots and build the combined Seurat object; MultiOmicIntegration for downstream WNN clustering) — https://github.com/GenomicsMachineLearning/SpaMTP
- **Cardinal** (Reads and preprocesses the MALDI-TOF imzML data before spot binning (SpaMTP bridges Cardinal processing into Seurat-style analysis)) — https://github.com/GenomicsMachineLearning/SpaMTP
- **Seurat** (Underlying object model and clustering (FindNeighbors/FindClusters) on the spot-mapped assays; WNN basis for MultiOmicIntegration) — https://github.com/GenomicsMachineLearning/SpaMTP
- **ms1_id** (Optional upstream pseudo-MS/MS annotation of imaging MS data (imzML) against indexed GNPS libraries before mapping metabolite identities) — https://github.com/Philipbear/ms1_id

## Examples

```
devtools::install_github("GenomicsMachineLearning/SpaMTP")  # then: AlignSpatialOmics() + MapSpatialOmics() to bin MALDI pixels onto 55 µm Visium spots
```

## Evaluation signals

- Spot-binned vs raw-pixel expression of a spatially patterned metabolite (e.g. dopamine, HMDB:HMDB0000073) shows concordant spatial structure when plotted with ImageMZPlot
- Resulting Seurat object contains both 'SPM' and 'SPT' assays with one-to-one spot correspondence across modalities
- ST-only clustering on the 'SPT' assay serves as a baseline; the integrated (0.6/0.4 metabolite/gene weighting) clustering should reveal additional structure (e.g. two striatal spot populations with opposing metabolic activity, lesioned vs intact) missed by ST-only clustering
- All MALDI pixels are accounted for in the binning step with no unassigned/ambiguous colocalisation to Visium spots
- Integrated clusters localise to expected anatomy (e.g. striatum) when visualised spatially

## Limitations

- Spot binning aggregates MALDI pixels into ST spot resolution (55 µm for Visium), sacrificing single-pixel spatial detail available in SM data
- Existing tools like METASPACE and pySM do not apply to most TOF data, so annotation quality for FMP-10 derivative peaks depends on SpaMTP's own annotation pipeline
- The multi-omic weighting (0.6/0.4) is dataset-optimised, not universal; modality weights may need re-tuning for other tissue/instrument combinations
- No explicit SpaMTP version number is given in the article; pin the GitHub commit or Apptainer image (e.g. spamtp-1.1.sif) for reproducibility

## Evidence

- [readme] SpaMTP bridges Cardinal and Seurat for integrative spatial-omics analysis.: [evidence span withheld — anchor status: external_doc]
- [readme] SpaMTP designed for integrative SM/ST analysis in R.: [evidence span withheld — anchor status: external_doc]
