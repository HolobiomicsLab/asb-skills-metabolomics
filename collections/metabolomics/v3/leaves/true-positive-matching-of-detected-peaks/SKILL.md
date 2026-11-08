---
name: true-positive-matching-of-detected-peaks
description: Use when evaluating peak-detection accuracy of an LC-MS processing tool
  on synthetic mzML data with a known inserted-peak ground truth, and you need per-class
  true-positive rates.
when_to_use_negative:
- Input is already a validated feature table without raw mzML files — re-detection
  and matching would not reflect tool performance
- The dataset has no ground-truth inserted-peak list, so true-positive calling is
  impossible; use alternative QC metrics instead
- The goal is metabolite identification or annotation rather than peak-detection accuracy
  benchmarking
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MassCube
  role: Detects peaks from synthetic mzML files via mass-trace construction (signal
    clustering with Gaussian-filter-assisted edge detection, sigma = 1.2, prominence
    = 0.1) whose output is matched against the ground-truth list
  repo: https://github.com/huaxuyu/masscube/
- name: Python
  role: Runtime environment for installing MassCube, running the untargeted workflow,
    performing the m/z and RT matching, and writing the accuracy CSV
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-60640-5
    title: MassCube improves accuracy for metabolomics data processing from raw files
      to phenotype classifiers
  source_content_hash: sha256:a4509ea4782180c9449c4d2723ca422794cf6eb6d38f2ba2686a74c1ef878ad3
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
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# true-positive matching of detected peaks

## Summary

A benchmark-validation procedure that matches peaks detected by MassCube's untargeted workflow against a published ground-truth list of inserted peaks (m/z + retention time agreement) to compute true-positive accuracy for single, double/co-eluting, and overall peak classes.

## When to use

Use when evaluating peak-detection accuracy of an LC-MS processing tool on synthetic mzML data with a known inserted-peak ground truth, and you need per-class true-positive rates.

## When NOT to use

- Input is already a validated feature table without raw mzML files — re-detection and matching would not reflect tool performance
- The dataset has no ground-truth inserted-peak list, so true-positive calling is impossible; use alternative QC metrics instead
- The goal is metabolite identification or annotation rather than peak-detection accuracy benchmarking

## Inputs

- Synthetic mzML benchmark files with known inserted peaks (Zenodo deposit 14159704)
- Published inserted-peak ground-truth list (13,500 single peaks + 13,500 double/co-eluting peaks)
- MassCube feature table produced by the untargeted metabolomics workflow

## Outputs

- peak_detection_accuracy.csv with true-positive accuracy for single peaks, double (co-eluting) peaks, and overall
- Per-file true-positive assignments of detected peaks to inserted-peak entries

## How to apply

Install MassCube from PyPI ('pip install masscube') and download the synthetic mzML benchmark plus the inserted-peak ground-truth list from Zenodo deposit 14159704 (13,500 single peaks and 13,500 double/co-eluting peaks). Load each mzML file with read_raw_file_to_obj and run the untargeted workflow (UntargetedMetabolomicsWorkflow / process_single_file), which detects peaks by constructing mass traces through signal clustering with Gaussian-filter-assisted edge detection using sigma = 1.2 and prominence = 0.1. For each detected feature, call it a true positive when both its m/z and retention time agree with an entry in the inserted-peak list. Compute accuracy separately for single inserted peaks, double (co-eluting) inserted peaks, and overall across both classes, then write the three values to peak_detection_accuracy.csv. Rationale: matching on both m/z and RT axes avoids crediting spurious detections, and class-stratified reporting exposes degradation on co-eluting peaks that an overall average would mask.

## Related tools

- **MassCube** (Detects peaks from synthetic mzML files via mass-trace construction (signal clustering with Gaussian-filter-assisted edge detection, sigma = 1.2, prominence = 0.1) whose output is matched against the ground-truth list) — https://github.com/huaxuyu/masscube/
- **Python** (Runtime environment for installing MassCube, running the untargeted workflow, performing the m/z and RT matching, and writing the accuracy CSV)

## Examples

```
from masscube import read_raw_file_to_obj, UntargetedMetabolomicsWorkflow; data = read_raw_file_to_obj('synthetic_benchmark.mzML'); wf = UntargetedMetabolomicsWorkflow(); features = wf.process_single_file(data)
```

## Evaluation signals

- peak_detection_accuracy.csv contains exactly three rows/values: single-peak accuracy, double-peak accuracy, and overall accuracy
- On this benchmark, expected reference values are mean double-peak accuracy 95.2% and overall average accuracy 96.5% for MassCube; large deviations indicate a mismatched ground-truth list or changed peak-detection parameters
- Every true-positive call must satisfy agreement on both m/z and retention time; peaks matching on only one axis must not be counted
- The number of matched inserted-peak entries cannot exceed 13,500 single + 13,500 double ground-truth peaks
- Detected-peak counts should be consistent between the feature table and the matching report (no silent peak dropping)

## Limitations

- Human-defined rules for peak shapes, reflected in algorithmic parameters such as sigma and prominence, cannot account for all peak-shape scenarios
- Synthetic data cannot capture characteristics of convoluted experimental data such as varying scan numbers across peaks and peak tailing
- External validated benchmarking datasets with large-scale ground-truth data remain scarce beyond this synthetic deposit

## Evidence

- [results] On the synthetic mzML benchmark, MassCube achieved the highest double-peak accuracy (mean 95.2%) and overall average accuracy of 96.5%: [evidence span withheld — anchor status: not_found]
- [abstract] peaks are detected by constructing mass traces through signal clustering and Gaussian-filter assisted edge detection: "peaks are detected by constructing mass traces through signal clustering and Gaussian-filter assisted edge detection"
- [methods] Feature detection, gap filling, and aligned feature grouping then reload this cache instead of reparsing the source.: "Feature detection, gap filling, and aligned feature grouping then reload this cache instead of reparsing the source."
- [abstract] Peak detection in MassCube achieves 100% signal coverage with comprehensive reporting of chromatographic metadata: "Peak detection in MassCube achieves 100% signal coverage with comprehensive reporting of chromatographic metadata for quality assurance."
- [discussion] Human-defined rules for peak shapes, reflected in algorithmic parameters, cannot account for all such scenarios.: "Human-defined rules for peak shapes, reflected in algorithmic parameters, cannot account for all such scenarios."
