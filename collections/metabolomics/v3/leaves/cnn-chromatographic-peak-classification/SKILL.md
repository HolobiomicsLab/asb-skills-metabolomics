---
name: cnn-chromatographic-peak-classification
description: Use when raw LC-HRMS profile-mode chromatograms (local maxima in rt x
  mz space) must be separated into true chromatographic peaks versus irrelevant background
  signals, and peak-center/bounding-box annotations are needed for downstream feature
  grouping.
when_to_use_negative:
- Input is already a feature table or peak-picked feature list rather than LC-HRMS
  profile-mode chromatograms — PeakBot operates on profile-mode data via local-maxima
  detection.
- Chromatograms contain co-eluting isomers within the reference features, since the
  gradient-descend reference extraction requires no co-eluting compounds (isomers).
- Fewer than 100 distinct reference features (including isotopologs) are available,
  so the CNN cannot be trained to high-quality peak-picking results.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: PeakBot
  role: Python package implementing local-maxima detection, standardized rt x mz area
    extraction, and CNN-based classification of chromatographic peaks vs background
    signals
  repo: https://github.com/christophuv/PeakBot
- name: TensorFlow
  role: Package implementing the CNN model (convolutional and pooling layers) used
    to report peak-type, peak-center, and bounding-box
  repo: https://www.tensorflow.org/
- name: Anaconda
  role: Recommended virtual environment to install CUDA dependencies (CUDA toolkit,
    cuDNN) for GPU-based training and inference
- name: CUDA toolkit
  role: Enables GPU-accelerated generation of training instances and faster CNN inference
    on LC-HRMS chromatograms
- name: cuDNN
  role: Library required alongside CUDA toolkit for the NVIDIA GPU used by PeakBot's
    TensorFlow model
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btac344
    title: 'PeakBot: machine-learning-based chromatographic peak picking'
  source_content_hash: sha256:c2048c969aecdfec92c61fe32e06ca648c9f30bc566f6fea7538e77c5141e3f0
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
  - CUDA toolkit
  - MS-Dial
  - PeakBot
  - Python
  - TensorFlow
  - XCMS
  - cuDNN
  - peakOnly
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# cnn-chromatographic-peak-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classify every local-maxima-derived standardized rt x mz area from an LC-HRMS chromatogram as a chromatographic peak versus background signal (walls, etc.) using a PeakBot convolutional neural network, also predicting peak-center and bounding-box for true peaks.

## When to use

Use when raw LC-HRMS profile-mode chromatograms (local maxima in rt x mz space) must be separated into true chromatographic peaks versus irrelevant background signals, and peak-center/bounding-box annotations are needed for downstream feature grouping.

## When NOT to use

- Input is already a feature table or peak-picked feature list rather than LC-HRMS profile-mode chromatograms — PeakBot operates on profile-mode data via local-maxima detection.
- Chromatograms contain co-eluting isomers within the reference features, since the gradient-descend reference extraction requires no co-eluting compounds (isomers).
- Fewer than 100 distinct reference features (including isotopologs) are available, so the CNN cannot be trained to high-quality peak-picking results.

## Inputs

- LC-HRMS profile-mode chromatogram files
- PeakBot reference feature TSV list (columns: RT, MZ, LeftRTBorder, RightRTBorder, MZDeviation)
- PeakBot wall background TSV list (columns: MZ, Rtstart, Rtend, MZdeviation)
- PeakBot other-background TSV list (columns: RTStart, RTEnd, MZLow, MZHigh)
- PeakBot LC-HRMS settings (noiseLevel, minIntensity, minRT, maxRT, RTpeakWidth, intraScanMaxAdjacentSignalDifferencePPM, interScanMaxSimilarSignalDifferencePPM)

## Outputs

- Per-local-maximum classification: chromatographic peak vs background signal type
- Peak-center and bounding-box for detected chromatographic peaks
- FeatureML and tab-separated-values files of detections, plus optional detection-example images for user verification

## How to apply

First detect all local signal maxima in the profile-mode chromatogram and extract each as a super-sampled standardized two-dimensional area (retention-time versus m/z). Feed these standardized areas into a PeakBot CNN model (TensorFlow 2.5.0, Python 3.8) that reports whether each local maximum is the apex of a chromatographic peak (including left/right isomeric compounds) or background (e.g. walls, signals present throughout large parts of the chromatogram). For classified peaks the model also suggests a bounding-box and peak-center. The model must be trained beforehand on at least 100 reference features specified in a TSV file with columns RT, MZ, LeftRTBorder, RightRTBorder, MZDeviation, together with optional wall/background TSV lists, using GPU (CUDA) based generation of training instances. Apply with peakbot.cuda.preProcessChromatogram to extract the standardized areas and peakbot.runPeakBot to classify them.

## Related tools

- **PeakBot** (Python package implementing local-maxima detection, standardized rt x mz area extraction, and CNN-based classification of chromatographic peaks vs background signals) — https://github.com/christophuv/PeakBot
- **TensorFlow** (Package implementing the CNN model (convolutional and pooling layers) used to report peak-type, peak-center, and bounding-box) — https://www.tensorflow.org/
- **Anaconda** (Recommended virtual environment to install CUDA dependencies (CUDA toolkit, cuDNN) for GPU-based training and inference)
- **CUDA toolkit** (Enables GPU-accelerated generation of training instances and faster CNN inference on LC-HRMS chromatograms)
- **cuDNN** (Library required alongside CUDA toolkit for the NVIDIA GPU used by PeakBot's TensorFlow model)

## Examples

```
from peakbot.cuda import preProcessChromatogram; from peakbot import runPeakBot; areas = preProcessChromatogram(chromatogram); results = runPeakBot(areas, model='my_model')
```

## Evaluation signals

- Verify the trained model reaches the reported accuracy of 0.99 on training and independent validation datasets for discriminating chromatographic peaks from background signals.
- Confirm at least 100 reference features were supplied before training; PeakBot's example workflow states this minimum for high-quality results.
- Check that classification outputs include peak-type, peak-center, and bounding-box, and that example detections export to images for user verification.
- Cross-check that wall annotations from background samples can be exported as featureML and TSV files and compiled into msms_exclusion_list.tsv for downstream MS/MS coverage analysis.

## Limitations

- PeakBot requires a CUDA-enabled NVIDIA graphics card plus CUDA toolkit and cuDNN libraries for efficient operation; CPU-only runs are supported but slower.
- Reference features must not contain co-eluting compounds (isomers), as this breaks the gradient-descend-based reference value estimation.
- Users must manually specify hard-to-evaluate parameters such as intraScanMaxAdjacentSignalDifferencePPM and interScanMaxSimilarSignalDifferencePPM (or estimate them via estimateParameters.py), making these error-prone inputs.
- PeakBot classifies only the chromatographic-peak vs background distinction reported by the CNN; it does not perform MS2-level spectral identification or quantification.

## Evidence

- [readme] The main functions to detect chromatographic peaks in a LC-HRMS chromatogram with a PeakBot model are peakbot.cuda.preProcessChromatogram for extracting the standardized areas and peakbot.runPeakBot: [evidence span withheld — anchor status: not_found]
