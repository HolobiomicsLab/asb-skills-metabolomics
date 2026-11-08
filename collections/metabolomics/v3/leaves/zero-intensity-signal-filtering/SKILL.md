---
name: zero-intensity-signal-filtering
description: Use when converting mzML or vendor raw files (.d, .raw) to the MZA HDF5
  structure and spectra must be stored as jagged 1D arrays with zero-intensity points
  removed before writing Arrays_intensity/Arrays_mz datasets.
when_to_use_negative:
- Input is already an .mza file — zero-intensity values are omitted at conversion
  time, so re-running the filter on converted data would only compound the earlier
  -intensityThreshold loss.
- Downstream analysis requires the full dense spectrum including zero-intensity bins
  (e.g., fixed-grid rebinning or total-spectrum reconstruction) — the stored jagged
  arrays permanently lack zero-intensity points.
- Quantification must retain every sub-threshold signal — the -intensityThreshold
  option irreversibly discards signals that do not exceed the minimum intensity (e.g.,
  20).
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: pymzML
  role: Parses the input mzML to extract each spectrum's m/z and intensity arrays
    that are then filtered for zero-intensity values
- name: h5py
  role: Writes and read-back-validates the filtered spectra as 1D HDF5 datasets named
    by scan number in the Arrays_intensity/Arrays_mz groups
- name: MZA
  role: Stand-alone converter executable that applies zero-intensity omission (and
    the optional -intensityThreshold) while building the .mza HDF5 file
  repo: https://github.com/PNNL-m-q/mza
- name: hdf5plugin
  role: Python dependency used with h5py when accessing the converted .mza files in
    the example scripts
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00313
    title: 'MZA: A Data Conversion Tool to Facilitate Software Development and Artificial
      Intelligence Research in Multidimensional Mass Spectrometry'
  source_content_hash: sha256:81371dba931351942f29408b5fa1d7487d064f5759dacc7c85b968dcef146ce1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mza
  tools:
  - MZA
  - Python
  - h5py
  - hdf5plugin
  - pymzML
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# zero-intensity-signal-filtering

## Summary

A filtering rule applied during mzML-to-MZA conversion that omits zero-intensity data points from every spectrum, so each scan is stored as a compact jagged 1D m/z array and 1D intensity array (named by its Scan value) in the MZA HDF5 structure, optionally tightened further with the -intensityThreshold minimum.

## When to use

Use when converting mzML or vendor raw files (.d, .raw) to the MZA HDF5 structure and spectra must be stored as jagged 1D arrays with zero-intensity points removed before writing Arrays_intensity/Arrays_mz datasets.

## When NOT to use

- Input is already an .mza file — zero-intensity values are omitted at conversion time, so re-running the filter on converted data would only compound the earlier -intensityThreshold loss.
- Downstream analysis requires the full dense spectrum including zero-intensity bins (e.g., fixed-grid rebinning or total-spectrum reconstruction) — the stored jagged arrays permanently lack zero-intensity points.
- Quantification must retain every sub-threshold signal — the -intensityThreshold option irreversibly discards signals that do not exceed the minimum intensity (e.g., 20).

## Inputs

- mzML spectra parsed with pymzML (m/z and intensity arrays per scan, with scan number, MS level, retention time, ion mobility arrival time metadata)
- Vendor raw files convertible by mza: Agilent '.d' (with or without ion mobility), Bruker ion mobility '.d', Thermo '.raw'
- Optional -intensityThreshold value (minimum intensity that must be exceeded for signals to be kept, e.g. 20)

## Outputs

- Arrays_intensity/<Scan> 1D HDF5 datasets containing only nonzero-intensity values per spectrum
- Arrays_mz/<Scan> 1D HDF5 datasets with the corresponding m/z values (jagged, row-aligned with the intensity arrays)
- Arrays_mzbin/<Scan> 1D index datasets plus Full_mz_array for ion mobility spectra
- Metadata HDF5 dataset with one row per spectrum whose Scan column keys the array datasets

## How to apply

During the mzML-to-MZA conversion path, parse each spectrum with pymzML to obtain its m/z and intensity arrays plus metadata (scan number, MS level, retention time, IonMobilityBin, IonMobilityTime), then discard any data point whose intensity is zero so only nonzero signals are written. Write the surviving values as two jagged 1D HDF5 datasets named by the scan's "Scan" value in the Arrays_intensity and Arrays_mz groups, with row alignment preserved between the pair. Optionally raise the cutoff above zero with the -intensityThreshold argument (e.g., -intensityThreshold 20) so only signals exceeding the minimum intensity are included in the output .mza file. For ion mobility spectra, apply the same zero-intensity removal but store the surviving m/z values as indexes into the shared Full_mz_array (Arrays_mzbin) instead of raw m/z values. Rationale: omitting zeros keeps the HDF5 datasets compact and read performance fast, since the metadata table row plus the paired 1D arrays fully reconstruct each spectrum.

## Related tools

- **pymzML** (Parses the input mzML to extract each spectrum's m/z and intensity arrays that are then filtered for zero-intensity values)
- **h5py** (Writes and read-back-validates the filtered spectra as 1D HDF5 datasets named by scan number in the Arrays_intensity/Arrays_mz groups)
- **MZA** (Stand-alone converter executable that applies zero-intensity omission (and the optional -intensityThreshold) while building the .mza HDF5 file) — https://github.com/PNNL-m-q/mza
- **hdf5plugin** (Python dependency used with h5py when accessing the converted .mza files in the example scripts)

## Examples

```
mza -file test_data\LowHigh_PC_160_180_frames1-10.d -intensityThreshold 20
```

## Evaluation signals

- Read the output .mza back with h5py and assert that no dataset under Arrays_intensity or Arrays_mz contains intensity values equal to zero.
- Check schema consistency: every "Scan" value in the metadata table has matching, correctly named datasets in Arrays_intensity and Arrays_mz (e.g., Arrays_intensity/630, Arrays_mz/630).
- Verify per scan that the length of the intensity array equals the length of its m/z array (or its Arrays_mzbin index array for IM spectra), preserving row alignment.
- Compare point counts against the -intensityThreshold setting: converting with -intensityThreshold 20 must yield strictly fewer (or equal) stored points per scan than zero-filtering alone.
- Reconstruct a spectrum's nonzero data by zipping its mz/mzbin and intensity arrays and confirm the m/z and intensity pairing is unchanged relative to the source mzML after removing zero-intensity points.

## Limitations

- Zero-intensity omission and the -intensityThreshold cutoff are irreversible at conversion time; low-abundance signals below the minimum are permanently absent from the .mza file.
- Because zero-intensity points are dropped, the stored spectra are jagged and cannot directly reconstruct a dense, fixed-m/z-grid spectrum without the shared Full_mz_array/index scheme used for ion mobility data.
- The MZA paper notes improvements in speed and file size are still needed, and reports no quantitative benchmarks for the conversion or filtering step.

## Evidence

- [other] For each spectrum, extract the m/z and intensity arrays with pymzML and filter out zero-intensity values so spectra are stored as jagged 1D arrays.: [evidence span withheld — anchor status: not_found]
