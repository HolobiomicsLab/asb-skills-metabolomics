---
name: log-scale-plot-annotation
description: Use when the task is to graphically summarize two-dimensional FDR (fdr2d)
  multiple-testing results for metabolomics biomarker discovery as volcano/tornado
  scatter plots with fdr2d isolines overlaid.
when_to_use_negative:
- Input is a raw metabolomics data file rather than an fdr2d results table with mean
  differences, standard errors, and p-values per feature
- The analysis goal is metabolite identification via the hierarchical EM model, not
  visualization of fdr2d biomarker discovery results
- A one-dimensional FDR procedure is in use, since this skill specifically visualizes
  two-dimensional FDR (fdr2d) isolines
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Programming environment used to load the fdr2d results table, compute log(standard
    error) and -log(p value), and assemble the two-panel figure
  repo: jjs3098/CNU-Bioinformatics-Lab
- name: ggplot2
  role: R graphics package used to draw the volcano and tornado scatter panels with
    overlaid fdr2d isolines and axis labels
  repo: jjs3098/CNU-Bioinformatics-Lab
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo9050103
    title: 'MetPC: Metabolite Pipeline Consisting of Metabolite Identification and
      Biomarker Discovery Under the Control of Two-Dimensional FDR'
  source_content_hash: sha256:669ff200ca6914bd528aecd6252e96793ac3988089de817cd13701fb43aad259
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jjs3098/CNU-Bioinformatics-Lab
  tools:
  - R
  - ggplot2
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# log-scale-plot-annotation

## Summary

Reproduces Figure 8 of the MetPC paper by plotting the Schisandra chinensis (Korea vs China) fdr2d biomarker discovery results as a two-panel graphic: a volcano plot (mean difference vs log(standard error)) and a tornado plot (mean difference vs -log(p value)), each overlaid with fdr2d isolines to delineate rejected hypotheses.

## When to use

Use when the task is to graphically summarize two-dimensional FDR (fdr2d) multiple-testing results for metabolomics biomarker discovery as volcano/tornado scatter plots with fdr2d isolines overlaid.

## When NOT to use

- Input is a raw metabolomics data file rather than an fdr2d results table with mean differences, standard errors, and p-values per feature
- The analysis goal is metabolite identification via the hierarchical EM model, not visualization of fdr2d biomarker discovery results
- A one-dimensional FDR procedure is in use, since this skill specifically visualizes two-dimensional FDR (fdr2d) isolines

## Inputs

- fdr2d analysis results table for the Schisandra chinensis (Korea vs China) metabolomics dataset with per-feature mean differences, standard errors, p-values, and fdr2d values

## Outputs

- figure8_fdr2d_plots.png — a single two-panel graphic with a volcano plot (log standard error vs mean difference) and a tornado plot (-log p-value vs mean difference), each with fdr2d isolines

## How to apply

Load the fdr2d analysis results table for the Schisandra chinensis dataset, which contains per-feature mean differences, standard errors, p-values, and fdr2d values. For the volcano panel, compute log(standard error) for each feature and scatter it on the y-axis against mean difference on the x-axis; for the tornado panel, compute -log(p value) on the y-axis against the same x-axis. Overlay the two-dimensional false discovery rate isolines on both panels so that features rejected at a given fdr2d level fall inside the delineated decision region. In the reference implementation, fdr2d = 0.05 rejects 1092 null hypotheses (with 1326, 1782, and 2120 at levels 0.1, 0.2, and 0.3), so the isolines should make the rejection region visibly shrink or grow as the level changes. Label the x-axis 'mean difference' and the y-axes 'log standard error' and '-log p-value', combine both panels into a single graphic, and save it as figure8_fdr2d_plots.png using R with ggplot2.

## Related tools

- **R** (Programming environment used to load the fdr2d results table, compute log(standard error) and -log(p value), and assemble the two-panel figure) — jjs3098/CNU-Bioinformatics-Lab
- **ggplot2** (R graphics package used to draw the volcano and tornado scatter panels with overlaid fdr2d isolines and axis labels) — jjs3098/CNU-Bioinformatics-Lab

## Examples

```
Rscript figure8_fdr2d_plots.R --input fdr2d_results.csv --output figure8_fdr2d_plots.png
```

## Evaluation signals

- The saved figure contains both a volcano panel (log standard error on y-axis) and a tornado panel (-log p-value on y-axis), both sharing mean difference on the x-axis, matching the reported layout of Figure 8
- fdr2d isolines are visible on both panels and the number of rejected hypotheses at fdr2d = 0.05 is 1092 (1326, 1782, and 2120 at levels 0.1, 0.2, and 0.3)
- Axis labels read 'mean difference' (x) and 'log standard error' / '-log p-value' (y) as specified
- Output file figure8_fdr2d_plots.png exists and renders both panels with the specified orientations

## Limitations

- The visualization is specific to fdr2d outputs of the MetPC biomarker discovery procedure and presumes the results table already contains mean differences, standard errors, p-values, and fdr2d values
- Rejected-hypothesis counts (1092 at fdr2d = 0.05; 1326, 1782, 2120 at 0.1/0.2/0.3) apply to the Schisandra chinensis Korea-vs-China comparison and will differ for other datasets
- No discussion section text was available for the article, so practical caveats about plot interpretation beyond the figure description are not documented

## Evidence

- [results] Figure 8 volcano/tornado layout: [evidence span withheld — anchor status: not_found]
- [results] fdr2d rejection counts: [evidence span withheld — anchor status: not_found]
- [abstract] fdr2d biomarker discovery control: "our procedure controls two-dimensional false discovery rate (fdr2d) when testing for multiple hypotheses simultaneously."
