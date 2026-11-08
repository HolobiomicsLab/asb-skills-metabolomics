---
name: tornado-plot-generation
description: Use when a per-feature fdr2d results table (mean difference, standard
  error, p-value, fdr2d) needs to be visualized as Figure 8-style tornado/volcano
  panels to summarize biomarker discovery decisions.
when_to_use_negative:
- Input is raw unprocessed metabolomics data rather than a per-feature fdr2d results
  table with mean differences, standard errors, and p-values
- The analysis goal is metabolite identification via the hierarchical EM model, not
  graphical summarization of fdr2d biomarker discovery results
- Only a single-panel plot is needed and the fdr2d decision regions should not be
  displayed as isolines
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: R
  role: Scripting environment used to compute -log(p value) and log(standard error)
    transforms and assemble the two-panel figure
  repo: https://github.com/jjs3098/CNU-Bioinformatics-Lab
- name: ggplot2
  role: R graphics package used to draw the scatter points and overlay the fdr2d isolines
    on the tornado and volcano panels
  repo: https://github.com/jjs3098/CNU-Bioinformatics-Lab
- name: MetPC
  role: Upstream pipeline that produces the fdr2d biomarker discovery results table
    consumed by the tornado plot
  repo: https://github.com/jjs3098/CNU-Bioinformatics-Lab
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo9050103
    title: 'MetPC: Metabolite Pipeline Consisting of Metabolite Identification and
      Biomarker Discovery Under the Control of Two-Dimensional FDR'
  source_content_hash: sha256:89640e49ea1196cbd307c2dbecb143913639b97d14a589da8da3ec68fb98edf0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jjs3098/CNU-Bioinformatics-Lab
  tools:
  - MetPC
  - R
  - ggplot2
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# tornado-plot-generation

## Summary

Generates the tornado (volcano-style) plot from MetPC's fdr2d biomarker discovery output for the Schisandra chinensis Korea-vs-China data, scattering -log(p value) against mean difference with fdr2d isolines overlaid (Figure 8), alongside its companion log(standard error) volcano panel.

## When to use

Use when a per-feature fdr2d results table (mean difference, standard error, p-value, fdr2d) needs to be visualized as Figure 8-style tornado/volcano panels to summarize biomarker discovery decisions.

## When NOT to use

- Input is raw unprocessed metabolomics data rather than a per-feature fdr2d results table with mean differences, standard errors, and p-values
- The analysis goal is metabolite identification via the hierarchical EM model, not graphical summarization of fdr2d biomarker discovery results
- Only a single-panel plot is needed and the fdr2d decision regions should not be displayed as isolines

## Inputs

- fdr2d analysis results table for the Schisandra chinensis (Korea vs China) dataset with columns: mean difference, standard error, p-value, fdr2d
- fdr2d threshold levels for isolines (e.g. 0.05, 0.1, 0.2, 0.3)

## Outputs

- Tornado panel scatter plot: -log(p value) vs mean difference with overlaid fdr2d isolines
- Companion volcano panel: log(standard error) vs mean difference with overlaid fdr2d isolines
- Combined Figure 8-style graphic (e.g. figure8_fdr2d_plots.png)

## How to apply

Load the fdr2d analysis results table for the Schisandra chinensis (Korea vs China) dataset, which contains per-feature mean differences, standard errors, p-values, and fdr2d values. Compute -log10(p-value) for each feature and draw the tornado panel as a scatter plot of -log(p value) on the y-axis versus mean difference on the x-axis. In parallel, compute log(standard error) and draw the companion volcano panel on the same x-axis. Overlay two-dimensional false discovery rate (fdr2d) isolines on both panels to delineate the biomarker decision regions; at fdr2d = 0.05, 1092 null hypotheses are rejected (1326, 1782, and 2120 at levels 0.1, 0.2, and 0.3). Combine both panels into a single Figure 8-style graphic and save it (e.g. figure8_fdr2d_plots.png) for validation against the reported layout.

## Related tools

- **R** (Scripting environment used to compute -log(p value) and log(standard error) transforms and assemble the two-panel figure) — https://github.com/jjs3098/CNU-Bioinformatics-Lab
- **ggplot2** (R graphics package used to draw the scatter points and overlay the fdr2d isolines on the tornado and volcano panels) — https://github.com/jjs3098/CNU-Bioinformatics-Lab
- **MetPC** (Upstream pipeline that produces the fdr2d biomarker discovery results table consumed by the tornado plot) — https://github.com/jjs3098/CNU-Bioinformatics-Lab

## Examples

```
ggplot(res, aes(x=mean_difference, y=-log10(p_value))) + geom_point() + geom_contour(aes(z=fdr2d), breaks=c(0.05,0.1,0.2,0.3)) + labs(y='-log p-value', x='mean difference')
```

## Evaluation signals

- The saved figure contains both panels with the specified axis orientations: -log(p value) (tornado) and log(standard error) (volcano) on the y-axis, mean difference on the x-axis
- fdr2d isolines are visibly overlaid on both panels, delineating the biomarker discovery decision regions
- The number of points labeled as rejected at fdr2d = 0.05 matches the reported 1092 (with 1326, 1782, and 2120 at 0.1, 0.2, and 0.3)
- Every feature row in the results table appears as exactly one point in each panel (no dropped features)
- The combined figure matches the reported layout of Figure 8 of the MetPC article

## Limitations

- Isoline rejection counts depend on the chosen fdr2d level; thresholds other than 0.05/0.1/0.2/0.3 change which hypotheses appear rejected
- The plot is a visualization layer only — it does not perform the fdr2d multiple-testing control itself, which is done upstream by the MetPC biomarker discovery procedure
- The skill is grounded in the Schisandra chinensis Korea-vs-China dataset; other two-class comparisons require re-deriving the results table first

## Evidence

- [other] fdr2d isolines and rejection counts: [evidence span withheld — anchor status: not_found]
- [abstract] fdr2d control in biomarker discovery: "our procedure controls two-dimensional false discovery rate (fdr2d) when testing for multiple hypotheses simultaneously."
- [abstract] MetPC tool scope: "we developed a new bioinformatics tool that includes a few of preprocessing steps and biomarker discovery procedure."
