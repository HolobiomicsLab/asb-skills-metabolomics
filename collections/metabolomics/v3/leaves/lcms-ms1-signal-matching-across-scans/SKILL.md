---
name: lcms-ms1-signal-matching-across-scans
description: Use when starting nontargeted LC-MS feature detection from raw MS1 scans
  (mzML or Bruker timsTOF .d) and you need 100% signal coverage rather than peak-shape-filtered
  peak picking.
when_to_use_negative:
- Input is already a processed feature table or aligned peak list — this skill operates
  on raw per-scan m/z/intensity signals, not on downstream feature tables.
- The goal is MS/MS-based metabolite identification — MassCube does not directly perform
  metabolite identification; this skill only constructs MS1 mass traces.
- Strict peak-shape filtering or quality-weighted feature detection is required (e.g.
  for a tightly curated targeted report) — this approach deliberately accepts noisy,
  non-Gaussian features to reach 100% signal coverage.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MassCube
  role: Python package whose untargeted peak detection module performs the scan-to-scan
    m/z signal matching into ROIs/mass traces and writes the TXT feature table.
  repo: https://github.com/huaxuyu/masscube
- name: Python
  role: Runtime for MassCube; read_raw_file_to_obj loads mzML or .d scans into data
    objects whose scans expose signals and inv_mobility arrays.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-60640-5
    title: MassCube improves accuracy for metabolomics data processing from raw files
      to phenotype classifiers
  source_content_hash: sha256:1c15bf415b3143dd3f53e00dc2a951d17b57df02e75b3f82909fbc0d72d28cfe
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
yes_rate: 0.9090909090909091
n_claims: 11
---

# lcms-ms1-signal-matching-across-scans

## Summary

MassCube builds LC-MS1 features by matching m/z signals scan-to-scan into mass traces (ROIs) with a 0.01 Da tolerance, so every detected MS1 signal is assigned to exactly one feature without imposing peak-shape, fluctuation, or minimum-scan-number requirements.

## When to use

Use when starting nontargeted LC-MS feature detection from raw MS1 scans (mzML or Bruker timsTOF .d) and you need 100% signal coverage rather than peak-shape-filtered peak picking.

## When NOT to use

- Input is already a processed feature table or aligned peak list — this skill operates on raw per-scan m/z/intensity signals, not on downstream feature tables.
- The goal is MS/MS-based metabolite identification — MassCube does not directly perform metabolite identification; this skill only constructs MS1 mass traces.
- Strict peak-shape filtering or quality-weighted feature detection is required (e.g. for a tightly curated targeted report) — this approach deliberately accepts noisy, non-Gaussian features to reach 100% signal coverage.

## Inputs

- Raw LC-MS1 data in mzML format (m/z and intensity signal arrays per MS1 scan)
- Bruker timsTOF TDF2 DDA-PASEF .d directories with calibrated m/z (read via read_raw_file_to_obj; optionally with inverse reduced ion mobility 1/K0)

## Outputs

- Single-file TXT feature table with one row per unique m/z signal (ROI/feature), per-scan presence and intensity columns, and a peak_shape column placeholder
- Versioned .mcraw workflow cache under the project tmp directory storing decoded MS1/MS2 metadata and concatenated peak arrays for reuse by gap filling and aligned feature grouping

## How to apply

Load raw MS1 scans with MassCube's read_raw_file_to_obj so each scan exposes its m/z and intensity signal arrays. Initialize one region of interest (ROI) per m/z signal detected in the first MS1 scan. For each subsequent scan, match incoming signals to open ROIs within a 0.01 Da m/z tolerance, extending matched ROIs and opening new ROIs for unmatched signals; this scan-to-scan clustering constructs mass traces through signal clustering and Gaussian-filter assisted edge detection. Close any ROI that has not matched a signal for 30 consecutive scans (the gap threshold). Assemble a single-file TXT feature table with one row per unique m/z signal, recording per-scan presence and intensity so each ROI's intensities form its extracted ion chromatogram; the peak_shape column is retained for compatibility but is not populated unless Params.output_peak_shape = True. The rationale is that features in LC-MS experiments can be noisy and often lack clean Gaussian shapes, so avoiding peak-shape, signal-fluctuation, or scan-number requirements guarantees that every detected MS1 signal is assigned to a feature for 100% coverage.

## Related tools

- **MassCube** (Python package whose untargeted peak detection module performs the scan-to-scan m/z signal matching into ROIs/mass traces and writes the TXT feature table.) — https://github.com/huaxuyu/masscube
- **Python** (Runtime for MassCube; read_raw_file_to_obj loads mzML or .d scans into data objects whose scans expose signals and inv_mobility arrays.)

## Examples

```
from masscube import read_raw_file_to_obj; data = read_raw_file_to_obj('sample.mzML')
```

## Evaluation signals

- The TXT feature table exists and parses as tabular text with one row per unique m/z signal and per-scan intensity columns.
- Every MS1 scan signal is represented by exactly one ROI row — 100% signal coverage with no unassigned signals.
- No ROI remains open after 30 consecutive scans without a matched signal (gap threshold respected).
- Cross-scan m/z matching stays within the 0.01 Da tolerance; matched signals do not jump outside it.
- A .mcraw cache is written under the project tmp directory so downstream gap filling and aligned feature grouping reload it instead of reparsing the source.

## Limitations

- Single-file TXT output keeps the peak_shape column for compatibility but does not embed full chromatographic traces by default; set Params.output_peak_shape = True when traces are explicitly needed.
- Human-defined rules for peak shapes, reflected in algorithmic parameters, cannot account for all peak-shape scenarios.
- The workflow cache (.mcraw) is not an archival substitute — the original vendor data should still be retained as the archival source.
- The reported 1/K0 applies the stored static mobility calibration but not the vendor's proprietary per-frame pressure compensation.

## Evidence

- [abstract] peaks are detected by constructing mass traces through signal clustering and Gaussian-filter assisted edge detection: "peaks are detected by constructing mass traces through signal clustering and Gaussian-filter assisted edge detection"
- [abstract] Peak detection in MassCube achieves 100% signal coverage: "Peak detection in MassCube achieves 100% signal coverage"
- [full_text] MassCube clusters all detected MS signals to unique ions without imposing peak shape/signal fluctuation/scan number requirements: [evidence span withheld — anchor status: not_found]
- [readme] read_raw_file_to_obj loads scans with m/z and intensity signals: "data = read_raw_file_to_obj("sample.d")
scan = data.scans[data.ms1_idx_arr[0]]
print(scan.signals)             # columns: m/z, intensity"
- [readme] downstream steps reload the .mcraw cache instead of reparsing the source: "Feature detection, gap filling, and aligned feature grouping then reload this
cache instead of reparsing the source."
