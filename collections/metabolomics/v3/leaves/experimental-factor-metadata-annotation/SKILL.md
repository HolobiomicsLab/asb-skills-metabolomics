---
name: experimental-factor-metadata-annotation
description: Use when a filtered NMRProcFlow bucket table must be turned into a samples-by-buckets
  data matrix whose rows (spectra) carry class-label columns for experimental-factor
  levels, e.g. before export to MetaboAnalyst 3.0 or BioStatFlow.
when_to_use_negative:
- Input is still raw FID data (Bruker or Agilent/Varian) that has not been baseline-corrected,
  calibrated, realigned, and binned by NMRProcFlow
- The dataset has no experimental-factor levels or experimental design, so there is
  no class-label metadata to annotate spectra with
- The matrix is already a class-labeled feature table accepted directly by the downstream
  statistical tool
edam_operation: http://edamontology.org/operation_0336
tools:
- name: NMRProcFlow
  role: Produces the filtered bucket table and associates spectra with experimental-factor
    levels; its documented export format yields the MetaboAnalyst 3.0 / BioStatFlow-compatible
    matrix
  repo: https://github.com/INRA/NMRProcFlow
- name: R
  role: Scripting language used to reshape the bucket table and append factor-level
    class-label columns before writing the CSV export
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/s11306-017-1178-y
    title: 'NMRProcFlow: a graphical and interactive tool dedicated to 1D spectra
      processing for NMR-based metabolomics'
  source_content_hash: sha256:d520ca7cc3d767b2c1ce1b894e5473279e727e494a8acbf6eebfe8fbc02cac03
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/INRA/NMRProcFlow
  tools:
  - NMRProcFlow
  - R
  edam_operation: http://edamontology.org/operation_0336
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# experimental-factor-metadata-annotation

## Summary

Attach experimental-factor level information to each processed 1D NMR spectrum so the bucket table exported from NMRProcFlow becomes a class-labeled samples-by-buckets matrix interpretable by statistical tools such as MetaboAnalyst 3.0 and BioStatFlow.

## When to use

Use when a filtered NMRProcFlow bucket table must be turned into a samples-by-buckets data matrix whose rows (spectra) carry class-label columns for experimental-factor levels, e.g. before export to MetaboAnalyst 3.0 or BioStatFlow.

## When NOT to use

- Input is still raw FID data (Bruker or Agilent/Varian) that has not been baseline-corrected, calibrated, realigned, and binned by NMRProcFlow
- The dataset has no experimental-factor levels or experimental design, so there is no class-label metadata to annotate spectra with
- The matrix is already a class-labeled feature table accepted directly by the downstream statistical tool

## Inputs

- Filtered bucket table from NMRProcFlow spectra processing (buckets filtered by signal-to-noise ratio at the output step)
- Experimental-factor level information associated with each spectrum (experimental design metadata)

## Outputs

- metaboanalyst_data_matrix.csv: samples-by-buckets matrix with ppm-interval bucket labels as column headers, spectral sample identifiers as rows, and experimental-factor class-label columns appended

## How to apply

Start from the bucket table produced by NMRProcFlow's spectra processing pipeline (ppm-scale calibration, baseline correction, realignment of ppm areas, binning, and signal-to-noise bucket filtering at the output step). Reshape it into a samples-by-buckets matrix, using the ppm interval bucket labels as column headers and the spectral sample identifiers as rows. For each spectrum, append its associated experimental-factor level(s) as additional class-label columns, mirroring how NMRProcFlow's spectra viewer groups spectra by color based on their factor level. Write the annotated matrix to metaboanalyst_data_matrix.csv in the export format NMRProcFlow documents as compatible with MetaboAnalyst 3.0 and BioStatFlow, so downstream statistics can be run without further file conversion.

## Related tools

- **NMRProcFlow** (Produces the filtered bucket table and associates spectra with experimental-factor levels; its documented export format yields the MetaboAnalyst 3.0 / BioStatFlow-compatible matrix) — https://github.com/INRA/NMRProcFlow
- **R** (Scripting language used to reshape the bucket table and append factor-level class-label columns before writing the CSV export)

## Examples

```
docker pull docker.io/nmrprocflow/npflow && sh ./npflow start
```

## Evaluation signals

- metaboanalyst_data_matrix.csv exists and parses as valid CSV
- Row count of the matrix equals the number of spectra in the bucket table
- Column count equals the number of retained buckets plus the number of class-label columns
- Each spectrum's class-label value matches its experimental-factor level in the experimental design
- The matrix loads in MetaboAnalyst 3.0 or BioStatFlow without manual reformatting or file conversion

## Limitations

- The number of retained buckets depends on signal-to-noise ratio filtering, which in NMRProcFlow can be applied either during computing or at the output step, changing the matrix dimensions
- Spectra alignment and other processing challenges in 1D NMR require user expertise; automated batch-mode workflows usable by non-experts have not reached full maturity, so factor-level grouping may reflect interactive, user-driven choices
- The annotated matrix is only as good as the experimental design metadata supplied; NMRProcFlow is a spectra processing tool, not a design-management system

## Evidence

- [abstract] Biologists and NMR spectroscopists can easily interact and develop synergies by visualizing the NMR spectra along with their corresponding experimental-factor levels, thus setting a bridge between: "Biologists and NMR spectroscopists can easily interact and develop synergies by visualizing the NMR spectra along with their corresponding experimental-factor levels, thus setting a bridge between"
- [readme] It allows the user to visually explore the spectra overlaid or stacked, to zoom on intensity scale, to group set of spectra by color based on their factor level.: "It allows the user to visually explore the spectra overlaid or stacked, to zoom on intensity scale, to group set of spectra by color based on their factor level."
- [results] in NMRProcFlow it is possible to filter buckets based on this ratio either during the computing process, or at the output step.: "in NMRProcFlow it is possible to filter buckets based on this ratio either during the computing process, or at the output step."
- [methods] The NMRProcFlow open source software provides an efficient GUI tool for spectra processing from 1D NMR metabolomics data: "The NMRProcFlow open source software provides an efficient GUI tool for spectra processing from 1D NMR metabolomics data"
