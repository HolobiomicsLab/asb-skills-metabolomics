---
name: metabolite-competition-ratio-calculation
description: Use when you have per-cell metabolite intensities plus centroid coordinates
  from segmented TOF-SIMS/IMC data and need to quantify how metabolite abundance in
  one cell type depends on neighboring cells or distance to structures like CD31+
  vessels.
when_to_use_negative:
- Input is raw TOF-SIMS or IMC image data that has not yet been segmented and extracted
  to per-cell intensities (use notebooks 01-04 first)
- IMC and SIMS modalities are not yet registered to a common coordinate frame (use
  notebook 02 registration first)
- The question is cell-type identification or joint embedding of protein-metabolite
  states rather than neighbor-dependent metabolite competition (use notebook 05 VAE
  embedding instead)
tools:
- name: scSpaMet
  role: Framework providing the competition analysis notebook (06 Metabolite analysis
    in different regions), the 'spatial' folder with custom spatial interaction functions,
    and 'utils.py' plotting/IO helpers
  repo: github.com/coskunlab/ScSpaMet
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-43917-5
    title: Single-cell spatial metabolomics with cell-type specific protein profiling
      for tissue systems biology
  source_content_hash: sha256:d911d769de6feaecd08b1b41c0eef6416b39b428cc5843ffb43d5870e2d6a726
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scSpaMet
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# metabolite-competition-ratio-calculation

## Summary

Computes per-cell metabolite competition ratios (a cell's own TOF-SIMS intensity divided by the mean intensity of its spatial neighbors) to quantify local metabolite competition between neighboring single cells, such as tumor cells vs T-cells, in human tissues profiled with scSpaMet.

## When to use

Use when you have per-cell metabolite intensities plus centroid coordinates from segmented TOF-SIMS/IMC data and need to quantify how metabolite abundance in one cell type depends on neighboring cells or distance to structures like CD31+ vessels.

## When NOT to use

- Input is raw TOF-SIMS or IMC image data that has not yet been segmented and extracted to per-cell intensities (use notebooks 01-04 first)
- IMC and SIMS modalities are not yet registered to a common coordinate frame (use notebook 02 registration first)
- The question is cell-type identification or joint embedding of protein-metabolite states rather than neighbor-dependent metabolite competition (use notebook 05 VAE embedding instead)

## Inputs

- Per-cell metabolite intensity table (TOF-SIMS mass channels, e.g. 148.0 m/z Methionine, 145.0 m/z Glutamine) from notebook 04 single-cell extraction
- Cell centroid coordinates from single-cell segmentation
- Registered region-of-interest assignments mapping cells to imaging regions (notebook 02)
- Cell-type labels (e.g., CD68+/T-cells vs tumor cells) from single-cell proteomics clustering
- Marker annotations of structures such as CD31+ endothelial cells for distance analysis

## Outputs

- Per-cell metabolite competition-ratio table (own intensity / mean neighbor intensity per channel)
- Competition-ratio spatial maps rendered per imaging region
- Channel-level comparisons (e.g., higher tumor-cell vs T-cell ratios with distance from CD31+ cells)

## How to apply

Load the per-cell metabolite intensity table and cell centroid coordinates from the single-cell extraction outputs (notebook 04) together with registered region-of-interest assignments (notebook 02). Build spatial neighbor sets for each cell using the custom spatial interaction functions in the 'spatial' folder of scSpaMet, defining neighborhoods either by a 20-um radius around the centroid or by direct cell-cell contact. For each metabolite channel (e.g., 148.0 m/z Methionine, 42.0 m/z lipid, 145.0 m/z Glutamine) compute the competition ratio as the cell's own intensity divided by the mean neighbor intensity, following the competition analysis in notebook '06 Metabolite analysis in different regions'. Render per-cell ratios back onto tissue coordinates as spatial maps per imaging region using the plotting helpers in 'utils.py'; stratify by cell type (CD68+/T-cell vs tumor cells) and distance to CD31+ endothelial cells to interpret the ratios.

## Related tools

- **scSpaMet** (Framework providing the competition analysis notebook (06 Metabolite analysis in different regions), the 'spatial' folder with custom spatial interaction functions, and 'utils.py' plotting/IO helpers) — github.com/coskunlab/ScSpaMet

## Examples

```
jupyter nbconvert --to notebook --execute notebooks/06\ Metabolite\ analysis\ in\ different\ regions.ipynb
```

## Evaluation signals

- A competition-ratio spatial map file is produced for every region of interest
- All competition-ratio values are finite and positive, indicating each cell had at least one measurable neighbor
- Ratios computed for named channels (e.g., 148.0 m/z Methionine, 42.0 m/z lipid, 145.0 m/z Glutamine) reproduce the published pattern of higher tumor-cell vs T-cell expression at greater distance from CD31+ endothelial cells
- Spatial maps align with the registered IMC/SIMS coordinates so cell types (CD68+/T-cells, tumor cells) overlay correctly on the ratio maps

## Limitations

- Each cell must have at least one measurable neighbor within the defined radius or contact definition; isolated cells yield undefined ratios
- Metabolite annotation and coverage for lipids, amino acids, and metabolic pathways from TOF-SIMS remains challenging, so ratio interpretation depends on channel annotations
- Relatively small imaging regions from tissue samples (a trade-off between imaging speed and spatial resolution in TOF-SIMS) limit the spatial extent of the competition maps

## Evidence

- [abstract] abstract: "The scSpaMet analysis revealed cell type-dependent metabolite profiles and local metabolite competition of neighboring single cells in human tissues"
- [results] results: "Single-cell segmentation was used to extract the protein and metabolite expression levels and their spatial locations"
- [readme] readme: "06 Metabolite analysis in different regions (metabolite difference, distance analysis, competition analysis)"
- [readme] readme_spatial: ""spatial" folder contains custom code for spatial interaction functions"
