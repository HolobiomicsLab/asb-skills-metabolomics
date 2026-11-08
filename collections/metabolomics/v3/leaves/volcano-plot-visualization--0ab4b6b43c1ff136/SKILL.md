---
name: volcano-plot-visualization
description: Use when a lipidomics dataset has been analyzed with lipidr's de_analysis/de_design
  (two-group, multi-group, or confounder-adjusted limma models) and the logFC/p-value
  results per lipid molecule need to be visualized as a volcano plot, or when reproducing
  published volcano plots from lipidr.
when_to_use_negative:
- The goal is QC of samples or measured lipids — use lipidr's PCA/box-plot QC and
  %CV filtering instead of a volcano plot
- The input is raw Skyline transition-level data that has not yet been summarized
  and normalized (e.g. PQN) — volcano plots summarize de_analysis output, not unprocessed
  intensities
- The objective is lipid set enrichment (LSEA of lipid classes, chain lengths, or
  unsaturation patterns) rather than per-molecule differential expression
edam_topics:
- http://edamontology.org/topic_3407
- http://edamontology.org/topic_2269
tools:
- name: lipidr
  role: R/Bioconductor package whose de_analysis/de_design functions fit limma models
    on loaded clinical variables and whose volcano-plot function visualizes the results
  repo: github.com/ahmohamed/lipidr
- name: limma
  role: Linear-modeling engine underlying the differential analysis whose logFC/p-value
    output the volcano plot displays; users needing custom design matrices are referred
    to the Limma User Guide
- name: Skyline
  role: Source of exported transition-level CSV results that lipidr reads in before
    DE analysis and volcano plotting
provenance:
  source_task_ids:
  - task_007
  - task_008
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00082
    title: 'lipidr: A Software Tool for Data Mining and Analysis of Lipidomics Datasets'
  source_content_hash: sha256:f98e3d8f5419b3b95658ab51959494ba0bd72e5efd4bf7627172e2657e5dba78
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Skyline
  - limma
  - lipidr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3407
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# volcano-plot-visualization

## Summary

Visualize lipidr differential-analysis results as volcano plots of log2 fold change versus statistical significance, using the plotting functions that ship with lipidr's limma-based de_analysis workflow. This turns per-lipid DE tables into an interpretable display of up- and down-regulated lipid classes (e.g. PCs/PGs up, CLs/TGs down in cancer tissue).

## When to use

Use when a lipidomics dataset has been analyzed with lipidr's de_analysis/de_design (two-group, multi-group, or confounder-adjusted limma models) and the logFC/p-value results per lipid molecule need to be visualized as a volcano plot, or when reproducing published volcano plots from lipidr workflows.

## When NOT to use

- The goal is QC of samples or measured lipids — use lipidr's PCA/box-plot QC and %CV filtering instead of a volcano plot
- The input is raw Skyline transition-level data that has not yet been summarized and normalized (e.g. PQN) — volcano plots summarize de_analysis output, not unprocessed intensities
- The objective is lipid set enrichment (LSEA of lipid classes, chain lengths, or unsaturation patterns) rather than per-molecule differential expression

## Inputs

- Skyline-exported CSV files (e.g. A1_data.csv from lipidr extdata)
- Numerical intensity matrix CSV (lipids as rows, samples as columns) with a matching sample-annotation CSV (e.g. clin.csv)
- LipidomicsExperiment/Lipidome object with loaded clinical variables (e.g. SampleType, Race, diet group)
- de_analysis/de_design results table with logFC and p-values per lipid (e.g. HighFat_water vs NormalDiet_water, measure='Area'; ST001111 Cancer-Benign and Cancer-Metastasis contrasts)

## Outputs

- Volcano plot of log2 fold change versus statistical significance per lipid, saved as a PNG file
- Optionally, the exported differential-expression results table (CSV) that the volcano plot summarizes

## How to apply

Import the lipidomics data into R as a LipidomicsExperiment (via read_skyline for Skyline CSV exports or as_lipidomics_experiment for numerical matrices, adding sample annotation with add_sample_annotation), then run de_analysis with an explicit contrast such as 'HighFat_water - NormalDiet_water' with measure='Area', or de_design(d, ~ Race + SampleType, coef='SampleTypeCancer') to adjust for a covariate. Extract the per-lipid results (logFC and p-values) and plot log2 fold change against statistical significance using lipidr's volcano-plot function; significant molecules are those below the standard adjusted-p-value threshold. Decision point: when the contrast yields a large number of significantly altered lipids, render the volcano plot without per-molecule labels (as in the HighFat vs NormalDiet vignette); with fewer hits, label individual lipids. Use the plot to verify reported class-level regulation patterns — e.g. PCs and PGs up-regulated, CLs and TGs down-regulated in cancer vs benign ST001111 samples — and to compare covariate-adjusted versus two-group contrasts, where similar patterns indicate only a small confounder effect.

## Related tools

- **lipidr** (R/Bioconductor package whose de_analysis/de_design functions fit limma models on loaded clinical variables and whose volcano-plot function visualizes the results) — github.com/ahmohamed/lipidr
- **limma** (Linear-modeling engine underlying the differential analysis whose logFC/p-value output the volcano plot displays; users needing custom design matrices are referred to the Limma User Guide)
- **Skyline** (Source of exported transition-level CSV results that lipidr reads in before DE analysis and volcano plotting)

## Examples

```
d <- read_skyline("Skyline_export.csv"); d <- add_sample_annotation(d, "clin.csv"); de_analysis(d, HighFat_water - NormalDiet_water, measure="Area")
```

## Evaluation signals

- The exported DE results table (CSV) and the volcano plot show the same set of significant molecules (adjusted p-value below the standard threshold) at the plot extremes
- Reproduced ST001111 volcano plots match the reported class-level pattern: PCs and PGs up-regulated and CLs and TGs down-regulated in cancer vs benign, with only minute differences between cancer and metastasis
- The covariate-adjusted (de_design ~ Race + SampleType) volcano plot shows a pattern similar to the two-group SampleType-only comparison, indicating a small Race effect
- For the diet workflow (HighFat_water vs NormalDiet_water, measure='Area'), the plot shows a large number of significantly altered lipids, consistent with rendering it without labels
- The rendered plot reflects the explicit contrast fitted (e.g. SampleTypeCancer coefficient, HighFat_water - NormalDiet_water), not an unlabeled default comparison

## Limitations

- When a contrast produces a large number of significantly altered lipids, the volcano plot is shown without labeling individual molecules, reducing per-lipid readability
- Differential analysis and its volcano visualization are restricted to clinical variables loaded into the LipidomicsExperiment via add_sample_annotation; more complex design matrices require consulting the Limma User Guide
- Volcano plots summarize limma-based linear-model output; they do not themselves validate normalization choices (e.g. PQN) or sample-quality filtering by %CV

## Evidence

- [readme] Univariate analysis can be performed using any of the loaded clinical variables, which can be readily visualized as volcano plots. Multi-group comparisons and adjusting for confounding variables is also supported: [evidence span withheld — anchor status: external_doc]
