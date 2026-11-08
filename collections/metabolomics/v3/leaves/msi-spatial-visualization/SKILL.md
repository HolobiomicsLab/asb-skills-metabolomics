---
name: msi-spatial-visualization
description: Use when you have a pixel-by-ion peak table (e.g., 53,812 pixels x 2,654
  ions with X/Y coordinates in the first two columns) and need to visualize graph-based
  clustering (Louvain) results as spatial maps to validate preprocessing quality against
  tissue morphology.
when_to_use_negative:
- The peak table has already been spatially clustered and rendered — re-running adds
  no information
- The goal is quality assessment/preprocessing of raw MSI data itself (use SMQVP's
  quality visualization modules) rather than visualizing downstream clustering results
- The data lacks pixel X/Y coordinates, so cluster labels cannot be mapped back to
  spatial positions
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: SMQVP
  role: Produces the preprocessed peak table whose clustering results are visualized
    spatially and compared against the raw-data maps
  repo: https://github.com/mzlab-research/SMQVP
- name: R
  role: Environment used to run Louvain clustering on the ion-intensity matrices and
    render the spatial cluster maps
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3390/metabo15060354
    title: 'SMQVP: A Web Application for Spatial Metabolomics Quality Visualization
      and Processing'
  source_content_hash: sha256:e9830cbb5eb2cb981da6cc94027f780d7d33a87a1983d068dfb77bdefc361b8d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SMQVP
  tools:
  - R
  - SMQVP
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# msi-spatial-visualization

## Summary

Render spatially resolved visualizations of mass-spec imaging clustering results by mapping pixel cluster labels back to X/Y coordinates, so that spatial regions (e.g., mouse brain neuroanatomy) can be compared against H&E-stained morphology as a quality check on preprocessing.

## When to use

Use when you have a pixel-by-ion peak table (e.g., 53,812 pixels x 2,654 ions with X/Y coordinates in the first two columns) and need to visualize graph-based clustering (Louvain) results as spatial maps to validate preprocessing quality against tissue morphology.

## When NOT to use

- The peak table has already been spatially clustered and rendered — re-running adds no information
- The goal is quality assessment/preprocessing of raw MSI data itself (use SMQVP's quality visualization modules) rather than visualizing downstream clustering results
- The data lacks pixel X/Y coordinates, so cluster labels cannot be mapped back to spatial positions

## Inputs

- Peak table matrix (pixels x ions) with pixel X and Y coordinates in the first two columns and m/z intensities in subsequent columns (e.g., OMIX009541 AFADESI mouse brain table, 53,812 pixels x 2,654 ions)
- SMQVP-preprocessed peak table (noise ions removed, background pixels removed)
- H&E-stained tissue section image for morphological reference

## Outputs

- Spatial cluster map for the raw peak table (Figure 3A equivalent), rendered in R
- Spatial cluster map for the SMQVP-preprocessed peak table (Figure 3B equivalent)
- Comparison of cluster-map regions against the H&E-stained morphology

## How to apply

Load the peak table (raw or SMQVP-preprocessed) with pixel X and Y coordinates in the first two columns and ion intensities in the remaining columns. Run Louvain graph-based clustering in R, treating pixels as observations and ions as features, independently on raw and preprocessed matrices. Map the resulting cluster labels back to pixel coordinates and render a spatial cluster map for each version (e.g., Figure 3A raw vs Figure 3B preprocessed equivalents). Judge success by whether the processed-data map yields distinct spatial regions closely resembling the H&E-stained mouse brain section, while the raw-data map fails to reflect known neuroanatomical structures; this morphological-alignment judgment is made via expert_review or llm_judge.

## Related tools

- **SMQVP** (Produces the preprocessed peak table whose clustering results are visualized spatially and compared against the raw-data maps) — https://github.com/mzlab-research/SMQVP
- **R** (Environment used to run Louvain clustering on the ion-intensity matrices and render the spatial cluster maps)

## Examples

```
library(igraph); comm <- cluster_louvain(graph_from_adjacency_matrix(W, mode='undirected', weighted=TRUE)); plot(coords$x, coords$y, col=membership(comm), pch=15)
```

## Evaluation signals

- The processed-data spatial cluster map shows distinct spatial regions closely resembling the H&E-stained mouse brain morphology
- The raw-data spatial cluster map fails to reflect known neuroanatomical structures (expected negative control)
- Cluster labels are successfully mapped back to the pixel X/Y coordinates from the first two columns of the peak table
- Both raw and preprocessed matrices are clustered independently and rendered as parallel Figure 3A/3B-equivalent maps for direct comparison
- Morphological-alignment judgment is recorded via expert_review or llm_judge rather than an arbitrary visual impression

## Limitations

- The morphological-alignment judgment is qualitative, relying on expert_review or llm_judge rather than a quantitative metric
- Preprocessing history matters: without SMQVP noise-ion filtering and background-pixel removal, the cluster map may not reflect tissue morphology
- The demonstrated use case is AFADESI mouse brain data in positive ion mode; transferability to other MSI platforms or tissues is not established in this context

## Evidence

- [abstract] Abstract: "This rigorous preprocessing resulted in improved clustering outcomes that more accurately reflected the underlying tissue morphology compared with analyses performed on unprocessed data."
- [results] Results: "The processed data aligns better with tissue morphology, highlighting the importance of thorough preprocessing in spatial metabolomics analysis."
- [readme] README: "First two columns: X and Y coordinates of the pixels"
