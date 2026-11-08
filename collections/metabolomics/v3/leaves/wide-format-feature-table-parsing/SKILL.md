---
name: wide-format-feature-table-parsing
description: Use when the input is a SECIMTools wide-format dataset (sample IDs in
  columns, metabolomics features in rows) plus a design file assigning samples to
  groups, before running PCA, ANOVA, random forest, or Elastic Net tools.
when_to_use_negative:
- Input is already a tool output (e.g. a per-sample scores CSV or per-feature loadings
  CSV) rather than a raw wide-format feature table
- Data are in long/tidy format (one measurement per row) and must first be pivoted
  to the SECIMTools wide layout
- The task is peak picking or feature identification — the article notes peak picking
  is particular to each platform and handled by upstream tools such as Galaxy-M and
  Workflow4Metabolomics
edam_operation: http://edamontology.org/operation_2935
tools:
- name: SECIMTools
  role: Source of the standard wide-format dataset and design-file conventions consumed
    by its analysis, classification, and visualization tools
  repo: https://github.com/SECIMTools
- name: Python
  role: pandas-based loading of the wide-format dataset and design file in standalone
    mode
- name: Galaxy
  role: Web platform into which the same dataset/design file tables are imported when
    tools are run as Galaxy wrappers
provenance:
  source_task_ids:
  - task_004
  - task_007
  - task_008
  - task_006
  source_papers:
  - doi: 10.1186/s12859-018-2134-1
    title: 'SECIMTools: a suite of metabolomics data analysis tools'
  source_content_hash: sha256:eb1b30efeb4d45a3172b7896dd3d35af05231b7ed68a30762b42fb3d4f7781ab
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Galaxy
  - Python
  - SECIMTools
  edam_operation: http://edamontology.org/operation_2935
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# wide-format-feature-table-parsing

## Summary

Parse SECIMTools' standard wide-format metabolomics feature table (samples as columns, metabolomics features as rows keyed by unique feature identifiers) together with its design file that maps each sample to sample characteristics such as treatment group, providing the shared input format that lets all SECIMTools analysis, classification, and visualization tools be arbitrarily linked.

## When to use

Use when the input is a SECIMTools wide-format dataset (sample IDs in columns, metabolomics features in rows) plus a design file assigning samples to groups, before running PCA, ANOVA, random forest, or Elastic Net tools.

## When NOT to use

- Input is already a tool output (e.g. a per-sample scores CSV or per-feature loadings CSV) rather than a raw wide-format feature table
- Data are in long/tidy format (one measurement per row) and must first be pivoted to the SECIMTools wide layout
- The task is peak picking or feature identification — the article notes peak picking is particular to each platform and handled by upstream tools such as Galaxy-M and Workflow4Metabolomics

## Inputs

- Wide-format metabolomics dataset (CSV; samples as columns, features as rows with unique feature identifiers)
- Design file mapping each sample identifier to sample characteristics such as treatment group

## Outputs

- Feature matrix with unique feature identifiers as row index and sample identifiers as columns
- Sample-to-group annotation series joined from the design file
- Optional combined analysis frame (features + design labels) ready for downstream SECIMTools tools

## How to apply

Load the wide-format table with pandas in Python so each row is a metabolomics feature with a unique feature identifier and each column is a sample; load the accompanying design file and relate each sample column to its sample characteristics (e.g. treatment group or binary class label). This two-file format is the standard data format of the suite: SECIMTools deliberately relies on standard data formats and a set of utilities so tools can be linked into novel workflows, and the same tables are consumed whether a tool runs standalone or wrapped in Galaxy. Preserve the feature identifiers as the row index and sample identifiers as column names, since downstream SECIMTools-style outputs (scores CSVs keyed by sample ID, loadings/selection tables keyed by feature ID) are matched against them. Center and (optionally) scale the feature matrix only at the point where a specific analysis (e.g. SVD-based PCA) requires it, not during parsing.

## Related tools

- **SECIMTools** (Source of the standard wide-format dataset and design-file conventions consumed by its analysis, classification, and visualization tools) — https://github.com/SECIMTools
- **Python** (pandas-based loading of the wide-format dataset and design file in standalone mode)
- **Galaxy** (Web platform into which the same dataset/design file tables are imported when tools are run as Galaxy wrappers)

## Examples

```
import pandas as pd; X = pd.read_csv('wide_dataset.csv', index_col=0); design = pd.read_csv('design.csv', index_col=0)
```

## Evaluation signals

- The parsed feature matrix has samples as columns and features as rows, with each row carrying a unique feature identifier
- Every sample column in the feature table resolves to a design-file entry mapping it to a group such as treatment group
- The script runs end-to-end without error and downstream tool outputs (scores CSV with sample identifier and PC1-PC3, loadings CSV with feature identifier) align row/column-wise with the parsed inputs
- The same parsed tables feed multiple SECIMTools-style tools (PCA, ANOVA, random forest, Elastic Net) unchanged, demonstrating the standard-format linkage property

## Limitations

- SECIMTools expects feature matrices starting from peak-picked feature tables; feature identification ('peak picking') is particular to the technological properties of each platform and out of scope for this parsing skill
- The suite provides no single capture technology for the entire metabolome, so column semantics (feature identifiers, sample names) depend on the upstream platform and preprocessing pipeline
- No version number for SECIMTools is resolved in the discussion; verify the installed secimtools package version before reproducing layouts

## Evidence

- [abstract] Standard data formats and a set of utilities allow arbitrary linkages between tools to encourage novel workflow designs.: "Standard data formats and a set of utilities allow arbitrary linkages between tools to encourage novel workflow designs."
- [readme] The SECIMTools a set of python tools that are available both as standalone and wrapped for use in Galaxy.: "The SECIMTools a set of python tools that are available both as standalone and wrapped for use in Galaxy."
- [methods] we packaged all tools as a python package available from https://pypi.python.org/pypi/secimtools: "we packaged all tools as a python package available from https://pypi.python.org/pypi/secimtools"
- [intro] The feature identification, or "peak picking" is particular to the technological properties of each platform: [evidence span withheld — anchor status: not_found]
