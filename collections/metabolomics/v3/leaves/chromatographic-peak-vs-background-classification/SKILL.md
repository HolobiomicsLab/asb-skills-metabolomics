---
name: chromatographic-peak-vs-background-classification
description: Use when you have profile-mode LC-HRMS chromatograms and a user-defined
  reference-feature list (ground truth) and need to distinguish true chromatographic
  peaks from background/walls with a custom-trained CNN.
when_to_use_negative:
- Input is already a peak table or feature matrix rather than profile-mode LC-HRMS
  chromatogram data — PeakBot operates on raw profile-mode data via local-maxima extraction
- No user-defined reference-feature list (ground truth) is available and fewer than
  100 reference features can be specified
- Reference chromatograms contain co-eluting compounds (isomers) at the reference-feature
  locations, which breaks the gradient-descend property estimation
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PeakBot
  role: Python package that extracts standardized rt-vs-m/z areas from LC-HRMS profile-mode
    data, trains the CNN, and classifies peak vs background
  repo: https://github.com/christophuv/PeakBot
- name: PeakBot_Example
  role: Example scripts (train.py, detect.py, group.py) showing training and untargeted
    peak detection with a trained model
  repo: https://github.com/christophuv/PeakBot_Example
- name: TensorFlow 2.5.0
  role: Implements the CNN model (convolutional and pooling layers) that outputs peak-type,
    -center, and -bounding-box
  repo: https://www.tensorflow.org/
- name: Anaconda
  role: Recommended virtual environment for installation since all CUDA dependencies
    can automatically be installed there
- name: CUDA toolkit
  role: Required (with an Nvidia CUDA-enabled graphics card) for GPU-accelerated training-instance
    generation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btac344
    title: 'PeakBot: machine-learning-based chromatographic peak picking'
  source_content_hash: sha256:b37aa64dd9a834effba607d88f96a7cc4f527acb5f93827602de9dae214ce914
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
  - PeakBot
  - PeakBot_Example
  - Python
  - TensorFlow 2.5.0
  - cuDNN
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# chromatographic-peak-vs-background-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train and apply PeakBot, a TensorFlow 2.5.0 CNN that classifies local-maxima extracted from LC-HRMS profile-mode chromatograms as true chromatographic peaks (with left/right isomeric compounds) versus background signals (e.g., walls), reporting peak-type, peak-center, and bounding-box.

## When to use

Use when you have profile-mode LC-HRMS chromatograms and a user-defined reference-feature list (ground truth) and need to distinguish true chromatographic peaks from background/walls with a custom-trained CNN.

## When NOT to use

- Input is already a peak table or feature matrix rather than profile-mode LC-HRMS chromatogram data — PeakBot operates on raw profile-mode data via local-maxima extraction
- No user-defined reference-feature list (ground truth) is available and fewer than 100 reference features can be specified
- Reference chromatograms contain co-eluting compounds (isomers) at the reference-feature locations, which breaks the gradient-descend property estimation

## Inputs

- Profile-mode LC-HRMS chromatograms for training and detection
- User-defined reference-feature list as TSV (template_Peaks.tsv: RT, MZ, LeftRTBorder, RightRTBorder, MZDeviation)
- Wall background list as TSV (template_Walls.tsv: MZ, Rtstart, Rtend, MZdeviation)
- Other-background region list as TSV (template_Backgrounds.tsv: RTStart, RTEnd, MZLow, MZHigh)
- Chromatogram settings: noiseLevel, minIntensity, minRT, maxRT, RTpeakWidth, intraScanMaxAdjacentSignalDifferencePPM, interScanMaxSimilarSignalDifferencePPM

## Outputs

- Custom-trained PeakBot CNN model (TensorFlow 2.5.0) with training (T) and validation (V, iT, iV, eV) loss/metric values
- Peak-type classification per local maximum (chromatographic peak with left/right isomeric compounds vs background)
- Peak-center and bounding-box for detected chromatographic peaks
- Grouped feature table and featureML file across samples (group.py)
- Exported detection example images for user verification

## How to apply

First extract reference features from training chromatograms using PeakBot's smoothing and gradient-descend peak pre-detection, estimating peaks' borders and centers, then match detected peaks across samples to the user-defined ground-truth TSV list (columns RT, MZ, LeftRTBorder, RightRTBorder, MZDeviation) and extend it with isotopologs of the same compound. Provide separate TSVs for wall backgrounds (MZ, Rtstart, Rtend, MZdeviation) and other background regions (RTStart, RTEnd, MZLow, MZHigh) so the model learns to differentiate true chromatographic peaks from irrelevant background information. Specify chromatogram settings (noiseLevel, minIntensity, minRT, maxRT, RTpeakWidth, intraScanMaxAdjacentSignalDifferencePPM, interScanMaxSimilarSignalDifferencePPM), generate a large set of augmented training instances (peaks plus distraction peaks/backgrounds) with the GPU (CUDA) implementation, and train the CNN via peakbot.trainPeakBotModel. Supply at least 100 reference features, which may be isotopologs of the same compound, and cover the different peak forms (fronting, tailing) present in the data. For detection, run peakbot.cuda.preProcessChromatogram to extract standardized rt-vs-m/z areas, then peakbot.runPeakBot to classify each local maximum and suggest bounding-box and peak-center; optionally group features across samples with peakbot.cuda.KNNalignFeatures and peakbot.cuda.groupFeatures.

## Related tools

- **PeakBot** (Python package that extracts standardized rt-vs-m/z areas from LC-HRMS profile-mode data, trains the CNN, and classifies peak vs background) — https://github.com/christophuv/PeakBot
- **PeakBot_Example** (Example scripts (train.py, detect.py, group.py) showing training and untargeted peak detection with a trained model) — https://github.com/christophuv/PeakBot_Example
- **TensorFlow 2.5.0** (Implements the CNN model (convolutional and pooling layers) that outputs peak-type, -center, and -bounding-box) — https://www.tensorflow.org/
- **Anaconda** (Recommended virtual environment for installation since all CUDA dependencies can automatically be installed there)
- **CUDA toolkit** (Required (with an Nvidia CUDA-enabled graphics card) for GPU-accelerated training-instance generation)

## Examples

```
pip install git+https://github.com/christophuv/PeakBot
```

## Evaluation signals

- peakType_ACCPeakNoPeak on the independent validation set (iV) must exceed 0.99, matching the reported accuracy of 0.99
- Training-vs-validation metric/loss gap must be below 5% between T, V, iT, and iV sets (no overfitting)
- At least 100 reference features must be provided for training to reach high-quality peak-picking results
- Exported detection examples as images must be visually verified by the user
- Grouping step (KNNalignFeatures/groupFeatures) should produce a coherent data matrix and featureML file across samples

## Limitations

- Care must be taken that no co-eluting compounds (isomers) are present when the gradient-descend approach determines reference-feature values per chromatogram
- The parameters intraScanMaxAdjacentSignalDifferencePPM and interScanMaxSimilarSignalDifferencePPM are difficult to evaluate, though a helper script (estimateParameters.py) or TOPPView can help
- A large number of training instances is required for high model performance; without a CUDA GPU, instance generation is slow
- Different chromatographic peak forms (fronting, tailing, etc.) must be represented in the reference features or the model will not recognize them correctly

## Evidence

- [abstract] high performance discriminating peak vs background: "In training and independent validation datasets used for development, PeakBot achieved a high performance with respect to discriminating between chromatographic peaks and background signals"
