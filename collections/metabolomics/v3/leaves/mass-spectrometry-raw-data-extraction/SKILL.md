---
name: mass-spectrometry-raw-data-extraction
description: Use when raw MS data stored in the MZA format (HDF5-based, converted
  from vendor .d/.raw/mzML via the mza converter) must be read into numpy arrays,
  especially multidimensional IMS-MS data.
when_to_use_negative:
- The input is still in a proprietary vendor format (Thermo .raw, Agilent/Bruker .d)
  or mzML that has not yet been converted to MZA — convert with the mza tool first.
- The analysis operates on already-processed feature tables or peak-picked results
  rather than unprocessed (raw) spectral data, which is mzapy's declared scope (an
  interface to unprocessed MS data).
- The data is plain LC-MS without an IMS dimension and an existing LC-MS tool with
  well-established algorithms already covers the workflow.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mzapy
  role: Python package providing the interface to unprocessed MS data in MZA format;
    used to open files, read cached metadata, and collect MS1/XIC/ATD/MS-MS arrays
  repo: https://github.com/PNNL-m-q/mzapy
- name: MZA (mza converter)
  role: Stand-alone command-line executable that converts vendor raw MS files into
    the MZA HDF5 data structure consumed by mzapy
  repo: https://github.com/PNNL-m-q/mza
- name: h5py
  role: Underlying HDF5 access dependency mzapy relies on to read MZA files
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c01653
    title: '<i>mzapy</i>: An Open-Source Python Library Enabling Efficient Extraction
      and Processing of Ion Mobility Spectrometry-Mass Spectrometry Data in the MZA
      File Format'
  source_content_hash: sha256:be3097aad43faec3d4eeda50913c1172a56546dc4eb2578deb6f75330651c6cc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mzapy
  tools:
  - MZA
  - MZA (mza converter)
  - h5py
  - mzapy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# mass-spectrometry-raw-data-extraction

## Summary

Extract unprocessed mass spectrometry data — including the ion mobility spectrometry (IMS) dimension — from MZA (HDF5) files using the mzapy Python package, retaining m/z, intensity, retention time, and drift/arrival time arrays for target precursors or extracted ion chromatograms.

## When to use

Use when raw MS data stored in the MZA format (HDF5-based, converted from vendor .d/.raw/mzML via the mza converter) must be read into numpy arrays, especially multidimensional IMS-MS data.

## When NOT to use

- The input is still in a proprietary vendor format (Thermo .raw, Agilent/Bruker .d) or mzML that has not yet been converted to MZA — convert with the mza tool first.
- The analysis operates on already-processed feature tables or peak-picked results rather than unprocessed (raw) spectral data, which is mzapy's declared scope (an interface to unprocessed MS data).
- The data is plain LC-MS without an IMS dimension and an existing LC-MS tool with well-established algorithms already covers the workflow.

## Inputs

- MZA file (HDF5-based) with metadata table plus Arrays_intensity, Arrays_mz, Arrays_mzbin, Full_mz_array groups
- Vendor raw MS file (Agilent '.d', Bruker ion mobility '.d', Thermo '.raw', mzML) already converted to MZA
- Target precursor identifiers (precursor m/z, scan numbers, retention time window)

## Outputs

- numpy arrays or DataFrames of m/z, intensity, retention time, and drift/arrival time (collect_ms1_arrays_by_rt, collect_xic_arrays_by_mz, collect_atd_arrays_by_rt_mz)
- MS/MS (MS2) spectra for DDA precursors via MsmsReaderDda.get_msms_spectrum
- Serialized arrays (e.g., extracted_precursor_arrays.npz with mz, i, rt, dt fields)

## How to apply

First convert vendor data (Agilent '.d', Bruker IM '.d', Thermo '.raw', mzML) to MZA with the mza executable, optionally applying an -intensityThreshold. In Python, open the MZA (HDF5) file with mzapy.MZA.__init__, optionally setting cache_metadata_headers (pass [] for the smallest memory footprint). Read the cached metadata table (columns such as Scan, MSLevel, RetentionTime, PrecursorMonoisotopicMz, IonMobilityTime) to locate scan numbers for the target precursor; drop erroneous scans via mzapy.dda.MsmsReaderDda if needed. Extract MS1 arrays with collect_ms1_arrays_by_rt / collect_ms1_df_by_rt, XICs with collect_xic_arrays_by_mz, arrival time distributions with collect_atd_arrays_by_rt_mz, or DDA MS/MS spectra with mzapy.dda.MsmsReaderDda.get_msms_spectrum — note IM spectra store m/z as mzbins indexes into Full_mz_array, and IonMobilityBin=0 is the total frame spectrum. Close the file handle with mzapy.MZA.close and serialize the arrays.

## Related tools

- **mzapy** (Python package providing the interface to unprocessed MS data in MZA format; used to open files, read cached metadata, and collect MS1/XIC/ATD/MS-MS arrays) — https://github.com/PNNL-m-q/mzapy
- **MZA (mza converter)** (Stand-alone command-line executable that converts vendor raw MS files into the MZA HDF5 data structure consumed by mzapy) — https://github.com/PNNL-m-q/mza
- **h5py** (Underlying HDF5 access dependency mzapy relies on to read MZA files)

## Examples

```
mza -file test_data/LowHigh_PC_160_180_frames1-10.d -intensityThreshold 20
```

## Evaluation signals

- The MZA file opens with mzapy.MZA and metadata rows map scan numbers to the expected MSLevel, RetentionTime, and IonMobilityTime columns.
- Extracted arrays contain all expected dimensions (m/z, intensity, rt, and drift/arrival time where IMS data is present) and span the precursor's scan range.
- For IM spectra, m/z values resolve through Arrays_mzbin indexes into the shared Full_mz_array, and IonMobilityBin=0 entries correspond to summed frame spectra.
- Zero-intensity values are absent from extracted arrays, consistent with the MZA storage convention of omitting them.
- The file handle is closed with mzapy.MZA.close and the serialized .npz loads without error and contains the named fields.

## Limitations

- Ion mobility spectra store the m/z dimension as mzbins indexes into a single Full_mz_array rather than direct m/z values, requiring an extra resolution step.
- Files with too many spectra may be partitioned (MzaPath column), so array access must go through the correct partition group (e.g., Arrays_intensity1/630).
- IonMobilityBin = 0 represents the total frame spectrum (IM dimension ignored) and must be excluded if per-arrival-time data is required.
- Analyzing IMS data has been challenging and limited the full utility of these measurements; incorporating the additional IMS dimension requires upgrading existing computational pipelines.

## Evidence

- [readme] MZA is a stand-alone and self-contained command-line executable which converts multidimensional mass spectrometry (MS) data from files in proprietary vendor formats to the MZA simple data structure based on the HDF5 format.: [evidence span withheld — anchor status: external_doc]
- [readme] For IM spectra the m/z dimension is stored as indexes (mzbins): [evidence span withheld — anchor status: external_doc]
