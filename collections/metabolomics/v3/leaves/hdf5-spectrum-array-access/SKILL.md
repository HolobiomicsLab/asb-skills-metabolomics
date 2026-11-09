---
name: hdf5-spectrum-array-access
description: Use when raw MS data has been converted to .mza format and a script must
  read individual spectra (m/z, intensity, IM arrival time, retention time) directly
  from the HDF5 structure in Python or R.
when_to_use_negative:
- The raw file has not yet been converted — run MZA data conversion first; this skill
  only reads .mza files, not vendor .d/.raw or mzML inputs directly
- Downstream analysis expects a feature table or processed peak matrix rather than
  raw per-spectrum jagged arrays
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: MZA
  role: Defines the HDF5 file structure (.mza) whose metadata table and per-spectrum
    array groups this skill reads
  repo: https://github.com/PNNL-m-q/mza
- name: h5py
  role: Python HDF5 library used to open .mza files and access metadata and array
    datasets
  repo: https://github.com/PNNL-m-q/mza
- name: rhdf5
  role: R Bioconductor HDF5 package used for equivalent .mza access in R scripts
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00313
    title: 'MZA: A Data Conversion Tool to Facilitate Software Development and Artificial
      Intelligence Research in Multidimensional Mass Spectrometry'
  source_content_hash: sha256:e705b69668eb8493953af008d69bc450f7e80b32745b36385f83b83707882385
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mza
  tools:
  - HDF5
  - MZA
  - Python
  - h5py
  - rhdf5
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# hdf5-spectrum-array-access

## Summary

Access raw mass-spectrometry spectra stored in MZA's HDF5 (.mza) file structure by reading the metadata table plus per-spectrum 1D arrays (Arrays_intensity, Arrays_mz, or Arrays_mzbin with Full_mz_array) via generic HDF5 libraries like h5py or rhdf5. This loose-coupling access pattern bypasses vendor Windows-bound libraries and mzML's inefficient XML so Python/R scripts can retrieve spectra directly.

## When to use

Use when raw MS data has been converted to .mza format and a script must read individual spectra (m/z, intensity, IM arrival time, retention time) directly from the HDF5 structure in Python or R.

## When NOT to use

- The raw file has not yet been converted — run MZA data conversion first; this skill only reads .mza files, not vendor .d/.raw or mzML inputs directly
- Downstream analysis expects a feature table or processed peak matrix rather than raw per-spectrum jagged arrays

## Inputs

- .mza HDF5 file produced by MZA conversion from Agilent .d, Bruker .d, Thermo .raw, or mzML
- Optional selection criteria on metadata columns (e.g., Scan, MSLevel, RetentionTime in minutes, IonMobilityTime in ms)

## Outputs

- Per-spectrum 1D intensity arrays (from Arrays_intensity)
- Per-spectrum 1D m/z arrays (from Arrays_mz) or m/z-bin indexes plus Full_mz_array (for IM spectra)
- Spectrum metadata table (Scan, MSLevel, Polarity, Activation, RetentionTime, IonMobilityTime, TIC, etc.)

## How to apply

Open the .mza file with h5py (Python) or rhdf5 (R) and read the metadata table where each row is a spectrum with Scan, MSLevel, RetentionTime (minutes), IonMobilityTime (ms for DT/SLIM), etc.; use the Scan value to locate the corresponding datasets. Retrieve the intensity array from the Arrays_intensity HDF5 group (e.g., Arrays_intensity/630 for Scan 630) and the m/z values either from Arrays_mz or, for IM spectra, from Arrays_mzbin indexes resolved against the shared Full_mz_array dataset. If MzaPath is a partition index (files with too many spectra are partitioned into multiple groups), access arrays as Arrays_intensity1/630, Arrays_mzbin1/630. Respect the conventions that zero-intensity values are omitted (jagged arrays) and IonMobilityBin = 0 marks the total frame spectrum that ignores the IM dimension; CCS calibration coefficients, when present, are HDF5 datasets (CCScalDT, CCScalSLIM, CCScalTIMS).

## Related tools

- **MZA** (Defines the HDF5 file structure (.mza) whose metadata table and per-spectrum array groups this skill reads) — https://github.com/PNNL-m-q/mza
- **h5py** (Python HDF5 library used to open .mza files and access metadata and array datasets) — https://github.com/PNNL-m-q/mza
- **rhdf5** (R Bioconductor HDF5 package used for equivalent .mza access in R scripts)

## Examples

```
import h5py, hdf5plugin; f = h5py.File('file.mza','r'); mz = f['Full_mz_array'][:][f['Arrays_mzbin/630'][:]]; inten = f['Arrays_intensity/630'][:]
```

## Evaluation signals

- Every Scan value in the metadata table resolves to an intensity dataset named Arrays_intensity/<Scan> (and Arrays_mz or Arrays_mzbin/<Scan>) — missing datasets indicate wrong partition or group naming
- Length of each intensity array matches the length of its paired m/z or mzbin array (jagged arrays with zero-intensity values omitted)
- For IM spectra, mzbins are indexes into the shared Full_mz_array dataset; resolved m/z values fall within the Full_mz_array range
- Metadata columns carry expected units: RetentionTime in minutes, IonMobilityTime in milliseconds for DT/SLIM (1/k0 in Vs/cm2 for TimsTOF)
- Partitioned files: arrays must be accessed with the MzaPath index (e.g., Arrays_intensity1/630 when MzaPath=1)

## Limitations

- Files with too many spectra are partitioned into multiple groups; storing many datasets in one group slows HDF5 reading, so the MzaPath partition index must be honored
- Spectra are stored omitting zero-intensity values as jagged arrays, so retrieved arrays must be paired with their m/z values — array lengths vary per spectrum
- IonMobilityBin = 0 datasets represent the total frame spectrum (IM dimension ignored), and must not be treated as individual IM bins

## Evidence
