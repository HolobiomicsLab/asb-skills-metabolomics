---
name: sample-metadata-derivation-from-column-headers
description: Use when a metabolomics feature table (csv or Excel, features in rows
  / samples in columns) originates from XCMS, MS-DIAL, MZmine, Compound Discoverer,
  or Progenesis QI and sample metadata must be attached before upload to MetaboReport.
when_to_use_negative:
- The input is already a normalized/parsed feature table with consistent sample columns
  and attached metadata — no header derivation is needed
- The workflow target is downstream MetaboReport analysis (statistical analysis, visualization,
  HTML report generation) rather than input parsing/upload
- The data are raw instrument files (e.g. mzML) or peak-picking outputs not among
  the supported tool exports (XCMS, MS-DIAL, MZmine, Compound Discoverer, Progenesis
  QI)
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetaboReport
  role: Target web app whose data uploading module consumes the parsed feature table
    and metadata
  repo: https://github.com/YonghuiDong/MetaboReport
- name: R
  role: Runtime for loading, header-normalizing, and writing the parsed feature table
    (csv output functions)
- name: readxl
  role: R package used to read Excel-format feature tables before header normalization
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae373
    title: 'MetaboReport: from metabolomics data analysis to comprehensive reporting'
  source_content_hash: sha256:1225e856effa75bdca2421ec7dd7f9c4bd9e32ebce4e6473eea12ba7af64fb8e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YonghuiDong/MetaboReport
  tools:
  - MetaboReport
  - R
  - readxl
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# Sample metadata derivation from column headers

## Summary

Derives sample metadata for metabolomics feature tables (features in rows, samples in columns) either from the sample column names or from a separate metadata table, normalizing tool-specific header conventions from XCMS, MS-DIAL, MZmine, Compound Discoverer, and Progenesis QI exports so MetaboReport can ingest them.

## When to use

Use when a metabolomics feature table (csv or Excel, features in rows / samples in columns) originates from XCMS, MS-DIAL, MZmine, Compound Discoverer, or Progenesis QI and sample metadata must be attached before upload to MetaboReport.

## When NOT to use

- The input is already a normalized/parsed feature table with consistent sample columns and attached metadata — no header derivation is needed
- The workflow target is downstream MetaboReport analysis (statistical analysis, visualization, HTML report generation) rather than input parsing/upload
- The data are raw instrument files (e.g. mzML) or peak-picking outputs not among the supported tool exports (XCMS, MS-DIAL, MZmine, Compound Discoverer, Progenesis QI)

## Inputs

- Metabolomics feature table in csv or Excel format (features in rows, samples in columns)
- Tool-specific exports from XCMS, MS-DIAL, MZmine, Compound Discoverer, or Progenesis QI
- Optional separate sample metadata table (when metadata is not embedded in sample column names)
- cancerCell demo MetaboReport feature table (reference/validation input)

## Outputs

- parsed_feature_table.csv with normalized sample columns and attached metadata fields
- In-memory parsed feature table in R with metadata consistent with the source table

## How to apply

Load the feature table from csv or Excel (use R with the readxl package for Excel input; the cancerCell demo MetaboReport dataset is a suitable reference for the expected shape). Inspect the sample column headers to determine whether metadata (e.g. sample group/condition) is embedded in the names, and otherwise pair the feature table with a separate metadata table. Normalize the tool-specific header conventions of the supported exports (XCMS, MS-DIAL, MZmine, Compound Discoverer, Progenesis QI) into a consistent sample-column scheme with attached metadata fields. Write the parsed table to parsed_feature_table.csv using R's csv output functions so it can be passed to MetaboReport's data uploading module. Do not fabricate samples or features: every sample column and metadata field in the output must trace back to the input.

## Related tools

- **MetaboReport** (Target web app whose data uploading module consumes the parsed feature table and metadata) — https://github.com/YonghuiDong/MetaboReport
- **R** (Runtime for loading, header-normalizing, and writing the parsed feature table (csv output functions))
- **readxl** (R package used to read Excel-format feature tables before header normalization)

## Examples

```
library(readxl); ft <- read_excel("cancerCell.xlsx"); write.csv(ft, "parsed_feature_table.csv", row.names = FALSE)
```

## Evaluation signals

- parsed_feature_table.csv exists and parses cleanly in R
- Sample columns in the output csv are consistent with the sample columns of the input demo (e.g. cancerCell) table — no fabricated or dropped samples
- Attached metadata fields match the information derivable from the sample column names or the separate metadata table, with no invented entries
- Feature rows are preserved one-to-one between input and parsed output
- Header conventions from the source tool export (XCMS, MS-DIAL, MZmine, Compound Discoverer, Progenesis QI) are fully normalized with no residual tool-specific prefixes/suffixes

## Limitations

- Metadata extraction depends on the sample column naming being consistent and parseable; malformed or ambiguous headers require a separate metadata table
- The skill covers the supported exporter formats only (XCMS, MS-DIAL, MZmine, Compound Discoverer, Progenesis QI); other export layouts are not covered
- MetaboReport's public package API is intentionally small (run_app()); analysis/IO helpers are internal and may change, so parsing should not rely on internal function contracts

## Evidence

- [other] Data uploading module accepts feature tables in csv/Excel with metadata embedded in sample column names or via separate metadata table: [evidence span withheld — anchor status: not_found]
- [other] Supported tool export header conventions: [evidence span withheld — anchor status: not_found]
- [other] Feature table layout of demo dataset: [evidence span withheld — anchor status: not_found]
- [readme] Public API stability note from README: "The supported public package API is intentionally small: `run_app()`. Analysis helpers are internal so their implementation can evolve without creating an accidental compatibility contract."
