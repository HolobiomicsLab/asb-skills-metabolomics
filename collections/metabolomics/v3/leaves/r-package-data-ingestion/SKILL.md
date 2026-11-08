---
name: r-package-data-ingestion
description: 'Use when importing targeted LC-MS peak-integrated tables into mzQuality:
  the input is a tab-delimited file (or SummarizedExperiment) with required columns
  aliquot, compound, area, type, injection_time, batch, compound_is and area_is that
  must be format-checked before QC.'
when_to_use_negative:
- The input is already a validated SummarizedExperiment and no re-check of mandatory
  columns is needed — skip readData and go straight to buildExperiment or analysis.
- The data are raw vendor spectra (e.g. SCIEX OS wiff exports) rather than peak-integrated
  tables; mzQuality processes peak-integrated data independently of vendor software
  and expects the tab-delimited table format.
- The goal is untargeted metabolomics peak processing (baseline noise filtering, RT
  alignment via tools like MZmine or OpenMS); mzQuality assumes peaks were already
  integrated upstream.
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mzQuality
  role: R package providing readData (column/format validation) and buildExperiment
    (SummarizedExperiment construction) for data ingestion
  repo: https://github.com/hankemeierlab/mzQuality
- name: R
  role: runtime environment in which mzQuality (R >= 4.0) is executed for ingestion
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/jasms.5c00073
    title: 'mzQuality: An Open-Source Software Tool for Quality Monitoring and Reporting
      of Targeted Mass Spectrometry Measurements'
  source_content_hash: sha256:757ed2a9b4ca43deb6fae8185a76752159fc3f90d9952234bef92157486d3e57
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hankemeierlab/mzQuality
  tools:
  - R
  - mzQuality
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# r-package-data-ingestion

## Summary

Ingest and validate tab-delimited metabolomics peak data in the mzQuality R package using readData, then convert it into the SummarizedExperiment object consumed by all downstream QC analyses.

## When to use

Use when importing targeted LC-MS peak-integrated tables into mzQuality: the input is a tab-delimited file (or SummarizedExperiment) with required columns aliquot, compound, area, type, injection_time, batch, compound_is and area_is that must be format-checked before QC.

## When NOT to use

- The input is already a validated SummarizedExperiment and no re-check of mandatory columns is needed — skip readData and go straight to buildExperiment or analysis.
- The data are raw vendor spectra (e.g. SCIEX OS wiff exports) rather than peak-integrated tables; mzQuality processes peak-integrated data independently of vendor software and expects the tab-delimited table format.
- The goal is untargeted metabolomics peak processing (baseline noise filtering, RT alignment via tools like MZmine or OpenMS); mzQuality assumes peaks were already integrated upstream.

## Inputs

- Tab-delimited text file with mandatory columns (aliquot, compound, area, type, injection_time, batch, compound_is, area_is; optional concentration column for calibration lines)
- Pre-built SummarizedExperiment object (alternative input)

## Outputs

- SummarizedExperiment object (the core mzQuality experiment used by doAnalysis and all plotting/QC functions)
- readData diagnostics confirming presence/format of mandatory columns

## How to apply

Locate the tab-delimited text file (e.g. inst/extdata/example.tsv shipped with mzQuality) and call readData(path), which verifies that all mandatory columns are present and correctly formatted — including aliquot, compound, area, type, injection_time, batch, and the internal-standard columns compound_is and area_is; a supplied concentration column carries known spike concentrations for calibration-line samples and NA for study samples. If the data is already a SummarizedExperiment it can be passed directly. Pass the validated data frame to buildExperiment(combined), which converts it into a SummarizedExperiment that serves as the core object for all subsequent doAnalysis calculations. The same ingestion path supports the package's built-in example of 105 compounds and 584 samples, so correctness can be judged by whether the returned SummarizedExperiment exposes assay/rowData/colData slots populated from the input table without errors.

## Related tools

- **mzQuality** (R package providing readData (column/format validation) and buildExperiment (SummarizedExperiment construction) for data ingestion) — https://github.com/hankemeierlab/mzQuality
- **R** (runtime environment in which mzQuality (R >= 4.0) is executed for ingestion)

## Examples

```
path <- system.file("extdata", "example.tsv", package = "mzQuality"); exp <- buildExperiment(readData(path))
```

## Evaluation signals

- readData returns without a 'missing or incorrect columns' error, confirming all mandatory columns (aliquot, compound, area, type, injection_time, batch) are present.
- buildExperiment yields a SummarizedExperiment whose assay/rowData/colData slots are populated and whose compounds are paired with their internal standards (compound_is / area_is).
- Calibration-line samples carry known values in the concentration column while study samples hold NA, matching the required split.
- The ingested experiment reproduces the package's example scale (105 compounds / 584 samples) when the shipped example.tsv is used.
- Downstream doAnalysis runs on the ingested object without failing on missing input fields.

## Limitations

- readData enforces a fixed schema — datasets missing the mandatory columns (or with incorrectly formatted values) are rejected at ingestion time.
- If calibration-line concentrations are omitted, doAnalysis cannot later compute absolute concentrations; the concentration column must be supplied at ingestion.
- mzQuality requires R version 4.0 or later, and installation of auxiliary dependencies (e.g. GenomeInfoDbData via BiocManager) is not fully automatic.
- The README is truncated mid-table in the provided context, so exact export/reporting details cannot be verified from documentation alone.

## Evidence

- [methods] readData validates mandatory columns before ingestion: "you can use the `readData` function. This function will read in your data and check for any missing or incorrect columns."
- [methods] buildExperiment creates the experiment object used for analysis: "The `buildExperiment` function will then take the data and create an experiment object that can be used for analysis."
- [readme] readData checks mandatory columns are present and correctly formatted: [evidence span withheld — anchor status: external_doc]
- [readme] buildExperiment converts data into a SummarizedExperiment used throughout the package: [evidence span withheld — anchor status: external_doc]
- [readme] Shipped example dataset of 105 compounds and 584 samples: [evidence span withheld — anchor status: external_doc]
- [readme] Built-in example can be loaded via system.file and processed by readData + buildExperiment: "path <- system.file("extdata", "example.tsv", package = "mzQuality")
exp <- buildExperiment(readData(path))"
- [readme] Both SummarizedExperiment and tab-delimited text file are accepted inputs: [evidence span withheld — anchor status: external_doc]
- [full_text] Example data is loadable via readData on extdata/example.tsv: [evidence span withheld — anchor status: not_found]
