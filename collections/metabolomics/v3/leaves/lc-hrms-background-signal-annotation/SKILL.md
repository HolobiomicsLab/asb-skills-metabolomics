---
name: lc-hrms-background-signal-annotation
description: Use when LC-HRMS chromatograms contain persistent background/wall signals
  that fragment preferentially in MS/MS, and you want to generate an exclusion list
  to boost MS/MS coverage of true peaks in subsequent runs.
when_to_use_negative:
- The input is already a processed feature table (PeakBot requires raw LC-HRMS profile-mode
  data).
- You need de novo peak detection without background/wall classification — plain PeakBot
  peak-picking suffices.
- No CUDA-enabled NVIDIA graphics card is available and the dataset is large enough
  that CPU-only training/generation would be prohibitively slow.
edam_operation: http://edamontology.org/operation_3694
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: PeakBot
  role: CNN-based peak/background classifier that annotates local maxima in profile-mode
    LC-HRMS chromatograms as chromatographic peaks or background (walls)
  repo: https://github.com/christophuv/PeakBot
- name: Python 3.8
  role: Runtime environment in which PeakBot is implemented
  repo: https://github.com/christophuv/PeakBot
- name: TensorFlow 2.5.0
  role: Framework implementing the convolutional neural network that classifies standardized
    rt x mz areas
- name: Anaconda
  role: Virtual environment manager used to install PeakBot and its CUDA dependencies
- name: CUDA toolkit
  role: GPU acceleration for training instance generation and CNN inference
- name: cuDNN
  role: GPU library required alongside CUDA toolkit for PeakBot's CUDA-based operations
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btac344
    title: 'PeakBot: machine-learning-based chromatographic peak picking'
  source_content_hash: sha256:2f315db7ebc73fd5829f6dfd98f95f814a37fa4bd27a08c0f9df7e7c70787837
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
  - Python
  - Python 3.8
  - TensorFlow 2.5.0
  - cuDNN
  edam_operation: http://edamontology.org/operation_3694
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7222222222222222
n_claims: 18
---

# lc-hrms-background-signal-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate LC-HRMS background 'wall' signals using PeakBot's CNN-based peak-picking framework so that persistent background features can be compiled into an MS/MS exclusion list, improving targeted MS/MS coverage of true chromatographic peaks.

## When to use

Use when LC-HRMS chromatograms contain persistent background/wall signals that fragment preferentially in MS/MS, and you want to generate an exclusion list to boost MS/MS coverage of true peaks in subsequent runs.

## When NOT to use

- The input is already a processed feature table (PeakBot requires raw LC-HRMS profile-mode data).
- You need de novo peak detection without background/wall classification — plain PeakBot peak-picking suffices.
- No CUDA-enabled NVIDIA graphics card is available and the dataset is large enough that CPU-only training/generation would be prohibitively slow.

## Inputs

- LC-HRMS profile-mode background/blank sample chromatograms
- Trained PeakBot CNN model
- Wall/background reference list in TSV format (columns: MZ, Rtstart, Rtend, MZdeviation)
- Anaconda environment yml file for Python 3.8 with TensorFlow 2.5.0 (GPU or CPU variant)

## Outputs

- featureML file with annotated background/wall detections
- Tab-separated-values (TSV) file of background detections
- msms_exclusion_list.tsv for MS/MS exclusion
- Detection example images for user verification
- MS/MS coverage counts of true chromatographic peaks in negative and positive ionization modes (with vs. without exclusion list)

## How to apply

Train or load a PeakBot CNN model in a Python 3.8 / TensorFlow 2.5.0 Anaconda environment (GPU support via CUDA toolkit and cuDNN). Feed LC-HRMS profile-mode background/blank samples through `peakbot.cuda.preProcessChromatogram` and `peakbot.runPeakBot` so that all local maxima are extracted as standardized rt x mz areas and classified by the CNN as either a chromatographic peak, a wall, or other background. Export the annotated background detections (featureML and TSV formats) and compile the wall annotations into an MS/MS exclusion list (msms_exclusion_list.tsv). Quantify MS/MS coverage of true peaks with and without the exclusion list; validated gains were 1356→1605 (+18%) in negative mode and 855→1810 (+112%) in positive mode.

## Related tools

- **PeakBot** (CNN-based peak/background classifier that annotates local maxima in profile-mode LC-HRMS chromatograms as chromatographic peaks or background (walls)) — https://github.com/christophuv/PeakBot
- **Python 3.8** (Runtime environment in which PeakBot is implemented) — https://github.com/christophuv/PeakBot
- **TensorFlow 2.5.0** (Framework implementing the convolutional neural network that classifies standardized rt x mz areas)
- **Anaconda** (Virtual environment manager used to install PeakBot and its CUDA dependencies)
- **CUDA toolkit** (GPU acceleration for training instance generation and CNN inference)
- **cuDNN** (GPU library required alongside CUDA toolkit for PeakBot's CUDA-based operations)

## Examples

```
conda create -f Windows10_python38_withGPU.yml; python detect.py; python group.py; python generateExclusionList.py
```

## Evaluation signals

- MS/MS coverage of true chromatographic peaks increases when the exclusion list is applied (reproduced values should match Supplementary Table S2: negative mode 1356→1605, positive mode 855→1810).
- The exported featureML file contains annotations where the CNN classified local maxima as background/wall rather than chromatographic peaks.
- Detection examples rendered to images show the model correctly labeling wall regions that span large parts of the chromatogram.
- The msms_exclusion_list.tsv file contains m/z and retention-time window entries consistent with the wall annotation output.
- Accuracy of the trained model on the validation dataset reaches ~0.99 for distinguishing chromatographic peaks from background signals.

## Limitations

- A minimum of 100 reference features is required to train a high-quality PeakBot model; fewer references degrade performance.
- PeakBot requires a CUDA-enabled NVIDIA graphics card plus CUDA toolkit and cuDNN libraries for the GPU-based training instance generation; without them only a slower CPU path is available.
- The wall/background reference list must be carefully specified (m/z, Rtstart, Rtend, MZdeviation) and narrow retention-time windows are needed to specifically train PeakBot for particular wall backgrounds.
- Co-eluting isomers must be avoided in the reference chromatograms when extracting reference feature values via the gradient-descent approach.

## Evidence

- [methods] Background wall annotation compiled into MS/MS exclusion list: [evidence span withheld — anchor status: not_found]
- [results] Exclusion list increases MS/MS coverage: [evidence span withheld — anchor status: not_found]
