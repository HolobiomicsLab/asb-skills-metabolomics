---
name: multi-tool-export-format-normalization
description: Use when a metabolomics feature table exported from XCMS, MS-DIAL, MZmine,
  Compound Discoverer, or Progenesis QI must be loaded into MetaboReport and needs
  tool-specific header conventions normalized into the features-in-rows/samples-in-columns
  csv layout.
when_to_use_negative:
- The input is already a normalized MetaboReport-format csv/Excel feature table with
  correctly embedded metadata
- The goal is downstream MetaboReport statistical analysis or HTML report generation
  rather than data ingestion
- The data are raw instrument files (e.g., mzML) rather than a processed feature table
  from one of the supported tools
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetaboReport
  role: Target application whose data uploading module defines the expected csv/Excel
    feature-table format and metadata conventions
  repo: https://github.com/YonghuiDong/MetaboReport
- name: R
  role: Implementation language for reading csv/Excel inputs, normalizing tool-specific
    headers, and writing parsed_feature_table.csv
- name: readxl
  role: R package used to read Excel-format feature tables exported by the supported
    tools
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae373
    title: 'MetaboReport: from metabolomics data analysis to comprehensive reporting'
  source_content_hash: sha256:90cd8a659a3a53e373febdfa70a76e95aaa9da9c0339d0993f6e6c35ba932209
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YonghuiDong/MetaboReport
  tools:
  - MetaboReport
  - R
  - readxl
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# multi-tool-export-format-normalization

## Summary

Normalizes feature tables exported by metabolomics processing tools (XCMS, MS-DIAL, MZmine, Compound Discoverer, Progenesis QI) into the csv/Excel feature-table format expected by MetaboReport's data uploading module, deriving sample metadata from column names or a separate metadata table.

## When to use

Use when a metabolomics feature table exported from XCMS, MS-DIAL, MZmine, Compound Discoverer, or Progenesis QI must be loaded into MetaboReport and needs tool-specific header conventions normalized into the features-in-rows/samples-in-columns csv layout.

## When NOT to use

- The input is already a normalized MetaboReport-format csv/Excel feature table with correctly embedded metadata
- The goal is downstream MetaboReport statistical analysis or HTML report generation rather than data ingestion
- The data are raw instrument files (e.g., mzML) rather than a processed feature table from one of the supported tools

## Inputs

- Feature table exported from XCMS, MS-DIAL, MZmine, Compound Discoverer, or Progenesis QI (csv or Excel, features in rows / samples in columns)
- Optional separate sample metadata table
- cancerCell demo metabolomics feature table (MetaboReport dataset) for validation

## Outputs

- parsed_feature_table.csv — normalized feature table with consistent sample columns and attached metadata fields

## How to apply

Load the feature table (features in rows, samples in columns) with R, using the readxl package when the source is Excel and csv readers for csv exports; the cancerCell MetaboReport demo dataset is a suitable reference input. Normalize the tool-specific header conventions of the supported exports (XCMS, MS-DIAL, MZmine, Compound Discoverer, Progenesis QI) so sample columns match MetaboReport's expected layout. Derive sample metadata either from the feature table's sample column names or attach it from a separate metadata table, since MetaboReport accepts metadata in either form. Write the normalized parsed feature table to parsed_feature_table.csv using R's csv output functions.

## Related tools

- **MetaboReport** (Target application whose data uploading module defines the expected csv/Excel feature-table format and metadata conventions) — https://github.com/YonghuiDong/MetaboReport
- **R** (Implementation language for reading csv/Excel inputs, normalizing tool-specific headers, and writing parsed_feature_table.csv)
- **readxl** (R package used to read Excel-format feature tables exported by the supported tools)

## Examples

```
ft <- readxl::read_excel("cancerCell.xlsx"); write.csv(ft, "parsed_feature_table.csv", row.names = FALSE)
```

## Evaluation signals

- parsed_feature_table.csv exists and parses cleanly as csv
- Sample columns in the output are consistent with the input demo table's sample columns (e.g., cancerCell)
- Attached metadata fields match those derivable from column names or the separate metadata table
- No fabricated sample or feature entries: features and samples in the output trace back to the input export

## Limitations

- Normalization is defined only for the five supported export conventions (XCMS, MS-DIAL, MZmine, Compound Discoverer, Progenesis QI); other tools' headers require manual adaptation
- MetaboReport's public package API is intentionally small (run_app()), so parsing/normalization helpers are internal and may evolve without a compatibility contract
- Metadata handling supports only two sources: sample column names of the feature table or a separate metadata table

## Evidence

- [abstract] MetaboReport is an R Shiny app (Beeley and Sukhdeve 2018) written in a modular way following the golem schema for production grade app development: "MetaboReport is an R Shiny app (Beeley and Sukhdeve 2018) written in a modular way following the golem schema for production grade app development"
- [readme] R/fct_*.R contains pure analysis, input/output, plotting, and report functions.: "`R/fct_*.R` contains pure analysis, input/output, plotting, and report functions."
- [readme] The supported public package API is intentionally small: run_app().: "The supported public package API is intentionally small: `run_app()`."
- [methods] you can use library(MetaboReport); run_app() or MetaboReport::run_app() to start the app.: "you can use `library(MetaboReport); run_app()` or `MetaboReport::run_app()` to start the app."
