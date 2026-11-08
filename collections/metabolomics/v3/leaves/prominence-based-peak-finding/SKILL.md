---
name: prominence-based-peak-finding
description: Use when raw LC-MS ROI/mass-trace intensity arrays must be split into
  discrete chromatographic peak segments with edge boundaries, without smoothing biasing
  the reported peak shapes.
when_to_use_negative:
- Input is already a feature table or peak list rather than raw/mass-trace ROI intensity
  arrays — this skill segments chromatograms, not tabulated features.
- The goal is adduct grouping, in-source fragment annotation, or alignment — those
  are downstream MassCube steps that operate on already-segmented peaks.
- 'Output must be smoothed signal: MassCube deliberately reports raw data because
  smoothing may bias data structures; do not substitute smoothed intensities into
  the final report.'
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MassCube
  role: Provides read_raw_file_to_obj to load raw LC-MS files and the peak-detection
    module in which Gaussian-filter-assisted, prominence-based edge finding segments
    each ROI into chromatographic peaks
  repo: https://github.com/huaxuyu/masscube/
- name: Python
  role: Implementation language of MassCube and the environment in which the Gaussian
    filter (sigma=1.2) and prominence-ratio (ratio=0.1) edge detection are executed
  repo: https://github.com/huaxuyu/masscube/
- name: NumPy
  role: Array backend for the ROI intensity arrays and concatenated peak arrays; MassCube's
    .mcraw cache stores scan-varying values and peak arrays as NumPy columns
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-60640-5
    title: MassCube improves accuracy for metabolomics data processing from raw files
      to phenotype classifiers
  source_content_hash: sha256:9c732331fa5907689777fe4870f552c606cd6dfd4da47545c5c9875ad2bdb948
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/huaxuyu/masscube
  tools:
  - MassCube
  - NumPy
  - Python
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# prominence-based-peak-finding

## Summary

Reconstructs MassCube's chromatographic peak segmentation: apply a Gaussian filter (sigma=1.2) to each ROI intensity array, select candidate peak edges as local minima under a prominence-ratio criterion (ratio=0.1), then cut the RAW (unsmoothed) signal into segments so that final reports are not biased by smoothing.

## When to use

Use when raw LC-MS ROI/mass-trace intensity arrays must be split into discrete chromatographic peak segments with edge boundaries, without smoothing biasing the reported peak shapes.

## When NOT to use

- Input is already a feature table or peak list rather than raw/mass-trace ROI intensity arrays — this skill segments chromatograms, not tabulated features.
- The goal is adduct grouping, in-source fragment annotation, or alignment — those are downstream MassCube steps that operate on already-segmented peaks.
- Output must be smoothed signal: MassCube deliberately reports raw data because smoothing may bias data structures; do not substitute smoothed intensities into the final report.

## Inputs

- Raw LC-MS file (e.g. mzML or Bruker timsTOF .d / TDF2 directory) parsed into ROI/mass-trace intensity arrays (m/z, intensity columns)
- Adjustable Gaussian sigma parameter (paper-optimized value: sigma=1.2)
- Prominence-ratio threshold for edge selection (paper-optimized value: ratio=0.1)

## Outputs

- Per-ROI chromatographic peak segments with start/end edge indices, retention-time boundaries, apex intensity, and peak_shape
- Segment table file (e.g. peak_segments.csv) with edge-boundary fields per ROI

## How to apply

Load each mass-trace ROI intensity array (m/z plus intensity columns) from the raw LC-MS file — e.g. via MassCube's read_raw_file_to_obj — and apply a 1D Gaussian filter with sigma=1.2 solely to enable edge detection, never for the final reporting values. On the smoothed array, find candidate peak edges as local minima selected by a prominence-ratio criterion with ratio=0.1; the smoothing exists only to make these edge candidates robust to noise. Map the detected edge indices back onto the RAW (unsmoothed) intensity array and cut it into distinct chromatographic peak segments bounded by start/end edge indices. Record per-segment metadata (retention-time boundaries, apex intensity, peak_shape) and write the segment table, e.g. to peak_segments.csv. The rationale, per the paper, is that smoothing 'may bias data structures', so raw data must be used for final reports while Gaussian-filter-assisted detection achieves comprehensive peak segmentation.

## Related tools

- **MassCube** (Provides read_raw_file_to_obj to load raw LC-MS files and the peak-detection module in which Gaussian-filter-assisted, prominence-based edge finding segments each ROI into chromatographic peaks) — https://github.com/huaxuyu/masscube/
- **Python** (Implementation language of MassCube and the environment in which the Gaussian filter (sigma=1.2) and prominence-ratio (ratio=0.1) edge detection are executed) — https://github.com/huaxuyu/masscube/
- **NumPy** (Array backend for the ROI intensity arrays and concatenated peak arrays; MassCube's .mcraw cache stores scan-varying values and peak arrays as NumPy columns)

## Examples

```
from masscube import read_raw_file_to_obj; data = read_raw_file_to_obj('sample.d'); scan = data.scans[data.ms1_idx_arr[0]]; print(scan.signals)
```

## Evaluation signals

- peak_segments.csv exists and contains non-empty segments with start/end edge-boundary fields per ROI
- Segments cover the ROI signal consistent with the paper's claim that 'Peak detection in MassCube achieves 100% signal coverage with comprehensive reporting of chromatographic metadata'
- Reported intensities and peak shapes match the RAW (unsmoothed) array, not the Gaussian-filtered array
- Number and positions of edges are reproducible when sigma is held at 1.2 and the prominence ratio at 0.1
- Each detected edge is a local minimum on the smoothed array satisfying the 0.1 prominence-ratio criterion

## Limitations

- Human-defined rules for peak shapes, reflected in algorithmic parameters, cannot account for all peak-shape scenarios — convoluted experimental data exhibit characteristics (varying scan numbers, peak tailing) hard to explore systematically with synthetic data.
- Smoothing may bias data structures, so the Gaussian-filtered signal must never be used for final reports; only edge detection may use it.
- Downstream steps (adduct/ISF grouping, alignment) require this segmentation first; wrong sigma or prominence ratio would propagate errors into feature tables.

## Evidence

- [abstract] Gaussian filter used only for edge detection; raw data used for final reports: "peaks are detected by constructing mass traces through signal clustering and Gaussian-filter assisted edge detection"
- [supplementary] Smoothing may bias data structures, so raw data are reported: [evidence span withheld — anchor status: not_found]
- [abstract] Peak detection achieves 100% signal coverage with chromatographic metadata: "Peak detection in MassCube achieves 100% signal coverage with comprehensive reporting of chromatographic metadata"
- [readme] README: MassCube provides highly accurate nontargeted peak detection and segmentation: "Highly accurate nontargeted peak detection and segmentation."
