---
name: lc-ms-roi-signal-segmentation
description: Use when raw LC-MS mzML or Bruker .d files contain noisy MS1 mass traces
  that must be cut into chromatographic peak segments without biasing final peak parameters,
  or when a peak detector must achieve complete signal coverage over each ROI.
when_to_use_negative:
- Input is already a processed feature table or peak list rather than raw LC-MS intensity
  data — segmentation must start from raw file imports.
- The goal is peak alignment or adduct/in-source-fragment grouping rather than detecting
  peak edges within individual ROIs — use MassCube's subsequent grouping/alignment
  modules instead.
- Vendor data cannot be decoded (e.g., unsupported Bruker TimsCompressionType or uncalibrated
  m/z models) — segmentation requires read_raw_file_to_obj to succeed first.
edam_operation: http://edamontology.org/operation_3694
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MassCube
  role: Provides the Python API (read_raw_file_to_obj, Gaussian filter σ = 1.2, prominence-ratio
    edge detection, peak_segments.csv writer) used to segment each ROI into chromatographic
    peak segments.
  repo: https://github.com/huaxuyu/masscube
- name: Python
  role: Runtime language in which MassCube is implemented, leveraging Python's numerical
    library ecosystem for the array-based smoothing and edge-detection steps.
  repo: https://github.com/huaxuyu/masscube/
- name: NumPy
  role: Array library used for storing and manipulating ROI intensity arrays and scan-varying
    peak arrays in the .mcraw cache before segmentation.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-60640-5
    title: MassCube improves accuracy for metabolomics data processing from raw files
      to phenotype classifiers
  source_content_hash: sha256:2ce60dd45462809c9af483fb9449b07a1e5e7ed583d603444fb57161746c9b09
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/huaxuyu/masscube
  tools:
  - MassCube
  - NumPy
  - Python
  edam_operation: http://edamontology.org/operation_3694
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# lc-ms-roi-signal-segmentation

## Summary

Segments LC-MS regions of interest (mass traces) into distinct chromatographic peak segments using Gaussian-filter-assisted edge detection on smoothed intensities while reporting metadata from the raw, unsmoothed signal, as implemented in MassCube.

## When to use

Use when raw LC-MS mzML or Bruker .d files contain noisy MS1 mass traces that must be cut into chromatographic peak segments without biasing final peak parameters, or when a peak detector must achieve complete signal coverage over each ROI.

## When NOT to use

- Input is already a processed feature table or peak list rather than raw LC-MS intensity data — segmentation must start from raw file imports.
- The goal is peak alignment or adduct/in-source-fragment grouping rather than detecting peak edges within individual ROIs — use MassCube's subsequent grouping/alignment modules instead.
- Vendor data cannot be decoded (e.g., unsupported Bruker TimsCompressionType or uncalibrated m/z models) — segmentation requires read_raw_file_to_obj to succeed first.

## Inputs

- Raw LC-MS data files readable by MassCube's read_raw_file_to_obj (mzML or supported Bruker .d / TDF2 directories)
- ROI intensity arrays (m/z and intensity columns) extracted from each constructed mass trace
- Gaussian filter sigma parameter (default optimized value σ = 1.2)
- Prominence-ratio criterion parameter (ratio = 0.1)

## Outputs

- peak_segments.csv with per-ROI segment rows including retention-time boundaries, start/end edge indices, apex intensity, and peak_shape metadata
- Distinct chromatographic peak segments covering the ROI signal (targeting 100% signal coverage)
- Segment metadata usable for downstream adduct/ISF grouping and gap filling

## How to apply

Load each raw LC-MS file with MassCube's read_raw_file_to_obj and extract the ROI intensity arrays (m/z and intensity columns) for each constructed mass trace. Apply a 1D Gaussian filter with an adjustable sigma (optimized to σ = 1.2) to each ROI intensity array solely to suppress noise prior to peak edge detection. Find candidate peak edges as local minima selected by a prominence-ratio criterion with ratio = 0.1 on the smoothed array, then map the detected edges back onto the RAW (unsmoothed) intensity array and cut it into distinct chromatographic peak segments bounded by start/end edge indices. Record per-segment metadata (retention-time boundaries, apex intensity, peak_shape) and write the segment table to peak_segments.csv. The rationale is that smoothing may bias data structures, so MassCube uses raw data for final reports while smoothing is reserved only for edge detection.

## Related tools

- **MassCube** (Provides the Python API (read_raw_file_to_obj, Gaussian filter σ = 1.2, prominence-ratio edge detection, peak_segments.csv writer) used to segment each ROI into chromatographic peak segments.) — https://github.com/huaxuyu/masscube
- **Python** (Runtime language in which MassCube is implemented, leveraging Python's numerical library ecosystem for the array-based smoothing and edge-detection steps.) — https://github.com/huaxuyu/masscube/
- **NumPy** (Array library used for storing and manipulating ROI intensity arrays and scan-varying peak arrays in the .mcraw cache before segmentation.)

## Examples

```
from masscube import read_raw_file_to_obj; data = read_raw_file_to_obj('sample.d'); scan = data.scans[data.ms1_idx_arr[0]]; print(scan.signals)  # columns: m/z, intensity
```

## Evaluation signals

- peak_segments.csv exists and contains non-empty rows for each processed ROI.
- Each segment row includes edge-boundary fields (start/end indices) traceable to the ROI intensity array.
- Segmentation covers the ROI signal consistent with MassCube's reported 100% signal coverage for peak detection.
- Retention-time boundaries and apex intensities in the segment table reflect values taken from the raw, unsmoothed intensity array rather than the Gaussian-filtered trace.
- Downstream modules (adduct/ISF grouping, gap filling, aligned feature grouping) can consume the .mcraw cache and segment output without re-parsing the source file.

## Limitations

- Human-defined rules for peak shapes, reflected in algorithmic parameters, cannot account for all peak-shape scenarios (e.g., varying scan numbers across peaks, peak tailing).
- The method requires the raw file to be parseable by MassCube; supported Bruker data are limited to TDF2 with MsMsType=8, specific m/z calibration models, and static TIMS calibration model 2.
- Single-file TXT output keeps the peak_shape column for compatibility but does not embed full chromatographic traces by default; set Params.output_peak_shape = True when those traces are explicitly needed.
- The original vendor data must be retained as the archival source because the .mcraw cache is only a workflow cache, not an archival format.

## Evidence

- [abstract] MassCube applies a Gaussian filter with an adjustable sigma value (optimized to σ = 1.2) solely for Gaussian-filter-assisted peak edge detection: "peaks are detected by constructing mass traces through signal clustering and Gaussian-filter assisted edge detection"
- [results] The Gaussian filter is used for edge detection only; raw data are used for final reports because smoothing may bias data structures.: "Peak detection in MassCube achieves 100% signal coverage with comprehensive reporting of chromatographic metadata"
- [readme] MassCube's reader decodes mzML or Bruker .d sources once and stores a versioned .mcraw cache that subsequent steps reload instead of re-parsing the source.: "Feature detection, gap filling, and aligned feature grouping then reload this cache instead of reparsing the source."
- [methods] Peak edges are local minima found on the smoothed array with a prominence-ratio criterion (ratio = 0.1), and segments are cut on the raw intensities.: "Classic software has focused on evaluating the rate of change in m/z abundance across a chromatographic peak for feature detection."
- [discussion] Human-defined peak-shape rules in the algorithmic parameters cannot cover all peak-shape scenarios such as varying scan numbers and peak tailing.: "Human-defined rules for peak shapes, reflected in algorithmic parameters, cannot account for all such scenarios."
