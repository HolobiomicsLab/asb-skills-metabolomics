---
name: pairwise-contrast-pvalue-computation
description: Use when you have a wide-format feature table (features as rows, samples
  as columns) plus a design file mapping samples to treatment groups, and you need
  per-feature pairwise group contrast p-values and binary significance flags.
when_to_use_negative:
- Input is not a wide-format feature table with a design file mapping samples to treatment
  groups — the SECIMTools ANOVA tool expects features as rows and samples as columns
  with unique feature identifiers.
- The analysis goal is non-parametric group comparison on non-normally distributed
  feature data — the tool's fixed ANOVA assumes normal and identically distributed
  errors, so the Kruskal-Wallis non-parametric test in the same suite would be the
  appropriate choice.
- Only a single group's data is available with no treatment-group design file, since
  pairwise contrasts between groups cannot be computed without group annotations.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: SECIMTools
  role: Source suite whose ANOVA tool performs the per-feature fully fixed single-factor
    ANOVA with all pairwise contrasts and generates volcano plots and model diagnostics;
    available standalone or wrapped for Galaxy.
  repo: https://github.com/SECIMTools
- name: Galaxy
  role: Web platform into which SECIMTools can be imported and the ANOVA tool executed
    as part of a workflow.
- name: Python
  role: Implementation language of the SECIMTools ANOVA tool (set of Python applications).
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-018-2134-1
    title: 'SECIMTools: a suite of metabolomics data analysis tools'
  source_content_hash: sha256:f4bd7f43c39bd9960a11e25a10ae373056bfb01d95715fa17450269ff70eefda
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Galaxy
  - Python
  - SECIMTools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# pairwise-contrast-pvalue-computation

## Summary

Computes per-feature pairwise group-contrast p-values and significance flags from a fully fixed single-factor ANOVA on a metabolomics feature table, as implemented in the SECIMTools ANOVA tool.

## When to use

Use when you have a wide-format feature table (features as rows, samples as columns) plus a design file mapping samples to treatment groups, and you need per-feature pairwise group contrast p-values and binary significance flags.

## When NOT to use

- Input is not a wide-format feature table with a design file mapping samples to treatment groups — the SECIMTools ANOVA tool expects features as rows and samples as columns with unique feature identifiers.
- The analysis goal is non-parametric group comparison on non-normally distributed feature data — the tool's fixed ANOVA assumes normal and identically distributed errors, so the Kruskal-Wallis non-parametric test in the same suite would be the appropriate choice.
- Only a single group's data is available with no treatment-group design file, since pairwise contrasts between groups cannot be computed without group annotations.

## Inputs

- Bundled SECIMTools example wide-format dataset (samples as columns, metabolomics features as rows with unique feature identifiers)
- Design file mapping each sample to its treatment group annotation

## Outputs

- Per-feature results table (anova_results.csv) with group means, ANOVA F-statistics, pairwise contrast p-values, and binary significance flag columns
- Volcano plots (log base 10 p-value against the difference between group means)
- Model diagnostics

## How to apply

Fit the fully fixed ANOVA model per feature across the treatment groups as specified by SECIMTools, assuming normal and identically distributed errors, with all pairwise contrasts calculated. Extract the raw p-value for each pairwise group contrast, then derive a binary significance flag column per contrast from that p-value. Alongside the p-values, compute per-feature group means and the ANOVA F-statistics. SECIMTools lists this ANOVA tool among its basic statistical analysis methods alongside PCA, PLS-DA, t-test, and Kruskal-Wallis, and the suite can be run standalone or wrapped in Galaxy; the paper reports p-values are returned with volcano plots rendering log base 10 p-value against the difference between group means.

## Related tools

- **SECIMTools** (Source suite whose ANOVA tool performs the per-feature fully fixed single-factor ANOVA with all pairwise contrasts and generates volcano plots and model diagnostics; available standalone or wrapped for Galaxy.) — https://github.com/SECIMTools
- **Galaxy** (Web platform into which SECIMTools can be imported and the ANOVA tool executed as part of a workflow.)
- **Python** (Implementation language of the SECIMTools ANOVA tool (set of Python applications).)

## Examples

```
python secimtools/anova.py --input feature_table.csv --design design_file.csv --out_dir ./anova_results
```

## Evaluation signals

- Output table contains one row per feature from the input wide-format dataset, keyed by the unique feature identifier.
- Each pairwise contrast produces exactly one raw p-value column and one binary significance flag column per feature.
- Group means computed per feature match the number of treatment groups defined in the design file.
- ANOVA F-statistics are present for every feature and are consistent with the reported p-values (small p-value implies large F-statistic).
- Volcano plot axes use log base 10 p-value against the difference between group means, matching the tool's specification.

## Limitations

- The tool assumes normal and identically distributed errors for the fixed ANOVA model; violation of these assumptions undermines the pairwise contrast p-values.
- The output is per-feature raw p-values without multiple-testing correction described in the surfaced material, so correction for genome/metabolome-wide testing is not part of this skill.
- No version number for SECIMTools is specified in the article discussion, so version-dependent behavior is unconfirmed.

## Evidence

- [abstract] SECIMTools includes basic statistical methods (PLS-DA, ANOVA, t-test, Kruskal-Wallis): "basic statistical analysis methods (partial least squares - discriminant analysis, analysis of variance, t-test, Kruskal-Wallis non-parametric test)"
- [abstract] SECIMTools is a set of Python applications available standalone and wrapped for Galaxy: "SECIMTools (SouthEast Center for Integrated Metabolomics) is a set of Python applications that are available both as standalone tools and wrapped for use in Galaxy."
- [abstract] SECIMTools includes visualization techniques including volcano plots for pairwise contrast p-values: "visualization techniques (hierarchical cluster heatmap, principal component analysis, modular modularity clustering)"
- [abstract] SECIMTools leverages the Galaxy platform for integrated workflows: "SECIMTools leverages the Galaxy platform and enables integrated workflows for metabolomics data analysis"
- [methods] SECIMTools code is available on GitHub under MIT license: "SECIMTools is open source, the code, is available on GitHub using the MIT license"
- [methods] secimtools is available as a PyPI package: "we packaged all tools as a python package available from https://pypi.python.org/pypi/secimtools"
- [readme] README confirms ANOVA among basic statistical analysis methods: "basic statistical analysis methods (partial least squares - discriminant analysis, analysis of variance)"
- [readme] README: SECIMTools runs standalone or via Galaxy: "SECIMTools project aims to develop a suite of tools for processing of metabolomics data, which can be run in a standalone mode or via Galaxy Genomics Framework."
