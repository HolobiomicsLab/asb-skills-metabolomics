---
name: allen-atlas-anatomical-mapping
description: Use when pixel-level clustering of a brain IMS feature matrix (e.g.,
  14,260 pixels x 3,044 m/z ions) must be validated against a reference anatomy atlas
  to confirm clusters recover known tissue divisions.
when_to_use_negative:
- Input is raw mass-spectrometry imaging data (not yet peak-picked into a pixel x
  m/z feature matrix) — run SMAnalyst's quality control and preprocessing first
- Goal is metabolite annotation or differential analysis between ROIs rather than
  validating spatial clustering against brain anatomy
- Tissue is not mouse brain (or has no comparable reference atlas), so Allen Atlas
  concordance cannot serve as a validation criterion
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: SMAnalyst
  role: 'Web platform that runs the UMAP-kmeans pixel clustering (one of its four
    Seurat-based algorithms: Seurat-LV, Seurat-LM, Seurat-SLM, UMAP-kmeans) and renders
    the spatial cluster map'
  repo: https://github.com/mzlab-research/SManalyst
- name: Seurat
  role: Clustering engine behind SMAnalyst's pixel-level clustering algorithms (Seurat
    5.1.0)
- name: R
  role: Runtime environment (>= v4.1.0) for local installation of the SMAnalyst Shiny
    app
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/biom15111562
    title: 'SMAnalyst: A Web Server for Spatial Metabolomic Data Analysis and Annotation'
  source_content_hash: sha256:7d8c1dc203785eb775ef042e7a63a1f8e17153c472dd30276f8813f4e46ec80c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SManalyst
  tools:
  - R
  - SMAnalyst
  - Seurat
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# allen-atlas-anatomical-mapping

## Summary

Annotates spatial metabolomics pixel-clustering results from a mouse brain coronal section against the Allen Mouse Brain Atlas, verifying that unsupervised clusters correspond to major anatomical divisions (cerebral cortex, hippocampus, midbrain, hindbrain, fiber tracts).

## When to use

Use when pixel-level clustering of a brain IMS feature matrix (e.g., 14,260 pixels x 3,044 m/z ions) must be validated against a reference anatomy atlas to confirm clusters recover known tissue divisions.

## When NOT to use

- Input is raw mass-spectrometry imaging data (not yet peak-picked into a pixel x m/z feature matrix) — run SMAnalyst's quality control and preprocessing first
- Goal is metabolite annotation or differential analysis between ROIs rather than validating spatial clustering against brain anatomy
- Tissue is not mouse brain (or has no comparable reference atlas), so Allen Atlas concordance cannot serve as a validation criterion

## Inputs

- Feature matrix CSV of a mouse brain coronal section: 14,260 spatial pixels x 3,044 unique m/z ion intensity columns, with columns 1-2 as X/Y pixel coordinates (NGDC OMIX009541 or SMAnalyst Tutorial Panel Example Data)
- Optional: noise-filtered ion subset (spatstat quadrat-test noise score >= 30)
- Allen Mouse Brain Atlas as anatomical reference

## Outputs

- Per-pixel cluster assignment table (14,260 rows, 25 cluster labels)
- Spatial pixel-cluster map over the tissue section (Figure 5B-style)
- Annotated atlas concordance map labeling cerebral cortex, hippocampus, midbrain, hindbrain, and fiber tracts

## How to apply

Load the Dataset 1 feature matrix (X/Y coordinate columns plus m/z intensity columns) from SMAnalyst's Tutorial Panel or NGDC OMIX accession OMIX009541, and optionally restrict to ions passing the noise filter (noise score = -log10 spatstat quadrat-test p-value, threshold >=30, removing ~39% of ions) since filtering improves cluster coherence. Run the UMAP-kmeans pixel clustering option — one of the four Seurat-based (Seurat 5.1.0) algorithms in SMAnalyst — with k=25 to match the 25 spatially heterogeneous categories reported for this dataset. Render the cluster assignments as a spatial pixel map over the tissue section (as in Figure 5B). Systematically compare the 25 clusters with the Allen Mouse Brain Atlas and annotate the major divisions (cerebral cortex, hippocampus, midbrain, hindbrain, fiber tracts) onto the cluster map. Rationale: anatomical concordance is the validation criterion showing that metabolomic spatial heterogeneity recapitulates neuroanatomy without using atlas labels during clustering.

## Related tools

- **SMAnalyst** (Web platform that runs the UMAP-kmeans pixel clustering (one of its four Seurat-based algorithms: Seurat-LV, Seurat-LM, Seurat-SLM, UMAP-kmeans) and renders the spatial cluster map) — https://github.com/mzlab-research/SManalyst
- **Seurat** (Clustering engine behind SMAnalyst's pixel-level clustering algorithms (Seurat 5.1.0))
- **R** (Runtime environment (>= v4.1.0) for local installation of the SMAnalyst Shiny app)

## Evaluation signals

- Output table contains cluster assignments for all 14,260 pixels across exactly 25 cluster labels
- Major anatomical divisions — cerebral cortex, hippocampus, midbrain, hindbrain, and fiber tracts — are qualitatively recognizable in the spatial cluster map
- Cluster map is consistent with the Allen Mouse Brain Atlas at the level of the paper's reported high concordance
- Spatial clusters are regionally coherent (contiguous tissue areas) rather than salt-and-pepper noise, especially when the noise-score >= 30 ion filter is applied
- Pixel coordinates in the assignment table match the X/Y columns of the input feature matrix

## Limitations

- Anatomical concordance is assessed qualitatively against the Allen Mouse Brain Atlas, not with a quantitative overlap metric
- k=25 is tuned to match the reported 25 spatially heterogeneous categories for this specific mouse brain dataset; other datasets may need different k
- The default noise filter (score >= 30) removes ~39% of ions, which improves cluster coherence but discards potentially informative low-spatial-autocorrelation peaks
- Clustering captures metabolomic heterogeneity only for the AFAD-ESI (+) mode, 100 micrometer resolution acquisition it was demonstrated on

## Evidence

- [readme] Pixel-level clustering: Identify tissue regions using 4 algorithms: Seurat-LV, Seurat-LM, Seurat-SLM, UMAP-kmeans: "Pixel-level clustering: Identify tissue regions using 4 algorithms: Seurat-LV, Seurat-LM, Seurat-SLM, UMAP-kmeans"
- [readme] Test dataset: Mouse brain coronal section ... 14,260 pixels x 3,044 ions: [evidence span withheld — anchor status: not_found]
- [intro] Adjust noise ion threshold (default: score ≥30): "Adjust noise ion threshold (default: score ≥30)"
- [readme] Raw data: NGDC OMIX Repository ID: OMIX009541: "Raw data: [NGDC OMIX Repository](https://ngdc.cncb.ac.cn/omix) ID: OMIX009541"
