---
name: feature-list-matching-across-lchrms-tools
description: Use when the same LC-HRMS profile-mode dataset has been processed by
  several peak-picking tools and you need to quantify which chromatographic features
  are found in common, missed, or exclusive per tool.
when_to_use_negative:
- Input is already a single merged feature table or peak matrix rather than raw profile-mode
  chromatograms needing independent peak picking per tool
- The analysis goal is MS/MS spectral identification or compound annotation rather
  than comparing peak-picking detection coverage
- Only one peak-picking tool is available or of interest, so no cross-tool overlap
  can be computed
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: PeakBot
  role: CNN-based peak-picking tool whose detections are one of the four feature lists
    compared; run via local-maxima detection and TensorFlow classification
  repo: https://github.com/christophuv/PeakBot
- name: XCMS
  role: Produces an independent peak-picking feature list on the same dataset for
    overlap computation
- name: MS-Dial
  role: Produces an independent peak-detection feature list on the same dataset for
    overlap computation
- name: peakOnly
  role: Produces an independent peak-detection feature list on the same dataset for
    overlap computation
- name: Python 3.8
  role: Implementation language for PeakBot and for the alignment/Venn-overlap computation
    script
- name: TensorFlow 2.5.0
  role: Runs the custom-trained PeakBot CNN that classifies local maxima as chromatographic
    peaks or background before feature lists are compared
- name: Anaconda
  role: Recommended virtual environment to install PeakBot so CUDA dependencies are
    resolved automatically
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btac344
    title: 'PeakBot: machine-learning-based chromatographic peak picking'
  source_content_hash: sha256:0ab83e37f10d383c0119bea23385d2e8a87602a768563a204ecf5139547e6486
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/christophuv/PeakBot
  repo_url: https://github.com/christophuv/PeakBot
  tools:
  - Anaconda
  - MS-Dial
  - PeakBot
  - Python
  - Python 3.8
  - TensorFlow 2.5.0
  - XCMS
  - peakOnly
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# feature-list-matching-across-lchrms-tools

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compares chromatographic peak-picking outputs from multiple LC-HRMS tools (PeakBot, XCMS, MS-Dial, peakOnly) by aligning their feature lists on retention time and m/z and computing Venn-overlap statistics to quantify tool agreement.

## When to use

Use when the same LC-HRMS profile-mode dataset has been processed by several peak-picking tools and you need to quantify which chromatographic features are found in common, missed, or exclusive per tool.

## When NOT to use

- Input is already a single merged feature table or peak matrix rather than raw profile-mode chromatograms needing independent peak picking per tool
- The analysis goal is MS/MS spectral identification or compound annotation rather than comparing peak-picking detection coverage
- Only one peak-picking tool is available or of interest, so no cross-tool overlap can be computed

## Inputs

- LC-HRMS profile-mode chromatogram files (e.g., porcine hepatic microsome / PHM dataset)
- Feature list 1: PeakBot detections (data matrix / featureML file)
- Feature list 2: XCMS peak-picking output
- Feature list 3: MS-Dial peak detection output
- Feature list 4: peakOnly peak detection output
- Alignment tolerances for retention time (seconds) and m/z

## Outputs

- Aligned four-way feature table keyed by retention time and m/z
- Venn-overlap statistics across the four tools (peakbot_overlap_stats.tsv)
- Per-tool detection-rate metrics (e.g., PeakBot total detection rate)

## How to apply

Run each peak-picking tool on the identical LC-HRMS dataset to produce four feature lists; for PeakBot, detect local signal maxima, extract super-sampled standardized areas (retention-time versus m/z), classify them with the TensorFlow CNN, and group detections into a data matrix and featureML file using k-nearest-neighbor alignment. Align the four feature lists by retention time and m/z tolerance, then compute Venn-overlap fractions across tools. On the PHM porcine hepatic microsome dataset the expected result is ~73% of features detected by all four tools, ~12% missed only by peakOnly, 4.7% exclusive to MS-Dial/peakOnly/PeakBot, and PeakBot detecting 94.65% of all features. Rationale: tool-specific peak-picking biases can silently drop real features, so cross-tool overlap is a direct quality metric for peak-picking pipelines.

## Related tools

- **PeakBot** (CNN-based peak-picking tool whose detections are one of the four feature lists compared; run via local-maxima detection and TensorFlow classification) — https://github.com/christophuv/PeakBot
- **XCMS** (Produces an independent peak-picking feature list on the same dataset for overlap computation)
- **MS-Dial** (Produces an independent peak-detection feature list on the same dataset for overlap computation)
- **peakOnly** (Produces an independent peak-detection feature list on the same dataset for overlap computation)
- **Python 3.8** (Implementation language for PeakBot and for the alignment/Venn-overlap computation script)
- **TensorFlow 2.5.0** (Runs the custom-trained PeakBot CNN that classifies local maxima as chromatographic peaks or background before feature lists are compared)
- **Anaconda** (Recommended virtual environment to install PeakBot so CUDA dependencies are resolved automatically)

## Examples

```
pip install git+https://github.com/christophuv/PeakBot
```

## Evaluation signals

- Computed four-tool Venn overlap on the PHM dataset matches the reported ~73% of features detected by all four approaches
- Fraction of features missed only by peakOnly is approximately 12%
- Fraction exclusive to MS-Dial/peakOnly/PeakBot is approximately 4.7%
- PeakBot's total detection rate across all features is 94.65%
- Alignment respects retention-time and m/z tolerance so identical chromatographic peaks are grouped rather than duplicated across tools

## Limitations

- Training a PeakBot CNN model requires a minimum of 100 reference features (isotopologs of the same compound can count) to achieve high-quality peak-picking results
- PeakBot GPU-based training-instance generation and inference require an NVIDIA CUDA-enabled graphics card with the CUDA toolkit and cuDNN libraries installed
- Users must supply dataset-specific settings such as noiseLevel, minIntensity, minRT/maxRT, RTpeakWidth, and intraScan/interScanMaxAdjacentSignalDifferencePPM, which are difficult to evaluate manually
- Reference feature values are recalculated per chromatogram via gradient descent, so they vary between samples; co-eluting isomers in reference chromatograms must be avoided

## Evidence

- [readme] The main functions to group detected features from several samples are peakbot.cuda.KNNalignFeatures for aligning the detected features using a k-nearest-neighbor approach and peakbot.cuda.groupFeatures: [evidence span withheld — anchor status: external_doc]
- [readme] A comparison has shown that at least 100 different reference features should be provided. However, this can also be different isotopologs of the same feature.: [evidence span withheld — anchor status: external_doc]
