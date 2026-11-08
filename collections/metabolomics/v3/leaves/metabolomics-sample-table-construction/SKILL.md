---
name: metabolomics-sample-table-construction
description: Use when the input is a directory of raw Bruker NMR spectra in the original
  Bruker directory format (per-sample 'acqus' files and 'fid' binary traces) and you
  need one-row-per-sample parameter and FID tables before any FID-to-spectrum preprocessing.
when_to_use_negative:
- Input is already a preprocessed spectra/feature table or binned matrix — extraction
  is only the first stage before FID-to-spectrum processing.
- The data are not in Bruker format (e.g. already-converted mzML/CSV spectra); extract_params/extract_fids
  expect per-sample Bruker 'acqus' and 'fid' files.
- You only need downstream preprocessing (peak alignment, binning) on an existing
  fids_df with a matching acqus_df.
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: Protomix
  role: Provides extract_params and extract_fids functions that parse Bruker 'acqus'
    files and 'fid' traces into sample-indexed pandas DataFrames
  repo: https://github.com/mzniber/protomix
- name: pandas
  role: Underlying tabular structure for acqus_df and fids_df (one row per sample)
- name: numpy
  role: Numerical representation of the raw FID traces inside fids_df
- name: plotly
  role: Verification plot of the real part of an extracted raw FID (fids_df.iloc[spectrIndex])
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbae192
    title: 'Protomix: a Python package for 1H-NMR metabolomics data preprocessing'
  source_content_hash: sha256:a60b668477eef7eddc99e78707cb8b560c5699d4c3dfe358acfb32bdaef82ddd
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
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# metabolomics-sample-table-construction

## Summary

Builds sample-indexed pandas DataFrames from raw Bruker 1H-NMR data by parsing per-sample 'acqus' acquisition-parameter files and extracting raw 'fid' traces with Protomix, producing the acqus_df and fids_df tables that feed the entire downstream preprocessing chain.

## When to use

Use when the input is a directory of raw Bruker NMR spectra in the original Bruker directory format (per-sample 'acqus' files and 'fid' binary traces) and you need one-row-per-sample parameter and FID tables before any FID-to-spectrum preprocessing.

## When NOT to use

- Input is already a preprocessed spectra/feature table or binned matrix — extraction is only the first stage before FID-to-spectrum processing.
- The data are not in Bruker format (e.g. already-converted mzML/CSV spectra); extract_params/extract_fids expect per-sample Bruker 'acqus' and 'fid' files.
- You only need downstream preprocessing (peak alignment, binning) on an existing fids_df with a matching acqus_df.

## Inputs

- Root directory of raw Bruker NMR data in original Bruker directory format (per-sample 'acqus' parameter files and 'fid' binary FID files), e.g. the 'spectra' folder from Zenodo record 13467227
- For extract_fids: the acqus_df DataFrame produced by extract_params

## Outputs

- acqus_df: sample-indexed pandas DataFrame of Bruker acquisition parameters from 'acqus' files
- fids_df: sample-indexed pandas DataFrame of raw FID traces (one row per sample)
- Persisted CSV tables (acqus_table.csv, fids_table.csv)

## How to apply

Download and unzip the NMR dataset (e.g. Zenodo record 13467227) and point root_directory at the extracted 'spectra' folder, keeping the original Bruker directory format. First call px.extract_params(root_directory=main_directory) to parse each sample's 'acqus' acquisition-parameter file into acqus_df, an acquisition-parameter DataFrame indexed by sample. Then call px.extract_fids(root_directory=main_directory, acqus_df=acqus_df), passing the parameter table because extract_fids uses acqus_df to interpret each sample's raw FID trace; the result fids_df holds one row per sample with the raw FID values. Verify extraction by plotting the real part of a single raw FID (e.g. fids_df.iloc[spectrIndex]) with a plotting library such as Plotly, then persist the tables (acqus_table.csv, fids_table.csv) for downstream steps like group_delay_removal and fourier_transform.

## Related tools

- **Protomix** (Provides extract_params and extract_fids functions that parse Bruker 'acqus' files and 'fid' traces into sample-indexed pandas DataFrames) — https://github.com/mzniber/protomix
- **pandas** (Underlying tabular structure for acqus_df and fids_df (one row per sample))
- **numpy** (Numerical representation of the raw FID traces inside fids_df)
- **plotly** (Verification plot of the real part of an extracted raw FID (fids_df.iloc[spectrIndex]))

## Examples

```
acqus_df = px.extract_params(root_directory=main_directory); fids_df = px.extract_fids(root_directory=main_directory, acqus_df=acqus_df)
```

## Evaluation signals

- acqus_df and fids_df each contain one row per sample directory in the root_directory, with consistent sample indexing between the two tables.
- acqus_df holds values parsed from each sample's 'acqus' file; fids_df holds numeric FID arrays of the length implied by the acquisition parameters.
- extract_fids runs successfully with acqus_df as an input — it requires the parameter table to interpret each sample's FID, so a mismatched/missing acqus_df should fail or misalign.
- Plotting the real part of fids_df.iloc[spectrIndex] shows a plausible decaying NMR signal (Free Induction Decay) rather than zeros, NaNs, or garbage values.
- Tables persist cleanly to acqus_table.csv and fids_table.csv and round-trip through pandas read_csv for downstream steps.

## Limitations

- Extraction requires the original Bruker directory format with per-sample 'acqus' and 'fid' files; other vendor formats or pre-converted spectra are not covered by these functions.
- extract_fids depends on acqus_df — the parameter table must be extracted first and from the same root_directory so sample indices align.
- The extracted tables are raw: no group delay removal, solvent removal, or apodization has been applied yet, so signals are not comparable across samples until the full preprocessing chain runs.

## Evidence

- [methods] extract_params parses acqus files: "acqus_df = px.extract_params(root_directory=main_directory)"
- [methods] extract_fids takes acqus_df as input: "fids_df = px.extract_fids(root_directory=main_directory, acqus_df=acqus_df)"
- [methods] Dataset download from Zenodo: "!wget -O NMRDataset.zip https://zenodo.org/record/13467227/files/NMRDataset.zip?download=1"
- [abstract] Protomix scope includes data extraction: "It encompasses a suite of functionalities for data extraction, preprocessing, and interactive visualization."
- [readme] README describes Protomix purpose: "Protomix is a comprehensive Python package designed for preprocessing Nuclear Magnetic Resonance (NMR) data."
