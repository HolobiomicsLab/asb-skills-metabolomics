---
name: spectral-data-conversion-raw-to-mzml
description: Use when starting from vendor raw MS/MS files (e.g. downloaded from PRIDE
  as PRIDE dataset PXD015943) that must be converted to mzML before spectrum embedding,
  clustering, or spectral library searching.
when_to_use_negative:
- Input files are already in mzML (or other supported peak formats) — skip conversion
  and pass them directly to `gleams embed`.
- The goal is PSM identification rather than spectral embedding — use a database search
  tool such as MSGF+ on raw-compatible inputs instead of converting for GLEAMS.
- The task requires training-data compilation from MassIVE-KB, where spectra are retrieved
  per-scan from the MassIVE FTP server rather than converted wholesale from vendor
  raw files.
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: GLEAMS
  role: Downstream consumer of the converted mzML peak files; its `gleams embed` command
    reads MS/MS spectra from mzML files and exports 32-dimensional embeddings to NumPy
    (.npy) and metadata (.parquet) files
  repo: https://github.com/bittremieux/GLEAMS
- name: ProteoWizard msconvert
  role: File-format converter used to convert vendor raw mass spectrometry files to
    mzML peak files (named in the PXD015943 reproduction workflow)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41592-022-01496-1
    title: A learned embedding for efficient joint analysis of millions of mass spectra
  source_content_hash: sha256:1406dc7583db2b534aaf66e51566e253dfa5d3eb71d0912c4b8f6a1c3659f4e1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/GLEAMS
  tools:
  - GLEAMS
  - ProteoWizard msconvert
  - Python
  - conda
  - pip
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# spectral-data-conversion-raw-to-mzml

## Summary

Convert vendor raw mass spectrometry files into open mzML peak files so downstream GLEAMS embedding and clustering can consume them; this is the first data-preparation step before `gleams embed` generates 32-dimensional spectral embeddings.

## When to use

Use when starting from vendor raw MS/MS files (e.g. downloaded from PRIDE as PRIDE dataset PXD015943) that must be converted to mzML before spectrum embedding, clustering, or spectral library searching.

## When NOT to use

- Input files are already in mzML (or other supported peak formats) — skip conversion and pass them directly to `gleams embed`.
- The goal is PSM identification rather than spectral embedding — use a database search tool such as MSGF+ on raw-compatible inputs instead of converting for GLEAMS.
- The task requires training-data compilation from MassIVE-KB, where spectra are retrieved per-scan from the MassIVE FTP server rather than converted wholesale from vendor raw files.

## Inputs

- Vendor raw mass spectrometry files downloaded from a repository such as PRIDE (e.g. PXD015943 phosphoproteomics raw files)
- ProteoWizard msconvert (or equivalent raw-to-mzML converter)

## Outputs

- mzML peak files containing MS/MS spectra, one per converted raw file
- Converted spectra ready for `gleams embed` (which expects mzML inputs)

## How to apply

Download the raw mass spectrometry files from their archive (e.g. PRIDE for PXD015943) and convert each vendor raw file to mzML using a file-format converter such as ProteoWizard msconvert, preserving MS/MS peak lists. The mzML outputs are the required input format for GLEAMS: the `gleams embed *.mzML` command reads MS/MS spectra from all matched mzML files. Keep filenames consistent so the generated `GLEAMS_embed.parquet` metadata can be linked back to source runs, and confirm conversion fidelity by checking that scan counts and precursor masses in the mzML match the raw vendor files. Only after conversion should the trained model weights (e.g. gleams_82c0124b.hdf5 from the GLEAMS v0.3 release) be applied to embed the spectra.

## Related tools

- **GLEAMS** (Downstream consumer of the converted mzML peak files; its `gleams embed` command reads MS/MS spectra from mzML files and exports 32-dimensional embeddings to NumPy (.npy) and metadata (.parquet) files) — https://github.com/bittremieux/GLEAMS
- **ProteoWizard msconvert** (File-format converter used to convert vendor raw mass spectrometry files to mzML peak files (named in the PXD015943 reproduction workflow))

## Examples

```
msconvert PXD015943_raw/*.raw -o mzml/ && gleams embed mzml/*.mzML --embed_name GLEAMS_embed
```

## Evaluation signals

- Every raw vendor file yields a corresponding mzML file with no conversion failures before running `gleams embed`.
- Number of MS/MS spectra read from the mzML files matches the raw file contents (e.g. 18.6 million embedded spectra expected for PXD015943).
- `gleams embed` runs without format errors and produces `GLEAMS_embed.npy` of dimension n x 32 plus `GLEAMS_embed.parquet` metadata.
- Precursor m/z values and scan numbers in the mzML agree with the original vendor raw files, ensuring downstream pair generation (positive/negative labels within 10 ppm precursor mass tolerance) is traceable.

## Limitations

- GLEAMS requires Python 3.8, a Linux operating system, and a CUDA-enabled GPU, so the converted mzML files can only be embedded in that environment.
- Very large datasets (e.g. 669 million spectra processed with GLEAMS, or 18.6 million embedded spectra for PXD015943) make conversion and embedding resource-intensive; plan storage and compute accordingly.
- Pair labeling for evaluation depends on available identifications (e.g. Mascot PSMs for PXD015943); conversion alone does not produce peptide labels.

## Evidence

- [methods] Convert the vendor raw files to mzML peak files using a file-format converter (e.g. ProteoWizard msconvert).: [evidence span withheld — anchor status: not_found]
- [readme] This will read the MS/MS spectra from all matched mzML files and export the results to a two-dimensional NumPy array of dimension n x 32.: "This will read the MS/MS spectra from all matched mzML files and export the results to a two-dimensional NumPy array of dimension n x 32 in file `GLEAMS_embed.npy`"
- [results] Download the raw mass spectrometry data for the PXD015943 dataset from PRIDE.: [evidence span withheld — anchor status: not_found]
- [intro] GLEAMS requires Python 3.8, a Linux operating system, and a CUDA-enabled GPU.: "GLEAMS requires Python 3.8, a Linux operating system, and a CUDA-enabled GPU."
