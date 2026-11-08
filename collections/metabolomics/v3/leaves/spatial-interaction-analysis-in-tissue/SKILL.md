---
name: spatial-interaction-analysis-in-tissue
description: Use when you have per-cell metabolite intensities with centroid coordinates
  from segmented TOF-SIMS/IMC data and need to test whether metabolite competition
  between adjacent cells (e.g., tumor vs T-cell) varies with distance to CD31+ vessels.
when_to_use_negative:
- Input is raw TOF-SIMS/IMC image data that has not yet been segmented and reduced
  to per-cell intensities and centroids (run notebooks 01-04 first).
- The analysis goal is joint protein-metabolite embedding or trajectory inference
  (notebooks 05, 08), not spatial neighbor competition between cells.
- No spatial coordinate information or ROI registration is available, so neighbor
  relationships cannot be constructed.
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: scSpaMet
  role: Provides the 'spatial' folder of custom spatial interaction functions for
    building neighbor sets, notebook 06 competition analysis, and utils.py plotting
    helpers used to render competition-ratio spatial maps.
  repo: github.com/coskunlab/ScSpaMet
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-43917-5
    title: Single-cell spatial metabolomics with cell-type specific protein profiling
      for tissue systems biology
  source_content_hash: sha256:a2d0408dab3f81db6c3d3ed92f029c97154abd032328b81e85a92fca8e8bd4f0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scSpaMet
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# spatial-interaction-analysis-in-tissue

## Summary

Quantifies local metabolite competition between neighboring single cells (e.g., CD68+/T-cells vs tumor cells) in the lung tumor microenvironment by computing, per metabolite channel, each cell's intensity divided by the mean intensity of its spatial neighbors, then mapping the ratios back onto tissue coordinates relative to CD31+ endothelial cells.

## When to use

Use when you have per-cell metabolite intensities with centroid coordinates from segmented TOF-SIMS/IMC data and need to test whether metabolite competition between adjacent cells (e.g., tumor vs T-cell) varies with distance to CD31+ vessels.

## When NOT to use

- Input is raw TOF-SIMS/IMC image data that has not yet been segmented and reduced to per-cell intensities and centroids (run notebooks 01-04 first).
- The analysis goal is joint protein-metabolite embedding or trajectory inference (notebooks 05, 08), not spatial neighbor competition between cells.
- No spatial coordinate information or ROI registration is available, so neighbor relationships cannot be constructed.

## Inputs

- Per-cell metabolite intensity table (single-cell extraction outputs from notebook 04)
- Cell centroid coordinates per imaging region
- Registered region-of-interest assignments (notebook 02 IMC-SIMS registration outputs)
- Cell-type labels (e.g., CD68+/T-cell vs tumor cell) from single-cell proteomics clustering

## Outputs

- Per-cell competition-ratio table (cell intensity / mean neighbor intensity, per metabolite channel)
- Competition-ratio spatial maps, one per imaging region of interest
- Comparison of channel ratios by distance to CD31+ endothelial cells (e.g., higher Methionine 148.0 m/z in tumor cells far from vessels; Glutamine 145.0 m/z higher near vessels)

## How to apply

Load the per-cell metabolite intensity table and cell centroids from the single-cell extraction outputs (notebook 04) together with the registered region-of-interest assignments from notebook 02. Build neighbor sets for each cell using the custom spatial interaction functions in the 'spatial' folder of scSpaMet, choosing either a 20-um radius around the centroid or direct cell-cell contact as the neighborhood definition. For each metabolite channel and each cell, compute the competition ratio as the cell's own intensity divided by the mean neighbor intensity, following the competition analysis in notebook '06 Metabolite analysis in different regions'. Render the per-cell ratios back onto the tissue coordinates as one spatial map per imaging region with the plotting helpers in 'utils.py', stratifying by distance to CD31+ endothelial cells to compare channels such as 148.0 m/z Methionine, 42.0 m/z lipid, 94.0 m/z, 45 m/z lipid fragments, and 145.0 m/z Glutamine.

## Related tools

- **scSpaMet** (Provides the 'spatial' folder of custom spatial interaction functions for building neighbor sets, notebook 06 competition analysis, and utils.py plotting helpers used to render competition-ratio spatial maps.) — github.com/coskunlab/ScSpaMet

## Examples

```
from scipy.spatial import cKDTree; nbrs = cKDTree(centroids).query_ball_point(centroids, r=20.0); ratio = intensity / np.array([intensity[i].mean() for i in nbrs])
```

## Evaluation signals

- A competition-ratio spatial map file is produced for every region of interest.
- All ratio values are finite and positive, indicating each cell had at least one measurable neighbor.
- Channel-level comparisons reproduce the expected directional pattern: 148.0 m/z Methionine, 42.0 m/z lipid, 94.0 m/z, and 45 m/z lipid fragments higher in tumor cells farther from CD31+ cells, while 145.0 m/z Glutamine is higher in tumor cells near CD31+ cells.
- Neighbor sets built at the 20-um radius yield non-empty neighborhoods; switching to cell-cell contact neighborhoods should not produce empty neighbor lists for interior tissue cells.
- Maps overlay correctly on registered tissue coordinates, confirming ROI assignment consistency across IMC and SIMS modalities.

## Limitations

- The neighborhood definition (20-um radius vs direct cell-cell contact) is a decision point that changes competition-ratio values; results are sensitive to this parameter.
- TOF-SIMS imaging regions are relatively small due to the trade-off between imaging speed and spatial resolution, limiting the number of neighborhoods per tissue.
- Metabolite annotation from TOF-SIMS is incomplete, so some channels (e.g., 94.0 m/z, 45 m/z lipid fragments) may remain ambiguous.

## Evidence

- [readme] spatial interaction functions: "'spatial' folder contains custom code for spatial interaction functions"
- [readme] notebook 06: "06 Metabolite analysis in different regions (metabolite difference, distance analysis, competition analysis)"
- [abstract] cell type-dependent metabolite profiles and competition: "The scSpaMet analysis revealed cell type-dependent metabolite profiles and local metabolite competition of neighboring single cells in human tissues"
- [results] single-cell segmentation outputs: "Single-cell segmentation was used to extract the protein and metabolite expression levels and their spatial locations"
