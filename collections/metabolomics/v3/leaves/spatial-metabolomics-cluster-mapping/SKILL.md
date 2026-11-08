---
name: spatial-metabolomics-cluster-mapping
description: Use when you have a spatial metabolomics peak table (pixels x ion intensities
  with X/Y coordinates) and need to assess whether preprocessing (e.g. SMQVP noise/background
  removal) improves spatial segmentation relative to H&E tissue morphology.
when_to_use_negative:
- The data has not been quality-checked or preprocessed — clustering raw unprocessed
  peak tables is exactly the negative control this skill is designed to contrast against,
  not the primary result
- The input is a two-dimensional matrix without spatial pixel coordinates (first two
  columns X/Y), since cluster labels cannot be mapped to tissue space
- The goal is ion-level identification (isotopes, adducts) rather than pixel-level
  spatial segmentation — use SMQVP's isotopic/adduct modules instead
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: SMQVP
  role: Preprocesses the AFADESI peak table (background pixel removal, noise ion filtering)
    so that Louvain clustering on the cleaned matrix recovers tissue morphology; provides
    the input format (first two columns = X/Y coordinates, subsequent columns = m/z
    intensities)
  repo: https://github.com/mzlab-research/SMQVP
- name: R
  role: Environment in which Louvain graph-based clustering is run on the ion-intensity
    matrices and the spatial cluster maps are rendered
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3390/metabo15060354
    title: 'SMQVP: A Web Application for Spatial Metabolomics Quality Visualization
      and Processing'
  source_content_hash: sha256:35d1117520eaad51d04a847af177bed1b2ca9896a5b4357c288b28103a9860a7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SMQVP
  tools:
  - R
  - SMQVP
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# spatial-metabolomics-cluster-mapping

## Summary

Graph-based (Louvain) clustering of spatial metabolomics peak-table matrices — pixels as observations, ions as features — with cluster labels mapped back to pixel X/Y coordinates to render spatial cluster maps that are judged against H&E tissue morphology. Used to demonstrate that SMQVP-preprocessed AFADESI mouse brain data yields anatomically faithful regions while raw data does not.

## When to use

Use when you have a spatial metabolomics peak table (pixels x ion intensities with X/Y coordinates) and need to assess whether preprocessing (e.g. SMQVP noise/background removal) improves spatial segmentation relative to H&E tissue morphology.

## When NOT to use

- The data has not been quality-checked or preprocessed — clustering raw unprocessed peak tables is exactly the negative control this skill is designed to contrast against, not the primary result
- The input is a two-dimensional matrix without spatial pixel coordinates (first two columns X/Y), since cluster labels cannot be mapped to tissue space
- The goal is ion-level identification (isotopes, adducts) rather than pixel-level spatial segmentation — use SMQVP's isotopic/adduct modules instead

## Inputs

- Raw AFADESI peak table matrix (pixels x ions) with pixel X/Y coordinates in the first two columns, e.g. 53,812 pixels x 2,654 ions from OMIX009541
- SMQVP-preprocessed peak table (background/noise-filtered) with matching pixel coordinates
- H&E-stained tissue section image for morphological comparison

## Outputs

- Spatial cluster map for the raw peak table (Figure 3A equivalent), rendered in R
- Spatial cluster map for the SMQVP-preprocessed peak table (Figure 3B equivalent), rendered in R
- Per-pixel Louvain cluster labels mapped to X/Y coordinates

## How to apply

Load the peak table (e.g. the AFADESI mouse brain matrix of 53,812 pixels x 2,654 ions from OMIX009541, with pixel X and Y coordinates in the first two columns) for both the raw and SMQVP-preprocessed versions. Run Louvain graph-based clustering independently on each ion-intensity matrix, treating pixels as observations and ions as features. Map the cluster labels back to pixel coordinates and render a spatial cluster map per condition in R (raw-data map and preprocessed-data map). Compare both maps against the H&E-stained tissue section: the processed-data map should yield distinct spatial regions closely resembling the stained morphology, while the raw-data map should fail to reflect known neuroanatomical structures — this alignment judgment is made via expert review or an LLM judge. Rationale: SMQVP's removal of background pixels and noise ions prevents noise signals from corrupting the graph structure underlying clustering.

## Related tools

- **SMQVP** (Preprocesses the AFADESI peak table (background pixel removal, noise ion filtering) so that Louvain clustering on the cleaned matrix recovers tissue morphology; provides the input format (first two columns = X/Y coordinates, subsequent columns = m/z intensities)) — https://github.com/mzlab-research/SMQVP
- **R** (Environment in which Louvain graph-based clustering is run on the ion-intensity matrices and the spatial cluster maps are rendered)

## Examples

```
library(igraph); g <- graph_from_adjacency_matrix(as.matrix(dist(scale(peakTable[, -c(1:2)]))), weighted=TRUE, mode='undirected'); memb <- membership(cluster_louvain(g))
```

## Evaluation signals

- Preprocessed-data cluster map shows distinct spatial regions that closely resemble the H&E-stained mouse brain morphology (expert_review / llm_judge alignment)
- Raw-data cluster map fails to reflect known neuroanatomical structures, establishing the expected before/after contrast
- Cluster labels map one-to-one onto pixel X/Y coordinates so every pixel in the 53,812-pixel matrix is assigned a region
- Both raw and preprocessed matrices are clustered with the identical Louvain procedure so the comparison isolates the effect of SMQVP preprocessing
- Input peak tables conform to the SMQVP schema: X/Y coordinates in the first two columns followed by ion-intensity columns

## Limitations

- Morphological-alignment judgment is subjective and relies on expert review or an LLM judge rather than a quantitative metric
- Preprocessing quality (background delineation with the lasso tool, noise-ion filtering) directly gates clustering quality; poor region selection upstream will propagate
- No changelog or version specification is available for the SMQVP repository, complicating reproducibility of the preprocessing step

## Evidence

- [abstract] Preprocessing with SMQVP resulted in improved clustering outcomes: "This rigorous preprocessing resulted in improved clustering outcomes that more accurately reflected the underlying tissue morphology compared with analyses performed on unprocessed data."
- [results] SMQVP improves clustering via noise removal: "SMQVP has been demonstrated to effectively improve clustering results by removing noise signals and enhancing data quality."
- [readme] Input data format for peak tables: "The input for SMQVP is a peak table matrix with the following structure: First two columns: X and Y coordinates of the pixels; Subsequent columns: m/z values with corresponding intensities at each"
- [readme] Example dataset description: "The example data used in this study is spatial metabolomics data of a mouse brain acquired using the AFAD-ESI platform in positive ion mode. The data includes 53,812 pixels and 2,654 ion peaks."
- [readme] Background and noise removal features: "Noise Ion Filtering: Detect and filter out noise ions that may interfere with spatial pattern recognition."
