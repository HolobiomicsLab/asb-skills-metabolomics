---
name: mza-file-structure-validation
description: Reference for verifying the structure of an MZA file after conversion
  from Agilent .d, Bruker ion mobility .d, Thermo .raw, or mzML, before consuming
  it with h5py or rhdf5.
when_to_use_negative:
- Input is still a proprietary vendor raw file (Agilent .d, Bruker .d, Thermo .raw)
  or an mzML file that has not yet been converted to .mza
- The file is a generic HDF5 file from another tool with no MZA-specific groups (Arrays_intensity,
  Arrays_mz, Arrays_mzbin) or MZA metadata columns
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MZA
  role: Converter that produces the .mza file whose structure this skill validates
  repo: https://github.com/PNNL-m-q/mza
- name: h5py
  role: Python library used to open and inspect the HDF5-based .mza file
- name: rhdf5
  role: R package used to open and inspect the HDF5-based .mza file
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00313
    title: 'MZA: A Data Conversion Tool to Facilitate Software Development and Artificial
      Intelligence Research in Multidimensional Mass Spectrometry'
  source_content_hash: sha256:333a05c95253788689386669ea70500d0c4dd3b97bd2ccd04f1ea30bd42367ea
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mza
  tools:
  - MZA
  - Python
  - h5py
  - pymzML
  - rhdf5
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# mza-file-structure-validation

## Summary

Validates that a converted .mza (HDF5-based) mass spectrometry file contains the expected MZA layout — a metadata dataset plus Arrays_intensity/Arrays_mz (or Arrays_mzbin + Full_mz_array for ion mobility) groups whose 1D datasets are named by Scan value — so downstream Python/R tooling can rely on a well-formed structure.

## When to use

Reference for verifying the structure of an MZA file after conversion from Agilent .d, Bruker ion mobility .d, Thermo .raw, or mzML, before consuming it with h5py or rhdf5.

## When NOT to use

- Input is still a proprietary vendor raw file (Agilent .d, Bruker .d, Thermo .raw) or an mzML file that has not yet been converted to .mza
- The file is a generic HDF5 file from another tool with no MZA-specific groups (Arrays_intensity, Arrays_mz, Arrays_mzbin) or MZA metadata columns

## Inputs

- A .mza file produced by the MZA converter (HDF5 format)
- Expected metadata table column list (Scan, MzaPath, MSLevel, Polarity, Activation, CollisionEnergy, RetentionTime, PrecursorScan, PrecursorMonoisotopicMz, PrecursorCharge, IsolationWindowTargetMz, IsolationWindowLowerOffset, IsolationWindowUpperOffset, TIC, SpectrumTitle, IonMobilityFrame, IonMobilityBin, IonMobilityTime)

## Outputs

- Pass/fail verdict on whether the .mza file matches the expected MZA HDF5 schema
- Report of which groups (Arrays_intensity, Arrays_mz or Arrays_mzbin, Full_mz_array) and per-scan datasets are present and correctly shaped
- Confirmation that CCS calibration datasets (CCScalDT, CCScalSLIM, CCScalTIMS) are present when applicable

## How to apply

Open the .mza file with h5py (Python) or rhdf5 (R) and inspect its top-level contents. Confirm a metadata dataset exists where each row represents one spectrum and columns include Scan, MzaPath, MSLevel, Polarity, Activation, CollisionEnergy, RetentionTime, PrecursorScan, PrecursorMonoisotopicMz, PrecursorCharge, IsolationWindowTargetMz, IsolationWindowLowerOffset, IsolationWindowUpperOffset, TIC, SpectrumTitle, IonMobilityFrame, IonMobilityBin and IonMobilityTime. For non-IM data, check that the Arrays_intensity and Arrays_mz groups contain 1D datasets named by the Scan value (e.g. Arrays_intensity/630 and Arrays_mz/630), with spectra stored omitting zero-intensity values as jagged arrays. For ion mobility data, verify that Arrays_mzbin contains 1D index arrays pointing into a shared Full_mz_array dataset, treating IonMobilityBin = 0 as the total frame spectrum, and confirm the correct CCS calibration dataset (CCScalDT, CCScalSLIM or CCScalTIMS) is present if it was detected during conversion. If the file holds too many spectra, MZA may have created partition groups (e.g. Arrays_intensity1/630 when MzaPath=1) — validate against the MzaPath column rather than assuming a single group.

## Related tools

- **MZA** (Converter that produces the .mza file whose structure this skill validates) — https://github.com/PNNL-m-q/mza
- **h5py** (Python library used to open and inspect the HDF5-based .mza file)
- **rhdf5** (R package used to open and inspect the HDF5-based .mza file)

## Examples

```
python -c "import h5py; f=h5py.File('out.mza','r'); print(list(f.keys())); print(list(f['Arrays_intensity'].keys())[:5])"
```

## Evaluation signals

- Metadata dataset is present and its columns match the documented MZA schema (Scan, MSLevel, Polarity, Activation, RetentionTime, IonMobilityBin, etc.)
- Each metadata row's Scan value has a matching 1D dataset in Arrays_intensity and Arrays_mz (or Arrays_mzbin for IM data)
- For IM files, Full_mz_array exists and every Arrays_mzbin entry indexes into it; IonMobilityBin = 0 rows correspond to total frame spectra
- Stored intensity arrays contain no zero-intensity values (zero-intensity points were omitted during conversion)
- If MzaPath is non-empty (partitioned file), the corresponding partition group (e.g. Arrays_intensity1, Arrays_mzbin1) holds the scan-named datasets

## Limitations

- Too many datasets within a single group can slow HDF5 read performance, so large files may be partitioned — validation must handle partition groups rather than assuming a single flat group
- CCS calibration datasets (CCScalDT, CCScalSLIM, CCScalTIMS) are only included if detected during conversion, so their absence is not necessarily an error
- IonMobilityTime units differ by platform: arrival time in milliseconds for DT and SLIM, 1/k0 in Vs/cm2 for TimsTOF

## Evidence
