---
name: mzml-spectral-data-loading
description: Use when raw MS/MS peak files are in mzML format and need to be read
  into memory as spectra objects before embedding, spectral clustering, or similarity
  computation.
when_to_use_negative:
- Peak files are not in mzML format (e.g., raw vendor files that have not been converted)
- You need only metadata/identifications without peak arrays — use the MassIVE-KB
  metadata TSV directly instead of parsing mzML
- Input is already a NumPy embedding array or a parquet metadata table (loading is
  already done)
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: GLEAMS
  role: Provides the 'gleams embed' command that reads MS/MS spectra from mzML peak
    files and converts them to 32-dimensional embeddings
  repo: https://github.com/bittremieux/GLEAMS
- name: Python 3.8
  role: Required runtime for running GLEAMS' spectrum loading and embedding on a Linux
    system with a CUDA-enabled GPU
- name: conda
  role: Creates the environment with compiler tools and GPU runtime needed to run
    GLEAMS' mzML loading
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41592-022-01496-1
    title: A learned embedding for efficient joint analysis of millions of mass spectra
  source_content_hash: sha256:b1e626bab0c83c525aecf1c4a89b56c81a22b41e4b3b11a611f419f5e6f8b322
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/GLEAMS
  tools:
  - ANN-SoLo
  - GLEAMS
  - Python
  - Python 3.8
  - conda
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# mzml-spectral-data-loading

## Summary

Load MS/MS spectra from mzML peak files so they can be featurized and embedded (e.g., by GLEAMS) into a 32-dimensional NumPy array with matching per-spectrum metadata in a parquet file.

## When to use

Use when raw MS/MS peak files are in mzML format and need to be read into memory as spectra objects before embedding, spectral clustering, or similarity computation.

## When NOT to use

- Peak files are not in mzML format (e.g., raw vendor files that have not been converted)
- You need only metadata/identifications without peak arrays — use the MassIVE-KB metadata TSV directly instead of parsing mzML
- Input is already a NumPy embedding array or a parquet metadata table (loading is already done)

## Inputs

- MS/MS spectra in mzML peak files
- MassIVE-KB metadata TSV with 'filename' and 'scan' columns (for training-data retrieval)
- Spectra downloaded from the MassIVE FTP server

## Outputs

- GLEAMS_embed.npy: two-dimensional NumPy array (n x 32) of spectrum embeddings
- GLEAMS_embed.parquet: tabular metadata for the embedded spectra
- In-memory MS/MS spectrum objects read from matched mzML files

## How to apply

Provide the mzML peak files (e.g., glob 'gleams embed *.mzML') to the GLEAMS embed command, which reads the MS/MS spectra from all matched mzML files. GLEAMS converts each spectrum to a 32-dimensional embedding and exports results to a two-dimensional NumPy array of dimension n x 32 (n = number of MS/MS spectra read), saved as GLEAMS_embed.npy. A tabular metadata file GLEAMS_embed.parquet is also created containing corresponding metadata for the embedded spectra. Rationale: this loading step is the entry point of the GLEAMS pipeline — spectra must be read from peak files before they can be embedded by the neural network or clustered. For the GLEAMS training-data workflow, spectra are retrieved from the MassIVE FTP server using the 'filename' column of the MassIVE-KB metadata TSV and extracted by scan number (column 'scan').

## Related tools

- **GLEAMS** (Provides the 'gleams embed' command that reads MS/MS spectra from mzML peak files and converts them to 32-dimensional embeddings) — https://github.com/bittremieux/GLEAMS
- **Python 3.8** (Required runtime for running GLEAMS' spectrum loading and embedding on a Linux system with a CUDA-enabled GPU)
- **conda** (Creates the environment with compiler tools and GPU runtime needed to run GLEAMS' mzML loading)

## Examples

```
gleams embed *.mzML --embed_name GLEAMS_embed
```

## Evaluation signals

- GLEAMS_embed.npy exists and has shape (n, 32), where n equals the number of MS/MS spectra read from the mzML files
- GLEAMS_embed.parquet exists with one metadata row per embedded spectrum
- Spectra read from mzML correspond to the expected MassIVE-KB filename/scan pairs when working from the training metadata TSV
- The embedding run completes without GPU/runtime errors (Python 3.8, Linux, CUDA-enabled GPU required)

## Limitations

- GLEAMS requires Python 3.8, a Linux operating system, and a CUDA-enabled GPU
- The full GLEAMS training dataset (30 million PSMs from MassIVE-KB v1) is not available as a single download; spectra must be retrieved from the MassIVE FTP server by filename and scan number
- Model-weights download may hit Git LFS data-quota errors; weights must then be downloaded manually from the latest release into the data/ directory

## Evidence

- [readme] GLEAMS provides the `gleams embed` command to convert MS/MS spectra in peak files to 32-dimensional embeddings.: [evidence span withheld — anchor status: external_doc]
- [readme] Embedding output is an n x 32 NumPy array plus parquet metadata.: "This will read the MS/MS spectra from all matched mzML files and export the results to a two-dimensional NumPy array of dimension _n_ x 32 in file `GLEAMS_embed.npy`"
- [readme] Metadata parquet accompanies the embedding array.: "Additionally, a tabular file `GLEAMS_embed.parquet` will be created containing corresponding metadata for the embedded spectra."
- [readme] Runtime requirements for GLEAMS spectrum loading.: "GLEAMS requires Python 3.8, a Linux operating system, and a CUDA-enabled GPU."
- [readme] Training spectra are retrieved by filename and scan number from MassIVE.: "Using the filename (column "filename") you can then [retrieve the corresponding spectra from the MassIVE FTP server]"
