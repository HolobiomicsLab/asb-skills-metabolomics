---
name: roi-extension-mass-trace-building
description: Use when starting nontargeted LC-MS feature detection from raw MS1 scans
  and every detected m/z signal must be assigned to a feature before peak-shape or
  intensity filtering.
when_to_use_negative:
- Input is already an aligned feature table or gap-filled data matrix — ROI extension
  applies to raw MS1 scans only
- You need isotope/adduct grouping or MS/MS annotation — those are downstream steps
  that consume the ROI feature table
- You require a graphical interface for peak detection — MassCube has no GUI in the
  current version
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MassCube
  role: Python package providing read_raw_file_to_obj and the ROI clustering/extension
    algorithm for building mass traces from MS1 signals
  repo: https://github.com/huaxuyu/masscube
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-60640-5
    title: MassCube improves accuracy for metabolomics data processing from raw files
      to phenotype classifiers
  source_content_hash: sha256:7605b38326a58c29bbcdb7fc6d6d136c8d85d5d4381b4b28aafa59773e539d00
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/huaxuyu/masscube
  tools:
  - MassCube
  - Python
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# ROI extension mass-trace building

## Summary

Cluster m/z signals across continuous MS1 scans into open regions of interest (ROIs) without imposing peak-shape or fluctuation requirements, achieving 100% signal coverage for feature detection in MassCube.

## When to use

Use when starting nontargeted LC-MS feature detection from raw MS1 scans and every detected m/z signal must be assigned to a feature before peak-shape or intensity filtering.

## When NOT to use

- Input is already an aligned feature table or gap-filled data matrix — ROI extension applies to raw MS1 scans only
- You need isotope/adduct grouping or MS/MS annotation — those are downstream steps that consume the ROI feature table
- You require a graphical interface for peak detection — MassCube has no GUI in the current version

## Inputs

- Raw LC-MS/MS data in mzML format
- Bruker timsTOF .d directories (DDA-PASEF TDF2, calibrated m/z, inverse reduced ion mobility)

## Outputs

- Feature table with one row per unique m/z signal (ROI), recording per-scan presence and intensity
- Single-file TXT output retaining a peak_shape column for compatibility
- Versioned .mcraw workflow cache under the project tmp directory

## How to apply

Read the raw mzML (or supported Bruker .d TDF2) file with MassCube's read_raw_file_to_obj and extract each MS1 scan's m/z and intensity signal arrays. Initialize one ROI per signal detected in the first MS1 scan, then for each subsequent scan match incoming signals to open ROIs within a 0.01 Da m/z tolerance — extending matched ROIs and opening new ROIs for unmatched signals. Close any ROI that has not matched a signal for 30 consecutive scans (gap threshold). Assemble the feature table with one row per unique m/z signal (ROI) so each ROI's intensities form its extracted ion chromatogram; the rationale is that clustering all signals to unique ions guarantees no signal is left unassigned, unlike rate-of-change methods sensitive to noise.

## Related tools

- **MassCube** (Python package providing read_raw_file_to_obj and the ROI clustering/extension algorithm for building mass traces from MS1 signals) — https://github.com/huaxuyu/masscube

## Examples

```
from masscube import read_raw_file_to_obj; data = read_raw_file_to_obj('sample.mzML');
```

## Evaluation signals

- Every MS1 scan signal is represented by exactly one ROI row (100% signal coverage)
- The feature table exists and parses as tabular text (single-file TXT)
- No open ROI remains unextended past the 30-scan gap limit
- No requirement is imposed on peak shape, signal fluctuation, or scan number for ROI assignment

## Limitations

- ISFs cannot be automatically annotated when the parent ion is not detected due to complete fragmentation in the ion source
- Characteristics of convoluted experimental data (varying scan numbers across peaks, peak tailing) are difficult to explore systematically with synthetic data
- Single-file TXT output does not embed full chromatographic traces by default; set Params.output_peak_shape = True when those traces are explicitly needed
- The original vendor data should still be retained as the archival source even when the .mcraw cache is used

## Evidence

- [abstract] 100% signal coverage: "Peak detection in MassCube achieves 100% signal coverage with comprehensive reporting of chromatographic metadata"
- [abstract] signal clustering: "peaks are detected by constructing mass traces through signal clustering and Gaussian-filter assisted edge detection"
- [methods] feature detection: "Feature detection, gap filling, and aligned feature grouping"
- [readme] TXT output: "Single-file TXT output keeps the peak_shape column for compatibility but does not embed full chromatographic traces by default."
- [intro] rate-of-change limitation: "features in LC-MS experiments can be noisy and often lack"
- [readme] 1/K0 availability: "calibrated m/z and inverse reduced ion mobility (1/K0)"
