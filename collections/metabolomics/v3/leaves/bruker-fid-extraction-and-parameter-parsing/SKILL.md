---
name: bruker-fid-extraction-and-parameter-parsing
description: Use when the input is a directory of raw Bruker 1H-NMR sample folders
  (e.g. NMRDataset.zip from Zenodo record 13467227) that must be converted into FID
  and acqus tables before spectral preprocessing.
when_to_use_negative:
- Input is already a frequency-domain spectra table (spectra_df); start instead at
  phase_correction or later steps
- Data are in non-Bruker vendor formats (e.g. JCAMP-DX from other vendors or processed
  mzML) rather than native Bruker sample directories
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: Protomix
  role: Python package providing extract_params and extract_fids for parsing Bruker
    acquisition parameters and FIDs
  repo: https://github.com/mzniber/protomix
- name: Python 3
  role: Runtime environment; Protomix is written in Python 3 and installed via pip
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbae192
    title: 'Protomix: a Python package for 1H-NMR metabolomics data preprocessing'
  source_content_hash: sha256:393558fd7ec9d1d06cce4a1dfbd35478725cc3d15212351849dd596489de9aa0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzniber/protomix
  tools:
  - Protomix
  - Python
  - Python 3
  - numpy
  - pandas
  - pyicoshift
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# bruker-fid-extraction-and-parameter-parsing

## Summary

Ingest raw Bruker-formatted 1H-NMR acquisition directories by extracting acquisition parameters (acqus) and Free Induction Decays (FIDs) with Protomix, producing tabular DataFrames that seed the FID-to-spectrum preprocessing chain.

## When to use

Use when the input is a directory of raw Bruker 1H-NMR sample folders (e.g. NMRDataset.zip from Zenodo record 13467227) that must be converted into FID and acqus tables before spectral preprocessing.

## When NOT to use

- Input is already a frequency-domain spectra table (spectra_df); start instead at phase_correction or later steps
- Data are in non-Bruker vendor formats (e.g. JCAMP-DX from other vendors or processed mzML) rather than native Bruker sample directories

## Inputs

- Root directory of Bruker-formatted NMR sample folders (e.g. unzipped NMRDataset.zip from Zenodo record 13467227)
- Bruker acqus acquisition parameter files within each sample directory

## Outputs

- acqus_df: pandas DataFrame of acquisition parameters indexed per sample
- fids_df: pandas DataFrame of raw Free Induction Decays (time-domain signals) per sample

## How to apply

Unzip the dataset archive to obtain a root directory containing one subdirectory per sample in Bruker format. Call px.extract_params(root_directory=main_directory) to parse acquisition parameters into an acqus_df, then call px.extract_fids(root_directory=main_directory, acqus_df=acqus_df) to load the raw time-domain FIDs into a fids_df; the acqus_df supplies per-sample metadata (e.g. spectral width, points) needed by downstream steps such as zero_filling and fourier_transform. These two calls are the entry point of the Protomix workflow and must precede group_delay_removal, solvent_residuals_removal, apodization, and zero_filling. Verify each sample directory was parsed by checking row counts of both DataFrames match the number of Bruker sample folders (70 in the urine dataset case study).

## Related tools

- **Protomix** (Python package providing extract_params and extract_fids for parsing Bruker acquisition parameters and FIDs) — https://github.com/mzniber/protomix
- **Python 3** (Runtime environment; Protomix is written in Python 3 and installed via pip)

## Examples

```
import protomix as px; acqus_df = px.extract_params(root_directory=main_directory); fids_df = px.extract_fids(root_directory=main_directory, acqus_df=acqus_df)
```

## Evaluation signals

- Row counts of acqus_df and fids_df equal the number of Bruker sample directories (70 for the Zenodo 13467227 urine dataset)
- Each fids_df entry is a numeric time-domain array whose length is consistent with the acqus parameters (e.g. supports zero_filling to target_points=10000 later)
- No sample directories are skipped: extract_params/extract_fids complete without missing-acqus errors across the whole root_directory
- Downstream group_delay_removal runs without error, confirming acqus_df fields were parsed correctly

## Limitations

- Only Bruker-formatted raw data are handled by the extract_* functions; other vendor formats need conversion first
- The extraction step depends on a correctly unzipped directory layout (NMRDataset.zip -> Bruker spectra directory); a misplaced root_directory yields empty or misaligned DataFrames

## Evidence

- [methods] Extract acquisition parameters from Bruker data: "acqus_df = px.extract_params(root_directory=main_directory)"
- [methods] Extract FIDs using parsed acqus parameters: "fids_df = px.extract_fids(root_directory=main_directory, acqus_df=acqus_df)"
- [methods] Dataset source for the Bruker archive: "!wget -O NMRDataset.zip https://zenodo.org/record/13467227/files/NMRDataset.zip?download=1"
- [abstract] Protomix scope covers data extraction as a first-class feature: "It encompasses a suite of functionalities for data extraction, preprocessing, and interactive visualization."
