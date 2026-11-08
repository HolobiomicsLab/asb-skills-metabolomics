---
name: eic-image-label-matching
description: Use when raw Orbitrap/QTOF LC-MS data plus a curated set of manually
  labeled EIC images (double-peak vs single-peak ground truth) are available and peak-detection
  accuracy must be quantified per software tool.
when_to_use_negative:
- Input is an already-built feature table with no raw LC-MS data, since the skill
  requires running peak detection from raw files so every labeled EIC is covered by
  a detected peak
- Only synthetic benchmark data are available — the article notes 'convoluted experimental
  data also display characteristics that are difficult to explore in a systematic
  way using synthetic data'
- No manually curated ground-truth labels exist, since matching accuracy cannot be
  computed without the labeled EIC classes
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MassCube
  role: Python framework whose untargeted peak detection (mass-trace construction
    with signal clustering and Gaussian-filter-assisted edge detection, then peak
    grouping) supplies the detected peaks matched to the labeled EICs
  repo: https://github.com/huaxuyu/masscube
- name: MS-DIAL
  role: Comparator peak-detection tool benchmarked against MassCube on the same 722
    labeled EICs
- name: MZmine3
  role: Comparator peak-detection tool benchmarked against MassCube on the same 722
    labeled EICs
- name: XCMS
  role: Comparator peak-detection tool benchmarked against MassCube on the same 722
    labeled EICs
- name: Python
  role: Environment for installing MassCube via pip and computing per-category accuracy
    into CSV output
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-60640-5
    title: MassCube improves accuracy for metabolomics data processing from raw files
      to phenotype classifiers
  source_content_hash: sha256:dad76c456f69342643547e5116e18742df657ea0cc3a0072a4058612921b4eb9
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# eic-image-label-matching

## Summary

Benchmarks MassCube peak detection against manually labeled extracted ion chromatograms (EICs) by matching each detected peak to a ground-truth double-peak or single-peak label and computing per-category accuracy (MassCube: 93.5% double peaks, 97.3% single peaks; comparators MS-DIAL, MZmine3, XCMS).

## When to use

Use when raw Orbitrap/QTOF LC-MS data plus a curated set of manually labeled EIC images (double-peak vs single-peak ground truth) are available and peak-detection accuracy must be quantified per software tool.

## When NOT to use

- Input is an already-built feature table with no raw LC-MS data, since the skill requires running peak detection from raw files so every labeled EIC is covered by a detected peak
- Only synthetic benchmark data are available — the article notes 'convoluted experimental data also display characteristics that are difficult to explore in a systematic way using synthetic data'
- No manually curated ground-truth labels exist, since matching accuracy cannot be computed without the labeled EIC classes

## Inputs

- Experimental Orbitrap/QTOF LC-MS raw files (processed via MassCube)
- Zenodo record 14159704: 722 pre-labeled extracted-ion chromatogram PNG images
- Metadata list mapping each labeled EIC image to its ground-truth double-peak/single-peak class
- Optionally, outputs from comparator tools MS-DIAL, MZmine3, and xcms for benchmarking

## Outputs

- peak_detection_accuracy.csv with double-peak accuracy, single-peak accuracy, and per-image agreement flags
- Per-category accuracy scores (reference: 93.5% double peaks, 97.3% single peaks for MassCube)

## How to apply

Install MassCube from PyPI ('pip install masscube') and download Zenodo record 14159704, which contains the 722 pre-labeled EIC PNG images and the metadata list recording the ground-truth peak-shape class for each image. Run the MassCube untargeted workflow — peak detection by constructing mass traces through signal clustering and Gaussian-filter-assisted edge detection, followed by peak grouping — on the associated experimental Orbitrap/QTOF LC-MS raw files so that every labeled EIC is covered by a detected peak. Match each MassCube-detected peak to its corresponding labeled EIC via the metadata list and assign the predicted double-peak or single-peak class. Compute per-category accuracy as the fraction of labeled EICs whose detected peak agrees with the ground truth, yielding the double-peak accuracy (reference 93.5%) and single-peak accuracy (reference 97.3%), and record per-image agreement flags in peak_detection_accuracy.csv.

## Related tools

- **MassCube** (Python framework whose untargeted peak detection (mass-trace construction with signal clustering and Gaussian-filter-assisted edge detection, then peak grouping) supplies the detected peaks matched to the labeled EICs) — https://github.com/huaxuyu/masscube
- **MS-DIAL** (Comparator peak-detection tool benchmarked against MassCube on the same 722 labeled EICs)
- **MZmine3** (Comparator peak-detection tool benchmarked against MassCube on the same 722 labeled EICs)
- **XCMS** (Comparator peak-detection tool benchmarked against MassCube on the same 722 labeled EICs)
- **Python** (Environment for installing MassCube via pip and computing per-category accuracy into CSV output)

## Examples

```
pip install masscube
```

## Evaluation signals

- All 722 manually labeled EICs are matched to a MassCube-detected peak (MassCube reports 100% signal coverage with chromatographic metadata)
- Computed double-peak accuracy falls near the reported reference of 93.5%
- Computed single-peak accuracy falls near the reported reference of 97.3%
- Per-image agreement flags are present in peak_detection_accuracy.csv and every labeled image has a predicted class assigned
- MassCube scores exceed comparator accuracies from MS-DIAL, MZmine3, and XCMS run on the same labeled set

## Limitations

- Human-defined rules for peak shapes, reflected in algorithmic parameters, cannot account for all peak-shape scenarios
- Experimental data display characteristics that are difficult to explore systematically with synthetic data, such as varying scan numbers across peaks and peak tailing
- External validated benchmarking datasets with large-scale ground-truth data are still lacking, so this labeled EIC set is a rare resource for such benchmarking

## Evidence

- [abstract] peaks are detected by constructing mass traces through signal clustering and Gaussian-filter assisted edge detection: "peaks are detected by constructing mass traces through signal clustering and Gaussian-filter assisted edge detection"
- [abstract] MassCube outperforms MS-DIAL, MZmine3 and XCMS in speed, isomer detection, and accuracy.: "MassCube outperforms MS-DIAL, MZmine3 or XCMS for speed, isomer detection, and accuracy."
- [abstract] Peak detection in MassCube achieves 100% signal coverage with comprehensive reporting of chromatographic metadata: "Peak detection in MassCube achieves 100% signal coverage with comprehensive reporting of chromatographic metadata"
- [readme] Highly accurate nontargeted peak detection and segmentation.: "It provides:

- Highly accurate nontargeted peak detection and segmentation."
