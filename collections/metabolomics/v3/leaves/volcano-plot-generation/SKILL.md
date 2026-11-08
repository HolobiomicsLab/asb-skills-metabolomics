---
name: volcano-plot-generation
description: Use when you have a per-feature fdr2d results table (mean differences,
  standard errors, p-values, fdr2d values) and need to graphically summarize rejected
  hypotheses as volcano/tornado scatter plots with fdr2d isolines.
when_to_use_negative:
- Input is raw or preprocessed metabolomics spectra rather than a computed fdr2d results
  table with per-feature statistics
- The goal is one-dimensional FDR (e.g., BH-adjusted p-values) rather than the paper's
  two-dimensional false discovery rate (fdr2d) control
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Computes log(standard error) and -log10(p-value) per feature and assembles
    the two-panel Figure 8 graphic from the fdr2d results table
  repo: jjs3098/CNU-Bioinformatics-Lab
- name: ggplot2
  role: Draws the volcano and tornado scatter panels with overlaid fdr2d isolines
    and axis labels, saved via ggsave to figure8_fdr2d_plots.png
  repo: jjs3098/CNU-Bioinformatics-Lab
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo9050103
    title: 'MetPC: Metabolite Pipeline Consisting of Metabolite Identification and
      Biomarker Discovery Under the Control of Two-Dimensional FDR'
  source_content_hash: sha256:e6f658ff2a4680620bad2260492ddf8f9199d9250581950d3afe439c4a796829
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jjs3098/CNU-Bioinformatics-Lab
  tools:
  - R
  - ggplot2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# volcano-plot-generation

## Summary

Generates Figure-8-style fdr2d visualization panels for the MetPC Schisandra chinensis (Korea vs China) biomarker discovery output: a volcano panel (log standard error vs mean difference) and a tornado panel (-log p value vs mean difference), each overlaid with two-dimensional FDR (fdr2d) isolines.

## When to use

Use when you have a per-feature fdr2d results table (mean differences, standard errors, p-values, fdr2d values) and need to graphically summarize rejected hypotheses as volcano/tornado scatter plots with fdr2d isolines.

## When NOT to use

- Input is raw or preprocessed metabolomics spectra rather than a computed fdr2d results table with per-feature statistics
- The goal is one-dimensional FDR (e.g., BH-adjusted p-values) rather than the paper's two-dimensional false discovery rate (fdr2d) control

## Inputs

- fdr2d analysis results table (per metabolite feature: mean difference, standard error, p-value, fdr2d value) for the Schisandra chinensis Korea vs China comparison

## Outputs

- figure8_fdr2d_plots.png — two-panel figure: volcano panel (log standard error vs mean difference) and tornado panel (-log p-value vs mean difference), both with fdr2d isolines
- counts of rejected null hypotheses at fdr2d thresholds 0.05, 0.1, 0.2, and 0.3 (1092, 1326, 1782, 2120)

## How to apply

Load the fdr2d analysis results table for the Schisandra chinensis (Korea vs China) dataset, which contains per-feature mean differences, standard errors, p-values, and fdr2d values. Compute log(standard error) per feature and draw the volcano panel as a scatter plot with log(standard error) on the y-axis and mean difference on the x-axis. Compute -log10(p-value) per feature and draw the tornado panel with -log10(p-value) on the y-axis versus mean difference on the x-axis. Overlay the two-dimensional false discovery rate (fdr2d) isolines on both panels to delineate the biomarker discovery decision regions; at fdr2d = 0.05, 1092 null hypotheses are rejected (1326, 1782, and 2120 at levels 0.1, 0.2, and 0.3). Combine both panels into a single Figure 8-style graphic with axis labels ('mean difference' on x; 'log standard error' and '-log p-value' on y) and save as figure8_fdr2d_plots.png.

## Related tools

- **R** (Computes log(standard error) and -log10(p-value) per feature and assembles the two-panel Figure 8 graphic from the fdr2d results table) — jjs3098/CNU-Bioinformatics-Lab
- **ggplot2** (Draws the volcano and tornado scatter panels with overlaid fdr2d isolines and axis labels, saved via ggsave to figure8_fdr2d_plots.png) — jjs3098/CNU-Bioinformatics-Lab

## Examples

```
Rscript -e "d <- read.csv('fdr2d_results.csv'); library(ggplot2); p <- ggplot(d, aes(mean_difference, log(standard_error))) + geom_point() + labs(x='mean difference', y='log standard error'); ggsave('figure8_fdr2d_plots.png')"
```

## Evaluation signals

- The saved figure contains both panels with the specified axis orientations: log(standard error) and -log p-value on the y-axes, mean difference on the x-axes
- fdr2d isolines are visible in both panels, delineating the biomarker discovery decision regions
- At the fdr2d = 0.05 isoline, the number of rejected hypotheses matches the reported 1092 (with 1326, 1782, 2120 at 0.1, 0.2, 0.3)
- Axis labels match the reported Figure 8 layout ('mean difference' on x; 'log standard error' and '-log p-value' on y)
- Every feature in the fdr2d results table appears as exactly one point in each panel (no features dropped silently)

## Limitations

- The plotted rejection counts are specific to the MetPC Schisandra chinensis (Korea vs China) analysis; other datasets will yield different counts at the same fdr2d levels
- The visualization summarizes fdr2d-controlled hypothesis testing and does not itself perform metabolite identification, which in MetPC relies on a hierarchical statistical model with an EM algorithm for latent variables

## Evidence

- [abstract] fdr2d isolines and rejection counts: "our procedure controls two-dimensional false discovery rate (fdr2d) when testing for multiple hypotheses simultaneously."
- [other] Figure 8 panel design: [evidence span withheld — anchor status: not_found]
- [other] rejection counts at fdr2d levels: [evidence span withheld — anchor status: not_found]
