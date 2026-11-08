---
name: metabolomics-tabular-data-import
description: Use when starting an mzQuality workflow from a tab-delimited peak-integrated
  table (e.g. from SCIEX OS dMRM integration) containing aliquot, compound, area,
  type, injection_time, and batch columns, and a SummarizedExperiment must be built
  before analysis.
when_to_use_negative:
- Input is already a SummarizedExperiment — you can pass it directly and skip the
  readData tab-delimited validation path
- Data is raw vendor peak-processing output (e.g. centroided spectra or profile data
  needing OpenMS/MZmine baseline filtering and retention-time alignment) rather than
  peak-integrated tabular areas; mzQuality does not process peaks
- Calibration-line concentrations must be supplied for concentration/R2 calculation
  — consult the Data Input vignette for the additional format requirements first
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: mzQuality
  role: Provides readData (column validation) and buildExperiment (SummarizedExperiment
    construction with auto-derived compound/IS ratio assay) for the import stage
  repo: https://github.com/hankemeierlab/mzQuality
- name: R
  role: Runtime environment; mzQuality is an R package requiring R 4.0 or later
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1021/jasms.5c00073
    title: 'mzQuality: An Open-Source Software Tool for Quality Monitoring and Reporting
      of Targeted Mass Spectrometry Measurements'
  source_content_hash: sha256:c9b134928504d54135b10cc760a74c6afb6c633b93dff095c5c8a2c64430e715
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hankemeierlab/mzQuality
  tools:
  - R
  - mzQuality
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# metabolomics-tabular-data-import

## Summary

Import vendor-neutral, peak-integrated targeted metabolomics data into mzQuality by validating mandatory columns with readData and constructing the SummarizedExperiment object (with auto-derived compound/internal-standard ratio assay) via buildExperiment.

## When to use

Use when starting an mzQuality workflow from a tab-delimited peak-integrated table (e.g. from SCIEX OS dMRM integration) containing aliquot, compound, area, type, injection_time, and batch columns, and a SummarizedExperiment must be built before analysis.

## When NOT to use

- Input is already a SummarizedExperiment — you can pass it directly and skip the readData tab-delimited validation path
- Data is raw vendor peak-processing output (e.g. centroided spectra or profile data needing OpenMS/MZmine baseline filtering and retention-time alignment) rather than peak-integrated tabular areas; mzQuality does not process peaks
- Calibration-line concentrations must be supplied for concentration/R2 calculation — consult the Data Input vignette for the additional format requirements first

## Inputs

- Tab-delimited peak-integrated metabolomics table (e.g. example.tsv) with mandatory columns: aliquot, compound, area, type, injection_time, batch; compound_is/area_is columns support IS-ratio derivation
- Alternatively, an existing SummarizedExperiment object
- Bundled example dataset: 105 compounds and 584 samples (419 study samples across six batches)

## Outputs

- SummarizedExperiment object suitable for mzQuality doAnalysis
- Serialized experiment object (e.g. experiment_object.rds via saveRDS)

## How to apply

Locate the input (bundled example via system.file("extdata/example.tsv", package = "mzQuality") or your own tab-delimited file) and call mzQuality's readData, which reads the table and checks that all mandatory columns are present and correctly formatted — the required columns are aliquot, compound, area, type, injection_time, and batch. Pass the validated data frame to buildExperiment, which converts it into the SummarizedExperiment object used throughout mzQuality for all calculations. Supply contract arguments such as compoundColumn, aliquotColumn, primaryAssay (area), secondaryAssay (area_is), typeColumn, qc, and secondaryIndex when defaults do not match your table; mzQuality automatically derives the compound-to-internal-standard ratio assay from the primary and secondary area assays, and if secondaryAssay is absent its value is set to 1. Serialize the resulting experiment (e.g. with saveRDS) so downstream steps such as doAnalysis can reload it. Rationale: mzQuality processes peak-integrated data independently of vendor software, so this import stage is the single entry point that enforces schema correctness before any QC calculations.

## Related tools

- **mzQuality** (Provides readData (column validation) and buildExperiment (SummarizedExperiment construction with auto-derived compound/IS ratio assay) for the import stage) — https://github.com/hankemeierlab/mzQuality
- **R** (Runtime environment; mzQuality is an R package requiring R 4.0 or later)

## Examples

```
path <- system.file("extdata", "example.tsv", package = "mzQuality"); exp <- buildExperiment(readData(path))
```

## Evaluation signals

- readData completes its mandatory-column check with no missing or incorrect columns (aliquot, compound, area, type, injection_time, batch all present)
- buildExperiment returns a SummarizedExperiment object (verify with class())
- The compound/IS ratio assay is auto-derived; when secondaryAssay (area_is) is absent, its value is set to 1 rather than erroring
- The saved RDS reloads as an experiment object usable by downstream functions such as doAnalysis
- Reloaded object's assay/colData/rowData slots are populated with expected assay names (e.g. area, area_is, ratio)

## Limitations

- readData strictly enforces mandatory columns — files missing or misnaming aliquot, compound, area, type, injection_time, or batch will fail validation
- If the secondaryAssay (internal standard area) is missing from the input, mzQuality substitutes a value of 1, which silently disables IS-based correction semantics
- Any NA value is treated as an absent data point and will not be used for the calculations; the bundled example dataset is not identified by name or accession in the paper

## Evidence

- [readme] Once your files are ready, you can use the `readData` function to read in your data. It will check if all mandatory columns are present and if the data is in the correct format.: [evidence span withheld — anchor status: external_doc]
- [readme] Finally, the function `buildExperiment` will convert the data into a SummarizedExperiment object. This is the object that mzQuality uses to perform all calculations and analyses.: [evidence span withheld — anchor status: external_doc]
- [methods] mzQuality builds a SummarizedExperiment via buildExperiment from a tab-delimited file read by readData, specifying compoundColumn, aliquotColumn, primaryAssay (area), secondaryAssay (area_is), typeColumn, qc, and secondaryIndex; it auto-derives the compound/IS ratio assay, and if secondaryAssay is absent its value is set to 1.: [evidence span withheld — anchor status: not_found]
- [abstract] It processes peak-integrated data independently of vendor software and provides essential quality control features, including batch correction, outlier detection, and background signal assessment.: "It processes peak-integrated data independently of vendor software and provides essential quality control features"
- [readme] an example dataset containing 105 compounds and 584 samples has been added. These can be loaded by running the following code. The result is a SummarizedExperiment, which is the core object that mzQuality uses.: [evidence span withheld — anchor status: external_doc]
