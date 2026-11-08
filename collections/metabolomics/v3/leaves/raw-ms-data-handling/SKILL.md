---
name: raw-ms-data-handling
description: Use when raw multidimensional LC-MS/LC-IM-MS data (e.g. .d, .raw) must
  be converted to an open format for feature engineering, machine learning, or visualization
  outside vendor Windows-bound software.
when_to_use_negative:
- Input is already an open-format file (.mza, or plain mzML/XML) that downstream tools
  can read directly without conversion
- The analysis requires vendor-native processing or the optimized parallel conversion
  speeds of ProteoWizard for mzXML/mzML/mz5 — the paper notes 'improvements in speed
  and file size are still needed'
- The workflow needs lossy compression or proteomics-specific packaging such as MS2AI,
  which discards information and does not support the ion mobility dimension
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MZA
  role: stand-alone command-line executable that converts vendor raw MS files to the
    HDF5-based .mza format
  repo: https://github.com/PNNL-m-q/mza
- name: HDF5
  role: binary hierarchical file format underlying the .mza file structure, enabling
    cross-platform, cross-language access
  repo: https://github.com/PNNL-m-q/mza
- name: Docker
  role: mzacontainer image for running wine mza.exe on Unix-like operating systems
  repo: https://github.com/PNNL-m-q/mza
- name: Wine
  role: compatibility layer (tested 6.0.3, with wine-mono 7.3.0) needed to run mza.exe
    on Ubuntu 22.04.1 without Docker
  repo: https://github.com/PNNL-m-q/mza
- name: h5py
  role: Python library for reading .mza HDF5 datasets in example access scripts
  repo: https://github.com/PNNL-m-q/mza
- name: rhdf5
  role: R package for reading .mza HDF5 datasets in example access scripts
  repo: https://github.com/PNNL-m-q/mza
- name: CCSz-demo
  role: example scripts converting IM-MS data to CCS/z space and visualizing mobility
    distributions from .mza files
  repo: https://github.com/EMSL-Computing/CCSz-demo
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00313
    title: 'MZA: A Data Conversion Tool to Facilitate Software Development and Artificial
      Intelligence Research in Multidimensional Mass Spectrometry'
  source_content_hash: sha256:143290bc13fea792078da470912f143a0d3346eefe6cb5a5d8064689add53eb4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mza
  tools:
  - CCSz-demo
  - Docker
  - HDF5
  - MZA
  - Wine
  - h5py
  - mz5
  - mzMLb
  - mzXML
  - rhdf5
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# raw-ms-data-handling

## Summary

Convert proprietary vendor raw mass spectrometry files (Agilent .d, Bruker TIMS .d, Thermo .raw, mzML) into the HDF5-based MZA (.mza) format so raw MS spectra can be accessed cross-platform from Python, R, or any language with generic HDF5 libraries.

## When to use

Use when raw multidimensional LC-MS/LC-IM-MS data (e.g. .d, .raw) must be converted to an open format for feature engineering, machine learning, or visualization outside vendor Windows-bound software.

## When NOT to use

- Input is already an open-format file (.mza, or plain mzML/XML) that downstream tools can read directly without conversion
- The analysis requires vendor-native processing or the optimized parallel conversion speeds of ProteoWizard for mzXML/mzML/mz5 — the paper notes 'improvements in speed and file size are still needed'
- The workflow needs lossy compression or proteomics-specific packaging such as MS2AI, which discards information and does not support the ion mobility dimension

## Inputs

- Agilent .d files (with or without ion mobility)
- Bruker ion mobility .d files (TimsTOF)
- Thermo .raw files
- mzML files

## Outputs

- .mza file (HDF5-based: metadata table + Arrays_intensity/Arrays_mz groups, or Arrays_mzbin/Full_mz_array for IM spectra)
- Metadata table per spectrum with columns such as Scan, MSLevel, Activation, RetentionTime, PrecursorMonoisotopicMz, IonMobilityTime
- CCS calibration coefficients as HDF5 datasets (CCScalDT, CCScalSLIM, CCScalTIMS) when detected

## How to apply

Download the mza.exe executable from the PNNL-m-q/mza releases page; it is stand-alone and requires no dependencies on Windows 10. Convert a single file with 'mza -file <raw_file> -intensityThreshold 20', or batch-convert all .d/.raw/.mzML files in a directory by adding '-extension .d'. On Unix-like systems, either build the mzacontainer Docker image and run 'wine mza.exe' inside it, or install Wine (tested 6.0.3) plus wine-mono.msi (tested 7.3.0) and run mza through Wine directly. The -intensityThreshold parameter sets the minimum intensity that must be exceeded for signals to be included in the output .mza file. The output .mza file stores a metadata table plus jagged 1D arrays in the Arrays_intensity and Arrays_mz HDF5 groups (Arrays_mzbin/Full_mz_array for IM data), named by Scan value, and can be read with h5py in Python or rhdf5 in R.

## Related tools

- **MZA** (stand-alone command-line executable that converts vendor raw MS files to the HDF5-based .mza format) — https://github.com/PNNL-m-q/mza
- **HDF5** (binary hierarchical file format underlying the .mza file structure, enabling cross-platform, cross-language access) — https://github.com/PNNL-m-q/mza
- **Docker** (mzacontainer image for running wine mza.exe on Unix-like operating systems) — https://github.com/PNNL-m-q/mza
- **Wine** (compatibility layer (tested 6.0.3, with wine-mono 7.3.0) needed to run mza.exe on Ubuntu 22.04.1 without Docker) — https://github.com/PNNL-m-q/mza
- **h5py** (Python library for reading .mza HDF5 datasets in example access scripts) — https://github.com/PNNL-m-q/mza
- **rhdf5** (R package for reading .mza HDF5 datasets in example access scripts) — https://github.com/PNNL-m-q/mza
- **CCSz-demo** (example scripts converting IM-MS data to CCS/z space and visualizing mobility distributions from .mza files) — https://github.com/EMSL-Computing/CCSz-demo

## Examples

```
mza -file test_data\LowHigh_PC_160_180_frames1-10.d -intensityThreshold 20
```

## Evaluation signals

- The output .mza file contains a metadata table whose rows correspond 1:1 to spectra, with a unique Scan number per spectrum
- Arrays_intensity/<scan> and Arrays_mz/<scan> (or Arrays_mzbin/<scan> for IM) datasets exist in the HDF5 groups for every Scan in the metadata table
- No spectrum below the -intensityThreshold value (e.g. 20) appears in the stored intensity arrays; zero-intensity values are omitted
- Large files use partitioned groups (MzaPath index, e.g. Arrays_intensity1/630) rather than excessive datasets in a single group, preserving read performance
- Example Python/R scripts from the repo and the CCSz-demo repository successfully open the converted file and reproduce mobility-distribution figures

## Limitations

- Improvements in speed and file size are still needed; MZA was slower to convert than optimized parallel ProteoWizard conversions to mzXML, mzML, and mz5
- Ion mobility m/z dimension is stored as indexes (mzbins) into a shared Full_mz_array, which is specific to IM data and not a general spectral representation
- Having too many datasets within a single HDF5 group can slow data reading; large files require partitions (MzaPath index)
- No version number or changelog is stated for the MZA tool, complicating reproducibility
- IonMobilityTime units vary by platform: arrival time in milliseconds for DT and SLIM, 1/k0 in Vs/cm2 for TimsTOF

## Evidence
