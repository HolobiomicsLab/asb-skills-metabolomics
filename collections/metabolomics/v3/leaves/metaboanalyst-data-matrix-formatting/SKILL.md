---
name: metaboanalyst-data-matrix-formatting
description: Use when a processed 1D NMR bucket table (ppm-interval buckets after
  signal-to-noise filtering) must be converted into a samples-by-buckets CSV that
  online statistics tools like MetaboAnalyst 3.0 or BioStatFlow can ingest directly.
when_to_use_negative:
- Input is still raw FID data (Bruker or Varian) that has not yet gone through NMRProcFlow's
  calibration, baseline correction, realignment, and binning steps
- The data matrix is already in a MetaboAnalyst 3.0 / BioStatFlow-compatible export
  format, in which case reformatting adds no value
- The dataset is not bucketed 1D NMR spectral data (e.g., an LC-MS peak table), since
  the ppm-interval bucket labeling scheme would not apply
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: NMRProcFlow
  role: Source of the S/N-filtered bucket table and documentation of the MetaboAnalyst
    3.0 / BioStatFlow-compatible export format
  repo: https://github.com/INRA/NMRProcFlow
- name: R
  role: Scripted reshaping of the bucket table into the samples-by-buckets matrix
    and writing of the CSV export
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/s11306-017-1178-y
    title: 'NMRProcFlow: a graphical and interactive tool dedicated to 1D spectra
      processing for NMR-based metabolomics'
  source_content_hash: sha256:59dae00fdc2ec121e8256489a2228faf463cd92de2d37726aa900e531c7cfc9a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/INRA/NMRProcFlow
  tools:
  - NMRProcFlow
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# metaboanalyst-data-matrix-formatting

## Summary

Reshapes the filtered bucket table produced by NMRProcFlow's 1D NMR spectra processing pipeline into a samples-by-buckets data matrix with experimental-factor class-label columns, written in the export format NMRProcFlow documents as compatible with MetaboAnalyst 3.0 and BioStatFlow, so processed spectral data reaches online statistical tools without file conversion.

## When to use

Use when a processed 1D NMR bucket table (ppm-interval buckets after signal-to-noise filtering) must be converted into a samples-by-buckets CSV that online statistics tools like MetaboAnalyst 3.0 or BioStatFlow can ingest directly.

## When NOT to use

- Input is still raw FID data (Bruker or Varian) that has not yet gone through NMRProcFlow's calibration, baseline correction, realignment, and binning steps
- The data matrix is already in a MetaboAnalyst 3.0 / BioStatFlow-compatible export format, in which case reformatting adds no value
- The dataset is not bucketed 1D NMR spectral data (e.g., an LC-MS peak table), since the ppm-interval bucket labeling scheme would not apply

## Inputs

- NMRProcFlow bucket table produced by the spectra processing pipeline, with buckets filtered on signal-to-noise ratio at the output step
- Experimental-factor level annotations associated with each spectrum (sample identifiers)

## Outputs

- metaboanalyst_data_matrix.csv: a samples-by-buckets CSV with ppm-interval bucket labels as column headers, spectral sample identifiers as rows, and class-label columns for experimental-factor levels

## How to apply

Start from the bucket table emitted by NMRProcFlow after its processing chain (ppm-scale calibration, baseline correction, realignment of ppm areas, binning, and filtering of buckets based on signal-to-noise ratio at the output step). Reshape the table so each row is a spectral sample identifier and each column header is a ppm interval bucket label, keeping only the S/N-filtered buckets so the matrix reflects the retained variables. Append the experimental-factor level information associated with each spectrum as class-label columns, since NMRProcFlow links spectra to experimental-factor levels to bridge experimental design and statistical analyses. Write the result to metaboanalyst_data_matrix.csv in the export format NMRProcFlow documents as compatible with MetaboAnalyst 3.0 and BioStatFlow. Validate that the file parses as CSV, row count equals the number of spectra, and column count equals retained buckets plus the class-label columns.

## Related tools

- **NMRProcFlow** (Source of the S/N-filtered bucket table and documentation of the MetaboAnalyst 3.0 / BioStatFlow-compatible export format) — https://github.com/INRA/NMRProcFlow
- **R** (Scripted reshaping of the bucket table into the samples-by-buckets matrix and writing of the CSV export)

## Evaluation signals

- metaboanalyst_data_matrix.csv exists and parses as valid CSV
- Row count of the matrix equals the number of spectra in the processed set
- Column count equals the number of retained (S/N-filtered) buckets plus the class-label columns
- Column headers are ppm interval bucket labels matching the bucket table, and sample rows carry the spectral sample identifiers
- Class-label values correspond to the experimental-factor levels recorded for each spectrum

## Limitations

- Only buckets retained by the signal-to-noise filtering step appear as columns, so the matrix content depends on the S/N filter being applied during computing or at output
- The compatibility claim in this context covers MetaboAnalyst 3.0 and BioStatFlow; other downstream tools may require different formats
- Producing a usable bucket table depends on upstream interactive processing; the article notes that automated batch-mode processing workflows for 1D NMR spectra have not yet reached full maturity and spectra alignment requires user expertise

## Evidence

- [abstract] Biologists and NMR spectroscopists can easily interact and develop synergies by visualizing the NMR spectra along with their corresponding experimental-factor levels, thus setting a bridge between: "Biologists and NMR spectroscopists can easily interact and develop synergies by visualizing the NMR spectra along with their corresponding experimental-factor levels, thus setting a bridge between"
- [results] in NMRProcFlow it is possible to filter buckets based on this ratio either during the computing process, or at the output step.: "in NMRProcFlow it is possible to filter buckets based on this ratio either during the computing process, or at the output step."
- [readme] the realignment of a set of ppm areas, and the binning (Intelligent, variable size or fixed width): "the realignment of a set of ppm areas, and the binning (Intelligent, variable size or fixed width)"
