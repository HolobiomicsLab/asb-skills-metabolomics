---
name: dem-table-visualization-with-volcano-plots-and-heatmaps
description: Use when you have a table of differentially expressed m/z peaks (with
  m/z, annotation, logFC, FDR) from SpaMTP's pseudo-bulked edgeR DEM analysis and
  need publication-ready visual summaries.
when_to_use_negative:
- Input is raw unannotated imzML peak intensities with no prior DEM testing — visualising
  before the pseudo-bulked edgeR step yields meaningless plots
- Data is spatial transcriptomics gene expression rather than m/z metabolite peaks
  — the DEM volcano/heatmap functions target metabolite peak tables
- The DEM table has fewer than a handful of significant m/z peaks after FDR filtering
  — a volcano plot adds little over the ranked table itself
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: SpaMTP
  role: Provides the DEM analysis that produces dem_results_table.csv and the in-built
    volcano plot and heatmap visualisation functions that render it
  repo: https://github.com/GenomicsMachineLearning/SpaMTP
- name: edgeR
  role: Differential testing engine behind the pseudo-bulked DEM analysis whose output
    is visualised; installed as a SpaMTP Bioconductor dependency
- name: Seurat
  role: Seurat Class Object foundation on which the annotated m/z intensity matrix
    and cluster labels are stored before DEM and plotting
- name: R
  role: Language in which SpaMTP's DEM and visualisation functions are run; conda
    environment with r-base=4.4 and bioconductor-edger supports the workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2024.10.31.621429
    title: 'SpaMTP: Integrative Statistical Analysis and Visualisation of Spatial
      Metabolomics and Transcriptomics data'
  source_content_hash: sha256:6349a361e7f4383cd6a5e13df691e70cedf413936e93724fac0bab53df7f5419
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/GenomicsMachineLearning/SpaMTP
  tools:
  - R
  - Seurat
  - SpaMTP
  - edgeR
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# DEM table visualization with volcano plots and heatmaps

## Summary

Visualise the differentially expressed m/z peaks output of SpaMTP's DEM analysis (dem_results_table.csv) using SpaMTP's in-built volcano plot and heatmap functions to convey fold-change, significance, and cluster-specific intensity patterns.

## When to use

Use when you have a table of differentially expressed m/z peaks (with m/z, annotation, logFC, FDR) from SpaMTP's pseudo-bulked edgeR DEM analysis and need publication-ready visual summaries.

## When NOT to use

- Input is raw unannotated imzML peak intensities with no prior DEM testing — visualising before the pseudo-bulked edgeR step yields meaningless plots
- Data is spatial transcriptomics gene expression rather than m/z metabolite peaks — the DEM volcano/heatmap functions target metabolite peak tables
- The DEM table has fewer than a handful of significant m/z peaks after FDR filtering — a volcano plot adds little over the ranked table itself

## Inputs

- dem_results_table.csv — ranked table of differentially expressed m/z peaks with per-m/z statistics (fold-change, FDR) and annotation columns from SpaMTP's DEM analysis
- SpaMTP MultiAssay/Seurat-style spatial metabolomics object with annotated m/z intensity matrix and spatial region/cluster labels (e.g. mouse urinary bladder MALDI imzML data)

## Outputs

- Volcano plot of differentially expressed m/z peaks (logFC vs significance, labelled with metabolite annotations)
- Heatmap of m/z peak intensities across spatial clusters/regions showing cluster-specific expression patterns

## How to apply

Run SpaMTP's pseudo-bulked DEM workflow first: pseudo-bulk pixels into pools per spatial region (e.g. urothelium, muscle, adventitial layer of the mouse urinary bladder MALDI dataset) with a fixed seed, run edgeR on the summed relative intensities per m/z, and write the ranked DEM table with annotation columns to dem_results_table.csv (~70 significant m/z peaks expected, including PC(34:1) up in urothelium and SM(34:1) up in muscle). Then plot the DEM table with SpaMTP's volcano plot function to display logFC against statistical significance per m/z, and use the heatmap function to render m/z-by-cluster relative intensity patterns across the tissue-specific regions. The volcano plot surfaces which annotated metabolites (e.g. lipids flagged by RefineLipids) drive group separation, while the heatmap confirms cluster-specific enrichment visually. Both visualisations consume the same dem_results_table.csv as input, so annotation columns added by the annotation pipeline (e.g. RaMP/HMDB names) flow through to the plots.

## Related tools

- **SpaMTP** (Provides the DEM analysis that produces dem_results_table.csv and the in-built volcano plot and heatmap visualisation functions that render it) — https://github.com/GenomicsMachineLearning/SpaMTP
- **edgeR** (Differential testing engine behind the pseudo-bulked DEM analysis whose output is visualised; installed as a SpaMTP Bioconductor dependency)
- **Seurat** (Seurat Class Object foundation on which the annotated m/z intensity matrix and cluster labels are stored before DEM and plotting)
- **R** (Language in which SpaMTP's DEM and visualisation functions are run; conda environment with r-base=4.4 and bioconductor-edger supports the workflow)

## Examples

```
Rscript -e 'devtools::install_github("GenomicsMachineLearning/SpaMTP"); dem <- read.csv("dem_results_table.csv"); SpaMTP::PlotVolcanoDE(dem); SpaMTP::PlotHeatmapDE(dem)'
```

## Evaluation signals

- dem_results_table.csv contains roughly 70 significantly differentially expressed m/z peaks between the three tissue clusters, matching the reported Table S5 result
- Volcano plot shows PC(34:1) with positive logFC (up-regulated in urothelium) and SM(34:1) with logFC indicating up-regulation in muscle
- Heatmap rows (m/z peaks) show visually distinct intensity blocks aligned to the urothelium, muscle, and adventitial layer cluster labels
- Every plotted m/z row carries a non-empty annotation column inherited from the RaMP/HMDB annotation pipeline output
- Plot axes and labels resolve to logFC (fold-change) and statistical significance (FDR) values that match the values stored in dem_results_table.csv

## Limitations

- The article documents the paired targeted metabolomics comparison section as 'To come!', so paired targeted-data refinement of DEM tables is not yet available
- MS1-ID/GNPS annotation is separate from the DEM plot path; unannotated m/z rows will plot without informative metabolite labels
- No versioned SpaMTP release identifier is given in the article text — pin the GitHub commit or the spamtp-1.1.sif Apptainer image for reproducibility

## Evidence

- [abstract] SpaMTP includes a suite of visualisation functions for spatial metabolomics: [evidence span withheld — anchor status: not_found]
- [results] Volcano plot and heatmap are the stated visualisation outputs: [evidence span withheld — anchor status: not_found]
- [readme] SpaMTP installs with edgeR as a dependency: [evidence span withheld — anchor status: external_doc]
- [readme] SpaMTP build on a Seurat Class Object foundation: [evidence span withheld — anchor status: external_doc]
