---
name: gaussian-filter-signal-smoothing
description: 'Use when you have raw LC-MS ROI intensity arrays (m/z + intensity columns
  from a mass trace) and need robust chromatographic peak segmentation: smoothing
  precedes edge detection, never final quantification.'
when_to_use_negative:
- Input is already a segmented feature table (e.g. aligned feature table or gap-filled
  peak table) — smoothing/edge detection would re-segment existing peaks rather than
  refine raw traces.
- 'You need final reported peak intensities or peak areas for downstream statistics:
  MassCube explicitly uses the raw (unsmoothed) data for final reports because smoothing
  may bias data structures.'
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MassCube
  role: Python library whose `read_raw_file_to_obj` exposes per-ROI m/z/intensity
    signal columns; its untargeted peak detection applies Gaussian-filter assisted
    edge detection with σ = 1.2 and ratio = 0.1 prominence on mass traces, then cuts
    raw intensities into peak segments.
  repo: https://github.com/huaxuyu/masscube
- name: Python
  role: Language in which the 1-D Gaussian filter, local-minima prominence edge detection
    (ratio = 0.1), and raw-intensity segmentation are implemented.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-60640-5
    title: MassCube improves accuracy for metabolomics data processing from raw files
      to phenotype classifiers
  source_content_hash: sha256:52c04329dfa0ae4a63716bd387739fdd7281f7564f7e78334a42c6c5b54b71d7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/huaxuyu/masscube
  tools:
  - MassCube
  - NumPy
  - Python
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# gaussian-filter-signal-smoothing

## Summary

A 1-D Gaussian filter (σ = 1.2) is applied solely to LC-MS ROI intensity arrays to assist chromatographic peak edge detection in MassCube; the raw, unsmoothed signal is retained for all final reporting because smoothing can bias data structures.

## When to use

Use when you have raw LC-MS ROI intensity arrays (m/z + intensity columns from a mass trace) and need robust chromatographic peak segmentation: smoothing precedes edge detection, never final quantification.

## When NOT to use

- Input is already a segmented feature table (e.g. aligned feature table or gap-filled peak table) — smoothing/edge detection would re-segment existing peaks rather than refine raw traces.
- You need final reported peak intensities or peak areas for downstream statistics: MassCube explicitly uses the raw (unsmoothed) data for final reports because smoothing may bias data structures.

## Inputs

- Raw LC-MS ROI intensity array per mass trace (columns: m/z, intensity), e.g. from MassCube's `read_raw_file_to_obj` on an mzML or Bruker timsTOF `.d` (TDF2/DDA-PASEF) file

## Outputs

- peak_segments.csv with per-ROI peak segments: start/end edge indices, retention-time boundaries, apex intensity, and peak_shape metadata
- 100% signal coverage peak detection report (chromatographic metadata for quality assurance), written against raw rather than smoothed intensities

## How to apply

Load each ROI intensity array (e.g. via MassCube's `read_raw_file_to_obj`, which exposes scan signals as m/z/intensity columns) and apply a 1-D Gaussian filter with the article-optimized sigma value σ = 1.2 to suppress noise before edge detection. Find candidate peak edges as local minima on the smoothed array using a prominence-ratio criterion with ratio = 0.1. Map the detected edges back onto the RAW (unsmoothed) intensity array and cut it into distinct chromatographic peak segments bounded by start/end edge indices, recording per-segment metadata (retention-time boundaries, apex intensity, `peak_shape`). The rationale, per the abstract and README, is that 'peaks are detected by constructing mass traces through signal clustering and Gaussian-filter assisted edge detection', while final reports use raw data because smoothing may bias data structures.

## Related tools

- **MassCube** (Python library whose `read_raw_file_to_obj` exposes per-ROI m/z/intensity signal columns; its untargeted peak detection applies Gaussian-filter assisted edge detection with σ = 1.2 and ratio = 0.1 prominence on mass traces, then cuts raw intensities into peak segments.) — https://github.com/huaxuyu/masscube
- **Python** (Language in which the 1-D Gaussian filter, local-minima prominence edge detection (ratio = 0.1), and raw-intensity segmentation are implemented.)

## Examples

```
from masscube import read_raw_file_to_obj; from scipy.ndimage import gaussian_filter1d; data = read_raw_file_to_obj('sample.mzML'); sig = data.scans[data.ms1_idx_arr[0]].signals; sm = gaussian_filter1d(sig[:,1], sigma=1.2)
```

## Evaluation signals

- Verify peak_segments.csv exists, is non-empty, and every segment carries edge-boundary fields (start/end indices) per ROI.
- Confirm segmentation covers the ROI signal consistent with MassCube's reported 100% signal coverage for peak detection (no raw intensities left unassigned to a segment).
- Check that the smoothed array is used only for edge detection — final per-peak values (apex intensity, area) must derive from the raw intensity array, not the Gaussian-filtered one.
- Confirm σ = 1.2 and the prominence ratio = 0.1 were applied; σ = 1.2 is the article's optimized setting for noise suppression before edge detection.
- If using the `.mcraw` workflow cache, verify the cache version and that raw vendor files remain the archival source.

## Limitations

- Smoothing is only for edge detection; using smoothed data for final reports may bias data structures, so raw data must be reloaded for quantification.
- Human-defined rules for peak shapes, reflected in algorithmic parameters, cannot account for all peak-shape scenarios (e.g. varying scan numbers across peaks, peak tailing).
- Convoluted experimental data display characteristics difficult to explore systematically with synthetic data, so synthetic-benchmark tuning may not generalize.
- Currently supported Bruker data are TDF2 (`TimsCompressionType=2`), DDA-PASEF (`MsMsType=8`), m/z calibration model 1 with `dC2=0` and `C3=C4=0`, and static TIMS calibration model 2 — other formats may need the .mcraw cache or vendor SDK.

## Evidence

- [abstract] peaks are detected by constructing mass traces through signal clustering and Gaussian-filter assisted edge detection: "peaks are detected by constructing mass traces through signal clustering and Gaussian-filter assisted edge detection"
- [results] MassCube applies a Gaussian filter with an adjustable sigma value (optimized to σ = 1.2) solely for Gaussian-filter-assisted peak edge detection, and it uses the raw data instead of smoothed data for the final reports, since smoothing may bias data structures.: [evidence span withheld — anchor status: not_found]
- [abstract] Peak detection in MassCube achieves 100% signal coverage with comprehensive reporting of chromatographic metadata: "Peak detection in MassCube achieves 100% signal coverage with comprehensive reporting of chromatographic metadata"
- [readme] Single-file TXT output keeps the `peak_shape` column for compatibility but does not embed full chromatographic traces by default. Set `Params.output_peak_shape = True` when those traces are explicitly needed.: "Single-file TXT output keeps the `peak_shape` column for compatibility but does not embed full chromatographic traces by default."
