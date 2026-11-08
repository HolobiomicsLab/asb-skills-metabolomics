---
name: hdf5-jagged-array-storage-design
description: Use when converting multidimensional MS data (LC-MS, LC-IM-MS with DIA/alternative
  activation) from vendor raw formats or mzML into a language-agnostic binary structure
  so spectra can be read in Python or R without vendor libraries.
when_to_use_negative:
- The input is already an .mza file or another efficiently indexed binary format —
  reconversion wastes time and file size, both of which the authors note still need
  improvement
- The downstream workflow requires the lossless full XML provenance or vendor-specific
  metadata of mzML rather than a simplified metadata table
- The target is proteomics-specific AI pipelines already served by MS2AI, which the
  authors describe as limited to LC-MS proteomics with lossy representations
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: MZA
  role: Stand-alone converter executable that writes the described HDF5 file structure
    from vendor raw formats or mzML
  repo: https://github.com/PNNL-m-q/mza
- name: h5py
  role: Python library used to read back and validate the metadata table, Arrays_intensity/Arrays_mz
    groups, and scan-named 1D datasets
- name: rhdf5
  role: R package used to access .mza arrays from R scripts
- name: pymzML
  role: Library for parsing mzML input spectra, their metadata, and m/z–intensity
    arrays during conversion
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00313
    title: 'MZA: A Data Conversion Tool to Facilitate Software Development and Artificial
      Intelligence Research in Multidimensional Mass Spectrometry'
  source_content_hash: sha256:4cb1684d97c01284d1d3cde18580bcde3b56b3d6bc425ee1dfb3fa95ac0d6bce
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mza
  tools:
  - HDF5
  - MZA
  - Python
  - h5py
  - pymzML
  - rhdf5
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# hdf5-jagged-array-storage-design

## Summary

Design of a simple HDF5-based file structure (as in MZA) that stores MS spectra as a metadata table plus per-spectrum 1D jagged arrays (intensities and m/z) in HDF5 groups, enabling cross-platform, cross-language access with generic libraries like h5py and rhdf5.

## When to use

Use when converting multidimensional MS data (LC-MS, LC-IM-MS with DIA/alternative activation) from vendor raw formats or mzML into a language-agnostic binary structure so spectra can be read in Python or R without vendor libraries.

## When NOT to use

- The input is already an .mza file or another efficiently indexed binary format — reconversion wastes time and file size, both of which the authors note still need improvement
- The downstream workflow requires the lossless full XML provenance or vendor-specific metadata of mzML rather than a simplified metadata table
- The target is proteomics-specific AI pipelines already served by MS2AI, which the authors describe as limited to LC-MS proteomics with lossy representations

## Inputs

- Vendor raw MS files (Agilent '.d', Bruker TIMS '.d', Thermo '.raw') or mzML
- Per-spectrum m/z and intensity arrays extracted with pymzML
- Optional intensity threshold (-intensityThreshold, e.g. 20) to filter low-abundance signals

## Outputs

- .mza HDF5 file: metadata table dataset with columns Scan, MzaPath, MSLevel, Polarity, Activation, CollisionEnergy, RetentionTime, precursor/isolation fields, TIC, IonMobilityFrame/Bin/Time
- Arrays_intensity HDF5 group of 1D datasets named by scan number
- Arrays_mz group of 1D datasets named by scan number (or Arrays_mzbin group of index arrays plus Full_mz_array for IM spectra)
- Optional CCScalDT, CCScalSLIM, CCScalTIMS datasets when CCS calibration coefficients are detected

## How to apply

Parse each input spectrum (e.g., with pymzML from mzML) to build a metadata table where each row holds per-spectrum properties such as Scan, MSLevel, Activation, CollisionEnergy, RetentionTime, IonMobilityTime. Write the metadata as an HDF5 dataset and create two groups, Arrays_intensity and Arrays_mz, each holding a 1D dataset named by the scan's 'Scan' value; spectra are stored as jagged arrays omitting zero-intensity values. For ion mobility spectra, build a single shared Full_mz_array and store per-spectrum indexes in an Arrays_mzbin group instead of raw m/z values, mapping IonMobilityBin=0 to the total frame spectrum. If the file has too many spectra, partition arrays across multiple groups (e.g., Arrays_intensity1/630 for MzaPath=1) because HDF5 reads more efficiently from multiple groups than from one group containing many datasets. Validate by reading the .mza back with h5py and confirming the metadata table, group structure, and scan-named dataset shapes.

## Related tools

- **MZA** (Stand-alone converter executable that writes the described HDF5 file structure from vendor raw formats or mzML) — https://github.com/PNNL-m-q/mza
- **h5py** (Python library used to read back and validate the metadata table, Arrays_intensity/Arrays_mz groups, and scan-named 1D datasets)
- **rhdf5** (R package used to access .mza arrays from R scripts)
- **pymzML** (Library for parsing mzML input spectra, their metadata, and m/z–intensity arrays during conversion)

## Examples

```
mza -file test_data\LowHigh_PC_160_180_frames1-10.d -intensityThreshold 20
```

## Evaluation signals

- Reading the .mza back with h5py returns the metadata table, the Arrays_intensity and Arrays_mz (or Arrays_mzbin) groups, and 1D datasets named by the metadata 'Scan' values
- Each metadata row corresponds to exactly one spectrum and its scan-named datasets in both groups have matching shapes (jagged, zero-intensity values removed)
- For IM spectra, Arrays_mzbin indexes are within [0, len(Full_mz_array)-1] and IonMobilityBin=0 rows correspond to the total frame spectrum
- Scan=630 (per README example) resolves to Arrays_intensity/630 and Arrays_mz/630, or Arrays_intensity1/630 and Arrays_mzbin1/630 when MzaPath=1
- Metadata units hold: RetentionTime in minutes, IonMobilityTime in ms (DT/SLIM) or 1/k0 in Vs/cm2 (TimsTOF)

## Limitations

- Files with too many spectra in a single HDF5 group slow reading; partitions (MzaPath index) must be created for large files
- Improvements in speed and file size are still needed for MZA
- IM spectra store m/z as indexes to a shared Full_mz_array, so the structure assumes a common m/z axis across spectra in the file

## Evidence

- [methods] validation workflow: [evidence span withheld — anchor status: not_found]
