---
name: spatial-metabolomics-pattern-visualization
description: Use when an IMS feature matrix (pixel coordinates + ion intensities)
  has been clustered by spatial co-expression and the patterns must be mapped back
  onto tissue pixels to identify region-enriched expression patterns.
when_to_use_negative:
- Input is raw instrument data or peak-picked spectra rather than a pixel-by-ion feature
  matrix with X/Y coordinate columns
- The goal is pixel-level tissue region clustering (Seurat-LV/Seurat-LM/Seurat-SLM/UMAP-kmeans)
  rather than metabolite-level spatial co-expression patterns
- Ions have not passed the noise-ion QC threshold, so maps would visualize noise rather
  than spatial patterns
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3361
tools:
- name: SMAnalyst
  role: Web/shiny platform performing metabolite-level spatial co-expression clustering
    and interactive visualization/export of ion distribution images and pattern maps
  repo: https://github.com/mzlab-research/SManalyst
- name: SpaGene
  role: Algorithm underlying metabolite-level clustering by spatial co-expression,
    producing the pattern groups rendered in the map
  repo: https://github.com/mzlab-research/SManalyst
- name: ggplot2
  role: R plotting package used by SMAnalyst to render spatial pattern visualizations
  repo: https://github.com/mzlab-research/SManalyst
- name: tidyverse
  role: R package suite bundled with SMAnalyst for data wrangling of the pixel/ion
    matrix prior to plotting
  repo: https://github.com/mzlab-research/SManalyst
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/biom15111562
    title: 'SMAnalyst: A Web Server for Spatial Metabolomic Data Analysis and Annotation'
  source_content_hash: sha256:021399983f2854f66718b91fa7e9f3998ac87a517f92c8680e030d0ac1b2ec20
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SManalyst
  tools:
  - R (>=)
  - SMAnalyst
  - SpaGene
  - ggplot2
  - tidyverse
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# spatial-metabolomics-pattern-visualization

## Summary

Renders metabolite-level spatial co-expression clusters (e.g., SpaGene pattern groups) as spatial maps over the tissue section, so that region-enriched patterns such as cortex-high or tissue-edge patterns can be visually identified and validated.

## When to use

Use when an IMS feature matrix (pixel coordinates + ion intensities) has been clustered by spatial co-expression and the patterns must be mapped back onto tissue pixels to identify region-enriched expression patterns.

## When NOT to use

- Input is raw instrument data or peak-picked spectra rather than a pixel-by-ion feature matrix with X/Y coordinate columns
- The goal is pixel-level tissue region clustering (Seurat-LV/Seurat-LM/Seurat-SLM/UMAP-kmeans) rather than metabolite-level spatial co-expression patterns
- Ions have not passed the noise-ion QC threshold, so maps would visualize noise rather than spatial patterns

## Inputs

- Feature matrix CSV: columns 1-2 = pixel X/Y coordinates, subsequent columns = ion intensities per m/z, one row per spatial pixel (e.g., 14,260 pixels x 3,044 ions)
- Noise-filtered tissue pixel subset (noise-ion filter score >= 30)
- Metabolite-level (SpaGene) cluster assignments per pixel or per ion

## Outputs

- Spatial map of metabolite co-expression patterns over the tissue section (analogous to Figure 5A / Supplementary Figure S11), exportable as PNG
- Table (CSV) listing the specific ions contained within each pattern cluster

## How to apply

Load the feature matrix (e.g., the 14,260-pixel × 3,044-ion mouse brain AFAD-ESI(+) dataset at 100 µm pixel size from NGDC OMIX OMIX009541 or SMAnalyst's Tutorial Panel example data), retain tissue pixels and ions passing the noise-ion filter (default score ≥ 30), and run metabolite-level spatial co-expression clustering with SpaGene as implemented in SMAnalyst. Export the table of ions assigned to each pattern cluster, then render a per-pixel map of cluster assignments colored by pattern group over the X/Y tissue coordinates, using tidyverse/ggplot2 (SMAnalyst is an R ≥ 4.1.0 shinydashboard app). Correctness is judged by whether the map resolves exactly the expected number of major patterns (eight for the mouse brain Dataset 1) and reproduces biologically interpretable enrichment, such as Pattern 4 high in the cerebral cortex and low in midbrain, and Pattern 7 enriched at tissue edge regions.

## Related tools

- **SMAnalyst** (Web/shiny platform performing metabolite-level spatial co-expression clustering and interactive visualization/export of ion distribution images and pattern maps) — https://github.com/mzlab-research/SManalyst
- **SpaGene** (Algorithm underlying metabolite-level clustering by spatial co-expression, producing the pattern groups rendered in the map) — https://github.com/mzlab-research/SManalyst
- **ggplot2** (R plotting package used by SMAnalyst to render spatial pattern visualizations) — https://github.com/mzlab-research/SManalyst
- **tidyverse** (R package suite bundled with SMAnalyst for data wrangling of the pixel/ion matrix prior to plotting) — https://github.com/mzlab-research/SManalyst

## Examples

```
install.packages(c("shinydashboard","ggplot2","shiny","shinythemes","tidyverse","reshape2","pheatmap","SpaGene"))  # then run metabolite clustering + pattern maps via the SMAnalyst app
```

## Evaluation signals

- Clustering resolves exactly the reported number of major metabolite spatial expression patterns (eight for the mouse brain Dataset 1)
- One pattern (Pattern 4) shows high intensity in the cerebral cortex and low intensity in the midbrain, matching the published figure
- One pattern (Pattern 7) is uniquely enriched at tissue edge regions
- Ion-membership table exists for each pattern cluster and ions are assigned to exactly one pattern
- Pixel coordinates in the map match the input feature matrix rows (14,260 tissue pixels for the example dataset)

## Limitations

- Visualization depends on prior QC (noise-ion threshold) and clustering; visualizing unfiltered ions will surface noise ions in the pattern map
- Pattern numbering/interpretation is dataset-specific and must be re-derived for new tissues or acquisition modes
- Large matrices (>14,000 pixels, >3,000 ions) are at the demonstrated performance limit of the platform

## Evidence

- [abstract] dual-dimension spatial pattern discovery (metabolite co-expression and pixel clustering): "dual-dimension spatial pattern discovery (metabolite co-expression and pixel clustering)"
- [readme] Metabolite-level clustering readme: "**Metabolite-level clustering**: Group ions by spatial co-expression patterns"
- [readme] Spatial analysis usage step: "**Spatial Analysis**:
    *   Run metabolite/pixel clustering"
- [readme] Example dataset description: "Acquisition: AFAD-ESI (+) mode, 100μm resolution
*   Content: 14,260 pixels × 3,044 ions"
