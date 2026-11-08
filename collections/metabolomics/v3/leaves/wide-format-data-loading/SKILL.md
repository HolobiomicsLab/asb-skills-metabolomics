---
name: wide-format-data-loading
description: Use when the input is a wide-formatted metabolomics feature table (samples
  as columns, features as rows) plus a design file, to be fed into any SECIMTools
  tool or Galaxy-wrapped step.
when_to_use_negative:
- The input is a raw instrument file or peak-picked output — SECIMTools expects a
  feature table produced after peak picking / feature identification, not vendor formats.
- The table is long/stacked format (one observation per row) rather than wide format
  with samples as columns.
- The analysis is not a metabolomics feature-table workflow (SECIMTools is scoped
  to metabolomics data processing, which the authors note spans sample preparation,
  data acquisition, preprocessing, and analysis).
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SECIMTools
  role: Defines and consumes the wide formatted feature table and design file as its
    standard input formats for all tools
  repo: https://github.com/SECIMTools
- name: Python
  role: Runtime for standalone SECIMTools tools and for parsing the tab-separated
    table and design file
  repo: https://pypi.python.org/pypi/secimtools
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-018-2134-1
    title: 'SECIMTools: a suite of metabolomics data analysis tools'
  source_content_hash: sha256:7353f94bccc165b76d442dfcaf123cd8cba0398cc90a9e6b4f46a41cf50c3018
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - SECIMTools
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# wide-format-data-loading

## Summary

Loads SECIMTools-style metabolomics feature tables in wide format — samples as columns, metabolomics features as rows — together with the companion design file, so downstream SECIMTools tools (hierarchical clustering heatmap, PCA, flags, etc.) receive a consistent, tool-parseable input.

## When to use

Use when the input is a wide-formatted metabolomics feature table (samples as columns, features as rows) plus a design file, to be fed into any SECIMTools tool or Galaxy-wrapped step.

## When NOT to use

- The input is a raw instrument file or peak-picked output — SECIMTools expects a feature table produced after peak picking / feature identification, not vendor formats.
- The table is long/stacked format (one observation per row) rather than wide format with samples as columns.
- The analysis is not a metabolomics feature-table workflow (SECIMTools is scoped to metabolomics data processing, which the authors note spans sample preparation, data acquisition, preprocessing, and analysis).

## Inputs

- wide formatted metabolomics feature table (tab-separated, samples as columns, features as rows, identifier column with feature labels)
- companion design file mapping sample identifiers to grouping variables

## Outputs

- parsed feature table with feature identifiers as row labels and sample names as column labels
- parsed design file with sample identifiers matched to the grouping variable

## How to apply

Read the wide formatted table with row identifiers for metabolomics features and column headers for samples, using tab separation to match SECIMTools' standard data formats. Load the companion design file in parallel and attach its grouping variable to sample identifiers, since tools such as the hierarchical clustering heatmap use design-file groups for sample coloring. Verify that the identifier column is parsed as labels (strings) rather than numeric data, and that sample column names match the design file entries exactly before passing the table onward. Because SECIMTools relies on standard data formats and utilities to allow arbitrary linkages between tools, keep the wide table unchanged — do not transpose, impute, or re-scale at this stage; those are separate SECIMTools tools (e.g. Imputation, Data Normalization and Re-Scaling).

## Related tools

- **SECIMTools** (Defines and consumes the wide formatted feature table and design file as its standard input formats for all tools) — https://github.com/SECIMTools
- **Python** (Runtime for standalone SECIMTools tools and for parsing the tab-separated table and design file) — https://pypi.python.org/pypi/secimtools

## Examples

```
import pandas as pd; df = pd.read_csv("feature_table.tsv", sep="\t"); design = pd.read_csv("design_file.tsv", sep="\t")
```

## Evaluation signals

- Parsed table has an identifier column plus numeric feature-value columns, with one column per sample and one row per metabolomics feature.
- Every sample column header in the feature table matches a sample identifier in the design file.
- Identifier column values remain strings (feature labels), not coerced to numeric.
- Downstream SECIMTools tool runs without error on the loaded table (e.g. the hierarchical clustering tool completes and writes its distance-matrix CSV and heatmap PNG).

## Limitations

- The article does not specify a version number for SECIMTools or Galaxy, nor does it resolve the GitHub reference [27] to a repository URL in the text.
- Untargeted metabolomics has no single platform or technology capturing the entire metabolome, so wide-format column counts and feature identifiers vary by platform; loaders must tolerate heterogeneous feature tables.

## Evidence

- [abstract] Standard data formats and a set of utilities allow arbitrary linkages between tools to encourage novel workflow designs.: "Standard data formats and a set of utilities allow arbitrary linkages between tools to encourage novel workflow designs."
- [readme] SECIMTools python tools standalone/Galaxy: "The SECIMTools a set of python tools that are available both as standalone and wrapped for use in Galaxy."
- [intro] SECIMTools standalone or Galaxy processing goal: "SECIMTools project aims to develop a suite of tools for processing of metabolomics data, which can be run in a standalone mode or via Galaxy"
