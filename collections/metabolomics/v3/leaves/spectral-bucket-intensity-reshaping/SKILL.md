---
name: spectral-bucket-intensity-reshaping
description: Use when a processed 1D NMR bucket table from NMRProcFlow must be handed
  off to an online statistical tool (MetaboAnalyst 3.0 / BioStatFlow) and you need
  a ready-to-import samples-by-buckets matrix with experimental-factor class labels.
when_to_use_negative:
- The data is already a samples-by-features matrix with class labels — no reshaping
  or export step is needed
- The spectra have not yet been binned (e.g. only raw FID or unaligned spectra are
  available); NMRProcFlow's binning and SNR bucket filtering must come first
- The data is not 1D NMR metabolomics (NMRProcFlow targets 1D 1H & 13C spectra from
  Bruker and Agilent/Varian FID)
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: NMRProcFlow
  role: Produces the SNR-filtered bucket table and defines the documented export format
    compatible with MetaboAnalyst 3.0 and BioStatFlow
  repo: https://github.com/INRA/NMRProcFlow
- name: R
  role: Open source language (also used to implement NMRProcFlow's NMRspec module)
    in which the bucket-table reshaping and CSV export can be scripted
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/s11306-017-1178-y
    title: 'NMRProcFlow: a graphical and interactive tool dedicated to 1D spectra
      processing for NMR-based metabolomics'
  source_content_hash: sha256:b80d447656161c5f955837fff03f3aa33f50486dd1a9ea3a4a2d0cb91726edf4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/INRA/NMRProcFlow
  tools:
  - NMRProcFlow
  - R
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# spectral-bucket-intensity-reshaping

## Summary

Reshapes a filtered NMRProcFlow bucket table (ppm-interval buckets x spectra) into a samples-by-buckets data matrix with class-label columns, exported in the CSV format NMRProcFlow documents as directly compatible with MetaboAnalyst 3.0 and BioStatFlow, avoiding intermediate file conversions.

## When to use

Use when a processed 1D NMR bucket table from NMRProcFlow must be handed off to an online statistical tool (MetaboAnalyst 3.0 / BioStatFlow) and you need a ready-to-import samples-by-buckets matrix with experimental-factor class labels.

## When NOT to use

- The data is already a samples-by-features matrix with class labels — no reshaping or export step is needed
- The spectra have not yet been binned (e.g. only raw FID or unaligned spectra are available); NMRProcFlow's binning and SNR bucket filtering must come first
- The data is not 1D NMR metabolomics (NMRProcFlow targets 1D 1H & 13C spectra from Bruker and Agilent/Varian FID)

## Inputs

- Filtered bucket table from NMRProcFlow 1D NMR spectra processing (ppm interval buckets vs spectra, SNR-filtered at output)
- Experimental-factor level metadata associated with each spectrum

## Outputs

- metaboanalyst_data_matrix.csv: samples-by-buckets data matrix with ppm interval bucket labels as column headers, spectral sample identifiers as rows, and experimental-factor class-label columns, compatible with MetaboAnalyst 3.0 and BioStatFlow

## How to apply

Start from the filtered bucket table produced by NMRProcFlow's spectra processing pipeline (calibration of the ppm scale, baseline correction, realignment of ppm areas, and Intelligent/variable-size/fixed-width binning), ensuring buckets were filtered on signal-to-noise ratio at the output step. Reshape the table so rows are spectral sample identifiers and columns are ppm interval bucket labels. Append one class-label column per experimental-factor level so downstream tools can interpret the experimental design alongside the spectra. Write the result to metaboanalyst_data_matrix.csv in NMRProcFlow's documented export format for MetaboAnalyst 3.0 and BioStatFlow, which is why this skill minimizes file conversions.

## Related tools

- **NMRProcFlow** (Produces the SNR-filtered bucket table and defines the documented export format compatible with MetaboAnalyst 3.0 and BioStatFlow) — https://github.com/INRA/NMRProcFlow
- **R** (Open source language (also used to implement NMRProcFlow's NMRspec module) in which the bucket-table reshaping and CSV export can be scripted)

## Examples

```
git clone https://github.com/INRA/NMRProcFlow.git && docker pull docker.io/nmrprocflow/npflow
```

## Evaluation signals

- metaboanalyst_data_matrix.csv exists and parses as valid CSV
- Row count equals the number of processed spectra
- Column count equals the number of retained (SNR-filtered) buckets plus the class-label columns
- Column headers correspond to ppm interval bucket labels from the NMRProcFlow bucket table
- Each spectrum's row carries its experimental-factor level values in the class-label columns

## Limitations

- Bucket retention depends on the signal-to-noise ratio filter, which NMRProcFlow can apply either during the computing process or at the output step — the export only contains buckets that survived filtering
- NMRProcFlow's documented downstream-tool compatibility is stated for BioStatFlow and MetaboAnalyst 3.0 specifically; other statistical tools may need format adjustments
- Spectra alignment quality affects bucket correspondences; the paper notes that spectra alignment remains a challenge requiring user expertise

## Evidence

- [results] in NMRProcFlow it is possible to filter buckets based on this ratio either during the computing process, or at the output step.: "in NMRProcFlow it is possible to filter buckets based on this ratio either during the computing process, or at the output step."
- [abstract] Biologists and NMR spectroscopists can easily interact and develop synergies by visualizing the NMR spectra along with their corresponding experimental-factor levels: "Biologists and NMR spectroscopists can easily interact and develop synergies by visualizing the NMR spectra along with their corresponding experimental-factor levels"
- [readme] the realignment of a set of ppm areas, and the binning (Intelligent, variable size or fixed width): "the realignment of a set of ppm areas, and the binning (Intelligent, variable size or fixed width)"
- [abstract] NMRProcFlow was developed to be a graphical and interactive 1D NMR (1H & 13C) spectra processing tool.: "NMRProcFlow was developed to be a graphical and interactive 1D NMR (1H & 13C) spectra processing tool."
