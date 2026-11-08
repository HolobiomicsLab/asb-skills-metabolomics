---
name: bruker-acqus-parameter-extraction
description: Use when starting from raw Bruker-format 1H-NMR directories where each
  sample folder contains an 'acqus' parameter file and an 'fid' trace, and you need
  a structured parameter table before FID extraction.
when_to_use_negative:
- Input is already preprocessed spectra or a feature/binned table rather than raw
  Bruker directories with 'acqus' files
- Data are not in Bruker format (e.g. already-converted spectra tables), since extract_params
  expects the per-sample directory structure
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3272
- http://edamontology.org/topic_0593
tools:
- name: Protomix
  role: Provides the extract_params function that parses per-sample Bruker 'acqus'
    files into the acqus_df parameter table
  repo: https://github.com/mzniber/protomix
- name: pandas
  role: Underlying data structure for the sample-indexed acqus_df parameter table
    returned by extract_params
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbae192
    title: 'Protomix: a Python package for 1H-NMR metabolomics data preprocessing'
  source_content_hash: sha256:431dd2ab76a828a7c3845ad158449dba89c1cb5d7b885fccf0b5c466d6085aca
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
  - http://edamontology.org/topic_3272
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# bruker-acqus-parameter-extraction

## Summary

Parse per-sample Bruker 'acqus' acquisition parameter files from a spectra directory into a sample-indexed pandas DataFrame (acqus_df) using Protomix extract_params; this table then drives all downstream FID extraction and preprocessing steps.

## When to use

Use when starting from raw Bruker-format 1H-NMR directories where each sample folder contains an 'acqus' parameter file and an 'fid' trace, and you need a structured parameter table before FID extraction.

## When NOT to use

- Input is already preprocessed spectra or a feature/binned table rather than raw Bruker directories with 'acqus' files
- Data are not in Bruker format (e.g. already-converted spectra tables), since extract_params expects the per-sample directory structure

## Inputs

- Root directory of raw Bruker NMR data in the original 'spectra' folder layout (one subfolder per sample, each containing an 'acqus' file)
- NMR dataset zip from Zenodo record 13467227 (NMRDataset.zip)

## Outputs

- acqus_df: a sample-indexed pandas DataFrame of Bruker acquisition parameters parsed from per-sample 'acqus' files

## How to apply

Download the NMR dataset (e.g. Zenodo record 13467227), unzip it, and set the root directory to the extracted 'spectra' folder while keeping the original Bruker directory layout. Call px.extract_params(root_directory=main_directory), which walks the directory tree, reads each sample's 'acqus' acquisition parameter file, and returns acqus_df — a pandas DataFrame with one row per sample. Do not modify the directory structure, since extraction relies on the standard Bruker layout to associate parameters with samples. Keep acqus_df because it is a required input to the next stage, px.extract_fids, and later to group_delay_removal, zero_filling, and fourier_transform.

## Related tools

- **Protomix** (Provides the extract_params function that parses per-sample Bruker 'acqus' files into the acqus_df parameter table) — https://github.com/mzniber/protomix
- **pandas** (Underlying data structure for the sample-indexed acqus_df parameter table returned by extract_params)

## Examples

```
import protomix as px; acqus_df = px.extract_params(root_directory=main_directory); fids_df = px.extract_fids(root_directory=main_directory, acqus_df=acqus_df)
```

## Evaluation signals

- acqus_df has one row per sample directory under the root 'spectra' folder
- The DataFrame can be saved as acqus_table.csv without losing sample indices
- Passing acqus_df to px.extract_fids(root_directory=main_directory, acqus_df=acqus_df) succeeds and yields fids_df with one row per sample
- Plotting fids_df.iloc[spectrIndex] with px.plot shows a plausible raw FID trace for a chosen sample

## Limitations

- Requires the original Bruker directory format; renaming or restructuring sample folders breaks parameter-to-sample association
- Extraction is the first stage of the Protomix pipeline — subsequent steps (extract_fids, group_delay_removal, zero_filling, fourier_transform) all consume acqus_df, so errors here propagate downstream

## Evidence

- [methods] extract_params call: "acqus_df = px.extract_params(root_directory=main_directory)"
- [methods] extract_fids consumes acqus_df: "fids_df = px.extract_fids(root_directory=main_directory, acqus_df=acqus_df)"
- [methods] dataset download source: "!wget -O NMRDataset.zip https://zenodo.org/record/13467227/files/NMRDataset.zip?download=1"
- [abstract] package purpose: "It encompasses a suite of functionalities for data extraction, preprocessing, and interactive visualization."
