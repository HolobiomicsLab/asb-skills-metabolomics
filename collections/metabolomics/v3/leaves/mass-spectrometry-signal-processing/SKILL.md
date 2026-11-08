---
name: mass-spectrometry-signal-processing
description: Use when a 1D MS signal (e.g., an extracted ion chromatogram or arrival
  time distribution from MZA data) needs peak detection, interpolation, or peak-area
  quantification, and detections must be validated against known peak positions.
when_to_use_negative:
- The data are still proprietary vendor formats (.raw, Agilent .d) not yet converted
  to MZA — use the mza converter first
- The analysis targets multidimensional IMS data where the m/z dimension is stored
  as mzbins indexes, rather than a single 1D signal suited to find_peaks_1d_localmax
- Only raw data extraction from MZA is needed with no peak finding, interpolation,
  or calibration — mzapy's core MZA interface suffices without the peaks module
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: mzapy
  role: Provides mzapy.peaks.find_peaks_1d_localmax and companion functions (find_peaks_1d_gauss,
    lerp_1d, calc_peak_area) used to detect and quantify peaks in 1D MS signals
  repo: https://github.com/PNNL-m-q/mzapy
- name: scipy
  role: find_peaks_1d_localmax uses scipy.signal.find_peaks internally to detect local-maximum
    peaks
- name: numpy
  role: Used to generate synthetic 1D test signals (seeded RNG, sum of Gaussian peaks
    plus noise floor) and to map peak indices to x positions and heights
- name: mza
  role: Converts vendor MS files (.raw, .d, .mzML) to the MZA/HDF5 structure that
    upstream signals for peak finding are extracted from
  repo: https://github.com/PNNL-m-q/mza
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c01653
    title: '<i>mzapy</i>: An Open-Source Python Library Enabling Efficient Extraction
      and Processing of Ion Mobility Spectrometry-Mass Spectrometry Data in the MZA
      File Format'
  source_content_hash: sha256:1c408b1023f23fe482d5e728c5c318ad4b1fec3902fdcea33930d5bc706c850a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mzapy
  tools:
  - Python
  - mza
  - mzapy
  - numpy
  - scipy
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# mass-spectrometry-signal-processing

## Summary

Applies mzapy's peak-processing utilities (mzapy.peaks.find_peaks_1d_localmax, find_peaks_1d_gauss, lerp_1d, calc_peak_area) to detect and characterize peaks in 1D mass spectrometry signals, such as extracted ion chromatograms collected via collect_xic_arrays_by_mz from MZA/HDF5 data.

## When to use

Use when a 1D MS signal (e.g., an extracted ion chromatogram or arrival time distribution from MZA data) needs peak detection, interpolation, or peak-area quantification, and detections must be validated against known peak positions.

## When NOT to use

- The data are still proprietary vendor formats (.raw, Agilent .d) not yet converted to MZA — use the mza converter first
- The analysis targets multidimensional IMS data where the m/z dimension is stored as mzbins indexes, rather than a single 1D signal suited to find_peaks_1d_localmax
- Only raw data extraction from MZA is needed with no peak finding, interpolation, or calibration — mzapy's core MZA interface suffices without the peaks module

## Inputs

- 1D intensity-vs-x arrays (e.g., extracted ion chromatogram from collect_xic_arrays_by_mz, or a synthetic Gaussian-peak signal on an m/z axis)
- MZA file (HDF5-based) read via mzapy/h5py when working from raw converted vendor data (.raw, .d, .mzML)

## Outputs

- Detected peak list (e.g., peak_list.csv with columns index, x, height)
- Peak positions and heights mapped from detected indices to signal-axis coordinates
- Optionally: interpolated signals (lerp_1d), peak areas (calc_peak_area), and Gaussian PSNR values (calc_gauss_psnr)

## How to apply

Install mzapy from PyPI via pip (or clone github.com/PNNL-m-q/mzapy) and import the mzapy.peaks module, whose documented functions include find_peaks_1d_localmax (which wraps scipy.signal.find_peaks for local-maximum peak detection), find_peaks_1d_gauss, lerp_1d/lerp_2d for interpolation, calc_gauss_psnr, and calc_peak_area. A representative workflow: generate or load a 1D signal on an m/z-like axis (e.g., a synthetic sum of well-resolved Gaussian peaks plus a small noise floor, seeded RNG), run mzapy.peaks.find_peaks_1d_localmax to detect local-maximum peaks, map returned peak indices back to x positions and heights, and write the detections to a table such as peak_list.csv with columns index, x, height. Validate by comparing each recovered x position and height to ground-truth peaks within a stated tolerance; for 5 known ground-truth peaks, peak_list.csv must contain exactly 5 matching rows. The rationale is that mzapy's pure-Python implementation with minimal, largely standardized dependencies (numpy, scipy, h5py) makes these utilities reusable in multiomics application development.

## Related tools

- **mzapy** (Provides mzapy.peaks.find_peaks_1d_localmax and companion functions (find_peaks_1d_gauss, lerp_1d, calc_peak_area) used to detect and quantify peaks in 1D MS signals) — https://github.com/PNNL-m-q/mzapy
- **scipy** (find_peaks_1d_localmax uses scipy.signal.find_peaks internally to detect local-maximum peaks)
- **numpy** (Used to generate synthetic 1D test signals (seeded RNG, sum of Gaussian peaks plus noise floor) and to map peak indices to x positions and heights)
- **mza** (Converts vendor MS files (.raw, .d, .mzML) to the MZA/HDF5 structure that upstream signals for peak finding are extracted from) — https://github.com/PNNL-m-q/mza

## Examples

```
pip install mzapy; from mzapy.peaks import find_peaks_1d_localmax; peaks = find_peaks_1d_localmax(x, y)
```

## Evaluation signals

- peak_list.csv exists and contains exactly one row per expected (ground-truth) peak — e.g., exactly 5 rows for 5 well-resolved Gaussian peaks
- Every recovered x position matches a ground-truth peak position within the specified tolerance
- Recovered peak heights match ground-truth heights within the specified tolerance
- No spurious detections appear in noise-only regions of the signal (noise floor stays below the local-maximum detection threshold)
- Downstream mzapy.peaks functions succeed on the same signal, e.g., calc_peak_area returns a finite area consistent with the detected peak extent

## Limitations

- The local-maximum finder assumes well-resolved peaks; heavily overlapping peaks would require find_peaks_1d_gauss instead, and no algorithmic parameters or thresholds are specified in the source documentation
- The README exposes no changelog for the repository, so API stability of mzapy.peaks functions across versions is not documented
- Functions are documented only via autodoc directives (e.g., autofunction stubs) in the article text; concrete parameter names and defaults must be read from the mzapy.readthedocs.io documentation

## Evidence
