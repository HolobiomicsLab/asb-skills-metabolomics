---
name: grouped-boxplot-visualization-of-diversity-metrics
description: Use when FT-ICR MS peak tables with assigned molecular formulas have
  per-sample abundance-based diversity (Shannon, Gini-Simpson, Chao1) and functional
  diversity (Rao's quadratic entropy) values that must be compared across user-defined
  sample groups.
when_to_use_negative:
- The input is raw, unprocessed FT-ICR MS spectra — molecular formula assignment must
  be performed before using MetaboDirect
- The goal is multivariate statistics, Van Krevelen diagrams, elemental/molecular
  class composition plots, or mass-difference transformation networks rather than
  chemodiversity index comparison across groups
- Peak abundances have not been normalized (the test_normalization/SPANS companion
  script should first be used to select the best normalization method)
edam_operation: http://edamontology.org/operation_2940
edam_topics:
- http://edamontology.org/topic_3272
- http://edamontology.org/topic_0091
tools:
- name: MetaboDirect
  role: Pipeline whose chemodiversity step computes the diversity indices and generates
    grouped box plots automatically as part of its 6 major steps
  repo: https://github.com/Coayala/MetaboDirect
- name: Python
  role: Implementation language in which MetaboDirect computes diversity metrics and
    renders box plots
- name: pandas
  role: Used to assemble per-sample diversity values into summary tables and write
    them out as .csv files
- name: Matplotlib
  role: Rendering engine for the grouped box plot figures of each diversity index
- name: seaborn
  role: Statistical plotting library used for the grouped box plots of diversity indices
- name: NumPy
  role: Numerical computation support for the diversity calculations feeding the box
    plots
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s40168-023-01476-3
    title: 'MetaboDirect: an analytical pipeline for the processing of FT-ICR MS-based
      metabolomic data'
  source_content_hash: sha256:d04343867dc9c085d813047bea020e191889f47aa6ac7ed8703ccbb5ddb93777
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Coayala/MetaboDirect
  tools:
  - Matplotlib
  - MetaboDirect
  - NumPy
  - Python
  - pandas
  - seaborn
  edam_operation: http://edamontology.org/operation_2940
  edam_topics:
  - http://edamontology.org/topic_3272
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# grouped-boxplot-visualization-of-diversity-metrics

## Summary

Visualizes per-sample chemodiversity indices (Shannon, Gini-Simpson, Chao1, Rao's Q) as box plots grouped by user-defined grouping variables (e.g., phage treatment, incubation condition), as implemented in MetaboDirect's chemodiversity analysis step for FT-ICR MS metabolomic data.

## When to use

Use when FT-ICR MS peak tables with assigned molecular formulas have per-sample abundance-based diversity (Shannon, Gini-Simpson, Chao1) and functional diversity (Rao's quadratic entropy) values that must be compared across user-defined sample groups.

## When NOT to use

- The input is raw, unprocessed FT-ICR MS spectra — molecular formula assignment must be performed before using MetaboDirect
- The goal is multivariate statistics, Van Krevelen diagrams, elemental/molecular class composition plots, or mass-difference transformation networks rather than chemodiversity index comparison across groups
- Peak abundances have not been normalized (the test_normalization/SPANS companion script should first be used to select the best normalization method)

## Inputs

- Processed FT-ICR MS peak abundance table with assigned molecular formulas, sum-normalized using the method selected by the test_normalization/SPANS companion script
- Per-sample diversity index values (Shannon, Gini-Simpson, Chao1, Rao's quadratic entropy)
- User-specified grouping variable/column (e.g., phage treatment, incubation condition)

## Outputs

- Box plot figures (one per diversity index) grouped by the user-defined grouping variable, saved to the chemodiversity output folder
- Per-sample chemodiversity summary tables exported as .csv files containing all diversity indices and grouping columns

## How to apply

After MetaboDirect computes per-sample abundance-based diversity (Shannon, Gini-Simpson, Chao1) on sum-normalized raw peak intensities and functional diversity (Rao's quadratic entropy) from molecular-level trait vectors (elemental ratios, double-bond equivalents/AI_mod aromaticity, reactivity proxies), assemble all diversity values into a per-sample summary table exported as .csv with pandas. Group samples by the user-specified grouping variable — e.g., phage treatment in the bacterium-phage model system or incubation condition in the S. fallax leachate incubation experiment — and render box plots of each diversity index per group with Matplotlib/seaborn. Save all box plot figures and diversity .csv tables into the chemodiversity output folder of the pipeline. This step is part of MetaboDirect's chemodiversity analysis, one of its 6 major pipeline steps, and runs automatically as part of the single-line-of-code pipeline invocation.

## Related tools

- **MetaboDirect** (Pipeline whose chemodiversity step computes the diversity indices and generates grouped box plots automatically as part of its 6 major steps) — https://github.com/Coayala/MetaboDirect
- **Python** (Implementation language in which MetaboDirect computes diversity metrics and renders box plots)
- **pandas** (Used to assemble per-sample diversity values into summary tables and write them out as .csv files)
- **Matplotlib** (Rendering engine for the grouped box plot figures of each diversity index)
- **seaborn** (Statistical plotting library used for the grouped box plots of diversity indices)
- **NumPy** (Numerical computation support for the diversity calculations feeding the box plots)

## Examples

```
metabodirect -h
```

## Evaluation signals

- Every diversity metric (Shannon, Gini-Simpson, Chao1, Rao's Q) is computed for every sample in the normalized peak table
- The exported .csv files contain expected per-sample diversity values plus the grouping columns
- All box plot figures render without error and are saved in the chemodiversity output folder
- Box plot groupings reflect the user-specified grouping variable (e.g., phage treatment or incubation condition)
- The diversity tables and figures match the chemodiversity outputs described in the MetaboDirect documentation

## Limitations

- Visualization depends on upstream normalization: the test_normalization/SPANS companion script should be run first to select the best normalization method before chemodiversity analysis
- MetaboDirect requires Python (3.5+), R (4+), and Cytoscape (3.8+) with specific libraries; missing seaborn/Matplotlib/pandas dependencies will break figure generation
- Direct injection FT-ICR MS cannot separate chemical isomers and suffers signal suppression/enhancement that can confound downstream analysis, affecting the diversity values being plotted

## Evidence

- [abstract] Chemodiversity step computes abundance-based and functional diversity indices: [evidence span withheld — anchor status: not_found]
- [intro] Pipeline structure locating chemodiversity as a major step: "The MetaboDirect pipeline consists of 6 major steps/categories (Fig. 1): (i) data pre-processing, (ii) data diagnostics, (iii) data exploration, (iv) chemodiversity analysis, (v) statistical"
- [results] Normalization must precede analysis: "the first step was to use the “test_normalization” companion script to help determine which normalization method worked the best"
- [abstract] Automatic figure/table generation with minimal coding: "MetaboDirect is superior in that it requires a single line of code to launch a fully automated framework for the generation and visualization of a wide range of plots, with minimal coding experience"
- [results] Dataset context for grouping (bacterium-phage samples): "The bacterium-phage model system data set included metabolites or mass spectrometry peaks present in 36 samples from bacteria infected by two different phages"
- [readme] README dependency list relevant to plotting: "- numpy
- pandas
- seaborn"
