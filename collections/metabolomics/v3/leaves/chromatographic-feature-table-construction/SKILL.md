---
name: chromatographic-feature-table-construction
description: Use when starting from raw LC-MS/MS MS1 scans (mzML or Bruker .d) and
  a feature table with complete signal coverage is needed before adduct grouping,
  alignment, or annotation.
when_to_use_negative:
- Input is already an aligned feature table — downstream steps like gap filling or
  aligned feature grouping reload the .mcraw cache instead
- Peak detection requiring strict peak-shape quality filtering of features at detection
  time — MassCube assigns every signal to a feature and imposes no peak-shape requirements;
  shape-based curation must happen elsewhere
- Bruker .d data outside the supported profile (e.g. non-TDF2 compression, MsMsType
  other than 8, non-model-1 m/z calibration, or per-frame pressure-compensated 1/K0)
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MassCube
  role: Python framework implementing ROI-based MS1 signal clustering and mass-trace
    construction for peak detection
  repo: https://github.com/huaxuyu/masscube
- name: Python
  role: Language ecosystem in which MassCube is implemented and invoked (pip-installed
    masscube package)
  repo: https://github.com/huaxuyu/masscube/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-60640-5
    title: MassCube improves accuracy for metabolomics data processing from raw files
      to phenotype classifiers
  source_content_hash: sha256:53290404d8c90fea1022ac62a511b7928d5379f8a61a63c865209a4c239d4cf8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/huaxuyu/masscube
  tools:
  - MassCube
  - Python
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# chromatographic-feature-table-construction

## Summary

Reconstructs MassCube's ROI-based MS1 feature detection: cluster m/z signals across continuous MS1 scans into unique-ion ROIs so that 100% of detected MS1 signals are assigned to a feature, with no requirements on peak shape, signal fluctuation, or scan number.

## When to use

Use when starting from raw LC-MS/MS MS1 scans (mzML or Bruker .d) and a feature table with complete signal coverage is needed before adduct grouping, alignment, or annotation.

## When NOT to use

- Input is already an aligned feature table — downstream steps like gap filling or aligned feature grouping reload the .mcraw cache instead
- Peak detection requiring strict peak-shape quality filtering of features at detection time — MassCube assigns every signal to a feature and imposes no peak-shape requirements; shape-based curation must happen elsewhere
- Bruker .d data outside the supported profile (e.g. non-TDF2 compression, MsMsType other than 8, non-model-1 m/z calibration, or per-frame pressure-compensated 1/K0)

## Inputs

- Raw LC-MS data file in mzML format
- Bruker timsTOF TDF2 .d directory (DDA-PASEF, compression type 2, m/z calibration model 1, static TIMS calibration model 2)

## Outputs

- Single-file TXT feature table with one row per unique m/z signal (ROI), including per-scan presence/intensity and a peak_shape column placeholder
- Versioned .mcraw workflow cache under the project tmp directory for reuse by gap filling and aligned feature grouping

## How to apply

Load MS1 scans from the raw mzML (or supported Bruker TDF2 .d) file using MassCube's read_raw_file_to_obj, extracting each scan's m/z and intensity signal arrays. Initialize one ROI per m/z signal detected in the first MS1 scan, then for each subsequent scan match incoming signals to open ROIs within a 0.01 Da m/z tolerance: extend matched ROIs and open new ROIs for unmatched signals. Close any ROI that has not matched a signal for 30 consecutive scans (gap threshold). MassCube constructs mass traces through signal clustering with Gaussian-filter assisted edge detection, deliberately imposing no peak-shape, signal-fluctuation, or minimum-scan-count requirements so every MS1 signal is assigned — the design choice behind its claimed 100% signal coverage. Finally assemble the feature table with one row per ROI, recording per-scan presence and intensity so each ROI's intensities form its extracted ion chromatogram, and write the single-file TXT output (retaining the peak_shape column placeholder; set Params.output_peak_shape=True only if full traces are needed).

## Related tools

- **MassCube** (Python framework implementing ROI-based MS1 signal clustering and mass-trace construction for peak detection) — https://github.com/huaxuyu/masscube
- **Python** (Language ecosystem in which MassCube is implemented and invoked (pip-installed masscube package)) — https://github.com/huaxuyu/masscube/

## Examples

```
from masscube import read_raw_file_to_obj; data = read_raw_file_to_obj("sample.d")
```

## Evaluation signals

- The TXT feature table exists and parses as tabular text
- Every MS1 scan signal is represented by exactly one ROI row (100% signal coverage invariant)
- No open ROI remains unextended past the 30-consecutive-scan gap limit
- Single-file TXT output retains the peak_shape column for compatibility even when Params.output_peak_shape is False
- A versioned .mcraw cache is created under the project tmp directory and reused by downstream feature detection, gap filling, and aligned feature grouping

## Limitations

- Human-defined rules for peak shapes, reflected in algorithmic parameters, cannot account for all peak-shape scenarios
- Currently supported Bruker data are limited to TDF2 compression, DDA-PASEF (MsMsType=8), m/z calibration model 1 with dC2=0 and C3=C4=0, and static TIMS calibration model 2; reported 1/K0 excludes the vendor's per-frame pressure compensation
- Single-file TXT output does not embed full chromatographic traces by default; original vendor data should still be retained as the archival source
- Rate-of-change-based feature detection approaches in classic software are limited in robustness and sensitive to noise — the rationale for this ROI-clustering design, but shape-based false-positive recognition is delegated to downstream metadata

## Evidence

- [abstract] Peak detection in MassCube achieves 100% signal coverage with comprehensive reporting of chromatographic metadata: "Peak detection in MassCube achieves 100% signal coverage with comprehensive reporting of chromatographic metadata"
- [abstract] peaks are detected by constructing mass traces through signal clustering and Gaussian-filter assisted edge detection: "peaks are detected by constructing mass traces through signal clustering and Gaussian-filter assisted edge detection"
- [readme] Single-file TXT output keeps the `peak_shape` column for compatibility but does not embed full chromatographic traces by default.: "Single-file TXT output keeps the `peak_shape` column for compatibility but
  does not embed full chromatographic traces by default. Set
  `Params.output_peak_shape = True` when those traces are"
- [methods] Feature detection, gap filling, and aligned feature grouping then reload this cache instead of reparsing the source.: "Feature detection, gap filling, and aligned feature grouping then reload this
  cache instead of reparsing the source."
- [readme] The untargeted workflow decodes each mzML or supported Bruker `.d` source once and stores a versioned `.mcraw` cache under the project `tmp` directory.: "The untargeted workflow decodes each mzML or supported Bruker `.d` source once
and stores a versioned `.mcraw` cache under the project `tmp` directory."
