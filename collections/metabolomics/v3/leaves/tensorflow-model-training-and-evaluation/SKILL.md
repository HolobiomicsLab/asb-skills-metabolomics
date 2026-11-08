---
name: tensorflow-model-training-and-evaluation
description: Use when a custom CNN must be trained on LC-HRMS chromatograms to distinguish
  true chromatographic peaks from background signals (including walls) and evaluated
  on independent validation chromatograms.
when_to_use_negative:
- You only need conventional peak picking without training a model — PeakBot requires
  at least 100 user-provided reference features to train a custom model.
- Input is already peak-picked/centroided feature data rather than LC-HRMS profile-mode
  chromatograms.
- The target task is feature grouping/alignment of already-detected peaks rather than
  training and evaluating the CNN classifier itself.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: PeakBot
  role: Python package providing training-instance generation (peakbot.train.cuda.generateTestInstances),
    model training (peakbot.trainPeakBotModel), and reference-feature extraction from
    training chromatograms
  repo: https://github.com/christophuv/PeakBot
- name: TensorFlow
  role: Framework implementing the CNN model that outputs peak type, peak center,
    and bounding box (version 2.5.0)
  repo: https://www.tensorflow.org/
- name: Anaconda
  role: Recommended virtual environment manager; automatically installs CUDA dependencies
    via provided yml environment files
  repo: https://github.com/christophuv/PeakBot
- name: PeakBot_Example
  role: Example scripts (train.py, quickExample_GPU.py) demonstrating how to train
    a new PeakBot CNN model and generate training/validation datasets T, V, iT, iV,
    eV
  repo: https://github.com/christophuv/PeakBot_Example
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btac344
    title: 'PeakBot: machine-learning-based chromatographic peak picking'
  source_content_hash: sha256:ffd7a68a8aecad697c4b87bd4835b9cd6ae654b586f54989a3062598c7f2053b
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
  - TensorFlow
  - cuDNN
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# tensorflow-model-training-and-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train and evaluate a custom TensorFlow 2.5.0 CNN model that classifies chromatographic peak versus background in LC-HRMS profile-mode data, using augmented training instances and checking train/validation metric gaps to rule out overfitting.

## When to use

Use when a custom CNN must be trained on LC-HRMS chromatograms to distinguish true chromatographic peaks from background signals (including walls) and evaluated on independent validation chromatograms.

## When NOT to use

- You only need conventional peak picking without training a model — PeakBot requires at least 100 user-provided reference features to train a custom model.
- Input is already peak-picked/centroided feature data rather than LC-HRMS profile-mode chromatograms.
- The target task is feature grouping/alignment of already-detected peaks rather than training and evaluating the CNN classifier itself.

## Inputs

- LC-HRMS profile-mode chromatograms (training and independent validation sets, e.g. from MTBLS1358, MTBLS868, MTBLS797, or ST001450)
- Reference feature list (TSV: RT, MZ, LeftRTBorder, RightRTBorder, MZDeviation)
- Wall background list (TSV: MZ, Rtstart, Rtend, MZdeviation)
- Other background list (TSV: RTStart, RTEnd, MZLow, MZHigh)
- LC-HRMS settings: noiseLevel, minIntensity, minRT, maxRT, RTpeakWidth, intraScanMaxAdjacentSignalDifferencePPM, interScanMaxSimilarSignalDifferencePPM

## Outputs

- Trained PeakBot CNN model (peak type, peak center, bounding box predictions)
- Training/validation loss and metric values (peakType_ACCPeakNoPeak) for T, V, iT, iV datasets
- Exported example detection images for user verification

## How to apply

Generate training instances from reference chromatograms by matching smoothed gradient-descend pre-detected peaks to a user-defined ground-truth reference list (TSV with RT, MZ, LeftRTBorder, RightRTBorder, MZDeviation), extending it with isotopologs, and iteratively combining matched references with distraction peaks and background/wall augmentations on GPU (CUDA) — at least 100 reference features are needed. Train the CNN (implemented in TensorFlow 2.5.0 under Python 3.8, e.g. in an Anaconda virtual environment with CUDA toolkit and cuDNN) to output peak type (peak vs background), peak center, and bounding box. Evaluate on held-out validation datasets (V, iT, iV, eV), recording the peakType_ACCPeakNoPeak metric on the independent validation set iV. Judge success by accuracy >0.99 on iV and a training-vs-validation gap of less than 5 percentage points, indicating no overfitting. Optionally export example detections as images for user verification.

## Related tools

- **PeakBot** (Python package providing training-instance generation (peakbot.train.cuda.generateTestInstances), model training (peakbot.trainPeakBotModel), and reference-feature extraction from training chromatograms) — https://github.com/christophuv/PeakBot
- **TensorFlow** (Framework implementing the CNN model that outputs peak type, peak center, and bounding box (version 2.5.0)) — https://www.tensorflow.org/
- **Anaconda** (Recommended virtual environment manager; automatically installs CUDA dependencies via provided yml environment files) — https://github.com/christophuv/PeakBot
- **PeakBot_Example** (Example scripts (train.py, quickExample_GPU.py) demonstrating how to train a new PeakBot CNN model and generate training/validation datasets T, V, iT, iV, eV) — https://github.com/christophuv/PeakBot_Example

## Examples

```
curl https://raw.githubusercontent.com/christophuv/PeakBot_Example/main/quickExample_GPU.py > quickExample_GPU.py; python quickExample_GPU.py
```

## Evaluation signals

- peakType_ACCPeakNoPeak on the independent validation set (iV) exceeds 0.99
- Training-vs-validation loss/metric gap is below 5 percentage points (no overfitting)
- At least 100 reference features were used for training
- Exported example detection images visually confirm correct peak/center/bounding-box predictions
- Model correctly differentiates true chromatographic peaks from walls and other background types in augmented training instances

## Limitations

- A minimum of 100 reference features is required to achieve high-quality peak-picking results; fewer will degrade performance.
- Reference features must cover the different chromatographic peak forms (fronting, tailing) present in the data; care is required to avoid co-eluting compounds (isomers) in reference chromatograms.
- GPU training-instance generation and CNN training require a CUDA-enabled Nvidia graphics card with CUDA toolkit and cuDNN installed (CPU-only Anaconda environments are also provided).
- The intraScanMaxAdjacentSignalDifferencePPM and interScanMaxSimilarSignalDifferencePPM parameters are difficult to set; the estimateParameters.py script or TOPPView can help.

## Evidence

- [readme] A comparison has shown that at least 100 different reference features should be provided. However, this can also be different isotopologs of the same feature.: [evidence span withheld — anchor status: external_doc]
