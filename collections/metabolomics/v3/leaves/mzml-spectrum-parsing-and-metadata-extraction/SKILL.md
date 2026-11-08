---
name: mzml-spectrum-parsing-and-metadata-extraction
description: Use when converting vendor-raw or mzML MS data (Agilent .d, Bruker IM
  .d, Thermo .raw, mzML) into an HDF5-based structure so spectra can be read in Python/R
  without Windows-bound vendor libraries.
when_to_use_negative:
- Input is already an .mza file or any non-MS tabular feature table — the converter
  targets raw vendor files and mzML only
- The analysis requires lossless vendor-specific detail beyond the MZA schema (e.g.
  MS2AI-style proteomics-only pipelines or formats retaining nested vendor metadata
  not in the metadata table)
- Only a handful of spectra need inspection and writing XML-parsing code (e.g. direct
  pymzML access) is simpler than a full format conversion
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: pymzML
  role: Python library used to parse mzML spectra and extract metadata (scan number,
    MS level, activation, collision energy, retention time, IM arrival time) and m/z/intensity
    arrays
- name: h5py
  role: Python library used to write the .mza HDF5 file structure (metadata dataset,
    Arrays_intensity/Arrays_mz/Arrays_mzbin groups, Full_mz_array) and to read it
    back for validation
- name: MZA
  role: Stand-alone command-line converter executable producing the MZA HDF5 structure
    from vendor raw files and mzML
  repo: https://github.com/PNNL-m-q/mza
- name: ProteoWizard
  role: Toolkit referenced for converting vendor formats to mzML, the open input format
    this parsing path consumes
- name: rhdf5
  role: R package for accessing the converted .mza arrays from R scripts
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00313
    title: 'MZA: A Data Conversion Tool to Facilitate Software Development and Artificial
      Intelligence Research in Multidimensional Mass Spectrometry'
  source_content_hash: sha256:fe1bee34a81d947987ab362da521362bdee339fa4cfa5ac9d2d9f13c4761caca
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mza
  tools:
  - MZA
  - ProteoWizard
  - Python
  - h5py
  - mzML
  - pymzML
  - rhdf5
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# mzml-spectrum-parsing-and-metadata-extraction

## Summary

Parse mzML mass spectrometry files with pymzML and write the MZA HDF5 structure via h5py: a per-spectrum metadata table plus jagged 1D m/z and intensity arrays named by scan number, enabling cross-platform access to raw MS data from Python or R.

## When to use

Use when converting vendor-raw or mzML MS data (Agilent .d, Bruker IM .d, Thermo .raw, mzML) into an HDF5-based structure so spectra can be read in Python/R without Windows-bound vendor libraries.

## When NOT to use

- Input is already an .mza file or any non-MS tabular feature table — the converter targets raw vendor files and mzML only
- The analysis requires lossless vendor-specific detail beyond the MZA schema (e.g. MS2AI-style proteomics-only pipelines or formats retaining nested vendor metadata not in the metadata table)
- Only a handful of spectra need inspection and writing XML-parsing code (e.g. direct pymzML access) is simpler than a full format conversion

## Inputs

- mzML file (or Agilent .d, Bruker ion mobility .d, Thermo .raw vendor file)
- Optional parameters: -file (required), -extension (.mzML, .raw or .d for batch conversion), -intensityThreshold (minimum intensity for inclusion)

## Outputs

- .mza file (HDF5) containing: metadata table dataset with columns Scan, MzaPath, MSLevel, Polarity, Activation, CollisionEnergy, RetentionTime (minutes), precursor fields, TIC, IonMobilityFrame, IonMobilityBin, IonMobilityTime
- Arrays_intensity HDF5 group of 1D intensity datasets named by scan number (e.g. Arrays_intensity/630)
- Arrays_mz HDF5 group of 1D m/z datasets named by scan number (non-IM spectra)
- Arrays_mzbin HDF5 group of 1D m/z index datasets plus Full_mz_array dataset (IM spectra)
- Optional CCScalDT=[Tfix,Beta], CCScalSLIM=[C0..C3], CCScalTIMS=[C0..C9] HDF5 datasets

## How to apply

Read the input with pymzML, extracting per-spectrum scan number, MS level, polarity, activation type, collision energy, retention time, precursor fields, TIC, and ion mobility arrival time into a metadata table where each row is one spectrum. Extract each spectrum's m/z and intensity arrays and omit zero-intensity values so spectra are stored as two jagged 1D arrays. For ion mobility spectra, build a single shared Full_mz_array and store per-spectrum m/z bin indexes instead of raw m/z values, mapping IonMobilityBin=0 to the total frame spectrum. Write the metadata table as an HDF5 dataset and create Arrays_intensity, Arrays_mz (or Arrays_mzbin for IM), and Full_mz_array named by the 'Scan' value using h5py; partition into multiple groups (e.g. Arrays_intensity1/630) when a file has too many spectra, since HDF5 reads faster with many groups than many datasets in one group. Optionally apply an -intensityThreshold minimum-intensity cutoff to exclude signals below the threshold from the output .mza file, and embed CCS calibration coefficients (CCScalDT, CCScalSLIM, CCScalTIMS) as HDF5 datasets if detected.

## Related tools

- **pymzML** (Python library used to parse mzML spectra and extract metadata (scan number, MS level, activation, collision energy, retention time, IM arrival time) and m/z/intensity arrays)
- **h5py** (Python library used to write the .mza HDF5 file structure (metadata dataset, Arrays_intensity/Arrays_mz/Arrays_mzbin groups, Full_mz_array) and to read it back for validation)
- **MZA** (Stand-alone command-line converter executable producing the MZA HDF5 structure from vendor raw files and mzML) — https://github.com/PNNL-m-q/mza
- **ProteoWizard** (Toolkit referenced for converting vendor formats to mzML, the open input format this parsing path consumes)
- **rhdf5** (R package for accessing the converted .mza arrays from R scripts)

## Examples

```
mza -file test_data\LowHigh_PC_160_180_frames1-10.d -intensityThreshold 20
```

## Evaluation signals

- Read the .mza back with h5py and confirm the metadata table, the Arrays_intensity and Arrays_mz (or Arrays_mzbin + Full_mz_array) groups exist with the documented structure
- Confirm every 1D array dataset is named by its 'Scan' value from the metadata table (e.g. Arrays_intensity/630, Arrays_mz/630) and each metadata row maps to one spectrum
- Verify zero-intensity values are omitted and arrays are jagged 1D arrays of matching m/z and intensity lengths per scan
- For IM files, verify IonMobilityBin=0 corresponds to the total frame (summed) spectrum and stored m/z values are indexes into the shared Full_mz_array
- Spot-check retained intensities against the -intensityThreshold cutoff (e.g. all stored values exceed 20 when -intensityThreshold 20 was used)

## Limitations

- Improvements in conversion speed and output file size are still needed; no quantitative benchmarks of speed or file size are reported
- mzML's XML basis is inefficient for storing and accessing large amounts of spectra from multidimensional measurements, which motivates conversion but limits upstream parsing speed
- Having too many datasets within a single HDF5 group slows reading; large files require partitioned groups via the MzaPath column (e.g. Arrays_intensity1/630)
- The MZA repository provides no changelog and no stated version number for the tool
- mza.exe is Windows-native; on Unix-like systems it requires Wine or a Docker container (wine mza.exe ...)

## Evidence

- [methods] Parse the input mzML file with pymzML to extract each spectrum's scan number, MS level, activation type, collision energy, retention time, and ion mobility arrival time into a metadata table.: [evidence span withheld — anchor status: not_found]
