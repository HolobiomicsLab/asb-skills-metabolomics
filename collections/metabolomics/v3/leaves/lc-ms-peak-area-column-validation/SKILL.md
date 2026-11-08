---
name: lc-ms-peak-area-column-validation
description: Use when importing tab-delimited peak-integrated metabolomics data into
  mzQuality and you need to verify the presence of mandatory columns (aliquot, compound,
  area, type, injection_time, batch) before downstream quality analysis.
when_to_use_negative:
- Your input is already a SummarizedExperiment — mzQuality accepts this directly and
  readData's tab-delimited column validation is unnecessary.
- Your data is raw vendor output (e.g. mzML or SCIEX integrations) rather than a tab-delimited
  peak-integrated table — mzQuality processes peak-integrated data and cannot perform
  the integration itself.
- Your table lacks any of the mandatory columns (aliquot, compound, area, type, injection_time,
  batch) and you cannot supply them — readData will fail the format check and the
  import cannot proceed.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mzQuality
  role: Provides the readData function that validates mandatory columns in tab-delimited
    peak-integrated data and the buildExperiment function that constructs the SummarizedExperiment
    with the compound/IS ratio assay
  repo: https://github.com/hankemeierlab/mzQuality
- name: R
  role: Runtime environment required (version 4.0 or later) in which mzQuality is
    distributed as an R package and where readData/buildExperiment are called
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/jasms.5c00073
    title: 'mzQuality: An Open-Source Software Tool for Quality Monitoring and Reporting
      of Targeted Mass Spectrometry Measurements'
  source_content_hash: sha256:eec31a89bce1946fb1c0da05fbc363ec6ab69a08f12fe57182eaee8c95c06016
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hankemeierlab/mzQuality
  tools:
  - R
  - mzQuality
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# lc-ms-peak-area-column-validation

## Summary

Validate and read tab-delimited peak-integrated LC-MS metabolomics data with mzQuality's readData function, verifying mandatory columns (aliquot, compound, area, type, injection_time, batch) before constructing a SummarizedExperiment via buildExperiment.

## When to use

Use when importing tab-delimited peak-integrated metabolomics data into mzQuality and you need to verify the presence of mandatory columns (aliquot, compound, area, type, injection_time, batch) before downstream quality analysis.

## When NOT to use

- Your input is already a SummarizedExperiment — mzQuality accepts this directly and readData's tab-delimited column validation is unnecessary.
- Your data is raw vendor output (e.g. mzML or SCIEX integrations) rather than a tab-delimited peak-integrated table — mzQuality processes peak-integrated data and cannot perform the integration itself.
- Your table lacks any of the mandatory columns (aliquot, compound, area, type, injection_time, batch) and you cannot supply them — readData will fail the format check and the import cannot proceed.

## Inputs

- Tab-delimited text file of peak-integrated LC-MS measurements with mandatory columns aliquot, compound, area, type, injection_time, and batch (and optionally compound_is/area_is for internal standard support)
- SummarizedExperiment object (alternative accepted input format)

## Outputs

- Validated data frame returned by readData after the mandatory-column check
- SummarizedExperiment object built by buildExperiment, containing the primary area assay and the auto-derived compound/IS ratio assay
- Serialized experiment object (e.g. experiment_object.rds) suitable for downstream analysis functions such as doAnalysis

## How to apply

Load a tab-delimited file (e.g. the bundled example.tsv containing 105 compounds and 584 samples) with mzQuality's readData function, which verifies that all mandatory columns — aliquot, compound, area, type, injection_time, and batch — are present and correctly formatted; missing or incorrect columns abort the import. Pass the validated data frame to buildExperiment, which converts it into a SummarizedExperiment that is the core object mzQuality uses for all calculations and analyses. When constructing the experiment, specify the compoundColumn, aliquotColumn, primaryAssay (area), secondaryAssay (area_is), typeColumn, qc, and secondaryIndex arguments so mzQuality can auto-derive the compound-to-internal-standard (compound/IS) ratio assay; if the secondaryAssay is absent its value is set to 1. Serialize the resulting experiment object with saveRDS (e.g. experiment_object.rds) so downstream functions such as doAnalysis can consume it. The rationale is that readData catches schema defects early — any NA value is treated as an absent data point and excluded from later calculations, so structural validation must precede the quality pipeline.

## Related tools

- **mzQuality** (Provides the readData function that validates mandatory columns in tab-delimited peak-integrated data and the buildExperiment function that constructs the SummarizedExperiment with the compound/IS ratio assay) — https://github.com/hankemeierlab/mzQuality
- **R** (Runtime environment required (version 4.0 or later) in which mzQuality is distributed as an R package and where readData/buildExperiment are called)

## Examples

```
path <- system.file("extdata", "example.tsv", package = "mzQuality"); exp <- buildExperiment(readData(path))
```

## Evaluation signals

- readData completes without raising an error, confirming all mandatory columns (aliquot, compound, area, type, injection_time, batch) are present and correctly formatted.
- buildExperiment returns a SummarizedExperiment whose assays include the primary area assay and the auto-derived compound/IS ratio assay (compound/area_is).
- The saved RDS reloads as a SummarizedExperiment suitable for downstream calls such as doAnalysis without further reformatting.
- No NA values remain in the mandatory columns; any NA in the data is confirmed to be treated as an absent data point excluded from calculations.
- When secondaryAssay (area_is) is absent, verify that its value defaults to 1 so the ratio assay remains computable.

## Limitations

- If the secondaryAssay (area_is) is absent, its value is set to 1, which means the ratio assay degenerates to the raw area and internal-standard normalization cannot be applied.
- Any NA value in the data is treated as an absent data point and excluded from calculations, so structural validation cannot distinguish missing measurements from truly absent compounds.
- When a compound is not detected in LQC or SQC samples, mzQuality assumes it is not present and will not report it, which affects how downstream analysis should interpret empty entries.

## Evidence

- [methods] This function will read in your data and check for any missing or incorrect columns.: "you can use the `readData` function. This function will read in your data and check for any missing or incorrect columns."
- [methods] buildExperiment converts the data into a SummarizedExperiment: "The `buildExperiment` function will then take the data and create an experiment object that can be used for analysis."
- [abstract] mzQuality processes peak-integrated data independently of vendor software: "It processes peak-integrated data independently of vendor software and provides essential quality control features, including batch correction, outlier detection, and background signal assessment"
- [readme] readData checks mandatory columns and format: [evidence span withheld — anchor status: external_doc]
- [readme] SummarizedExperiment is the core object: [evidence span withheld — anchor status: external_doc]
