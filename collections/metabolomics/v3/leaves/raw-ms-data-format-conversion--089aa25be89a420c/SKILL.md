---
name: raw-ms-data-format-conversion
description: Use when raw MS data from LC, LC-IM-MS, or LC-SLIM-MS experiments is
  locked in Agilent '.d', Bruker TIMS, or Thermo '.raw' formats and downstream Python/R
  analysis needs cross-platform, lossless access to spectra arrays.
when_to_use_negative:
- Input is already mzML, mz5, or mzMLb and the goal is solely file-size comparison,
  since mz5 is reported to provide the smallest output while MZA files are larger
  than mz5 and mzMLb.
- The target application is LC-MS proteomics with lossy data representations and no
  IM dimension, where the MS2AI-style converter is more appropriate.
- The workflow requires instrument-vendor-specific features not covered by MZA's supported
  input formats (e.g., non-Agilent/Bruker/Thermo proprietary formats).
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MZA
  role: stand-alone command-line executable that converts proprietary vendor MS files
    to the MZA HDF5 data structure
  repo: https://github.com/PNNL-m-q/mza
- name: ProteoWizard
  role: toolkit used in benchmarking to convert raw files to mzML, mzXML, mz5 formats
    for comparison
- name: h5py
  role: Python library required to read the HDF5-based .mza output
- name: rhdf5
  role: R package required to read the HDF5-based .mza output
- name: hdf5plugin
  role: Python package required alongside h5py for reading MZA files
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00313
    title: 'MZA: A Data Conversion Tool to Facilitate Software Development and Artificial
      Intelligence Research in Multidimensional Mass Spectrometry'
  source_content_hash: sha256:67642b733ef9fa3813bb4e496431a7a4e6ea2315f474690838b7d8d3274172f0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mza
  tools:
  - MZA
  - ProteoWizard
  - h5py
  - hdf5plugin
  - mz5
  - mzXML
  - rhdf5
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# raw-ms-data-format-conversion

## Summary

Convert proprietary vendor mass spectrometry files (Agilent '.d', Bruker TIMS '.d', Thermo '.raw', mzML) into the open, HDF5-based MZA format so that raw multidimensional MS spectra become directly accessible from Python and R without vendor SDK dependencies.

## When to use

Use when raw MS data from LC, LC-IM-MS, or LC-SLIM-MS experiments is locked in Agilent '.d', Bruker TIMS, or Thermo '.raw' formats and downstream Python/R analysis needs cross-platform, lossless access to spectra arrays.

## When NOT to use

- Input is already mzML, mz5, or mzMLb and the goal is solely file-size comparison, since mz5 is reported to provide the smallest output while MZA files are larger than mz5 and mzMLb.
- The target application is LC-MS proteomics with lossy data representations and no IM dimension, where the MS2AI-style converter is more appropriate.
- The workflow requires instrument-vendor-specific features not covered by MZA's supported input formats (e.g., non-Agilent/Bruker/Thermo proprietary formats).

## Inputs

- Agilent MassHunter '.d' directory (with or without ion mobility)
- Bruker ion mobility '.d' directory (TIMS)
- Thermo '.raw' file
- mzML file

## Outputs

- HDF5-based .mza file containing a metadata table and jagged arrays per spectrum (Arrays_intensity, Arrays_mz, and optionally Arrays_mzbin/Full_mz_array)
- HDF5 datasets for CCS calibration coefficients (CCScalDT, CCScalSLIM, CCScalTIMS) when detected during conversion

## How to apply

Run the mza.exe executable (directly on Windows 10, or on Ubuntu 22.04 via Wine 6.0.3 + wine-mono 7.3.0, or inside the mzacontainer Docker image with WINEDEBUG=-all) with the required `-file` argument pointing at the raw file. For lossless conversion pass no `-intensityThreshold`, which stores spectra omitting zero-intensity values only; for compressed outputs set a numeric threshold such as 20 so that signals below it are excluded. Convert a single file with `mza -file <path>` or batch-convert all '.d' files in a directory by adding `-extension .d`. The output .mza file contains a metadata table plus jagged 1D arrays per spectrum stored in the `Arrays_intensity`, `Arrays_mz` (or `Arrays_mzbin` with `Full_mz_array` for IM data) HDF5 groups, indexed by the Scan value.

## Related tools

- **MZA** (stand-alone command-line executable that converts proprietary vendor MS files to the MZA HDF5 data structure) — https://github.com/PNNL-m-q/mza
- **ProteoWizard** (toolkit used in benchmarking to convert raw files to mzML, mzXML, mz5 formats for comparison)
- **h5py** (Python library required to read the HDF5-based .mza output)
- **rhdf5** (R package required to read the HDF5-based .mza output)
- **hdf5plugin** (Python package required alongside h5py for reading MZA files)

## Examples

```
wine mza -file test_data\LowHigh_PC_160_180_frames1-10.d -intensityThreshold 20
```

## Evaluation signals

- The output .mza file contains a metadata table and spectra stored in Arrays_intensity/Scan and Arrays_mz/Scan (or Arrays_mzbin/Scan with Full_mz_array) HDF5 groups.
- When run without -intensityThreshold the conversion is lossless except for zero-intensity values; setting -intensityThreshold 20 excludes signals at or below that minimum intensity.
- Batch conversion with -extension .d (or .mzML/.raw) produces one .mza per matching input file.
- For IM-enabled inputs, IonMobilityBin = 0 represents the total frame spectrum and the m/z dimension is stored as indexes (mzbins) into Full_mz_array.
- CCS calibration coefficient datasets (CCScalDT, CCScalSLIM, CCScalTIMS) appear in the output only when detected during conversion.

## Limitations

- MZA files are larger than mz5 and mzMLb and smaller than mzXML and mzML; mz5 provides the smallest output for all benchmarked files, and improvements in speed and file size are still needed.
- The executable is Windows-native and requires Wine (tested 6.0.3 with wine-mono 7.3.0 on Ubuntu 22.04) or a Docker container (mzacontainer) for Unix-like systems.
- Files with too many spectra may trigger partitioning (MzaPath index) because storing many datasets within a single HDF5 group can slow down reading performance.
- No changelog and no version number are documented in the repository, and no DOI, authors, or journal metadata are present in the repo README.

## Evidence

- [intro] Vendor-provided raw file libraries have complex, Windows-bounded dependencies that hinder rapid algorithm and tool development in Python and R, especially for novice developers: [evidence span withheld — anchor status: not_found]
