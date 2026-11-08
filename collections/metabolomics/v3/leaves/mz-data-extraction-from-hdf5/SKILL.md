---
name: mz-data-extraction-from-hdf5
description: Use when spectra from a converted MZA (HDF5) file need to be read programmatically
  — e.g., extracting per-scan m/z and intensity arrays from Arrays_mz/Arrays_intensity,
  or loading mzbins and Full_mz_array for ion mobility data — as the first step before
  calibration or peak finding.
when_to_use_negative:
- Input is still a proprietary vendor file (Thermo .raw, Agilent .d, Bruker .d) that
  has not yet been converted — run the mza converter first
- Data is already in another open format directly readable by the analysis pipeline
  (e.g., mzML) and no MZA conversion is planned
- The task is post-extraction processing such as mass calibration or peak finding
  — extraction is only the prerequisite step
tools:
- name: mzapy
  role: Python package providing the interface to unprocessed MS data in the MZA format
    (mzapy.MZA) and downstream utilities such as mzapy.calibration for extracted values
  repo: https://github.com/PNNL-m-q/mzapy
- name: mza
  role: Stand-alone command-line converter that produces MZA files from vendor formats
    (Agilent .d, Bruker timsTOF, Thermo .raw, mzML)
  repo: https://github.com/PNNL-m-q/mza
- name: h5py
  role: Generic HDF5 library used from Python to open MZA files and read the metadata
    table and per-scan arrays
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.3c01653
    title: '<i>mzapy</i>: An Open-Source Python Library Enabling Efficient Extraction
      and Processing of Ion Mobility Spectrometry-Mass Spectrometry Data in the MZA
      File Format'
  source_content_hash: sha256:956bf0a017f683ed0b1385d9ef57401db54f2269895719806c798c38dfa223f5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mzapy
  tools:
  - MZA
  - Python
  - h5py
  - mza
  - mzapy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# mz-data-extraction-from-hdf5

## Summary

Programmatic access to raw mass spectrometry data stored in the MZA file format — a simple, HDF5-based structure holding a metadata table plus per-scan 1D m/z and intensity arrays — using h5py or the mzapy Python package (mzapy.MZA).

## When to use

Use when spectra from a converted MZA (HDF5) file need to be read programmatically — e.g., extracting per-scan m/z and intensity arrays from Arrays_mz/Arrays_intensity, or loading mzbins and Full_mz_array for ion mobility data — as the first step before calibration or peak finding.

## When NOT to use

- Input is still a proprietary vendor file (Thermo .raw, Agilent .d, Bruker .d) that has not yet been converted — run the mza converter first
- Data is already in another open format directly readable by the analysis pipeline (e.g., mzML) and no MZA conversion is planned
- The task is post-extraction processing such as mass calibration or peak finding — extraction is only the prerequisite step

## Inputs

- MZA file (HDF5-based mass spectrometry data structure, converted from vendor .d/.raw/mzML by mza.exe)
- Metadata table (HDF5 dataset; one row per spectrum with Scan, MSLevel, Polarity, RetentionTime, IonMobilityTime, etc.)

## Outputs

- Per-scan 1D m/z array (e.g., Arrays_mz/630) and 1D intensity array (e.g., Arrays_intensity/630)
- For IM spectra: per-scan mzbins index array (Arrays_mzbin/630) plus the shared Full_mz_array dataset
- Arrays ready for downstream mzapy utilities (mass calibration, XIC collection, peak finding)

## How to apply

First ensure vendor data (Agilent '.d', Bruker ion mobility 'd', Thermo '.raw', or mzML) has been converted to MZA using the mza command-line converter, optionally with an -intensityThreshold to drop low signals at conversion time. Open the resulting .mza file with h5py (or via mzapy.MZA in Python): each row of the metadata table describes one spectrum with properties such as Scan, MSLevel, RetentionTime (minutes), and IonMobilityTime, and spectra are stored as two jagged 1D arrays under the Arrays_intensity and Arrays_mz HDF5 groups, named by the Scan value (e.g., Arrays_intensity/630, Arrays_mz/630). For IM spectra the m/z dimension is stored as indexes in Arrays_mzbin pointing into a shared Full_mz_array dataset. If the MzaPath column is non-empty (partitions are created when a file has too many spectra, which slows HDF5 reading), prepend the partition index to group names, e.g., Arrays_intensity1/630. Zero-intensity values are omitted, so extracted arrays are sparse by design; downstream steps like mass calibration (mzapy.calibration.MassCalibration) pair these extracted measured m/z values with reference calibrant masses.

## Related tools

- **mzapy** (Python package providing the interface to unprocessed MS data in the MZA format (mzapy.MZA) and downstream utilities such as mzapy.calibration for extracted values) — https://github.com/PNNL-m-q/mzapy
- **mza** (Stand-alone command-line converter that produces MZA files from vendor formats (Agilent .d, Bruker timsTOF, Thermo .raw, mzML)) — https://github.com/PNNL-m-q/mza
- **h5py** (Generic HDF5 library used from Python to open MZA files and read the metadata table and per-scan arrays)

## Examples

```
import h5py; f = h5py.File('sample.mza', 'r'); mz = f['Arrays_mz/630'][()]; inten = f['Arrays_intensity/630'][()]
```

## Evaluation signals

- Every Scan value in the metadata table resolves to a matching pair of 1D arrays under Arrays_mz and Arrays_intensity (with the MzaPath partition index prepended when non-empty)
- Arrays are jagged with zero-intensity values omitted — no constant-length arrays should appear across scans
- For IM spectra, Arrays_mzbin values are valid indexes into the shared Full_mz_array dataset
- Extracted measured m/z values, when paired with reference calibrant masses, yield ppm residuals consistent with mzapy/test unit-test tolerances after MassCalibration.fit

## Limitations

- Storing too many datasets within a single HDF5 group slows reading; MZA may create partitions (MzaPath column) that must be handled when indexing groups
- Zero-intensity values are omitted, so reconstructed spectra are sparse and any analysis must account for missing zero channels
- On Unix-like systems the mza converter requires Wine (or a Docker container with Wine), adding a platform dependency for the conversion step; an -intensityThreshold applied at conversion permanently excludes low-intensity signals from the MZA file

## Evidence

- [readme] Once converted, MZA files can be easily accessed from any programming language and operating system using generic HDF5 libraries available (e.g., h5py and rhdf5).: [evidence span withheld — anchor status: external_doc]
- [readme] Spectra are stored omitting zero-intensity values and as two jagged arrays, one in each corresponding group and named as the "Scan" value in the metadata table: [evidence span withheld — anchor status: external_doc]
- [readme] Partitions may be created for files with too many spectra. Having too many datasets within a single group can slow down data reading performance.: [evidence span withheld — anchor status: external_doc]
