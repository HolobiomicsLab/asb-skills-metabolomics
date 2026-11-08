---
name: nmr-bucket-table-export
description: Use when a processed 1D NMR bucket table (ppm-interval buckets per spectrum)
  must be handed off, without intermediate file conversions, to online statistical
  tools such as MetaboAnalyst 3.0 or BioStatFlow.
when_to_use_negative:
- The input is raw FID data from Bruker or Agilent (Varian) instruments — spectra
  processing (calibration, baseline correction, realignment, binning) must run in
  NMRProcFlow first; this skill only exports the already-processed bucket table.
- The bucket table is already in a statistical-tools-ready matrix format with class
  labels — re-exporting adds no value.
- The goal is to perform the downstream statistical analysis itself (e.g. PCA, clustering)
  rather than producing the import-compatible data matrix.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3383
tools:
- name: NMRProcFlow
  role: Produces the SNR-filtered bucket table and defines the export format documented
    as compatible with MetaboAnalyst 3.0 and BioStatFlow
  repo: https://github.com/INRA/NMRProcFlow
- name: R
  role: Can be used to reshape the bucket table into the samples-by-buckets matrix
    and append class-label columns before writing the CSV
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/s11306-017-1178-y
    title: 'NMRProcFlow: a graphical and interactive tool dedicated to 1D spectra
      processing for NMR-based metabolomics'
  source_content_hash: sha256:82b4f7ccf115e3d6055702beba0a23b31eadffb7739fda9f98e63280af83b439
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/INRA/NMRProcFlow
  tools:
  - NMRProcFlow
  - R
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3383
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# nmr-bucket-table-export

## Summary

Reshapes NMRProcFlow's SNR-filtered bucket table into a samples-by-buckets data matrix with experimental-factor class-label columns, exported in the format NMRProcFlow documents as compatible with MetaboAnalyst 3.0 and BioStatFlow, so processed spectral data reaches downstream statistics without file conversions.

## When to use

Use when a processed 1D NMR bucket table (ppm-interval buckets per spectrum) must be handed off, without intermediate file conversions, to online statistical tools such as MetaboAnalyst 3.0 or BioStatFlow.

## When NOT to use

- The input is raw FID data from Bruker or Agilent (Varian) instruments — spectra processing (calibration, baseline correction, realignment, binning) must run in NMRProcFlow first; this skill only exports the already-processed bucket table.
- The bucket table is already in a statistical-tools-ready matrix format with class labels — re-exporting adds no value.
- The goal is to perform the downstream statistical analysis itself (e.g. PCA, clustering) rather than producing the import-compatible data matrix.

## Inputs

- NMRProcFlow bucket table from processed 1D 1H (or 13C) spectra, with buckets filtered by signal-to-noise ratio at the output step
- ppm interval bucket labels used as column identifiers
- spectral sample identifiers (one per spectrum)
- experimental-factor level information associated with each spectrum

## Outputs

- metaboanalyst_data_matrix.csv: samples-by-buckets CSV with ppm interval labels as column headers, class-label columns from experimental factors, compatible with MetaboAnalyst 3.0 and BioStatFlow

## How to apply

Start from the filtered bucket table produced by NMRProcFlow's spectra processing pipeline (ppm scale calibration, baseline correction, realignment of ppm areas, and binning by Intelligent/variable size/fixed width), whose buckets can be filtered on signal-to-noise ratio either during the computing process or at the output step. Reshape the bucket table into a samples-by-buckets matrix, using the ppm interval bucket labels as column headers and the spectral sample identifiers as rows. Append the experimental-factor level information associated with each spectrum as class-label columns, since NMRProcFlow explicitly bridges spectra and their experimental-factor levels for statistical analysis. Write the matrix to metaboanalyst_data_matrix.csv in the export format NMRProcFlow documents as compatible with MetaboAnalyst 3.0 and BioStatFlow, minimizing file conversions in the handoff.

## Related tools

- **NMRProcFlow** (Produces the SNR-filtered bucket table and defines the export format documented as compatible with MetaboAnalyst 3.0 and BioStatFlow) — https://github.com/INRA/NMRProcFlow
- **R** (Can be used to reshape the bucket table into the samples-by-buckets matrix and append class-label columns before writing the CSV)

## Examples

```
docker pull docker.io/nmrprocflow/npflow && sh ./npflow start
```

## Evaluation signals

- metaboanalyst_data_matrix.csv exists and parses as valid CSV
- Row count of the exported matrix equals the number of processed spectra
- Column count equals the number of retained (SNR-filtered) buckets plus the class-label columns
- Column headers are the ppm interval bucket labels from the bucket table
- Class-label column values match the experimental-factor levels assigned to each spectrum

## Limitations

- Documented export compatibility is specific to MetaboAnalyst 3.0 and BioStatFlow; other downstream tools are not claimed to be supported by this format.
- The set of exported buckets depends on where signal-to-noise filtering is applied — during the computing process or at the output step — so the matrix contents can differ between the two filtering modes.
- The repository README covers installation and running via Docker but defers detailed export documentation to http://nmrprocflow.org; the agent must rely on NMRProcFlow's documented export format for exact column conventions.

## Evidence

- [results] Bucket filtering by signal-to-noise ratio at output: "in NMRProcFlow it is possible to filter buckets based on this ratio either during the computing process, or at the output step."
- [abstract] Bridge between spectra and experimental-factor levels for statistics: "Biologists and NMR spectroscopists can easily interact and develop synergies by visualizing the NMR spectra along with their corresponding experimental-factor levels"
- [readme] Spectra processing steps producing the bucket table: "the spectra processing includes: the calibration of the ppm scale, the base line correction (locally or  fully), the realignment of a set of ppm areas, and the binning"
- [abstract] Scope of NMRProcFlow processing for metabolomics: "NMRProcFlow (http://nmrprocflow.org), dedicated to metabolic fingerprinting and targeted metabolomics, covers all spectra processing steps including baseline correction"
