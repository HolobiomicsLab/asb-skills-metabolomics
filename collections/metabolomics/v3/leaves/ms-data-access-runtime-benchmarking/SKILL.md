---
name: ms-data-access-runtime-benchmarking
description: 'Decision support for choosing an MS data-access route: use when you
  must quantify how long XIC extraction takes from .mza/HDF5 files across a full LC-MS
  run versus vendor library access.'
when_to_use_negative:
- Input is still a vendor proprietary file (.d, .raw, mzML) — convert to .mza first
  or use the vendor library directly, since the benchmark compares the two access
  routes rather than converting data.
- The data are IM-frame spectra (Arrays_mzbin/Full_mz_array layout with IonMobilityBin
  indexes) rather than per-scan MS1 jagged m/z arrays — the scan-wise Arrays_intensity/Arrays_mz
  access pattern and timing loop assume the LC-MS layout.
- 'You need rapid interactive extraction where vendor software is faster: the paper
  reports MZA access at 2.14 min versus 13.24 s via the vendor library, so this skill
  documents the tradeoff rather than replacing vendor tools for speed-critical workflows.'
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MZA
  role: Provides the .mza HDF5 data structure whose scan-wise Arrays_intensity and
    Arrays_mz jagged arrays are read and timed during XIC extraction
  repo: https://github.com/PNNL-m-q/mza
- name: h5py
  role: Python library used to open the .mza file, read the metadata table and per-scan
    1D arrays
  repo: https://github.com/PNNL-m-q/mza
- name: hdf5plugin
  role: Registered plugin required to read the compressed HDF5 datasets in .mza files
  repo: https://github.com/PNNL-m-q/mza
- name: numpy
  role: Performs the intensity summation within isotope m/z windows across retained
    MS1 scans
  repo: https://github.com/PNNL-m-q/mza
- name: matplotlib
  role: Plots the extracted isotope XIC traces in a single fidelity-comparison figure
  repo: https://github.com/PNNL-m-q/mza
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00313
    title: 'MZA: A Data Conversion Tool to Facilitate Software Development and Artificial
      Intelligence Research in Multidimensional Mass Spectrometry'
  source_content_hash: sha256:8fa4c0848217fffcc097f670a172a108fcf57fda708cdb84276375da905eb448
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mza
  tools:
  - MZA
  - Python
  - h5py
  - hdf5plugin
  - matplotlib
  - numpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# ms-data-access-runtime-benchmarking

## Summary

Times the extraction of MS1 chromatograms (XICs) from MZA-converted HDF5 data in Python versus vendor software, quantifying the access-runtime tradeoff of the MZA loose-coupling data model.

## When to use

Decision support for choosing an MS data-access route: use when you must quantify how long XIC extraction takes from .mza/HDF5 files across a full LC-MS run versus vendor library access.

## When NOT to use

- Input is still a vendor proprietary file (.d, .raw, mzML) — convert to .mza first or use the vendor library directly, since the benchmark compares the two access routes rather than converting data.
- The data are IM-frame spectra (Arrays_mzbin/Full_mz_array layout with IonMobilityBin indexes) rather than per-scan MS1 jagged m/z arrays — the scan-wise Arrays_intensity/Arrays_mz access pattern and timing loop assume the LC-MS layout.
- You need rapid interactive extraction where vendor software is faster: the paper reports MZA access at 2.14 min versus 13.24 s via the vendor library, so this skill documents the tradeoff rather than replacing vendor tools for speed-critical workflows.

## Inputs

- .mza file (MZA/HDF5 conversion of a vendor LC-MS or LC-IM-MS raw file, e.g. from an LC-SLIM-MS acquisition)
- metadata table columns Scan, MSLevel, RetentionTime (and MzaPath if partitions exist)
- a target list of m/z windows / isotopic peaks (e.g. four isotope m/z values for a peptide at a given charge state)

## Outputs

- four (or N) extracted ion chromatograms as intensity-vs-retention-time traces spanning the full run
- a matplotlib figure of the isotope XICs (e.g. xic_vleeanqainpk.png)
- a measured XIC-generation runtime in minutes for the .mza/HDF5 route, comparable against the vendor-software runtime

## How to apply

Open the MZA-converted .mza file with h5py (with hdf5plugin registered) and load the metadata table, retaining MS1-level spectra (MSLevel = 1) sorted by RetentionTime in minutes. Define target m/z tolerance windows for the isotope set of interest — e.g., four isotopic peaks of the peptide VLEEANQAINPK (M+2H)2+, spaced ~0.5 Da at charge 2 — then for each retained MS1 scan read the 1D intensity and m/z jagged arrays from the Scan-named HDF5 datasets (Arrays_intensity/<scan>, Arrays_mz/<scan>) and sum intensities falling in each isotope window. Time the extraction loop with a wall-clock timer over the complete run (the paper reports 2.14 min for MZA access versus 13.24 s via the vendor library on the LC-SLIM-MS file) and record the measured time alongside the outputs. Plot the resulting intensity-vs-retention-time traces with matplotlib and validate that the four isotope XICs co-elute across the ~60 min run, mirroring the vendor fidelity comparison, before comparing runtimes.

## Related tools

- **MZA** (Provides the .mza HDF5 data structure whose scan-wise Arrays_intensity and Arrays_mz jagged arrays are read and timed during XIC extraction) — https://github.com/PNNL-m-q/mza
- **h5py** (Python library used to open the .mza file, read the metadata table and per-scan 1D arrays) — https://github.com/PNNL-m-q/mza
- **hdf5plugin** (Registered plugin required to read the compressed HDF5 datasets in .mza files) — https://github.com/PNNL-m-q/mza
- **numpy** (Performs the intensity summation within isotope m/z windows across retained MS1 scans) — https://github.com/PNNL-m-q/mza
- **matplotlib** (Plots the extracted isotope XIC traces in a single fidelity-comparison figure) — https://github.com/PNNL-m-q/mza

## Examples

```
import h5py, hdf5plugin, numpy as np
with h5py.File('LC-SLIM.mza','r') as f:
    meta = f['Metadata'][()]
    mz = f['Arrays_mz/630'][()]; inten = f['Arrays_intensity/630'][()]
```

## Evaluation signals

- The extracted isotope XICs show four resolved, co-eluting chromatographic profiles across the full LC-SLIM-MS run, consistent with the reported MZA-vs-vendor fidelity comparison.
- The measured generation time is recorded and directly comparable to the paper's reported 2.14 min for MZA access (vendor software: 13.24 s).
- Only spectra with MSLevel = 1 are included in the extraction, sorted by RetentionTime in minutes.
- Intensity sums come exclusively from the Scan-named HDF5 datasets (e.g. Arrays_intensity/630, Arrays_mz/630), respecting the MzaPath partition index if present.
- The XIC figure reproduces the vendor-software isotope profile for the same peptide, confirming no loss of chromatographic fidelity in the MZA route.

## Limitations

- Improvements in speed and file size are still needed for MZA; the access route is slower than vendor libraries (2.14 min vs 13.24 s in the paper's example).
- Having too many datasets within a single HDF5 group can slow data reading performance; MZA may partition arrays into indexed groups (MzaPath), which the access code must honor.
- No version number is stated for the MZA tool and no changelog is available in the repository, so reproducibility of the exact runtime depends on the release used.

## Evidence

- [results] MZA XIC extraction runtime reported: [evidence span withheld — anchor status: not_found]
