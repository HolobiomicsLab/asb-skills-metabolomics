---
name: bruker-directory-format-handling
description: Use when the input is a root directory of Bruker spectrometer folders
  (one subdirectory per sample containing 'acqus' and 'fid' files) and the goal is
  to build acqus_df and fids_df tables before FID-level preprocessing.
when_to_use_negative:
- Input is already preprocessed spectra (e.g. a spectra_df or binned/bin_df table)
  rather than raw Bruker directories — use downstream Protomix steps like fourier_transform,
  phase_correction, or binning instead
- Data are in a non-Bruker vendor format (no per-sample 'acqus'/'fid' files) — extract_params/extract_fids
  expect the original Bruker directory layout
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3272
tools:
- name: Protomix
  role: Provides px.extract_params and px.extract_fids, which parse 'acqus' files
    and 'fid' traces from the Bruker root directory into sample-indexed DataFrames
  repo: https://github.com/mzniber/protomix
- name: pandas
  role: Underlying DataFrame format for the returned acqus_df and fids_df tables
- name: numpy
  role: Array backend for the raw FID signal values held in fids_df rows
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbae192
    title: 'Protomix: a Python package for 1H-NMR metabolomics data preprocessing'
  source_content_hash: sha256:c1264da54db74eed7f07ee08214074fa861fa3799e6d3411db90fd7cdcbb853e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzniber/protomix
  tools:
  - Protomix
  - Python
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# bruker-directory-format-handling

## Summary

Reading raw Bruker NMR data directories so that per-sample 'acqus' acquisition-parameter files and 'fid' free-induction-decay traces are parsed into sample-indexed pandas DataFrames, forming the entry point of the Protomix 1H-NMR preprocessing pipeline.

## When to use

Use when the input is a root directory of Bruker spectrometer folders (one subdirectory per sample containing 'acqus' and 'fid' files) and the goal is to build acqus_df and fids_df tables before FID-level preprocessing.

## When NOT to use

- Input is already preprocessed spectra (e.g. a spectra_df or binned/bin_df table) rather than raw Bruker directories — use downstream Protomix steps like fourier_transform, phase_correction, or binning instead
- Data are in a non-Bruker vendor format (no per-sample 'acqus'/'fid' files) — extract_params/extract_fids expect the original Bruker directory layout

## Inputs

- Root directory in original Bruker format, with one subfolder per sample containing 'acqus' acquisition-parameter files and 'fid' FID files
- NMR dataset zip from Zenodo record 13467227 (NMRDataset), unzipped so the 'spectra' folder is the root

## Outputs

- acqus_df: sample-indexed pandas DataFrame of parsed Bruker acquisition parameters (one row per sample)
- fids_df: sample-indexed pandas DataFrame of raw FID traces, one row per sample, usable with px.group_delay_removal(fid_df=fids_df, acqus_df=acqus_df)

## How to apply

Point Protomix at the root directory holding the original Bruker directory format, e.g. the 'spectra' folder obtained by unzipping the Zenodo record 13467227 NMR dataset. Call px.extract_params(root_directory=main_directory) to walk the directory tree and parse each sample's 'acqus' file into a sample-indexed acquisition-parameter DataFrame (acqus_df). Then call px.extract_fids(root_directory=main_directory, acqus_df=acqus_df): extract_fids reads the raw 'fid' binary traces and uses acqus_df to interpret each sample's FID, returning fids_df with one row per sample. The two extraction functions are coupled — the parameter table is an explicit input to FID extraction — so extract_params must run first and both must target the same root directory.

## Related tools

- **Protomix** (Provides px.extract_params and px.extract_fids, which parse 'acqus' files and 'fid' traces from the Bruker root directory into sample-indexed DataFrames) — https://github.com/mzniber/protomix
- **pandas** (Underlying DataFrame format for the returned acqus_df and fids_df tables)
- **numpy** (Array backend for the raw FID signal values held in fids_df rows)

## Examples

```
acqus_df = px.extract_params(root_directory=main_directory); fids_df = px.extract_fids(root_directory=main_directory, acqus_df=acqus_df)
```

## Evaluation signals

- acqus_df has one row per sample subdirectory in the root directory (index aligned to sample folder names)
- fids_df has one row per sample and the same index alignment as acqus_df, since extract_fids consumes acqus_df
- Sanity-check a raw FID by plotting its real part, e.g. px.plot(fids_df.iloc[spectrIndex]), and confirm a decaying oscillatory trace rather than zeros or noise
- Persist both tables (acqus_table.csv, fids_table.csv) and confirm downstream calls (group_delay_removal, zero_filling) accept them without index-mismatch errors

## Limitations

- Extracted FIDs are raw (pre-group-delay-removal, pre-apodization); they must pass through group_delay_removal and other filter steps before Fourier transformation
- The extraction assumes the original Bruker directory format is preserved after unzipping; renamed or flattened directories will break per-sample discovery
- extract_fids depends on acqus_df to interpret each FID, so running extraction on a mismatched root directory or a stale parameter table will misinterpret traces

## Evidence

- [methods] extract_params: "acqus_df = px.extract_params(root_directory=main_directory)"
- [methods] extract_fids: "fids_df = px.extract_fids(root_directory=main_directory, acqus_df=acqus_df)"
- [methods] two-stage extraction relationship: "Extract Free Induction Decays (FID)"
- [methods] dataset provenance: "!wget -O NMRDataset.zip https://zenodo.org/record/13467227/files/NMRDataset.zip?download=1"
- [abstract] Protomix scope: "It encompasses a suite of functionalities for data extraction, preprocessing, and interactive visualization."
