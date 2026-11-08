---
name: lc-hrms-chromatographic-peak-detection
description: Use when you have LC-HRMS profile-mode chromatograms (e.g. RAW-converted
  retention-time vs m/z scans) and need untargeted, model-based detection of chromatographic
  peaks including fronting/tailing shapes and wall backgrounds.
when_to_use_negative:
- Input is already a peak table or feature matrix (PeakBot works on profile-mode LC-HRMS
  chromatograms, not pre-integrated data)
- You have fewer than 100 reference features to train a new model — the paper reports
  this minimum is needed to achieve high-quality peak-picking results
- Reference chromatograms contain co-eluting compounds (isomers), which the README
  explicitly warns against when specifying references
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: PeakBot
  role: 'Primary peak-picking framework: local-maxima detection, standardized-area
    extraction, CNN classification, feature export'
  repo: https://github.com/christophuv/PeakBot
- name: TensorFlow
  role: Implements the CNN model that classifies standardized areas and predicts peak-center
    and bounding-box
- name: Anaconda
  role: Virtual-environment installation so all CUDA dependencies can be installed
    automatically
- name: CUDA toolkit / cuDNN
  role: GPU-accelerated inference and training-instance generation for the CNN
- name: TOPPView (OpenMS toolbox)
  role: Visual verification of exported featureML files and estimation of ppm signal
    differences for parameter setting
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btac344
    title: 'PeakBot: machine-learning-based chromatographic peak picking'
  source_content_hash: sha256:8994708124a017a5aa7c7fad586d7f12cc2ace4f88aa0ad4efb774ebc382652a
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
  - CUDA toolkit / cuDNN
  - PeakBot
  - Python
  - TOPPView (OpenMS toolbox)
  - TensorFlow
  - cuDNN
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.782608695652174
n_claims: 23
---

# lc-hrms-chromatographic-peak-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Detect chromatographic peaks in LC-HRMS profile-mode chromatograms using PeakBot, a CNN-based peak picker that classifies local-maxima as peak apexes versus background and predicts peak-center and bounding-box. It replaces heuristic peak-picking with a machine-learning model trainable on user-supplied reference features.

## When to use

Use when you have LC-HRMS profile-mode chromatograms (e.g. RAW-converted retention-time vs m/z scans) and need untargeted, model-based detection of chromatographic peaks including fronting/tailing shapes and wall backgrounds.

## When NOT to use

- Input is already a peak table or feature matrix (PeakBot works on profile-mode LC-HRMS chromatograms, not pre-integrated data)
- You have fewer than 100 reference features to train a new model — the paper reports this minimum is needed to achieve high-quality peak-picking results
- Reference chromatograms contain co-eluting compounds (isomers), which the README explicitly warns against when specifying references

## Inputs

- LC-HRMS profile-mode chromatograms (retention-time vs m/z scan data)
- Trained PeakBot CNN model (TensorFlow)
- Reference feature list as tsv with columns RT, MZ, LeftRTBorder, RightRTBorder, MZDeviation (for training)
- Wall/background lists as tsv (MZ, Rtstart, Rtend, MZdeviation and RTStart, RTEnd, MZLow, MZHigh) for training

## Outputs

- Detected chromatographic peaks as a tab-separated-values peak-area table (e.g. wheatear_peak_areas.tsv)
- featureML files viewable in TOPPView from the OpenMS toolbox
- Per-peak annotations: peak-type, peak-center, and bounding-box
- Example detection images for user verification

## How to apply

Install PeakBot in an Anaconda virtual environment (Python 3.8, TensorFlow 2.5.0) via `pip install git+https://github.com/christophuv/PeakBot`; for GPU inference install a CUDA-enabled Nvidia card with the CUDA toolkit and cuDNN (e.g. via the Windows10_python38_withGPU.yml or Debian11_python38_withGPU.yml environment files). Run detection with the trained CNN model: PeakBot detects all local signal maxima, extracts each as a super-sampled standardized area (retention-time versus m/z), and the CNN reports peak-type (left/right isomeric compounds vs background), peak-center, and bounding-box. Use a CUDA-enabled GPU for the computationally intensive inference, setting exportBatchSize to 2048 (~4 GB GPU memory) or reducing it to 1024/512 with less memory; adapt blockdim/griddim to the streaming-processor count (e.g. quickFindCUDAParameters.py). Key chromatogram settings to specify are noiseLevel, minIntensity, minRT/maxRT, RTpeakWidth, and intraScanMaxAdjacentSignalDifferencePPM / interScanMaxSimilarSignalDifferencePPM (settable via estimateParameters.py or TOPPView from the OpenMS toolbox). Export detections as a tab-separated peak-area table and featureML files viewable in TOPPView, optionally exporting example detections to images for user verification.

## Related tools

- **PeakBot** (Primary peak-picking framework: local-maxima detection, standardized-area extraction, CNN classification, feature export) — https://github.com/christophuv/PeakBot
- **TensorFlow** (Implements the CNN model that classifies standardized areas and predicts peak-center and bounding-box)
- **Anaconda** (Virtual-environment installation so all CUDA dependencies can be installed automatically)
- **CUDA toolkit / cuDNN** (GPU-accelerated inference and training-instance generation for the CNN)
- **TOPPView (OpenMS toolbox)** (Visual verification of exported featureML files and estimation of ppm signal differences for parameter setting)

## Examples

```
pip install git+https://github.com/christophuv/PeakBot
```

## Evaluation signals

- CNN accuracy of 0.99 discriminating chromatographic peaks from background in training and independent validation datasets
- Mean relative standard deviation (RSD) of peak areas across technical replicate injections below 10%, with ≥90% of features having RSD below 20% (WheatEar repeatability benchmark)
- Exported featureML files load and display correctly in TOPPView from the OpenMS toolbox
- Example detection images exported to files allow visual user-verification of peak-type, peak-center, and bounding-box predictions
- Satisfying loss and metric values reached during model training before applying the model to new chromatograms

## Limitations

- Requires a CUDA-enabled Nvidia graphics card with the CUDA toolkit and cuDNN libraries for GPU inference; a noGPU Anaconda environment is available for CPU execution
- At least 100 reference features (possibly isotopologs of the same compound) covering the different peak forms are needed to train a new CNN model
- The parameters intraScanMaxAdjacentSignalDifferencePPM and interScanMaxSimilarSignalDifferencePPM are difficult to evaluate and must match the instrument's profile-mode signal spacing
- exportBatchSize of 2048 needs ~4 GB GPU memory; must be reduced to 1024/512 on smaller cards

## Evidence

- [readme] The main functions to detect chromatographic peaks in a LC-HRMS chromatogram with a PeakBot model are `peakbot.cuda.preProcessChromatogram` for extracting the standardized areas: [evidence span withheld — anchor status: external_doc]
- [readme] The model reports if the respective local maximum is the apex of a chromatographic peak: [evidence span withheld — anchor status: not_found]
