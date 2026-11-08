---
name: single-cell-spatial-map-visualization
description: Use when per-cell intensities and centroid coordinates are already extracted
  (notebook 04) and the goal is to visualize local metabolite competition or expression
  as a spatial map per imaging region relative to landmarks like CD31+ vessels.
when_to_use_negative:
- Input is raw TOF-SIMS or IMC image data before single-cell segmentation and intensity
  extraction — run notebook 04 first.
- The goal is multi-omics integration or joint embedding (VAE, notebooks 05/07) rather
  than spatial rendering of per-cell ratios.
- Regions of interest are not yet registered between IMC and SIMS modalities (notebook
  02 registration is a prerequisite).
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: scSpaMet
  role: Provides the 'spatial' folder for spatial interaction/neighborhood functions
    and 'utils.py' plotting and IO helpers used to compute and render per-cell competition-ratio
    spatial maps
  repo: github.com/coskunlab/ScSpaMet
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-43917-5
    title: Single-cell spatial metabolomics with cell-type specific protein profiling
      for tissue systems biology
  source_content_hash: sha256:bc6a58cca7f5694e6135b5237dd8c454a327b2502a4e579cbb55895f5066e86f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scSpaMet
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# single-cell-spatial-map-visualization

## Summary

Renders per-cell metabolite competition ratios (cell intensity / mean neighbor intensity, e.g., CD68+/T-cell vs tumor cell) back onto tissue centroid coordinates as per-region spatial maps using the scSpaMat 'utils.py' plotting helpers and 'spatial' neighborhood functions.

## When to use

Use when per-cell intensities and centroid coordinates are already extracted (notebook 04) and the goal is to visualize local metabolite competition or expression as a spatial map per imaging region relative to landmarks like CD31+ vessels.

## When NOT to use

- Input is raw TOF-SIMS or IMC image data before single-cell segmentation and intensity extraction — run notebook 04 first.
- The goal is multi-omics integration or joint embedding (VAE, notebooks 05/07) rather than spatial rendering of per-cell ratios.
- Regions of interest are not yet registered between IMC and SIMS modalities (notebook 02 registration is a prerequisite).

## Inputs

- Per-cell metabolite intensity table (from notebook 04 single-cell intensity extraction)
- Cell centroid x/y coordinates per imaging region
- Registered region-of-interest assignments (from notebook 02 IMC-SIMS registration)
- Cell-type/protein labels (e.g., CD68+/T-cell vs tumor cell, CD31+ endothelial cells)

## Outputs

- Per-region competition-ratio spatial map images (one map file per region of interest)
- Per-cell competition-ratio values (finite, positive) for each metabolite channel

## How to apply

Load the per-cell metabolite intensity table and cell centroid coordinates from the single-cell extraction outputs (notebook 04) together with the registered region-of-interest assignments (notebook 02). Build spatial neighbor sets for each cell with the custom functions in the 'spatial' folder of scSpaMet, defining neighborhoods by a 20-um radius around the centroid or by direct cell-cell contact. For each metabolite channel and cell, compute the competition ratio as the cell's own intensity divided by the mean neighbor intensity, following notebook '06 Metabolite analysis in different regions'. Render the per-cell ratios back onto tissue coordinates as one spatial map per imaging region using the plotting helpers in 'utils.py'. Rationale: mapping ratios onto tissue coordinates preserves the microenvironment context (distance to CD31+ endothelial cells) that drives tumor cell vs T-cell metabolic differences such as 148.0 m/z Methionine and 145.0 m/z Glutamine.

## Related tools

- **scSpaMet** (Provides the 'spatial' folder for spatial interaction/neighborhood functions and 'utils.py' plotting and IO helpers used to compute and render per-cell competition-ratio spatial maps) — github.com/coskunlab/ScSpaMet

## Examples

```
# in scSpaMat repo: compute neighbors with spatial module, then plot with utils.py
from spatial import spatial_interaction; from utils import plot_map
ratios = spatial_interaction(intensities, centroids, radius_um=20)
plot_map(centroids, ratios['148.0'], region='ROI_01')
```

## Evaluation signals

- A competition-ratio spatial map file is produced for every region of interest.
- All competition-ratio values are finite and positive, indicating each cell had at least one measurable neighbor.
- Maps overlay per-cell ratios on the correct tissue coordinates from the registered ROI assignments (notebook 02).
- Channel-specific patterns are consistent with the reported finding: 148.0 m/z Methionine and 42.0 m/z lipid higher in tumor cells further from CD31+ cells, 145.0 m/z Glutamine higher in tumor cells near CD31+ cells.

## Limitations

- TOF-SIMS imaging regions are relatively small due to the trade-off between imaging speed and spatial resolution, limiting the mapped field of view.
- Metabolite annotation coverage for lipids, amino acids, and metabolic pathways from TOF-SIMS is challenging, so mapped channel identities may be partial.
- The 20-um radius neighborhood is a fixed choice; results depend on this radius or the cell-cell contact definition.

## Evidence

- [results] Competition-ratio analysis revealed that mass channels 148.0 m/z Methionine, 42.0 m/z lipid, 94.0 m/z, and 45 m/z lipid fragments have higher expression in tumor cells compared to T-cells at a further distance from CD31+ endothelial cells: [evidence span withheld — anchor status: not_found]
- [readme] spatial folder contains custom code for spatial interaction functions: ""spatial" folder contains custom code for spatial interaction functions"
- [readme] utils.py contains plotting and io custom functions: ""utils.py" contains plotting and io custom functions"
- [readme] 06 Metabolite analysis in different regions (metabolite difference, distance analysis, competition analysis): "06 Metabolite analysis in different regions (metabolite difference, distance analysis, competition analysis)"
- [results] Single-cell segmentation was used to extract the protein and metabolite expression levels and their spatial locations: "Single-cell segmentation was used to extract the protein and metabolite expression levels and their spatial locations"
