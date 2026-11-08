---
name: fid-time-domain-signal-extraction
description: Use when the input is a directory of raw Bruker 1H-NMR spectra (each
  sample folder containing 'acqus' and 'fid' files) and you need time-domain FID data
  plus acquisition parameters as pandas DataFrames before any FID-domain preprocessing.
when_to_use_negative:
- Input is already processed spectra (frequency-domain spectra_df) rather than raw
  Bruker time-domain 'fid' files
- Data is in vendor formats other than Bruker 'acqus'/'fid' directory layout, which
  extract_params/extract_fids are not described as handling
- The goal is later-stage preprocessing (e.g. Icoshift peak alignment or binning)
  that operates on spectra, not raw FID extraction
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: Protomix
  role: Provides extract_params (parses 'acqus' into a parameter DataFrame) and extract_fids
    (reads 'fid' files using acqus_df to build the sample-indexed FID DataFrame)
  repo: https://github.com/mzniber/protomix
- name: pandas
  role: Backs the sample-indexed DataFrames returned by the extraction functions and
    the CSV exports (acqus_table.csv, fids_table.csv)
- name: numpy
  role: Numeric handling of the raw FID signal arrays (e.g. plotting the real part
    of a FID)
- name: plotly
  role: Interactive plotting used to visually verify an extracted raw FID
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbae192
    title: 'Protomix: a Python package for 1H-NMR metabolomics data preprocessing'
  source_content_hash: sha256:115b1037dccf496311c10ec0eac6d5d3145253159f7193ab0ed54f8d8448e239
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzniber/protomix
  tools:
  - Protomix
  - Python
  - numpy
  - pandas
  - plotly
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# fid-time-domain-signal-extraction

## Summary

Extract raw Bruker acquisition parameters ('acqus' files) and Free Induction Decay traces ('fid' files) from a Bruker-format spectra directory into sample-indexed pandas DataFrames, where the parameter table feeds the FID extraction step.

## When to use

Use when the input is a directory of raw Bruker 1H-NMR spectra (each sample folder containing 'acqus' and 'fid' files) and you need time-domain FID data plus acquisition parameters as pandas DataFrames before any FID-domain preprocessing.

## When NOT to use

- Input is already processed spectra (frequency-domain spectra_df) rather than raw Bruker time-domain 'fid' files
- Data is in vendor formats other than Bruker 'acqus'/'fid' directory layout, which extract_params/extract_fids are not described as handling
- The goal is later-stage preprocessing (e.g. Icoshift peak alignment or binning) that operates on spectra, not raw FID extraction

## Inputs

- Root directory of unzipped Bruker NMR dataset in original directory format (e.g. Zenodo record 13467227 'spectra' folder)
- Per-sample Bruker 'acqus' acquisition parameter files
- Per-sample Bruker 'fid' free induction decay files

## Outputs

- acqus_df: sample-indexed pandas DataFrame of acquisition parameters
- fids_df: sample-indexed pandas DataFrame of raw FID traces (one row per sample)
- acqus_table.csv and fids_table.csv exports

## How to apply

Set the root directory to the extracted 'spectra' folder keeping the original Bruker directory layout. First run px.extract_params(root_directory=main_directory) to parse the per-sample 'acqus' files into an acquisition-parameter DataFrame (acqus_df). Then run px.extract_fids(root_directory=main_directory, acqus_df=acqus_df), passing the parameter table because FID interpretation requires the acquisition parameters, producing fids_df with one row per sample. Verify extraction by plotting the real part of a raw FID (fids_df.iloc[spectrIndex]) with a plotting library, then persist the tables as acqus_table.csv and fids_table.csv.

## Related tools

- **Protomix** (Provides extract_params (parses 'acqus' into a parameter DataFrame) and extract_fids (reads 'fid' files using acqus_df to build the sample-indexed FID DataFrame)) — https://github.com/mzniber/protomix
- **pandas** (Backs the sample-indexed DataFrames returned by the extraction functions and the CSV exports (acqus_table.csv, fids_table.csv))
- **numpy** (Numeric handling of the raw FID signal arrays (e.g. plotting the real part of a FID))
- **plotly** (Interactive plotting used to visually verify an extracted raw FID)

## Examples

```
acqus_df = px.extract_params(root_directory=main_directory); fids_df = px.extract_fids(root_directory=main_directory, acqus_df=acqus_df)
```

## Evaluation signals

- acqus_df and fids_df have one row per sample directory in the root_directory
- Plotting the real part of a row (fids_df.iloc[spectrIndex]) shows a plausible decaying time-domain signal
- fids_df has the expected number of time-point columns per sample, consistent with the 'acqus' acquisition parameters
- acqus_table.csv and fids_table.csv are written and reloadable with pandas with identical shape to the DataFrames

## Limitations

- extract_fids requires the acqus_df from extract_params as input, so the two extraction steps are coupled and cannot be run independently
- Functions expect the original Bruker directory format under the root directory; data must first be downloaded and unzipped (e.g. from Zenodo record 13467227) before extraction

## Evidence

- [methods] extract_params and extract_fids workflow: "acqus_df = px.extract_params(root_directory=main_directory)"
- [methods] extract_fids consumes acqus_df: "fids_df = px.extract_fids(root_directory=main_directory, acqus_df=acqus_df)"
- [methods] Dataset source: "!wget -O NMRDataset.zip https://zenodo.org/record/13467227/files/NMRDataset.zip?download=1"
- [abstract] Package scope in Python: "It encompasses a suite of functionalities for data extraction, preprocessing, and interactive visualization."
- [intro] Protomix purpose: "Protomix provides the tools you need to prepare your NMR data for analysis"
