---
name: extracted-ion-chromatogram-extraction
description: Use when raw MS data has been converted to .mza and you need intensity-vs-retention-time
  traces for specific m/z windows (e.g., isotope envelopes) using h5py/numpy rather
  than vendor libraries.
when_to_use_negative:
- The .mza file contains ion mobility spectra stored as mzbins indexes (Arrays_mzbin/Full_mz_array)
  rather than direct m/z arrays, and you have not resolved the m/z binning — direct
  m/z-window filtering as described does not apply
- You need MS2/fragmentation traces (Activation, PrecursorMonoisotopicMz filtering)
  rather than MS1 chromatograms
- Peak-picking or centroid-level quantification is required — this skill sums raw
  profile intensities within m/z tolerance windows and performs no deconvolution
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MZA
  role: Defines the .mza HDF5 file structure (metadata table + Arrays_intensity/Arrays_mz
    jagged datasets) that the XIC loop reads
  repo: https://github.com/PNNL-m-q/mza
- name: h5py
  role: Python library used to open the .mza file and read the per-scan 1D m/z and
    intensity datasets
  repo: https://github.com/PNNL-m-q/mza
- name: hdf5plugin
  role: Registered plugin needed by h5py to decode the compressed arrays inside .mza
    files
  repo: https://github.com/PNNL-m-q/mza
- name: numpy
  role: Used for the m/z tolerance-window filtering and intensity summation per scan
- name: matplotlib
  role: Plots the four isotope XIC traces in a single figure mirroring the fidelity
    comparison of Fig. 3
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00313
    title: 'MZA: A Data Conversion Tool to Facilitate Software Development and Artificial
      Intelligence Research in Multidimensional Mass Spectrometry'
  source_content_hash: sha256:2d9455c5a9bf62cb17682f1042ba9df440377c96f4b7889bf5541a71a14e52e8
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# extracted-ion-chromatogram-extraction

## Summary

Extracts ion chromatograms for target m/z windows directly from MZA (.mza) HDF5 files in Python, bypassing vendor software. The paper demonstrates this by regenerating the four-isotope XICs of peptide VLEEANQAINPK (M+2H)2+ from a 60 min LC-SLIM-MS file.

## When to use

Use when raw MS data has been converted to .mza and you need intensity-vs-retention-time traces for specific m/z windows (e.g., isotope envelopes) using h5py/numpy rather than vendor libraries.

## When NOT to use

- The .mza file contains ion mobility spectra stored as mzbins indexes (Arrays_mzbin/Full_mz_array) rather than direct m/z arrays, and you have not resolved the m/z binning — direct m/z-window filtering as described does not apply
- You need MS2/fragmentation traces (Activation, PrecursorMonoisotopicMz filtering) rather than MS1 chromatograms
- Peak-picking or centroid-level quantification is required — this skill sums raw profile intensities within m/z tolerance windows and performs no deconvolution

## Inputs

- MZA-converted .mza file (HDF5-based, e.g., converted from an Agilent .d LC-SLIM-MS raw file)
- Metadata table inside the .mza file (Scan, MSLevel, RetentionTime, IonMobilityBin columns)
- Target m/z windows for the analyte isotopes (e.g., monoisotopic and first 3 isotopes of VLEEANQAINPK (M+2H)2+, ~0.5 Da spacing)

## Outputs

- Four intensity-vs-retention-time XIC traces spanning the full 60 min run (one per isotope peak)
- Matplotlib figure with the co-eluting isotope XICs (e.g., xic_vleeanqainpk.png)
- Measured XIC generation time for benchmarking against the reported 2.14 min MZA access time

## How to apply

Open the .mza file with h5py after registering hdf5plugin (required for decoding the compressed arrays), load the metadata table, and retain MS1-level spectra (MSLevel = 1) sorted by RetentionTime in minutes. Define the target m/z tolerance windows — for VLEEANQAINPK (M+2H)2+, four windows for the monoisotopic plus three isotope peaks spaced ~0.5 Da apart (doubly charged isotope spacing). For each retained scan, read the 1D jagged arrays from the Scan-named datasets (e.g., Arrays_intensity/<scan> and Arrays_mz/<scan>) and sum intensities whose m/z falls inside each isotope window, then assemble the four intensity-vs-retention-time traces across the full run. The key decision point is the m/z tolerance width per window and whether to filter by MSLevel and IonMobilityBin, since IM files store m/z as indexes into Full_mz_array via Arrays_mzbin rather than direct m/z values. Time the extraction loop for comparison against the paper's benchmark of 2.14 min for MZA in Python vs 13.24 s via the vendor software.

## Related tools

- **MZA** (Defines the .mza HDF5 file structure (metadata table + Arrays_intensity/Arrays_mz jagged datasets) that the XIC loop reads) — https://github.com/PNNL-m-q/mza
- **h5py** (Python library used to open the .mza file and read the per-scan 1D m/z and intensity datasets) — https://github.com/PNNL-m-q/mza
- **hdf5plugin** (Registered plugin needed by h5py to decode the compressed arrays inside .mza files) — https://github.com/PNNL-m-q/mza
- **numpy** (Used for the m/z tolerance-window filtering and intensity summation per scan)
- **matplotlib** (Plots the four isotope XIC traces in a single figure mirroring the fidelity comparison of Fig. 3)

## Examples

```
import h5py, hdf5plugin; f = h5py.File('LC-SLIM-MS.mza','r'); mz, inten = f['Arrays_mz/630'][:], f['Arrays_intensity/630'][:]
```

## Evaluation signals

- The figure shows four resolved isotope XICs with co-eluting chromatographic profiles across the full 60 min run, consistent with the reported MZA-vs-vendor fidelity comparison (Fig. 3)
- The set of retained scans equals the number of metadata rows with MSLevel = 1, sorted by RetentionTime in minutes
- Per scan, summed intensity per window equals the sum of Arrays_intensity values whose Arrays_mz falls within each isotope's tolerance window
- The measured extraction loop time is recorded and comparable in magnitude to the reported 2.14 min MZA access time (vendor software baseline: 13.24 s)
- Spectra are read without zero-intensity padding, matching MZA's stored representation

## Limitations

- MZA data access is slower than vendor software: 2.14 min for XICs of 4 isotopic peaks of VLEEANQAINPK across the complete LC-SLIM-MS file vs 13.24 s with the vendor library; the paper notes improvements in speed and file size are still needed
- Files with too many spectra may be partitioned into multiple HDF5 groups (MzaPath column, e.g., Arrays_intensity1/630), so the scan-to-group path must be resolved from the metadata table
- Zero-intensity values are omitted from stored spectra, so traces must be built from summed window intensities per scan, not dense m/z grids

## Evidence

- [results] Generating the XICs of the 4 isotopic peaks of VLEEANQAINPK (M+2H)2+ across the complete LC-SLIM-MS file using MZA in Python took 2.14 min, compared to 13.24 s with the vendor software.: [evidence span withheld — anchor status: not_found]
