---
name: fdr2d-isoline-visualization
description: Use when a metabolomics feature-level results table (mean difference,
  standard error, p-value, fdr2d per hypothesis) must be visualized as the paper's
  Figure 8 volcano/tornado pair with fdr2d isolines.
when_to_use_negative:
- Input lacks per-feature p-values or standard errors, so neither the tornado (-log
  p-value) nor volcano (log standard error) y-axis can be constructed
- The goal is metabolite identification or data preprocessing rather than visualizing
  fdr2d biomarker discovery results
- A single-panel or conventional 1D-FDR (BH q-value) volcano plot is sufficient and
  fdr2d isolines are not required
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3361
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Statistical environment used to compute log(standard error) and -log10(p-value)
    per feature and assemble the two-panel figure
  repo: jjs3098/CNU-Bioinformatics-Lab
- name: ggplot2
  role: R graphics package used to draw the scatter panels and overlay the fdr2d isolines
    before saving figure8_fdr2d_plots.png
  repo: jjs3098/CNU-Bioinformatics-Lab
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo9050103
    title: 'MetPC: Metabolite Pipeline Consisting of Metabolite Identification and
      Biomarker Discovery Under the Control of Two-Dimensional FDR'
  source_content_hash: sha256:f4bd38a46d38fd56f1af0b78f60c6d86c921e9647eb37c9301b8ce661504f8c3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jjs3098/CNU-Bioinformatics-Lab
  tools:
  - R
  - ggplot2
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3361
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# fdr2d-isoline-visualization

## Summary

Reproduces the Figure 8-style graphic of the MetPC biomarker discovery output: a two-panel scatter display (volcano panel with log standard error on y; tornado panel with -log p-value on y; both versus mean difference on x) with two-dimensional false discovery rate (fdr2d) isolines overlaid to delineate biomarker decision regions.

## When to use

Use when a metabolomics feature-level results table (mean difference, standard error, p-value, fdr2d per hypothesis) must be visualized as the paper's Figure 8 volcano/tornado pair with fdr2d isolines.

## When NOT to use

- Input lacks per-feature p-values or standard errors, so neither the tornado (-log p-value) nor volcano (log standard error) y-axis can be constructed
- The goal is metabolite identification or data preprocessing rather than visualizing fdr2d biomarker discovery results
- A single-panel or conventional 1D-FDR (BH q-value) volcano plot is sufficient and fdr2d isolines are not required

## Inputs

- fdr2d analysis results table with per-feature columns: mean difference, standard error, p-value, fdr2d (e.g., for the Schisandra chinensis Korea-vs-China comparison)
- chosen fdr2d isoline levels (0.05, 0.1, 0.2, 0.3 as reported)

## Outputs

- figure8_fdr2d_plots.png: two-panel graphic (volcano: mean difference vs log standard error; tornado: mean difference vs -log10 p-value) with fdr2d isolines overlaid
- counts of rejected null hypotheses per fdr2d level (expected: 1092 at 0.05; 1326, 1782, 2120 at 0.1, 0.2, 0.3)

## How to apply

Load the fdr2d analysis results table produced for the Schisandra chinensis (Korea vs China) dataset, which holds per-feature mean differences, standard errors, p-values, and fdr2d values. Build the volcano panel as a scatter of log(standard error) on the y-axis versus mean difference on the x-axis, then overlay the two-dimensional FDR isolines that define the rejection regions. Build the tornado panel as a scatter of -log10(p-value) on the y-axis versus mean difference on the x-axis with the same isolines. Combine both panels into one figure with axis labels 'mean difference' (x), 'log standard error' and '-log p-value' (y), and save it as figure8_fdr2d_plots.png. The article reports that at fdr2d = 0.05, 1092 null hypotheses are rejected (1326, 1782, and 2120 at levels 0.1, 0.2, and 0.3), so isolines should be drawn at these levels; implement the panels in R with ggplot2 as in the source workflow.

## Related tools

- **R** (Statistical environment used to compute log(standard error) and -log10(p-value) per feature and assemble the two-panel figure) — jjs3098/CNU-Bioinformatics-Lab
- **ggplot2** (R graphics package used to draw the scatter panels and overlay the fdr2d isolines before saving figure8_fdr2d_plots.png) — jjs3098/CNU-Bioinformatics-Lab

## Examples

```
Rscript -e 'g <- ggplot(fdr2d_results, aes(mean.difference, log(standard.error))) + geom_point() + labs(y="log standard error", x="mean difference"); ggsave("figure8_fdr2d_plots.png", plot=g)'
```

## Evaluation signals

- The saved figure contains exactly two scatter panels: volcano (y = log standard error) and tornado (y = -log10 p-value), both with mean difference on x
- Visible fdr2d isolines are present on both panels and correspond to the reported levels 0.05, 0.1, 0.2, and 0.3
- Rejection counts implied by the isolines match the article's reported numbers (1092 at fdr2d = 0.05; 1326, 1782, 2120 at 0.1, 0.2, 0.3)
- Axis labels read 'mean difference' on x and 'log standard error' / '-log p-value' on y, matching the Figure 8 layout
- Output file is saved as figure8_fdr2d_plots.png and renders both panels without missing points for any feature in the results table

## Limitations

- Isoline rejection counts are dataset-specific: the reported counts (1092/1326/1782/2120) apply to the Schisandra chinensis Korea-vs-China analysis and must be recomputed for other datasets
- Requires the complete per-feature fdr2d results table; if fdr2d values are not exported, isolines cannot be reconstructed
- The article's extracted index contains no discussion-section text or changelog, so edge-case guidance (e.g., tie handling, degenerate isolines) is not documented

## Evidence

- [results] Figure 8 plots each metabolite's hypothesis as a point with fdr2d isolines overlaid: [evidence span withheld — anchor status: not_found]
- [abstract] For biomarker metabolite discovery, our procedure controls two-dimensional false discovery rate (fdr2d) when testing for multiple hypotheses simultaneously.: "For biomarker metabolite discovery, our procedure controls two-dimensional false discovery rate (fdr2d) when testing for multiple hypotheses simultaneously."
- [abstract] we developed a new bioinformatics tool that includes a few of preprocessing steps and biomarker discovery procedure.: "we developed a new bioinformatics tool that includes a few of preprocessing steps and biomarker discovery procedure."
