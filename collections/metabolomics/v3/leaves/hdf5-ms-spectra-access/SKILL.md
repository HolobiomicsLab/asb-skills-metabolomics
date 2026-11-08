---
name: hdf5-ms-spectra-access
description: Use when a workflow needs programmatic access to raw LC-MS/LC-IM-MS spectra
  (e.g., XIC extraction, mobility plots, ML feature engineering) and vendor raw-file
  libraries are unavailable, Windows-bound, or too slow to iterate against.
when_to_use_negative:
- The task only needs peak lists or identifications already available from a search
  engine — reading raw spectra arrays adds conversion overhead without benefit
- 'Fast interactive XIC review against vendor software is required and the data are
  small: the article reports MZA access took 2.14 min for four-isotope XICs of VLEEANQAINPK
  across a 60 min LC-SLIM file versus 13.24 s via the vendor library'
- The downstream tool requires mzML/XML semantics or vendor-specific metadata not
  represented in the MZA metadata table
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: MZA
  role: Command-line converter producing the HDF5 (.mza) file structure and the access
    paradigm this skill follows
  repo: https://github.com/PNNL-m-q/mza
- name: h5py
  role: Python library used to open .mza files and read metadata and per-scan jagged
    arrays (with hdf5plugin registered)
  repo: https://github.com/PNNL-m-q/mza
- name: rhdf5
  role: R package for the same HDF5 access path in R example scripts
- name: hdf5plugin
  role: Registers HDF5 compression filters so compressed datasets in .mza files decode
    in h5py
- name: numpy
  role: Sum/filter intensities within m/z tolerance windows and assemble traces
- name: matplotlib
  role: Plot XICs and mobility distributions from retrieved arrays
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00313
    title: 'MZA: A Data Conversion Tool to Facilitate Software Development and Artificial
      Intelligence Research in Multidimensional Mass Spectrometry'
  source_content_hash: sha256:4e29108f38343f2c237ac6b14facd762c51a51ac312abe3b87d03cf9870d506a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mza
  tools:
  - HDF5
  - MZA
  - Python
  - h5py
  - hdf5plugin
  - matplotlib
  - numpy
  - rhdf5
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# hdf5-ms-spectra-access

## Summary

Read raw mass spectrometry spectra (MS1/MS2, LC and ion-mobility) directly from MZA's HDF5-based file structure in Python or R, using a metadata table plus jagged 1D intensity and m/z arrays per scan, enabling cross-platform access without vendor libraries.

## When to use

Use when a workflow needs programmatic access to raw LC-MS/LC-IM-MS spectra (e.g., XIC extraction, mobility plots, ML feature engineering) and vendor raw-file libraries are unavailable, Windows-bound, or too slow to iterate against.

## When NOT to use

- The task only needs peak lists or identifications already available from a search engine — reading raw spectra arrays adds conversion overhead without benefit
- Fast interactive XIC review against vendor software is required and the data are small: the article reports MZA access took 2.14 min for four-isotope XICs of VLEEANQAINPK across a 60 min LC-SLIM file versus 13.24 s via the vendor library
- The downstream tool requires mzML/XML semantics or vendor-specific metadata not represented in the MZA metadata table

## Inputs

- MZA-converted .mza file (HDF5 structure: metadata table + Arrays_intensity, Arrays_mz, Arrays_mzbin, Full_mz_array groups)
- Vendor raw MS file (Agilent .d, Bruker ion mobility .d, Thermo .raw) or mzML as conversion input
- Scan-level filter criteria (e.g., MSLevel, RetentionTime) and target m/z windows or precursor m/z values

## Outputs

- Per-scan 1D intensity and m/z numpy arrays retrieved from HDF5 datasets
- Extracted ion chromatograms (intensity vs RetentionTime traces) or mobility distributions plotted with matplotlib
- Timed extraction benchmarks (e.g., XIC generation duration in minutes)

## How to apply

Convert the vendor file (Agilent .d, Bruker IM .d, Thermo .raw, or mzML) to .mza with the MZA executable, optionally passing -intensityThreshold to drop low-intensity signals. In Python, open the .mza with h5py (registering hdf5plugin for compressed datasets) and load the metadata dataset, filtering rows by column values such as MSLevel=1 and sorting by RetentionTime (minutes). For each scan, read the jagged 1D arrays stored in the 'Arrays_intensity' and 'Arrays_mz' HDF5 groups, named by the scan's 'Scan' value (e.g., Arrays_intensity/630); for IM data read Arrays_mzbin indexes against Full_mz_array and respect MzaPath partition indexes (e.g., Arrays_intensity1/630) since large files are partitioned into multiple groups for read performance. Assemble per-scan arrays into traces or distributions with numpy and plot with matplotlib. Note spectra are stored with zero-intensity values omitted, so bounds checks on m/z windows operate on compressed arrays.

## Related tools

- **MZA** (Command-line converter producing the HDF5 (.mza) file structure and the access paradigm this skill follows) — https://github.com/PNNL-m-q/mza
- **h5py** (Python library used to open .mza files and read metadata and per-scan jagged arrays (with hdf5plugin registered)) — https://github.com/PNNL-m-q/mza
- **rhdf5** (R package for the same HDF5 access path in R example scripts)
- **hdf5plugin** (Registers HDF5 compression filters so compressed datasets in .mza files decode in h5py)
- **numpy** (Sum/filter intensities within m/z tolerance windows and assemble traces)
- **matplotlib** (Plot XICs and mobility distributions from retrieved arrays)

## Examples

```
import h5py, hdf5plugin, numpy as np; f = h5py.File('LC-SLIM.mza','r'); meta = f['Metadata'][()]; mz = f['Arrays_mz/630'][()]; inten = f['Arrays_intensity/630'][()]
```

## Evaluation signals

- Metadata filter returns only rows with MSLevel=1 for MS1 tasks and scans sorted ascending by RetentionTime in minutes
- Every Scan value in the filtered metadata resolves to datasets in Arrays_intensity and Arrays_mz (respecting any MzaPath partition suffix, e.g., Arrays_intensity1/<scan>)
- Extracted XICs show four resolved isotope peaks with co-eluting chromatographic profiles across the full run (e.g., 60 min LC-SLIM file)
- Measured XIC generation time is recorded and comparable in order of magnitude to the reported 2.14 min for MZA access (vs 13.24 s vendor)
- Plots render without KeyError/HDF5 filter errors, confirming hdf5plugin was registered before dataset reads

## Limitations

- Vendor-library access remains faster for interactive queries: 13.24 s (vendor) vs 2.14 min (MZA/Python) for the four-isotope VLEEANQAINPK XIC example
- Files with very many spectra are partitioned (MzaPath column) into multiple HDF5 groups because too many datasets in one group slows reading — access code must honor partitions
- Spectra omit zero-intensity values, so arrays are jagged/compressed and direct positional indexing against full m/z grids only applies to IM data via Arrays_mzbin/Full_mz_array
- Improvements in speed and file size were still needed at publication; conversion itself requires the mza.exe binary (Windows-native, or Wine/Docker on Unix)

## Evidence

- [results] MZA access versus vendor for XIC generation of peptide VLEEANQAINPK: [evidence span withheld — anchor status: not_found]
