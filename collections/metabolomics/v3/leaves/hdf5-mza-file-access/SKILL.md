---
name: hdf5-mza-file-access
description: Use when raw MS/IMS data has been converted to MZA (HDF5) and you need
  programmatic access to spectra, metadata headers, or ion mobility arrival-time arrays
  from Python or R.
when_to_use_negative:
- Data is still in a vendor format (Thermo .raw, Agilent .d, Bruker .d) or mzML —
  it must first be converted to MZA with the mza executable.
- You only need derived results (e.g., a feature table or calibrated peak list); mzapy's
  higher-level modules (XIC collection, peak finding, calibration) are a better fit
  than raw HDF5 access.
- The file is a generic HDF5 file not conforming to the MZA structure — the Arrays_intensity/Arrays_mz/Scan
  conventions will not match.
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: mzapy
  role: Python package providing an interface to unprocessed MS data in the MZA format,
    including caching metadata headers and collecting arrays
  repo: https://github.com/PNNL-m-q/mzapy
- name: MZA
  role: stand-alone command-line converter that transforms vendor MS files into the
    MZA HDF5 structure this skill reads
  repo: https://github.com/PNNL-m-q/mza
- name: h5py
  role: generic Python HDF5 library for direct access to the metadata dataset and
    Arrays_* groups
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c01653
    title: '<i>mzapy</i>: An Open-Source Python Library Enabling Efficient Extraction
      and Processing of Ion Mobility Spectrometry-Mass Spectrometry Data in the MZA
      File Format'
  source_content_hash: sha256:4fa1220f56b8d6ad0f27b06e524c58e409848be3e4eb4e3772762f0453891869
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mzapy
  tools:
  - MZA
  - h5py
  - mzapy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# hdf5-mza-file-access

## Summary

Reads multidimensional mass spectrometry data stored in the MZA format — a simple HDF5-based structure holding a metadata table plus per-spectrum 1D m/z and intensity arrays — either directly with generic HDF5 libraries (h5py, rhdf5) or via the mzapy package's MZA interface, which also handles the ion mobility spectrometry (IMS) dimension.

## When to use

Use when raw MS/IMS data has been converted to MZA (HDF5) and you need programmatic access to spectra, metadata headers, or ion mobility arrival-time arrays from Python or R.

## When NOT to use

- Data is still in a vendor format (Thermo .raw, Agilent .d, Bruker .d) or mzML — it must first be converted to MZA with the mza executable.
- You only need derived results (e.g., a feature table or calibrated peak list); mzapy's higher-level modules (XIC collection, peak finding, calibration) are a better fit than raw HDF5 access.
- The file is a generic HDF5 file not conforming to the MZA structure — the Arrays_intensity/Arrays_mz/Scan conventions will not match.

## Inputs

- MZA file (HDF5 format, .mza) produced by the mza converter from Thermo .raw, Agilent .d, Bruker ion mobility .d, or mzML
- Optional: list of metadata headers to cache (for mzapy.MZA cache_metadata_headers)

## Outputs

- Metadata table rows (Scan, MSLevel, Polarity, Activation, CollisionEnergy, RetentionTime, PrecursorScan, PrecursorMonoisotopicMz, IonMobilityFrame, IonMobilityBin, IonMobilityTime, etc.)
- 1D m/z arrays per scan (from Arrays_mz or Arrays_mzbin + Full_mz_array for IM spectra)
- 1D intensity arrays per scan (from Arrays_intensity)
- Optional CCS calibration coefficient datasets embedded at conversion (CCScalDT, CCScalSLIM, CCScalTIMS)

## How to apply

Open the .mza file as HDF5: either with mzapy's MZA reader (mzapy.MZA, optionally restricting cache_metadata_headers, e.g. passing [] for minimal memory footprint) or generically with h5py in Python / rhdf5 in R. Read the metadata table, where each row is a spectrum keyed by its unique Scan value and columns such as MSLevel, RetentionTime (minutes), PrecursorMonoisotopicMz, and IonMobilityTime describe the spectrum. Retrieve intensity values from the Arrays_intensity group and m/z values from the Arrays_mz group using the Scan value as the dataset name (e.g. Arrays_intensity/630); spectra are stored omitting zero-intensity values as jagged arrays. For ion-mobility spectra, m/z is stored as indexes (Arrays_mzbin) pointing into the shared Full_mz_array dataset, and IonMobilityBin=0 denotes the total frame spectrum with the IM dimension ignored. Close the file handle when done and respect MzaPath partitions if the file was split into multiple groups.

## Related tools

- **mzapy** (Python package providing an interface to unprocessed MS data in the MZA format, including caching metadata headers and collecting arrays) — https://github.com/PNNL-m-q/mzapy
- **MZA** (stand-alone command-line converter that transforms vendor MS files into the MZA HDF5 structure this skill reads) — https://github.com/PNNL-m-q/mza
- **h5py** (generic Python HDF5 library for direct access to the metadata dataset and Arrays_* groups)

## Examples

```
import mzapy; mza = mzapy.MZA('sample.mza', cache_metadata_headers=[])
```

## Evaluation signals

- Metadata table loads and contains the documented columns (Scan, MSLevel, RetentionTime, PrecursorMonoisotopicMz, IonMobilityTime); Scan values are unique per spectrum.
- For a given scan (e.g., Scan=630), the HDF5 datasets Arrays_intensity/630 and Arrays_mz/630 (or Arrays_mzbin/630 for IM scans) both exist and have equal lengths.
- Arrays contain no zero-intensity values, since MZA stores spectra omitting zero-intensity points as jagged arrays.
- For IM scans, m/z indexes in Arrays_mzbin fall within the bounds of the shared Full_mz_array dataset; IonMobilityBin=0 scans represent summed frame spectra.
- If the file uses partitions, MzaPath is non-empty and arrays are read from the corresponding partitioned group (e.g., Arrays_intensity1/630).

## Limitations

- Files with too many spectra may be partitioned into multiple groups (MzaPath column); reading without honoring the partition slows performance or fails to find datasets.
- For ion-mobility spectra the m/z dimension is stored as indexes into Full_mz_array rather than direct m/z values, so consumers must map indexes to the shared grid.
- IonMobilityTime units vary by instrument: arrival time in milliseconds for DT and SLIM, but 1/k0 in Vs/cm2 for TimsTOF.
- IonMobilityBin=0 rows represent total frame (summed) spectra, so they must be excluded when the IM dimension is required.

## Evidence

- [readme] MZA structure basics: [evidence span withheld — anchor status: external_doc]
- [readme] jagged array storage: [evidence span withheld — anchor status: external_doc]
- [readme] partitioning rationale: [evidence span withheld — anchor status: external_doc]
