---
name: synthetic-ground-truth-accuracy-evaluation
description: Use when quantifying true-positive peak-detection accuracy of an LC-MS
  feature-detection tool against a synthetic mzML benchmark with known inserted single
  and co-eluting (double) peaks, e.g. for benchmarking against MZmine3, xcms, or MS-DIAL.
when_to_use_negative:
- Input is real experimental LC-MS data with no ground-truth list of inserted peaks
  — accuracy cannot be computed without known true peaks
- Input is already an aligned feature table; this skill evaluates raw-file peak detection,
  not downstream table quality
- Goal is compound identification or MS/MS annotation accuracy rather than peak-detection
  accuracy — MassCube does not directly perform metabolite identification
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3368
- http://edamontology.org/topic_3520
tools:
- name: MassCube
  role: Python package that reads each synthetic mzML file and runs the untargeted
    peak-detection workflow (sigma = 1.2, prominence = 0.1) whose true-positive rates
    are scored
  repo: https://github.com/huaxuyu/masscube
- name: Python
  role: Runtime environment for installing MassCube via pip and computing/matching
    true positives against the ground-truth list
- name: MZmine3
  role: Comparator tool processed on the same synthetic mzML benchmark to contextualize
    accuracy (MassCube outperformed it)
- name: MS-DIAL
  role: Comparator tool processed on the same synthetic mzML benchmark to contextualize
    accuracy
- name: XCMS
  role: Comparator tool processed on the same synthetic mzML benchmark to contextualize
    accuracy
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-60640-5
    title: MassCube improves accuracy for metabolomics data processing from raw files
      to phenotype classifiers
  source_content_hash: sha256:cb5fc9a1f44bbfe8f353374234fb24401d8cbf77e939a2f2082eb744b5897e7e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/huaxuyu/masscube
  tools:
  - MS-DIAL
  - MZmine3
  - MassCube
  - Python
  - XCMS
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3368
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# synthetic-ground-truth-accuracy-evaluation

## Summary

Quantifies true-positive peak-detection accuracy of an LC-MS feature-detection algorithm by processing synthetic mzML files containing known inserted single and double (co-eluting) peaks and matching detected features against the published ground-truth list.

## When to use

Use when quantifying true-positive peak-detection accuracy of an LC-MS feature-detection tool against a synthetic mzML benchmark with known inserted single and co-eluting (double) peaks, e.g. for benchmarking against MZmine3, xcms, or MS-DIAL.

## When NOT to use

- Input is real experimental LC-MS data with no ground-truth list of inserted peaks — accuracy cannot be computed without known true peaks
- Input is already an aligned feature table; this skill evaluates raw-file peak detection, not downstream table quality
- Goal is compound identification or MS/MS annotation accuracy rather than peak-detection accuracy — MassCube does not directly perform metabolite identification

## Inputs

- Synthetic mzML benchmark files containing inserted single and double (co-eluting) peaks
- Published inserted-peak ground-truth list (m/z and retention time of inserted peaks) from Zenodo deposit 14159704
- Detected peak/feature list produced by the peak-detection algorithm under evaluation

## Outputs

- peak_detection_accuracy.csv with true-positive accuracy for single peaks, double peaks, and overall
- Per-class true-positive rates suitable for cross-tool comparison (MassCube, MZmine3, xcms, MS-DIAL)

## How to apply

Install MassCube from PyPI ('pip install masscube') and download the synthetic mzML benchmark files plus the published inserted-peak ground-truth list from Zenodo deposit 14159704 (13,500 single peaks and 13,500 double/co-eluting peaks). Load each mzML file with MassCube's read_raw_file_to_obj and run the untargeted metabolomics workflow (UntargetedMetabolomicsWorkflow / process_single_file), which constructs mass traces by signal clustering with Gaussian-filter-assisted edge detection using peak-detection parameters sigma = 1.2 and prominence = 0.1. Match each detected peak to the inserted-peak list using agreement in both m/z and retention time to call a true positive. Compute true-positive accuracy separately for single inserted peaks, double (co-eluting) inserted peaks, and overall across both classes, then write the three accuracy values to peak_detection_accuracy.csv. Compare the resulting accuracies against the other tools (MZmine3, xcms, MS-DIAL) processed on the same benchmark; MassCube reported the highest double-peak accuracy (mean 95.2%) and 96.5% overall average accuracy.

## Related tools

- **MassCube** (Python package that reads each synthetic mzML file and runs the untargeted peak-detection workflow (sigma = 1.2, prominence = 0.1) whose true-positive rates are scored) — https://github.com/huaxuyu/masscube
- **Python** (Runtime environment for installing MassCube via pip and computing/matching true positives against the ground-truth list)
- **MZmine3** (Comparator tool processed on the same synthetic mzML benchmark to contextualize accuracy (MassCube outperformed it))
- **MS-DIAL** (Comparator tool processed on the same synthetic mzML benchmark to contextualize accuracy)
- **XCMS** (Comparator tool processed on the same synthetic mzML benchmark to contextualize accuracy)

## Examples

```
pip install masscube; python -c "from masscube import read_raw_file_to_obj; data = read_raw_file_to_obj('synthetic_sample.mzML')"
```

## Evaluation signals

- Verify the benchmark contains 13,500 single inserted peaks and 13,500 double/co-eluting inserted peaks before scoring
- A detected peak counts as a true positive only when both m/z and retention time agree with an inserted peak; report per-class TPR, not a single pooled rate
- Expected reference values on this benchmark: MassCube double-peak accuracy mean 95.2% and overall average accuracy 96.5%, higher than MZmine3, xcms, and MS-DIAL
- All three accuracy values (single, double, overall) must be present in peak_detection_accuracy.csv and bounded in [0, 1]
- Re-running with identical sigma = 1.2 and prominence = 0.1 parameters should reproduce the same accuracy values

## Limitations

- Convoluted experimental data display characteristics that are difficult to explore systematically with synthetic data, such as varying scan numbers across peaks and peak tailing
- Human-defined rules for peak shapes, reflected in algorithmic parameters, cannot account for all peak-shape scenarios
- Results are specific to the synthetic benchmark; external validated datasets with large-scale ground-truth data remain scarce

## Evidence

- [abstract] peaks are detected by constructing mass traces through signal clustering and Gaussian-filter assisted edge detection: "peaks are detected by constructing mass traces through signal clustering and Gaussian-filter assisted edge detection"
- [abstract] MassCube outperforms MS-DIAL, MZmine3 or XCMS for speed, isomer detection, and accuracy.: "MassCube outperforms MS-DIAL, MZmine3 or XCMS for speed, isomer detection, and accuracy."
- [intro] Existing software has often not been thoroughly validated and benchmarked for data processing efficiency and accuracy: "Existing software has often not been thoroughly validated and benchmarked for data processing efficiency and accuracy"
- [discussion] convoluted experimental data also display characteristics that are difficult to explore in a systematic way using synthetic data: "convoluted experimental data also display characteristics that are difficult to explore in a systematic way using synthetic data"
- [readme] masscube is an integrated Python package for liquid chromatography-mass spectrometry (LC-MS) data processing.: "**masscube** is an integrated Python package for liquid chromatography-mass spectrometry (LC-MS) data processing."
