---
name: metabolomics-group-comparison-statistics
description: Use when the input is a metabolite-by-sample count data frame with sample
  metadata on one or two grouping factors and the goal is per-metabolite fold change,
  adjusted p values, and group-level visualization.
when_to_use_negative:
- Data are already a precomputed feature table with annotations and statistics — instead
  skip ahead to count_fold_changes or plotting.
- The experimental design requires more than two groups without ANOVA, since omu_summary
  is built for a two-group comparison (use omu_anova for multi-group designs).
- Compounds lack any KEGG identifier and keep_unknowns = TRUE cannot retain them for
  class annotation.
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: omu
  role: provides read.metabo, assign_hierarchy, omu_summary/omu_anova, count_fold_changes,
    KEGG_gather, and plotting wrappers for the full comparison workflow
  repo: github.com/connor-reid-tiffany/Omu
- name: R
  role: runtime environment for the omu package
- name: KEGGREST
  role: retrieves KEGG API data used by assign_hierarchy and KEGG_gather
- name: ggplot2
  role: rendering backend for plot_bar and plot_volcano outputs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1128/mra.00129-19
    title: omu, a Metabolomics Count Data Analysis Tool for Intuitive Figures and
      Convenient Metadata Collection
  source_content_hash: sha256:fb86ca2206df1fd78be83c34a2e16f796c0e1d4134cec1dc3668b12491631ec6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KEGGREST
  - R
  - ggplot2
  - omu
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5652173913043478
n_claims: 23
---

# metabolomics-group-comparison-statistics

## Summary

Two-group metabolomics statistics (per-compound fold change plus t-test or ANOVA) with Benjamini–Hochberg correction, KEGG class annotation, and downstream visual summaries, as implemented in the omu R package for count-style metabolomics tables.

## When to use

Use when the input is a metabolite-by-sample count data frame with sample metadata on one or two grouping factors and the goal is per-metabolite fold change, adjusted p values, and group-level visualization.

## When NOT to use

- Data are already a precomputed feature table with annotations and statistics — instead skip ahead to count_fold_changes or plotting.
- The experimental design requires more than two groups without ANOVA, since omu_summary is built for a two-group comparison (use omu_anova for multi-group designs).
- Compounds lack any KEGG identifier and keep_unknowns = TRUE cannot retain them for class annotation.

## Inputs

- metabolite-by-sample count data frame (e.g. c57_nos2KO_mouse_countDF packaged with omu)
- metadata file mapping samples to experimental factors (e.g. c57_nos2KO_mouse_metadata)
- KEGG compound identifiers per metabolite (unknowns allowed when keep_unknowns = TRUE)

## Outputs

- per-metabolite statistics table with log-transformed fold changes and BH-adjusted p values
- count table of metabolites significantly increased or decreased per KEGG compound class or subclass
- ggplot2 bar plot of class-level fold-change counts and a volcano plot

## How to apply

Load the metabolomics count data frame with omu's read.metabo and pair it with the metadata file. Attach compound hierarchy using assign_hierarchy with identifier = "KEGG" and keep_unknowns = TRUE so compounds lacking KEGG IDs are retained. For a two-group comparison, run omu_summary with log_transform = TRUE, test_type set to one of "students", "welch", or "mwu" (paired tests via paired = TRUE), and p_adjust = "BH" to obtain a Benjamini & Hochberg adjusted p value and fold change per metabolite; use omu_anova for more than two groups. Aggregate significant results by KEGG class with count_fold_changes (column = "Class", sig_threshold = 0.05) and render grouped bar plots with plot_bar (a ggplot2 object) or volcano plots with plot_volcano. Optionally call KEGG_gather to link metabolites to KEGG enzyme orthology and gene annotations for hypothesis generation.

## Related tools

- **omu** (provides read.metabo, assign_hierarchy, omu_summary/omu_anova, count_fold_changes, KEGG_gather, and plotting wrappers for the full comparison workflow) — github.com/connor-reid-tiffany/Omu
- **R** (runtime environment for the omu package)
- **KEGGREST** (retrieves KEGG API data used by assign_hierarchy and KEGG_gather)
- **ggplot2** (rendering backend for plot_bar and plot_volcano outputs)

## Examples

```
omu_summary(count_data = c57_nos2KO_mouse_countDF, metadata = c57_nos2KO_mouse_metadata, numerator = "Strep", denominator = "Mock", factor = "Treatment", log_transform = TRUE, p_adjust = "BH", test_type = "welch")
```

## Evaluation signals

- Output statistics table contains a BH-adjusted p value column and a fold-change column for every metabolite row.
- With sig_threshold = 0.05, class counts in the bar plot equal the number of compounds per KEGG class whose adjusted p value crosses the threshold.
- Bar plot is a valid ggplot2 object and saves as a PNG (e.g. class_fold_change_barplot.png) with Decrease/Increase grouping.
- Volcano plot axes reflect log fold change versus adjusted p value and align with omu_summary output.
- Compounds without KEGG numbers remain in the hierarchy table when keep_unknowns = TRUE was set.

## Limitations

- omu_summary handles a single two-group comparison; multi-factor designs require omu_anova instead.
- Class annotation depends on KEGG IDs; compounds without KEGG numbers are only kept if keep_unknowns = TRUE.
- read.metabo is the recommended loader, so data must conform to the expected omu count-data format.

## Evidence

- [abstract] two-group comparison statistics: "perform a student’s t test on each metabolite between two experimental groups, provide a fold-change value for each compound"
- [methods] BH adjustment parameter: "a p value adjustment method of Benjamini & Hochberg with the argument ```p_adjust = "BH"```"
- [methods] KEGG hierarchy assignment: "To assign hierarchical class data, use the ```assign_hierarchy``` function"
- [methods] keep unknowns behavior: "```keep_unknowns = TRUE``` keeps compounds without KEGG numbers"
- [methods] class count threshold: "```sig_threshold``` value will be filled red"
- [abstract] class count workflow: "provide a count table of every metabolite within a KEGG compound category that significantly increased or decreased between groups"
