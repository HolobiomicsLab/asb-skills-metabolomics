---
name: chromatographic-peak-shape-accuracy-benchmarking
description: Use when a peak-detection algorithm needs quantitative accuracy assessment
  against ground-truth manually labeled EIC peak-shape classes (double vs single peak)
  on experimental LC-MS data.
when_to_use_negative:
- The input is already an aligned feature table rather than raw LC-MS data requiring
  peak detection.
- Ground-truth peak-shape labels are unavailable, so no manual-label accuracy check
  can be computed.
- The goal is metabolite identification or annotation accuracy rather than chromatographic
  peak-shape detection accuracy.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MassCube
  role: Peak-detection tool under test; detects peaks via mass-trace construction
    with signal clustering and Gaussian-filter-assisted edge detection, then peak
    grouping
  repo: https://github.com/huaxuyu/masscube
- name: Python
  role: Environment for installing MassCube from PyPI and computing per-category accuracy
    scores
- name: MS-DIAL
  role: Comparison baseline whose peak-detection accuracy is benchmarked against the
    manually labeled EICs
- name: MZmine3
  role: Comparison baseline whose peak-detection accuracy is benchmarked against the
    manually labeled EICs
- name: XCMS
  role: Comparison baseline whose peak-detection accuracy is benchmarked against the
    manually labeled EICs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-60640-5
    title: MassCube improves accuracy for metabolomics data processing from raw files
      to phenotype classifiers
  source_content_hash: sha256:79fd4212cf74eb10e0190255f8ff8d449a74a94f406a9a5158780196b011b97a
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# chromatographic-peak-shape-accuracy-benchmarking

## Summary

Benchmarks an LC-MS peak-detection tool against manually labeled extracted ion chromatograms (EICs) by classifying detected peaks as double-peak or single-peak and computing per-category accuracy. Used here to reproduce MassCube's reported 93.5% double-peak and 97.3% single-peak accuracy versus MS-DIAL, MZmine3, and xcms.

## When to use

Use when a peak-detection algorithm needs quantitative accuracy assessment against ground-truth manually labeled EIC peak-shape classes (double vs single peak) on experimental LC-MS data.

## When NOT to use

- The input is already an aligned feature table rather than raw LC-MS data requiring peak detection.
- Ground-truth peak-shape labels are unavailable, so no manual-label accuracy check can be computed.
- The goal is metabolite identification or annotation accuracy rather than chromatographic peak-shape detection accuracy.

## Inputs

- Zenodo record 14159704: 722 manually labeled EIC PNG images with metadata list of ground-truth peak-shape classes (double-peak vs single-peak)
- Experimental Orbitrap/QTOF LC-MS raw files (e.g. mzML) underlying the labeled EICs
- Peak-detection tool outputs from MassCube, MS-DIAL, MZmine3, or xcms

## Outputs

- peak_detection_accuracy.csv with double-peak and single-peak accuracy scores and per-image agreement flags
- Per-category accuracy values comparable to reported references (double-peak 93.5%, single-peak 97.3%)

## How to apply

Install the tool under test (e.g. 'pip install masscube') and download Zenodo record 14159704, which contains 722 pre-labeled EIC PNG images plus a metadata list recording the ground-truth peak-shape class (double-peak or single-peak) for each image. Run the tool's untargeted LC-MS workflow — for MassCube, peaks are detected by constructing mass traces through signal clustering and Gaussian-filter assisted edge detection, followed by peak grouping — on the associated experimental Orbitrap/QTOF raw files so that every labeled EIC is covered by a detected peak. Match each detected peak back to its labeled EIC via the metadata list and assign the software's predicted peak-shape class. Compute per-category accuracy as the fraction of labeled EICs whose predicted class agrees with the ground-truth label, reporting double-peak accuracy (reference: 93.5%) and single-peak accuracy (reference: 97.3%) separately, and write per-image agreement flags alongside the summary scores.

## Related tools

- **MassCube** (Peak-detection tool under test; detects peaks via mass-trace construction with signal clustering and Gaussian-filter-assisted edge detection, then peak grouping) — https://github.com/huaxuyu/masscube
- **Python** (Environment for installing MassCube from PyPI and computing per-category accuracy scores)
- **MS-DIAL** (Comparison baseline whose peak-detection accuracy is benchmarked against the manually labeled EICs)
- **MZmine3** (Comparison baseline whose peak-detection accuracy is benchmarked against the manually labeled EICs)
- **XCMS** (Comparison baseline whose peak-detection accuracy is benchmarked against the manually labeled EICs)

## Examples

```
pip install masscube
```

## Evaluation signals

- Double-peak accuracy computed over the 722 labeled EICs matches the reported reference of 93.5% for MassCube.
- Single-peak accuracy matches the reported reference of 97.3% for MassCube.
- Every one of the 722 labeled EICs is matched to exactly one detected peak via the metadata list (100% signal coverage expected).
- peak_detection_accuracy.csv contains per-image agreement flags for all 722 EICs, not only summary scores.
- Re-running the same benchmark with MS-DIAL, MZmine3, or xcms yields lower accuracy than the MassCube result.

## Limitations

- Human-defined rules for peak shapes, reflected in algorithmic parameters, cannot account for all peak-shape scenarios.
- Convoluted experimental data display characteristics that are difficult to explore systematically with synthetic data, such as varying scan numbers across peaks and peak tailing.
- External validated benchmarking datasets with large-scale ground-truth data are still missing beyond this labeled set.

## Evidence

- [results] MassCube achieved the highest accuracy for double peaks (93.5%) and single peaks (97.3%), outperforming MS-DIAL, MZmine3, and xcms on the 722 manually labeled extracted ion chromatograms.: [evidence span withheld — anchor status: not_found]
- [abstract] Peak detection via mass traces: "peaks are detected by constructing mass traces through signal clustering and Gaussian-filter assisted edge detection"
- [abstract] Peak detection coverage claim: "Peak detection in MassCube achieves 100% signal coverage with comprehensive reporting of chromatographic metadata"
- [intro] Benchmarking gap rationale: "Existing software has often not been thoroughly validated and benchmarked for data processing efficiency and accuracy"
- [discussion] Peak-shape rule limitation: "Human-defined rules for peak shapes, reflected in algorithmic parameters, cannot account for all such scenarios."
- [readme] Installation command from README: [evidence span withheld — anchor status: not_found]
