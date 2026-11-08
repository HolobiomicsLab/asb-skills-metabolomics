---
name: chromatographic-peak-edge-detection
description: Use when processing raw LC-MS ROI intensity arrays (e.g. mzML or Bruker
  TDF2 .d via MassCube) and you need to split each mass trace into discrete chromatographic
  peak segments bounded by start/end edge indices.
when_to_use_negative:
- Input is already a feature table or aligned peak list — no ROI intensity arrays
  exist to segment
- You need smoothed chromatograms in the final report — MassCube deliberately reports
  raw (unsmoothed) data because smoothing may bias data structures
- Working with data from a single instrument profile such that vendor's proprietary
  per-frame pressure compensation is required for mobility-ordered traces (Bruker
  .d reader records mobility_pressure_compensated == False)
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MassCube
  role: Provides read_raw_file_to_obj for loading raw LC-MS files and implements Gaussian-filter-assisted
    peak edge detection on ROIs
  repo: https://github.com/huaxuyu/masscube
- name: NumPy
  role: Array operations for filtering ROI intensity arrays and cutting segments at
    edge indices
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-60640-5
    title: MassCube improves accuracy for metabolomics data processing from raw files
      to phenotype classifiers
  source_content_hash: sha256:2c8b256b24cd092f78c4906dc4025a372a97d3f7fa0505839f56cd72fa1698f1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/huaxuyu/masscube
  tools:
  - MassCube
  - NumPy
  - Python
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# chromatographic-peak-edge-detection

## Summary

Segment chromatographic peaks within each ROI (mass trace) by applying a 1D Gaussian filter (σ = 1.2) for edge detection only, then cutting the raw intensity array at the detected boundaries. MassCube uses this to achieve 100% signal coverage without letting smoothing bias the reported peak data.

## When to use

Use when processing raw LC-MS ROI intensity arrays (e.g. mzML or Bruker TDF2 .d via MassCube) and you need to split each mass trace into discrete chromatographic peak segments bounded by start/end edge indices.

## When NOT to use

- Input is already a feature table or aligned peak list — no ROI intensity arrays exist to segment
- You need smoothed chromatograms in the final report — MassCube deliberately reports raw (unsmoothed) data because smoothing may bias data structures
- Working with data from a single instrument profile such that vendor's proprietary per-frame pressure compensation is required for mobility-ordered traces (Bruker .d reader records mobility_pressure_compensated == False)

## Inputs

- Raw LC-MS data file (mzML or supported Bruker TDF2 .d, read via MassCube read_raw_file_to_obj)
- Per-ROI intensity arrays (m/z, intensity columns) extracted from mass traces constructed by signal clustering

## Outputs

- peak_segments.csv with per-segment metadata: retention-time boundaries, start/end edge indices, apex intensity, peak_shape
- Distinct chromatographic peak segments cut from the raw intensity array for each ROI

## How to apply

Load the raw LC-MS file with MassCube's read_raw_file_to_obj and extract the ROI intensity arrays (m/z, intensity columns) for each mass trace. Apply a 1D Gaussian filter with sigma=1.2 to each ROI intensity array to suppress noise prior to edge detection; smoothing is used solely for locating edges, not for reporting. Find candidate peak edges as local minima on the smoothed array selected by a prominence-ratio criterion with ratio=0.1. Map the detected edges back onto the raw (unsmoothed) intensity array and cut it into peak segments bounded by start/end edge indices, recording per-segment metadata (retention-time boundaries, apex intensity, peak_shape) to peak_segments.csv. The rationale is that smoothed data may bias data structures, so raw intensities are used for the final reports.

## Related tools

- **MassCube** (Provides read_raw_file_to_obj for loading raw LC-MS files and implements Gaussian-filter-assisted peak edge detection on ROIs) — https://github.com/huaxuyu/masscube
- **NumPy** (Array operations for filtering ROI intensity arrays and cutting segments at edge indices)

## Examples

```
from masscube import read_raw_file_to_obj; data = read_raw_file_to_obj("sample.d"); scan = data.scans[data.ms1_idx_arr[0]]; signals = scan.signals  # columns: m/z, intensity
```

## Evaluation signals

- peak_segments.csv exists and contains non-empty segments with edge-boundary fields (start/end indices) per ROI
- Segmentation covers the ROI signal consistent with MassCube's reported 100% signal coverage for peak detection
- Segment apex intensities and boundaries are taken from the raw (unsmoothed) intensity array, not the Gaussian-smoothed one
- Every detected edge pair yields a bounded segment with defined retention-time boundaries and a peak_shape annotation
- Parameters used match the optimized defaults: Gaussian sigma=1.2 and prominence ratio=0.1

## Limitations

- Human-defined rules for peak shapes, reflected in algorithmic parameters, cannot account for all peak-shape scenarios
- Smoothing may bias data structures, so it is used only for edge detection and never for final reported intensities
- Single-file TXT output keeps the peak_shape column but does not embed full chromatographic traces by default; set Params.output_peak_shape = True when traces are explicitly needed

## Evidence

- [abstract] Gaussian filter solely for edge detection with raw data for reports: "peaks are detected by constructing mass traces through signal clustering and Gaussian-filter assisted edge detection"
- [abstract] 100% signal coverage finding: "Peak detection in MassCube achieves 100% signal coverage with comprehensive reporting of chromatographic metadata"
- [readme] peak_shape column behavior in TXT output: "Single-file TXT output keeps the `peak_shape` column for compatibility but does not embed full chromatographic traces by default."
- [discussion] peak shape rules limitation: "Human-defined rules for peak shapes, reflected in algorithmic parameters, cannot account for all such scenarios."
