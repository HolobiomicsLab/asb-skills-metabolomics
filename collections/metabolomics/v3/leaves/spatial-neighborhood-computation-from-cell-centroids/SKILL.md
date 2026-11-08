---
name: spatial-neighborhood-computation-from-cell-centroids
description: Use when per-cell intensity tables and centroid coordinates are available
  and the goal is to quantify neighborhood-dependent behavior (e.g., metabolite competition
  between tumor cells and T-cells) as a function of proximity to CD31+ endothelial
  cells or cell-cell contact.
when_to_use_negative:
- Single-cell segmentation or intensity extraction has not yet been performed — the
  skill requires per-cell intensity tables and centroids as input, not raw TOF-SIMS/IMC
  images
- IMC and SIMS modalities are not yet registered to a common coordinate frame (notebook
  02 registration must precede spatial neighborhood computation)
- The analysis goal is cell-type clustering or VAE joint embedding rather than spatial
  neighbor-dependent metabolite competition
edam_operation: http://edamontology.org/operation_2940
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_3382
tools:
- name: scSpaMet
  role: Framework providing the 'spatial' folder with custom spatial interaction functions
    for neighborhood definition, notebook 06 for competition analysis, and 'utils.py'
    plotting/io helpers for rendering spatial maps
  repo: github.com/coskunlab/ScSpaMet
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-43917-5
    title: Single-cell spatial metabolomics with cell-type specific protein profiling
      for tissue systems biology
  source_content_hash: sha256:7d20355ad6e0b4e67640c606eeb9811276ac00a16720a166c6bfbe7e4c5b4092
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scSpaMet
  edam_operation: http://edamontology.org/operation_2940
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# spatial-neighborhood-computation-from-cell-centroids

## Summary

Builds per-cell spatial neighbor sets from single-cell centroid coordinates extracted from joint TOF-SIMS/IMC imaging, enabling local metabolite competition analysis (own intensity / mean neighbor intensity) between cell types such as CD68+/T-cells and tumor cells in lung tumor tissue.

## When to use

Use when per-cell intensity tables and centroid coordinates are available and the goal is to quantify neighborhood-dependent behavior (e.g., metabolite competition between tumor cells and T-cells) as a function of proximity to CD31+ endothelial cells or cell-cell contact.

## When NOT to use

- Single-cell segmentation or intensity extraction has not yet been performed — the skill requires per-cell intensity tables and centroids as input, not raw TOF-SIMS/IMC images
- IMC and SIMS modalities are not yet registered to a common coordinate frame (notebook 02 registration must precede spatial neighborhood computation)
- The analysis goal is cell-type clustering or VAE joint embedding rather than spatial neighbor-dependent metabolite competition

## Inputs

- Per-cell metabolite intensity table from single-cell extraction (notebook 04 outputs)
- Cell centroid coordinates (x, y) per imaging region
- Registered region-of-interest assignments from IMC-SIMS registration (notebook 02 outputs)
- Cell-type/phenotype labels (e.g., CD68+/T-cells, tumor cells) and marker channels such as CD31+ for endothelial distance

## Outputs

- Per-cell neighbor sets (20-um radius neighborhoods or direct contact pairs)
- Per-cell, per-channel competition ratios (own intensity / mean neighbor intensity)
- Competition-ratio spatial maps, one per imaging region of interest

## How to apply

Load the per-cell metabolite intensity table and cell centroid coordinates from the single-cell extraction outputs (notebook 04) together with the registered region-of-interest assignments from notebook 02. Use the custom spatial interaction functions in the 'spatial' folder of the scSpaMet repository to define each cell's neighborhood either by a 20-um radius around the centroid or by direct cell-cell contact. For each metabolite channel and cell, compute the competition ratio as the cell's own intensity divided by the mean neighbor intensity, following the competition analysis in notebook '06 Metabolite analysis in different regions'. Render the per-cell ratios back onto tissue coordinates as per-region spatial maps using the plotting helpers in 'utils.py'. The rationale is that mass channels such as 148.0 m/z Methionine and lipid fragments show competition behavior that depends on distance to CD31+ endothelial cells, so neighborhood definition directly controls the biological interpretation.

## Related tools

- **scSpaMet** (Framework providing the 'spatial' folder with custom spatial interaction functions for neighborhood definition, notebook 06 for competition analysis, and 'utils.py' plotting/io helpers for rendering spatial maps) — github.com/coskunlab/ScSpaMet

## Examples

```
# In a notebook 06 workflow with scSpaMet: from spatial.spatialInteraction import buildNeighborSets; neighbors = buildNeighborSets(centroids, radius_um=20)
```

## Evaluation signals

- A competition-ratio spatial map file is produced for every region of interest
- All computed competition-ratio values are finite and positive, indicating each cell had at least one measurable neighbor
- Per-cell competition ratios reproduce the reported directionality: mass channels 148.0 m/z Methionine, 42.0 m/z lipid, 94.0 m/z, and 45 m/z lipid fragments show higher expression in tumor cells versus T-cells at further distance from CD31+ endothelial cells, while 145.0 m/z Glutamine shows higher tumor-cell expression near CD31+ cells
- Neighborhood definition is explicitly recorded (20-um radius vs. direct contact) and consistent across regions being compared
- Centroid coordinates align with the registered ROI assignments from notebook 02 (cross-modality registration validated by Histone 3/Intercalator in IMC and Phosphate 79 m/z in TOF-SIMS)

## Limitations

- Neighborhood definition (20-um radius vs. direct contact) is a modeling choice that materially changes competition-ratio values; results should be interpreted relative to the chosen definition
- TOF-SIMS imaging regions are relatively small due to the trade-off between imaging speed and spatial resolution, limiting the number of neighborhoods per region
- Metabolite annotation for some channels (e.g., unnamed 94.0 m/z channel and lipid fragments) remains tentative, complicating biological interpretation of competition ratios
- Competition ratios require each cell to have at least one measurable neighbor; edge cells without neighbors yield undefined values

## Evidence

- [readme] Neighborhood definition by radius or contact: ""spatial" folder contains custom code for spatial interaction functions"
- [readme] Competition analysis located in notebook 06: "06 Metabolite analysis in different regions (metabolite difference, distance analysis, competition analysis)"
- [abstract] Article-level support for the underlying biological phenomenon: "The scSpaMet analysis revealed cell type-dependent metabolite profiles and local metabolite competition of neighboring single cells in human tissues"
- [results] Prerequisite single-cell extraction of intensities and locations: "Single-cell segmentation was used to extract the protein and metabolite expression levels and their spatial locations"
- [results] Registration prerequisite for aligning modalities before spatial analysis: "a cross-modality single-cell registration pipeline was developed utilizing Histone 3 and Intercalator markers in the IMC dataset, and Phosphate 79 m/z channels in TOF-SIMS dataset"
