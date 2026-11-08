---
name: metabolite-class-bar-plot-visualization
description: Use when you have omu_summary output (fold changes and BH-adjusted p
  values per metabolite) plus assign_hierarchy KEGG class metadata and need a class-level
  Decrease/Increase count figure.
when_to_use_negative:
- You have not yet run omu_summary or assign_hierarchy — the plot requires fold-change/p-value
  statistics joined to KEGG hierarchy metadata, not raw peak intensities.
- The comparison involves more than two experimental groups — omu_summary performs
  a t test between a numerator and denominator factor level (e.g., Strep vs Mock),
  so class-level increase/decrease counts are undefined for multi-group designs (consider
  omu_anova instead).
- You need compound-level (not class-level) visualization, e.g., a volcano plot via
  plot_volcano.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: omu
  role: Provides count_fold_changes to tabulate significant increase/decrease counts
    per KEGG class and plot_bar to render the grouped bar plot as a ggplot2 object
  repo: github.com/connor-reid-tiffany/Omu
- name: R
  role: Language in which omu is implemented and the pipeline is executed
- name: assign_hierarchy
  role: Supplies the KEGG hierarchical class metadata that count_fold_changes aggregates
    over (column = "Class")
- name: ggplot2
  role: Backend of plot_bar output; the figure is a ggplot2 object compatible with
    any ggplot2 themes
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1128/mra.00129-19
    title: omu, a Metabolomics Count Data Analysis Tool for Intuitive Figures and
      Convenient Metadata Collection
  source_content_hash: sha256:d80a5cb7587c1dbf91245fbce999fbc1574efac8b79eee6497f4064c15c20e19
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - assign_hierarchy
  - ggplot2
  - omu
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# metabolite-class-bar-plot-visualization

## Summary

Renders a grouped bar plot (ggplot2 object via omu's plot_bar) showing how many metabolites significantly increased or decreased between two experimental groups, aggregated by KEGG compound Class or Subclass, as in Fig. 1A of the omu paper.

## When to use

Use when you have omu_summary output (fold changes and BH-adjusted p values per metabolite) plus assign_hierarchy KEGG class metadata and need a class-level Decrease/Increase count figure.

## When NOT to use

- You have not yet run omu_summary or assign_hierarchy — the plot requires fold-change/p-value statistics joined to KEGG hierarchy metadata, not raw peak intensities.
- The comparison involves more than two experimental groups — omu_summary performs a t test between a numerator and denominator factor level (e.g., Strep vs Mock), so class-level increase/decrease counts are undefined for multi-group designs (consider omu_anova instead).
- You need compound-level (not class-level) visualization, e.g., a volcano plot via plot_volcano.

## Inputs

- omu_summary statistics table with adjusted p values and fold changes per metabolite (e.g., from c57_nos2KO_mouse_countDF)
- assign_hierarchy metadata mapping KEGG identifiers to compound Class/Subclass (keep_unknowns = TRUE retains compounds without KEGG numbers)

## Outputs

- Grouped bar plot (ggplot2 object) of significantly increased/decreased compound counts per KEGG class, saved as class_fold_change_barplot.png
- count_fold_changes count table of compounds per KEGG class category

## How to apply

First join the omu_summary statistics table (built with numerator 'Strep' vs denominator 'Mock' on the Treatment factor, log_transform = TRUE, p_adjust = 'BH', test_type = 'welch') with hierarchy metadata from assign_hierarchy(identifier = "KEGG", keep_unknowns = TRUE). Then apply count_fold_changes with column = "Class" and sig_threshold = 0.05 to tabulate how many compounds per class significantly increased or decreased. Finally call plot_bar on the count table, styling fills with dodgerblue2 for one direction and firebrick2 for the other with black outlines; the result is a ggplot2 object compatible with any ggplot2 themes, which you save as class_fold_change_barplot.png. The sig_threshold argument drives which bars are colored, so it must match the significance cutoff used upstream.

## Related tools

- **omu** (Provides count_fold_changes to tabulate significant increase/decrease counts per KEGG class and plot_bar to render the grouped bar plot as a ggplot2 object) — github.com/connor-reid-tiffany/Omu
- **R** (Language in which omu is implemented and the pipeline is executed)
- **assign_hierarchy** (Supplies the KEGG hierarchical class metadata that count_fold_changes aggregates over (column = "Class"))
- **ggplot2** (Backend of plot_bar output; the figure is a ggplot2 object compatible with any ggplot2 themes)

## Examples

```
count_df <- count_fold_changes(omu_summary(omu::c57_nos2KO_mouse_countDF, metadata = omu::c57_nos2KO_mouse_metadata, numerator = "Strep", denominator = "Mock", factor_of_interest = "Treatment", log_transform = TRUE, p_adjust = "BH", test_type = "welch"), column = "Class", sig_threshold = 0.05); plot_bar(count_df, fill = c("dodgerblue2", "firebrick2"))
```

## Evaluation signals

- class_fold_change_barplot.png exists as a valid PNG file.
- The bar heights equal counts from count_fold_changes(column = "Class", sig_threshold = 0.05) — counts must match the BH-adjusted p < 0.05 compounds in the omu_summary table.
- Bars are separated into Decrease and Increase groups per KEGG class, comparable to Fig. 1A.
- Fill colors are dodgerblue2 and firebrick2 with black outlines, and the figure behaves as a ggplot2 object (theme-compatible).
- Compounds lacking KEGG numbers are handled per keep_unknowns = TRUE rather than silently dropped from class counts.

## Limitations

- sig_threshold determines which bars are colored/filled, so an inconsistency with the upstream p-adjustment cutoff (e.g., p_adjust = "BH") yields a misleading figure.
- Counts are restricted to the KEGG compound category chosen (Class or Subclass); compounds without KEGG numbers are only retained if keep_unknowns = TRUE.
- The workflow is specific to two-group comparisons via omu_summary with test_type options "students", "welch", or "mwu"; paired designs need paired = TRUE.

## Evidence

- [abstract] provide a count table of every metabolite within a KEGG compound category that significantly increased or decreased between groups: "provide a count table of every metabolite within a KEGG compound category that significantly increased or decreased between groups"
- [methods] The figure is a ggplot2 object, so it is compatible with any ggplot2 themes: "The figure is a ggplot2 object, so it is compatible with any ggplot2 themes"
- [methods] ```sig_threshold``` value will be filled red: "```sig_threshold``` value will be filled red"
- [methods] ```keep_unknowns = TRUE``` keeps compounds without KEGG numbers: "```keep_unknowns = TRUE``` keeps compounds without KEGG numbers"
