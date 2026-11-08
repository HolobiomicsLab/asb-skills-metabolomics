---
name: fold-change-computation-between-experimental-groups
description: Use when a metabolomics count data frame has exactly two experimental
  groups to contrast (e.g., a Treatment factor with numerator/denominator levels like
  'Strep' vs 'Mock') and per-metabolite fold-change and t-test statistics are needed.
when_to_use_negative:
- The design has more than two experimental groups — use the omu_anova (anova_function)
  alternative instead
- The goal is assigning KEGG compound hierarchy metadata or counting significant metabolites
  per KEGG category (assign_hierarchy / count_fold_changes) rather than two-group
  testing
- The input is not metabolomics count data and has not been made appropriate for the
  model via omu's transform functions
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0092
tools:
- name: omu
  role: R package supplying omu_summary, which performs the two-group t test and fold-change
    computation on metabolomics count data
  repo: github.com/connor-reid-tiffany/Omu
- name: read.metabo
  role: omu function recommended for loading metabolomics data files into R before
    running omu_summary
  repo: github.com/connor-reid-tiffany/Omu
- name: R
  role: runtime environment in which the omu package and omu_summary are executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1128/mra.00129-19
    title: omu, a Metabolomics Count Data Analysis Tool for Intuitive Figures and
      Convenient Metadata Collection
  source_content_hash: sha256:b4e63db48f9907026dd55f7c8bf3b77049f003318be58cea5cd049bc9ec504a2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - omu
  - omu_summary
  - read.metabo
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# fold-change-computation-between-experimental-groups

## Summary

Per-metabolite two-group statistical testing with fold-change calculation using the omu R package's omu_summary function, which computes group means, standard error, standard deviation, fold change, log2 fold change, p values, and Benjamini & Hochberg adjusted p values for every metabolite in metabolomics count data.

## When to use

Use when a metabolomics count data frame has exactly two experimental groups to contrast (e.g., a Treatment factor with numerator/denominator levels like 'Strep' vs 'Mock') and per-metabolite fold-change and t-test statistics are needed.

## When NOT to use

- The design has more than two experimental groups — use the omu_anova (anova_function) alternative instead
- The goal is assigning KEGG compound hierarchy metadata or counting significant metabolites per KEGG category (assign_hierarchy / count_fold_changes) rather than two-group testing
- The input is not metabolomics count data and has not been made appropriate for the model via omu's transform functions

## Inputs

- metabolomics count data frame (e.g., c57_nos2KO_mouse_countDF) with one row per metabolite and columns per sample
- metadata file mapping samples to experimental factors (e.g., c57_nos2KO_mouse_metadata)
- user-supplied metabolomics files loaded via the read.metabo function

## Outputs

- per-metabolite statistics data frame containing group means, standard error, standard deviation, fold change, log2 fold change, p values, and BH-adjusted p values
- results table exported as omu_summary_results.csv

## How to apply

Load the metabolomics count data and metadata, ideally via omu's read.metabo function for user-supplied files or the shipped example dataset c57_nos2KO_mouse_countDF with c57_nos2KO_mouse_metadata. Call omu_summary with numerator and denominator set to the two group levels of the Factor column (e.g., numerator = 'Strep', denominator = 'Mock', Factor = 'Treatment', response_variable = 'Metabolite') and set log_transform = TRUE so sample counts are log transformed before testing. Choose test_type from the supported options — 'students' for a student's t test, or 'welch' or 'mwu' as demonstrated alternatives — and optionally set paired = TRUE for paired designs. Specify p_adjust = 'BH' to apply Benjamini & Hochberg correction to the per-metabolite p values. The function returns a data frame of per-metabolite statistics (means, standard error, standard deviation, fold change, log2 fold change, p value, adjusted p value) that can be written to a results CSV such as omu_summary_results.csv; note that omu_anova is the documented alternative when more than two groups are being compared.

## Related tools

- **omu** (R package supplying omu_summary, which performs the two-group t test and fold-change computation on metabolomics count data) — github.com/connor-reid-tiffany/Omu
- **read.metabo** (omu function recommended for loading metabolomics data files into R before running omu_summary) — github.com/connor-reid-tiffany/Omu
- **R** (runtime environment in which the omu package and omu_summary are executed)

## Examples

```
omu_summary(keep_data=c57_nos2KO_mouse_countDF, metadata=c57_nos2KO_mouse_metadata, numerator="Strep", denominator="Mock", Factor="Treatment", response_variable="Metabolite", log_transform=TRUE, p_adjust="BH", test_type="welch")
```

## Evaluation signals

- Output data frame contains a statistics row for every metabolite in the input count data frame (no metabolites dropped)
- Columns for fold change and log2 fold change are populated; with log_transform = TRUE the log2 fold-change column should match fold change computed on log-transformed counts
- Adjusted p values reflect Benjamini & Hochberg correction applied across all metabolites (p_adjust = 'BH')
- test_type used matches one of the documented options 'students', 'welch', or 'mwu', and numerator/denominator levels match the metadata Factor column
- Results file (omu_summary_results.csv) round-trips with the in-memory omu_summary data frame

## Limitations

- omu_summary is limited to two experimental groups; for multi-group designs the article directs users to omu_anova ('An alternative option to omu_summary is the omu_anova')
- test_type must be chosen among 'students', 'welch', or 'mwu'; 'students' is the student's t test while 'welch' is the variant demonstrated in the paper's workflow
- Compounds without KEGG numbers are only retained if keep_unknowns = TRUE is set elsewhere in the omu workflow, which may affect downstream per-category interpretation

## Evidence

- [abstract] two-group t test and fold change: "perform a student’s t test on each metabolite between two experimental groups, provide a fold-change value for each compound"
- [methods] log transform filter step: "The data can be log transformed using ```log_transform = TRUE```"
- [methods] BH adjustment filter step: "a p value adjustment method of Benjamini & Hochberg with the argument ```p_adjust = "BH"```"
- [methods] test_type options: "```test_type``` argument is one of "students", "welch", or "mwu""
- [methods] anova alternative for multi-group designs: "An alternative option to ```omu_summary``` is the ```omu_anova```"
