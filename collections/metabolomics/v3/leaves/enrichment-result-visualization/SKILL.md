---
name: enrichment-result-visualization
description: Use when you have a reporter_score_res object (pathway-level reporter
  scores with adjusted p-values) from GRSA and need ranked enrichment tables and figures
  showing which modules are up- or down-enriched per group.
when_to_use_negative:
- Before the GRSA step has produced a reporter_score_res object — visualization operates
  on aggregated pathway-level scores, not raw KO p-values
- When the input abundance table was prefiltered, since GRSA is threshold-free and
  expects full background features; downstream figures would not reflect the paper's
  design
- When you need method sensitivity/specificity benchmarking rather than visualizing
  one run's enrichment results
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3174
tools:
- name: ReporterScore
  role: Provides export_report_table, plot_report (rs_threshold, mode, show_ID, Pathway_description),
    and plot_enrich_res to export and visualize GRSA enrichment results
  repo: https://github.com/Asa12138/ReporterScore
- name: R
  role: Runtime environment in which the ReporterScore visualization and export functions
    are executed
- name: devtools
  role: Used to install ReporterScore and its pcutils dependency from GitHub before
    running the visualization workflow
  repo: github.com/Asa12138/pcutils
provenance:
  source_task_ids:
  - task_005
  - task_008
  - task_004
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae116
    title: Generalized reporter score-based enrichment analysis for omics data
  source_content_hash: sha256:c79ae9582c9041ce6356c268451c26bb3da2d865cb7f8bfe7c7259a3b7fac625
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Asa12138/ReporterScore
  tools:
  - R
  - ReporterScore
  - devtools
  - pcutils
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# enrichment-result-visualization

## Summary

Visualize and export pathway/module-level results from the ReporterScore GRSA pipeline, using plot_report, plot_enrich_res, and export_report_table to rank enriched modules by reporter score sign and adjusted p-value and to render age/cutotype/gestational-age enrichment trends.

## When to use

Use when you have a reporter_score_res object (pathway-level reporter scores with adjusted p-values) from GRSA and need ranked enrichment tables and figures showing which modules are up- or down-enriched per group.

## When NOT to use

- Before the GRSA step has produced a reporter_score_res object — visualization operates on aggregated pathway-level scores, not raw KO p-values
- When the input abundance table was prefiltered, since GRSA is threshold-free and expects full background features; downstream figures would not reflect the paper's design
- When you need method sensitivity/specificity benchmarking rather than visualizing one run's enrichment results

## Inputs

- reporter_score_res object from ReporterScore::reporter_score (pathway-level reporter scores, adjusted p-values, per-group results)
- Grouping metadata (aging groups, cutotypes, or continuous variable such as gestational age)
- Optional KEGG modulelist for Pathway_description labels

## Outputs

- Pathway-level enrichment result table exported via export_report_table (e.g., grsa_pregnancy_reporter_scores.csv, grsa_taxonomy_cutotype_results.csv)
- Enrichment figures from plot_report (bar mode with Pathway_description, rs_threshold, show_ID)
- Enrichment overview plots from plot_enrich_res (e.g., Figure 2C-D style grouped comparisons, pathway_reporter_scores.png)

## How to apply

After running reporter_score (or get_reporter_score) on a KO/gene/compound abundance table against the KEGG modulelist, inspect the resulting reporter_score_res object and rank enriched pathways by reporter score sign and adjusted p-value for each group (e.g., aging group, cutotype, gestational age). Export the pathway-level results with export_report_table(reporter_res, dir_name, overwrite) to produce a CSV table. Visualize enriched modules and group trends with plot_report, tuning rs_threshold, mode, show_ID, and Pathway_description to control which pathways are drawn and how they are labeled (e.g., bar mode showing pathway descriptions to reproduce Figure 5A-B), and use plot_enrich_res for enrichment overview plots. Validation is judgment-based: the exported tables/figures should recover the expected directionality — e.g., lipid A biosynthesis reporter scores increasing while d-glucuronate and related carbohydrate degradation scores decrease across young-to-old aging groups, or steroid hormone biosynthesis up-regulated with gestational age; absence of these directional trends flags the run for review.

## Related tools

- **ReporterScore** (Provides export_report_table, plot_report (rs_threshold, mode, show_ID, Pathway_description), and plot_enrich_res to export and visualize GRSA enrichment results) — https://github.com/Asa12138/ReporterScore
- **R** (Runtime environment in which the ReporterScore visualization and export functions are executed)
- **devtools** (Used to install ReporterScore and its pcutils dependency from GitHub before running the visualization workflow) — github.com/Asa12138/pcutils

## Examples

```
export_report_table(reporter_res, dir_name = "grsa_results", overwrite = TRUE); plot_report(reporter_res, rs_threshold = 1.64, mode = "bar", show_ID = FALSE, Pathway_description = TRUE)
```

## Evaluation signals

- Exported table contains one row per pathway per group with reporter score sign and adjusted p-value columns populated
- Ranking by reporter score sign reproduces reported directionality: lipid A biosynthesis increasing and d-glucuronate/carbohydrate degradation decreasing across young-to-old aging groups
- For the pregnancy metabolome application, plot_report (bar mode) shows steroid hormone biosynthesis and cortisol synthesis up-regulated and aromatic amino acid metabolism down-regulated with gestational age, matching Figure 5A-B
- Output figure files (e.g., pathway_reporter_scores.png) render with Pathway_description labels when show_ID/Pathway_description are set
- Absence of the expected directional trends in exported tables/figures flags the run for review

## Limitations

- GRSA exhibits slightly lower specificity, underscoring the necessity for additional experimental validations, so visualized enrichments should be treated as hypotheses
- Specificity of enrichment methods cannot be accurately evaluated because true negatives are hard to define due to incompleteness of databases
- No version number is stated for the ReporterScore package, limiting reproducibility of exact figure output
- Mechanisms behind aging-related skin microbiota functional changes require further study — the visualized trends do not explain causality

## Evidence

- [other] Export the pathway-level results with export_report_table(reporter_res, dir_name, overwrite) and visualize enriched modules and age trends with plot_report: [evidence span withheld — anchor status: not_found]
- [abstract] We implemented GRSA in an R package, ReporterScore, integrating with a powerful visualization module and updatable pathway databases: "We implemented GRSA in an R package, ReporterScore, integrating with a powerful visualization module and updatable pathway databases"
- [other] plot_report (bar mode, showing pathway descriptions) to reproduce the up-regulated (steroid hormone biosynthesis, cortisol synthesis): [evidence span withheld — anchor status: not_found]
- [discussion] GRSA exhibits a slightly lower specificity, underscoring the necessity for additional experimental validations.: "GRSA exhibits a slightly lower specificity, underscoring the necessity for additional experimental validations."
- [methods] devtools::install_github("Asa12138/ReporterScore")
library(ReporterScore): "devtools::install_github("Asa12138/ReporterScore")
library(ReporterScore)"
