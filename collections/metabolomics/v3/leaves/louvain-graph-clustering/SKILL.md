---
name: louvain-graph-clustering
description: Use when a spatial metabolomics peak table (pixels x ion intensities
  with X/Y coordinates) needs to be segmented into spatial tissue regions whose alignment
  with histology (e.g. H&E) must be assessed, or when comparing preprocessing impact
  on clustering.
when_to_use_negative:
- Input is raw mass-spectrometry imaging data (e.g. mzML/imzML) rather than a peak
  table matrix; preprocess with SMQVP first
- Goal is differential ion abundance statistics between regions rather than unsupervised
  spatial segmentation
- The peak table still contains background pixels and noise ions known to distort
  spatial pattern recognition
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Environment in which Louvain graph-based clustering is run on the ion-intensity
    matrix and spatial cluster maps are rendered
- name: SMQVP
  role: Produces the preprocessed peak table (background noise removal, noise ion
    filtering) whose Louvain clustering is compared against clustering of the raw
    table
  repo: github.com/mzlab-research/SMQVP
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3390/metabo15060354
    title: 'SMQVP: A Web Application for Spatial Metabolomics Quality Visualization
      and Processing'
  source_content_hash: sha256:3ecd31e76f58a7854c75df5870487b91635f62b712dbd665dae88d43ceca65b0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - SMQVP
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# louvain-graph-clustering

## Summary

Run Louvain graph-based community detection on spatial metabolomics pixel-by-ion peak tables to segment tissue into spatial clusters, used here to compare clustering of raw vs SMQVP-preprocessed AFADESI mouse brain data against H&E morphology.

## When to use

Use when a spatial metabolomics peak table (pixels x ion intensities with X/Y coordinates) needs to be segmented into spatial tissue regions whose alignment with histology (e.g. H&E) must be assessed, or when comparing preprocessing impact on clustering.

## When NOT to use

- Input is raw mass-spectrometry imaging data (e.g. mzML/imzML) rather than a peak table matrix; preprocess with SMQVP first
- Goal is differential ion abundance statistics between regions rather than unsupervised spatial segmentation
- The peak table still contains background pixels and noise ions known to distort spatial pattern recognition

## Inputs

- Raw AFADESI mouse brain peak table (53,812 pixels x 2,654 ion intensities, X/Y coordinates in first two columns) from OMIX009541
- SMQVP-preprocessed peak table (background/noise-filtered) from task_004
- H&E-stained mouse brain section image for morphological comparison

## Outputs

- Per-pixel Louvain cluster label assignments
- Spatial cluster map rendered in R for the raw peak table (Figure 3A equivalent)
- Spatial cluster map rendered in R for the preprocessed peak table (Figure 3B equivalent)

## How to apply

Load the peak table (e.g. the 53,812-pixel x 2,654-ion AFADESI mouse brain table with pixel X and Y coordinates in the first two columns) and treat pixels as observations and ions as features. Build a graph over pixels from the ion-intensity matrix (e.g. from a pixel-pixel similarity/distance representation) and run Louvain community detection (per refs 24-25 of the paper) in R. Map resulting cluster labels back to pixel X/Y coordinates and render a spatial cluster map for each dataset — raw (Figure 3A equivalent) and SMQVP-preprocessed (Figure 3B equivalent). The method's rationale is diagnostic: Louvain on unprocessed data should fail to recover neuroanatomical structure, while on SMQVP-denoised data (background noise removal and noise ion filtering applied) it should yield distinct regions matching the H&E-stained section. Validate the contrast via expert review or LLM-judged morphological alignment against the H&E image (Figure 2A reference).

## Related tools

- **R** (Environment in which Louvain graph-based clustering is run on the ion-intensity matrix and spatial cluster maps are rendered)
- **SMQVP** (Produces the preprocessed peak table (background noise removal, noise ion filtering) whose Louvain clustering is compared against clustering of the raw table) — github.com/mzlab-research/SMQVP

## Examples

```
Rscript -e 'library(igraph); g <- graph_from_adjacency_matrix(as.matrix(dist(peak_table[,-1:-2])), weighted=TRUE, mode="undirected"); plot(clusters SpatialMap of membership(cluster_louvain(g)), coords=peak_table[,1:2])'
```

## Evaluation signals

- Preprocessed-data cluster map shows distinct spatial regions closely resembling the H&E-stained tissue morphology
- Raw-data cluster map fails to reflect known neuroanatomical structures, confirming the expected contrast
- Every pixel in the 53,812-pixel table receives exactly one cluster label that maps back to a valid X/Y coordinate
- Morphological-alignment judgment between cluster maps and the H&E image is consistent under expert_review / llm_judge evaluation

## Limitations

- Morphological alignment is a qualitative judgment (expert review / LLM judge), not a numeric metric
- Clustering quality depends on prior SMQVP preprocessing; noise ions and background pixels can interfere with spatial pattern recognition
- No software version or license was specified for SMQVP, and no changelog exists for the repository

## Evidence

- [results] Louvain clustering of the SMQVP-preprocessed data yielded distinct spatial regions closely resembling the H&E staining morphology, whereas clustering of unprocessed data failed to reflect known neuroanatomical structures.: "SMQVP has been demonstrated to effectively improve clustering results by removing noise signals and enhancing data quality."
- [abstract] Louvain graph-based clustering run independently on each ion-intensity matrix with pixels as observations and ions as features.: "This rigorous preprocessing resulted in improved clustering outcomes that more accurately reflected the underlying tissue morphology compared with analyses performed on unprocessed data."
- [readme] First two columns: X and Y coordinates of the pixels; Subsequent columns: m/z values with corresponding intensities at each pixel: "The input for SMQVP is a peak table matrix with the following structure:
- First two columns: X and Y coordinates of the pixels
- Subsequent columns: m/z values with corresponding intensities at each"
- [readme] The example data includes 53,812 pixels and 2,654 ion peaks from AFADESI mouse brain in positive ion mode, available at OMIX009541.: "The data includes 53,812 pixels and 2,654 ion peaks. This peak table and raw data are available in the public repository https://ngdc.cncb.ac.cn/omix/release/OMIX009541"
