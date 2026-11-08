---
name: mza-hdf5-raw-data-access
description: Use when multidimensional MS data have already been converted to the
  MZA format and you need to retrieve raw spectra (1D m/z and intensity arrays) by
  scan number directly through generic HDF5 libraries like h5py or rhdf5.
when_to_use_negative:
- The raw data are still in a proprietary vendor format (.raw, .d, mzML) and have
  not yet been converted to MZA—run the mza.exe converter first.
- The task requires higher-level processing such as calibration, signal processing,
  peak finding, or plotting—use mzapy's dedicated modules instead of bare HDF5 reads.
- You need validated DDA MS/MS extraction semantics (e.g., excluding erroneous scans
  via drop_scans)—use mzapy.dda.MsmsReaderDda rather than ad hoc HDF5 access.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mza
  role: Command-line converter that transforms vendor MS files (Agilent .d, Bruker
    .d, Thermo .raw, mzML) into the MZA HDF5 structure that this skill reads
  repo: https://github.com/PNNL-m-q/mza
- name: h5py
  role: Python HDF5 library used to open .mza files and read the metadata table, Arrays_mz,
    Arrays_intensity, Arrays_mzbin, and Full_mz_array datasets
  repo: https://github.com/PNNL-m-q/mza
- name: mzapy
  role: Python package providing an interface to unprocessed MS data in the MZA format
    as an alternative to direct h5py access
  repo: https://github.com/PNNL-m-q/mzapy
- name: rhdf5
  role: R package for generic HDF5 access to MZA files, mentioned alongside h5py in
    the README
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c01653
    title: '<i>mzapy</i>: An Open-Source Python Library Enabling Efficient Extraction
      and Processing of Ion Mobility Spectrometry-Mass Spectrometry Data in the MZA
      File Format'
  source_content_hash: sha256:6b03fa7ca8d7a6704e610be9f9f373cfa1342a1214496439243c09c17b2caad5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mza
  tools:
  - HDF5
  - MZA
  - Python
  - h5py
  - mza
  - mzapy
  - rhdf5
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# mza-hdf5-raw-data-access

## Summary

Directly reads unprocessed mass spectrometry spectra from MZA files—a simple HDF5-based data structure consisting of a metadata table plus per-scan 1D m/z and intensity arrays—using generic HDF5 libraries such as h5py (Python) or rhdf5 (R), without needing vendor-specific readers.

## When to use

Use when multidimensional MS data have already been converted to the MZA format and you need to retrieve raw spectra (1D m/z and intensity arrays) by scan number directly through generic HDF5 libraries like h5py or rhdf5.

## When NOT to use

- The raw data are still in a proprietary vendor format (.raw, .d, mzML) and have not yet been converted to MZA—run the mza.exe converter first.
- The task requires higher-level processing such as calibration, signal processing, peak finding, or plotting—use mzapy's dedicated modules instead of bare HDF5 reads.
- You need validated DDA MS/MS extraction semantics (e.g., excluding erroneous scans via drop_scans)—use mzapy.dda.MsmsReaderDda rather than ad hoc HDF5 access.

## Inputs

- MZA file (.mza) created by the mza.exe converter from Agilent .d, Bruker ion mobility .d, Thermo .raw, or mzML input
- HDF5 library (h5py for Python, rhdf5 for R) plus numpy and hdf5plugin for decompression
- Target scan number (and optionally MzaPath partition index) from the metadata table

## Outputs

- 1D m/z array for the requested scan (from Arrays_mz or via Arrays_mzbin indexed into Full_mz_array for IM spectra)
- 1D intensity array for the requested scan (from Arrays_intensity), with zero-intensity values omitted
- Metadata attributes for the scan (MSLevel, RetentionTime, Activation, CollisionEnergy, PrecursorMonoisotopicMz, IonMobilityTime, TIC, etc.)

## How to apply

Open the .mza file with h5py (or rhdf5 in R) and first read the metadata table, where each row is a spectrum with columns such as Scan, MSLevel, RetentionTime, PrecursorMonoisotopicMz, IonMobilityBin, and MzaPath. To fetch a spectrum, look up its Scan value and read the 1D datasets at Arrays_intensity/<Scan> (intensities) and Arrays_mz/<Scan> (m/z values); for ion mobility spectra the m/z dimension is stored as indices in Arrays_mzbin/<Scan> that map into the shared Full_mz_array dataset. Honor the MzaPath column when present—partitions move arrays into numbered groups (e.g., Arrays_intensity1/630) because too many datasets in a single HDF5 group slows reading. Note that spectra are stored omitting zero-intensity values, and IonMobilityBin=0 represents the total frame spectrum with the IM dimension summed away.

## Related tools

- **mza** (Command-line converter that transforms vendor MS files (Agilent .d, Bruker .d, Thermo .raw, mzML) into the MZA HDF5 structure that this skill reads) — https://github.com/PNNL-m-q/mza
- **h5py** (Python HDF5 library used to open .mza files and read the metadata table, Arrays_mz, Arrays_intensity, Arrays_mzbin, and Full_mz_array datasets) — https://github.com/PNNL-m-q/mza
- **mzapy** (Python package providing an interface to unprocessed MS data in the MZA format as an alternative to direct h5py access) — https://github.com/PNNL-m-q/mzapy
- **rhdf5** (R package for generic HDF5 access to MZA files, mentioned alongside h5py in the README)

## Examples

```
import h5py, numpy as np; f = h5py.File('file.mza','r'); mz = np.array(f['Arrays_mz/630']); inten = np.array(f['Arrays_intensity/630'])
```

## Evaluation signals

- The scan number queried exists both in the metadata table and as datasets under the (possibly partitioned) Arrays_intensity and Arrays_mz groups.
- For IM scans, Arrays_mzbin indices are valid indexes into Full_mz_array (all values fall within the length of Full_mz_array).
- Returned intensity arrays contain no zero-intensity values, matching MZA's convention of omitting them.
- Metadata attributes for the retrieved scan (e.g., MSLevel, RetentionTime in minutes, IonMobilityBin) are consistent with the spectrum arrays read from the same Scan value and MzaPath partition.
- The same read succeeds when repeated through an independent library (h5py vs. rhdf5), confirming format-generic access.

## Limitations

- Zero-intensity values are omitted from stored spectra, so array lengths reflect thresholded/filtered peaks rather than a full m/z grid.
- Files with very many spectra may be partitioned (MzaPath column), so group paths must be resolved per scan; too many datasets in one HDF5 group degrades reading performance.
- For ion mobility spectra the m/z dimension is stored as mzbins indexes into Full_mz_array rather than literal m/z values, requiring an extra mapping step.
- IonMobilityBin=0 stores the total frame spectrum (IM dimension summed), which may not be what frame-level analysis requires.

## Evidence

- [readme] MZA simple data structure based on the HDF5 format: [evidence span withheld — anchor status: external_doc]
- [readme] Generic HDF5 access to MZA: [evidence span withheld — anchor status: external_doc]
- [readme] MZA file structure: [evidence span withheld — anchor status: external_doc]
- [readme] Partitioning caveat: [evidence span withheld — anchor status: external_doc]
