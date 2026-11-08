---
name: metabolomics-statistical-testing
description: Use when metabolomics count data (e.g. c57_nos2KO_mouse_countDF) must
  be tested for per-metabolite differences between two groups (e.g. Mock vs Strep)
  with BH-adjusted p-values and fold-change output for downstream plotting.
when_to_use_negative:
- When the comparison involves more than two experimental groups for a single factor
  — use omu_anova instead of omu_summary's two-group t-test framing.
- When compounds lack KEGG numbers and hierarchy-linked counting is required and keep_unknowns
  = TRUE cannot reconcile them — assign_hierarchy-based summaries depend on KEGG identifiers.
- When the goal is enzyme orthology or gene annotation retrieval rather than group-wise
  statistical testing — that is KEGG_gather's role.
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: omu
  role: Primary R package providing omu_summary (t-test + fold change), omu_anova,
    assign_hierarchy, count_fold_changes, and plot_volcano for the statistical testing
    workflow
  repo: github.com/connor-reid-tiffany/Omu
- name: R
  role: Runtime environment in which omu's statistical and plotting functions are
    executed
- name: ggplot2
  role: Renders the plot_volcano output as a ggplot2 object compatible with themes
    (e.g. theme_bw) for the volcano plot
- name: read.metabo
  role: Loads the raw metabolomics count table into R before statistical testing
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1128/mra.00129-19
    title: omu, a Metabolomics Count Data Analysis Tool for Intuitive Figures and
      Convenient Metadata Collection
  source_content_hash: sha256:95a59f9734628318b04b9e33ba268b9f7729def89aaf69d78f3b496e00a79df1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - ggplot2
  - omu
  - read.metabo
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# metabolomics-statistical-testing

## Summary

Statistically compare metabolomics count data between two experimental groups using omu's omu_summary (t-test/ANOVA variants) and count_fold_changes, producing per-metabolite adjusted p-values, log2 fold-changes, and KEGG-hierarchy-level significance counts that feed directly into visualization.

## When to use

Use when metabolomics count data (e.g. c57_nos2KO_mouse_countDF) must be tested for per-metabolite differences between two groups (e.g. Mock vs Strep) with BH-adjusted p-values and fold-change output for downstream plotting.

## When NOT to use

- When the comparison involves more than two experimental groups for a single factor — use omu_anova instead of omu_summary's two-group t-test framing.
- When compounds lack KEGG numbers and hierarchy-linked counting is required and keep_unknowns = TRUE cannot reconcile them — assign_hierarchy-based summaries depend on KEGG identifiers.
- When the goal is enzyme orthology or gene annotation retrieval rather than group-wise statistical testing — that is KEGG_gather's role.

## Inputs

- Metabolomics count data frame (e.g. c57_nos2KO_mouse_countDF loaded via read.metabo)
- Metadata file mapping samples to factors (e.g. c57_nos2KO_mouse_metadata with Treatment factor)
- Optional KEGG compound identifiers for hierarchy annotation (assign_hierarchy, identifier = 'KEGG')

## Outputs

- omu_summary result table with per-metabolite padj, log2FoldChange, standard error, and standard deviation
- count_fold_changes count table of significantly increased/decreased metabolites per KEGG compound category
- ggplot2 volcano plot object (via plot_volcano) highlighting compounds by hierarchy metadata

## How to apply

Load the metabolomics count table with omu's read.metabo function (or use the packaged example c57_nos2KO_mouse_countDF and c57_nos2KO_mouse_metadata), optionally assigning KEGG hierarchy metadata first via assign_hierarchy(count_data, keep_unknowns = TRUE, identifier = 'KEGG') so results can be linked to compound classes. Call omu_summary with numerator and denominator set to the two treatment groups (e.g. numerator = 'Strep', denominator = 'Mock'), response_variable = 'Metabolite', Factor = 'Treatment', log_transform = TRUE to log-transform the data, p_adjust = 'BH' for Benjamini & Hochberg correction, and test_type = 'welch' (alternatives: 'students' or 'mwu'; paired tests via paired = TRUE). omu_summary returns padj, log2FoldChange, standard error, and standard deviation for each metabolite; these are the inputs to plot_volcano for a volcano plot (Fig. 1B style) or to count_fold_changes, which builds a count table of metabolites significantly increased or decreased within KEGG compound categories. As an alternative to omu_summary, omu_anova performs per-metabolite ANOVA; transformation functions can be applied first to make the data more appropriate for the models.

## Related tools

- **omu** (Primary R package providing omu_summary (t-test + fold change), omu_anova, assign_hierarchy, count_fold_changes, and plot_volcano for the statistical testing workflow) — github.com/connor-reid-tiffany/Omu
- **R** (Runtime environment in which omu's statistical and plotting functions are executed)
- **ggplot2** (Renders the plot_volcano output as a ggplot2 object compatible with themes (e.g. theme_bw) for the volcano plot)
- **read.metabo** (Loads the raw metabolomics count table into R before statistical testing)

## Examples

```
omu_summary(count_data = c57_nos2KO_mouse_countDF, metadata = c57_nos2KO_mouse_metadata, numerator = "Strep", denominator = "Mock", response_variable = "Metabolite", Factor = "Treatment", log_transform = TRUE, p_adjust = "BH", test_type = "welch")
```

## Evaluation signals

- omu_summary output contains columns for padj, log2FoldChange, standard error, and standard deviation for every metabolite in the count data frame.
- p-values are Benjamini & Hochberg adjusted (p_adjust = 'BH') rather than raw, and the chosen test_type ('students', 'welch', or 'mwu') matches the intended test.
- count_fold_changes yields a count table per KEGG compound category restricted to metabolites significantly increased or decreased between the two groups.
- The volcano plot (plot_volcano, Fig. 1B structure) shows a dashed significance line at adjusted p value 0.05 with compounds highlighted by Class metadata and non-highlighted points labelled NA.
- Saved plot artifact (e.g. volcano_plot.png) exists, is a valid PNG, and was produced from the omu_summary statistical output rather than raw counts.

## Limitations

- Significance and fold-change summaries require a two-group numerator/denominator design; multi-group comparisons need the alternative omu_anova function.
- Hierarchy-linked counting and highlighting depend on KEGG identifiers; keep_unknowns = TRUE is needed to retain compounds without KEGG numbers, otherwise those compounds drop out.
- Data may need transformation (log_transform = TRUE or transform functions) before the models are appropriate.

## Evidence

- [abstract] perform a student’s t test on each metabolite between two experimental groups, provide a fold-change value for each compound: "perform a student’s t test on each metabolite between two experimental groups, provide a fold-change value for each compound"
- [methods] p_adjust: "a p value adjustment method of Benjamini & Hochberg with the argument ```p_adjust = "BH"```"
- [methods] test_type: "```test_type``` argument is one of "students", "welch", or "mwu""
- [methods] log_transform: "The data can be log transformed using ```log_transform = TRUE```"
- [abstract] count_fold_changes: "provide a count table of every metabolite within a KEGG compound category that significantly increased or decreased between groups"
- [methods] omu_anova alternative: "An alternative option to ```omu_summary``` is the ```omu_anova```"
- [methods] keep_unknowns: "```keep_unknowns = TRUE``` keeps compounds without KEGG numbers"
