---
name: lc-ms-feature-detection
description: Use when starting from raw LC-MS files (mzML or Bruker timsTOF .d/TDF2
  DDA-PASEF) and you need nontargeted peak detection with 100% signal coverage, edge-bounded
  peak segments, and feature quality metadata.
when_to_use_negative:
- Input is already a processed feature table or aligned peak matrix — feature detection
  must run from raw imports forward.
- Targeted analysis where peak boundaries and analytes are already known and only
  quantification of pre-defined transitions is required.
- Data that require a graphical user interface or direct metabolite identification,
  since MassCube does not provide a GUI or perform metabolite identification itself.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MassCube
  role: Python package that implements raw-file reading, mass-trace construction via
    signal clustering, Gaussian-filter (sigma=1.2) assisted edge detection, and peak
    segmentation using raw (unsmoothed) data for final reports.
  repo: https://github.com/huaxuyu/masscube
- name: Python
  role: Runtime ecosystem used by MassCube for array programming and numerical libraries;
    masscube is installed via pip.
  repo: https://github.com/huaxuyu/masscube
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-60640-5
    title: MassCube improves accuracy for metabolomics data processing from raw files
      to phenotype classifiers
  source_content_hash: sha256:92feb4cad5b2c6c19af317baf1e92ddb91c8ef61963f15d217eb0d6bd0020dac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/huaxuyu/masscube
  tools:
  - MassCube
  - NumPy
  - Python
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 21
---

# lc-ms-feature-detection

## Summary

Detect chromatographic peaks in untargeted LC-MS runs by constructing mass traces through signal clustering and Gaussian-filter-assisted edge detection, as implemented in MassCube, so that raw-data reports retain full signal coverage and chromatographic metadata without smoothing bias.

## When to use

Use when starting from raw LC-MS files (mzML or Bruker timsTOF .d/TDF2 DDA-PASEF) and you need nontargeted peak detection with 100% signal coverage, edge-bounded peak segments, and feature quality metadata.

## When NOT to use

- Input is already a processed feature table or aligned peak matrix — feature detection must run from raw imports forward.
- Targeted analysis where peak boundaries and analytes are already known and only quantification of pre-defined transitions is required.
- Data that require a graphical user interface or direct metabolite identification, since MassCube does not provide a GUI or perform metabolite identification itself.

## Inputs

- Raw LC-MS data file (mzML)
- Bruker timsTOF .d directory (TDF2, DDA-PASEF MsMsType=8, calibration model 1/2)
- MassCube project directory with tmp cache support

## Outputs

- Feature table with per-feature chromatographic metadata (retention-time boundaries, apex intensity, peak_shape, m/z, quality columns)
- Detected peak segments bounded by start/end edge indices per ROI/mass trace
- Single-file TXT output (optionally with embedded chromatographic traces when Params.output_peak_shape=True)

## How to apply

Load the raw file with MassCube's read_raw_file_to_obj (supports mzML and Bruker TDF2 directories with calibrated m/z and inverse reduced ion mobility), then extract ROI intensity arrays per mass trace. Construct mass traces through signal clustering, then apply a 1D Gaussian filter with sigma=1.2 to each ROI intensity array solely for edge detection: candidate peak edges are local minima selected by a prominence-ratio criterion with ratio=0.1 on the smoothed array. Map detected edges back onto the RAW (unsmoothed) intensity array and cut it into chromatographic peak segments bounded by start/end edge indices, recording per-segment metadata (retention-time boundaries, apex intensity, peak_shape). Rationale: smoothing may bias data structures, so final reports use raw data; MassCube reports 100% signal coverage with comprehensive chromatographic metadata for quality assurance.

## Related tools

- **MassCube** (Python package that implements raw-file reading, mass-trace construction via signal clustering, Gaussian-filter (sigma=1.2) assisted edge detection, and peak segmentation using raw (unsmoothed) data for final reports.) — https://github.com/huaxuyu/masscube
- **Python** (Runtime ecosystem used by MassCube for array programming and numerical libraries; masscube is installed via pip.) — https://github.com/huaxuyu/masscube

## Examples

```
from masscube import read_raw_file_to_obj; data = read_raw_file_to_obj('sample.d')
```

## Evaluation signals

- peak_segments.csv (or equivalent feature table) exists and is non-empty, with per-ROI edge-boundary fields (start/end indices) present.
- Segmentation covers the ROI signal consistent with MassCube's reported 100% signal coverage for peak detection.
- Final peak intensities are read from the RAW intensity array, not the Gaussian-smoothed array, verifying no smoothing bias in reported values.
- Output retains comprehensive chromatographic metadata per feature (RT boundaries, apex intensity, peak_shape) enabling downstream quality-control checks.
- When re-running from a .mcraw cache, feature detection, gap filling, and aligned feature grouping reload the cache instead of reparsing the source, and results match the original run.

## Limitations

- Human-defined rules for peak shapes, reflected in algorithmic parameters, cannot account for all peak-shape scenarios.
- Rate-of-change approaches are limited in robustness and sensitive to noise, requiring extensive manual inspection (challenge addressed by the trace-clustering + Gaussian-edge approach).
- The .mcraw cache is not an archival format — the original vendor data should still be retained as the archival source.
- Single-file TXT output does not embed full chromatographic traces by default; Params.output_peak_shape must be set to True when traces are needed.

## Evidence

- [abstract] peaks are detected by constructing mass traces through signal clustering and Gaussian-filter assisted edge detection: "peaks are detected by constructing mass traces through signal clustering and Gaussian-filter assisted edge detection"
- [abstract] Peak detection in MassCube achieves 100% signal coverage with comprehensive reporting of chromatographic metadata: "Peak detection in MassCube achieves 100% signal coverage with comprehensive reporting of chromatographic metadata"
- [other] MassCube applies a Gaussian filter with an adjustable sigma value (optimized to σ = 1.2) solely for Gaussian-filter-assisted peak edge detection: [evidence span withheld — anchor status: not_found]
- [methods] Feature detection, gap filling, and aligned feature grouping then reload this cache instead of reparsing the source.: "Feature detection, gap filling, and aligned feature grouping then reload this cache instead of reparsing the source."
- [readme] The original vendor data should still be retained as the archival source.: "The original vendor data should still be retained as the archival source."
