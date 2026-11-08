---
name: hdf5-spectra-storage
description: Use when raw LC/LC-IM-MS vendor files (.d, .raw, mzML) must be converted
  to a vendor-neutral format readable with generic HDF5 libraries (h5py, rhdf5) for
  downstream analysis or ML feature engineering.
when_to_use_negative:
- When the input already contains zero-intensity peaks that must be preserved verbatim,
  since MZA spectra are stored omitting zero-intensity values.
- When an existing mzML/mzXML/mz5/mzMLb workflow already meets file-size and speed
  requirements, since MZA files were larger than mz5 and mzMLb and slower than ProteoWizard's
  optimized parallel conversions.
- When the data is not multidimensional MS vendor data (e.g. already a feature table
  or a proteomics search result), since MZA's functionality targets raw multidimensional
  spectra.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MZA
  role: stand-alone command-line executable that converts vendor raw MS files into
    the HDF5-based .mza data structure
  repo: https://github.com/PNNL-m-q/mza
- name: h5py
  role: Python library for reading the .mza HDF5 datasets (metadata, intensity and
    m/z arrays) after conversion
- name: rhdf5
  role: R package used to access the .mza HDF5 groups and metadata table
- name: Docker
  role: containerizes the Wine-based execution of mza.exe on Unix-like systems
- name: Wine
  role: compatibility layer that lets the Windows-only MZA executable run on Ubuntu
    (tested on 6.0.3 with wine-mono 7.3.0)
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00313
    title: 'MZA: A Data Conversion Tool to Facilitate Software Development and Artificial
      Intelligence Research in Multidimensional Mass Spectrometry'
  source_content_hash: sha256:c7e9866c35aef7bfb009d4fd70042b06d954097b0c9febcb366986ee98e0d303
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mza
  tools:
  - Docker
  - HDF5
  - MZA
  - Wine
  - h5py
  - mz5
  - mzMLb
  - mzXML
  - rhdf5
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# hdf5-spectra-storage

## Summary

Converts raw multidimensional mass spectrometry data into a flat, HDF5-based MZA file structure (metadata table + jagged m/z and intensity arrays) that can be read cross-platform from Python or R without vendor libraries.

## When to use

Use when raw LC/LC-IM-MS vendor files (.d, .raw, mzML) must be converted to a vendor-neutral format readable with generic HDF5 libraries (h5py, rhdf5) for downstream analysis or ML feature engineering.

## When NOT to use

- When the input already contains zero-intensity peaks that must be preserved verbatim, since MZA spectra are stored omitting zero-intensity values.
- When an existing mzML/mzXML/mz5/mzMLb workflow already meets file-size and speed requirements, since MZA files were larger than mz5 and mzMLb and slower than ProteoWizard's optimized parallel conversions.
- When the data is not multidimensional MS vendor data (e.g. already a feature table or a proteomics search result), since MZA's functionality targets raw multidimensional spectra.

## Inputs

- Agilent '.d' file (with or without ion mobility)
- Bruker ion mobility 'd' file (with ion mobility)
- Thermo '.raw' file
- mzML file

## Outputs

- .mza file (HDF5-based) with metadata table, Arrays_intensity group, Arrays_mz group, and for IM data Arrays_mzbin group plus Full_mz_array
- Optional CCScalDT / CCScalSLIM / CCScalTIMS HDF5 datasets embedded when CCS calibration coefficients are detected

## How to apply

Run the MZA executable with '-file <raw_file>' plus optional '-intensityThreshold' (e.g. 20) to convert Agilent '.d', Bruker ion-mobility 'd', Thermo '.raw', or mzML inputs into a single .mza file. On Unix-like systems either build the Docker image and run 'wine mza.exe' with WINEDEBUG=-all, or install Wine 6.0.3 plus wine-mono 7.3.0 and invoke mza directly; on Windows the executable runs natively. The resulting .mza file contains one metadata table row per spectrum (scan number, MS level, activation type, retention time, ion mobility arrival time, precursor fields) and jagged 1D arrays grouped under Arrays_intensity and Arrays_mz named by the spectrum's Scan value; for IM data, Arrays_mzbin stores indices into a shared Full_mz_array, and IonMobilityBin=0 marks summed frame spectra. Zero-intensity values are omitted and spectra below the intensity threshold are excluded, so conversion is lossless with respect to nonzero peaks. CCS calibration coefficients (CCScalDT, CCScalSLIM, CCScalTIMS) are stored as HDF5 datasets when detected during conversion.

## Related tools

- **MZA** (stand-alone command-line executable that converts vendor raw MS files into the HDF5-based .mza data structure) — https://github.com/PNNL-m-q/mza
- **h5py** (Python library for reading the .mza HDF5 datasets (metadata, intensity and m/z arrays) after conversion)
- **rhdf5** (R package used to access the .mza HDF5 groups and metadata table)
- **Docker** (containerizes the Wine-based execution of mza.exe on Unix-like systems)
- **Wine** (compatibility layer that lets the Windows-only MZA executable run on Ubuntu (tested on 6.0.3 with wine-mono 7.3.0))

## Examples

```
wine mza -file test_data\LowHigh_PC_160_180_frames1-10.d -intensityThreshold 20
```

## Evaluation signals

- Every spectrum in the output .mza metadata table has a unique Scan value and corresponding Arrays_intensity/<Scan> and Arrays_mz/<Scan> (or Arrays_mzbin/<Scan>) HDF5 datasets.
- Metadata columns (Scan, MSLevel, Polarity, Activation, RetentionTime, IonMobilityTime, etc.) are populated and RetentionTime is in minutes, IonMobilityTime in ms for DT/SLIM or 1/k0 for TimsTOF.
- For IM data, IonMobilityBin=0 rows represent the summed frame spectrum and m/z values are stored as indices into a single shared Full_mz_array.
- No spectra below the '-intensityThreshold' cutoff appear in the output arrays; zero-intensity values are absent from every stored array.
- If the raw file contained CCS calibration data, CCScalDT/CCScalSLIM/CCScalTIMS datasets are present with the expected coefficient counts (2 for DT, 4 for SLIM, 10 for TimsTOF).

## Limitations

- Conversion speed and output file size still lag ProteoWizard's optimized parallel mzML/mzXML/mz5 conversions and mzMLb, respectively — improvements are explicitly needed.
- Files with too many spectra are partitioned into multiple HDF5 groups (MzaPath index), so array access must respect the MzaPath column rather than a single group path.
- Requires Wine (plus wine-mono) or Docker on Unix-like systems because the executable is a Windows binary; native Windows execution is limited to Windows 10 testing.
- Ion mobility m/z dimension is stored as indices into Full_mz_array, so downstream code must resolve bin indices before interpreting m/z values.

## Evidence
