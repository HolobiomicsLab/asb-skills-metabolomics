---
name: lipid-class-abundance-profiling
description: Use when profiling lipidomics abundance data and you need sample-level
  or group-level composition summaries at the lipid-class level rather than per lipid
  species, e.g. to compare class percentages across samples in a multi-group dataset.
when_to_use_negative:
- Only two-group data can conduct the 'Network' and 'Machine learning' workflows —
  but for profiling composition of individual lipid species rather than lipid classes,
  use char set to a different characteristic (e.g. species-level profiling).
- Data has not yet been processed into a SummarizedExperiment object via the tool
  functions (vignette('tool_function')) — lipid_profiling expects the processed object.
- The goal is differential expression, enrichment, network, or machine learning analysis
  rather than descriptive composition profiling.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: LipidSigR
  role: Provides lipid_profiling() which computes class-level (char='class') summary
    statistics and composition plots on processed SummarizedExperiment objects
  repo: https://github.com/BioinfOMICS/LipidSigR
- name: R
  role: Runtime environment (>= 4.3.0 required for running LipidSigR)
- name: dplyr
  role: Data manipulation when reading and preparing the abundance_multiGroup table
    before/after calling lipid_profiling
provenance:
  source_task_ids:
  - task_006
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbaf047
    title: 'LipidSigR: a R-based solution for integrated lipidomics data analysis
      and visualization'
  source_content_hash: sha256:89cc696ecbe0c725443964cfe74901a41a9332e00acf0000c2349ffc7759ecae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioinfOMICS/LipidSigR
  tools:
  - LipidSigR
  - R
  - dplyr
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# lipid-class-abundance-profiling

## Summary

Generates lipid-characteristic (class) composition profiling using LipidSigR's lipid_profiling() with char='class', producing a class abundance bar plot and a stacked horizontal composition chart that shows the percentage of each lipid class per sample.

## When to use

Use when profiling lipidomics abundance data and you need sample-level or group-level composition summaries at the lipid-class level rather than per lipid species, e.g. to compare class percentages across samples in a multi-group dataset.

## When NOT to use

- Only two-group data can conduct the 'Network' and 'Machine learning' workflows — but for profiling composition of individual lipid species rather than lipid classes, use char set to a different characteristic (e.g. species-level profiling).
- Data has not yet been processed into a SummarizedExperiment object via the tool functions (vignette('tool_function')) — lipid_profiling expects the processed object.
- The goal is differential expression, enrichment, network, or machine learning analysis rather than descriptive composition profiling.

## Inputs

- Processed SummarizedExperiment object (processed_se) from upstream LipidSigR profiling preprocessing
- Processed abundance_multiGroup lipid abundance data table (multi-group lipidomics data)

## Outputs

- Class-level abundance summary table extracted from the lipid_profiling object
- Lipid class abundance bar plot (PNG, fig.width=6, fig.height=5)
- Stacked horizontal lipid class composition chart showing percentage of lipid classes per sample (PNG, fig.width=6, fig.height=5)

## How to apply

Start from the processed SummarizedExperiment object produced upstream by the Profiling preprocessing step (the abundance_multiGroup table in the 'Case - multi-group data' vignette). In R (>= 4.3.0) with dplyr and utils loaded, call LipidSigR::lipid_profiling(processed_se, char='class') so all summary statistics are computed at the lipid-class level rather than per lipid species. Extract the class-level abundance summary table from the returned lipid_profiling object. Render the class abundance bar plot and the stacked horizontal lipid-composition chart (one stacked bar per group/sample showing percentage of each lipid class), e.g. as PNG figures with knitr::opts_chunk$set(fig.width = 6, fig.height = 5). Variability between samples in class composition should be visible in the stacked bar chart.

## Related tools

- **LipidSigR** (Provides lipid_profiling() which computes class-level (char='class') summary statistics and composition plots on processed SummarizedExperiment objects) — https://github.com/BioinfOMICS/LipidSigR
- **R** (Runtime environment (>= 4.3.0 required for running LipidSigR))
- **dplyr** (Data manipulation when reading and preparing the abundance_multiGroup table before/after calling lipid_profiling)

## Examples

```
LipidSigR::lipid_profiling(processed_se, char='class')
```

## Evaluation signals

- Both class-level Profiling figures (abundance bar plot and stacked composition chart) are generated without error.
- The stacked composition chart shows one stacked bar per sample/group with class axis labels present.
- The rendered plots visually match the corresponding Profiling-section plots of the 'Case - multi-group data' vignette.
- The stacked bars represent percentages of lipid classes summing to 100% per sample, with between-sample variability visible.
- All summary statistics were computed at lipid-class level, confirming char='class' was passed rather than species-level defaults.

## Limitations

- Requires R 4.3.0 or later; on Windows, LipidSigR is supported only on Windows 10 or later.
- Profiling composition outputs depend on upstream preprocessing into a processed SummarizedExperiment object; raw lipid abundance tables cannot be passed directly.
- Lipid characteristics available for profiling are limited to lipid class, chain length, unsaturation, hydroxyl groups, and fatty acid composition.

## Evidence

- [methods] class composition output: [evidence span withheld — anchor status: not_found]
- [methods] lipid_profiling call: [evidence span withheld — anchor status: not_found]
- [readme] R version requirement: "You need to have R 4.3.0 or a later version installed for running LipidSigR."
- [readme] profiling scope: [evidence span withheld — anchor status: external_doc]
- [intro] characteristics analyzed: "Each section offers unique aspects to analyzing lipidome profiling data based on various characteristics, including lipid class, chain length, unsaturation, hydroxyl groups, and fatty acid"
