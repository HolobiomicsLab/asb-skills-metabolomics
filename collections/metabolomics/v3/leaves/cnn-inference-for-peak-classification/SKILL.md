---
name: cnn-inference-for-peak-classification
description: Use when detecting chromatographic peaks in LC-HRMS profile-mode data
  with a trained CNN model instead of heuristics, e.g. for peak-area quantification
  across replicate injections.
when_to_use_negative:
- Input is already a peak table or feature matrix — PeakBot operates on profile-mode
  LC-HRMS chromatograms, not on pre-picked or centroided data
- The chromatographic peak shapes of interest differ substantially from those the
  CNN was trained on (fronting, tailing, etc.) and no retraining is possible
- The analysis requires de novo feature grouping only; use PeakBot's grouping functions
  (KNNalignFeatures, groupFeatures) rather than the classification step itself
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: PeakBot
  role: Python package providing the CNN peak-picking pipeline (preProcessChromatogram,
    runPeakBot) and GPU-accelerated instance generation
  repo: https://github.com/christophuv/PeakBot
- name: TensorFlow 2.5.0
  role: Implements the CNN model used to classify standardized rt x m/z areas as peak
    apex, background, or wall
  repo: https://www.tensorflow.org/
- name: Anaconda
  role: Virtual environment manager that automatically installs CUDA dependencies
    for the GPU inference path
- name: CUDA toolkit / cuDNN
  role: Required for CUDA-enabled Nvidia GPU inference and GPU-based generation of
    training instances
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btac344
    title: 'PeakBot: machine-learning-based chromatographic peak picking'
  source_content_hash: sha256:25513f45a961c6d67856225ebf2255f3e9f89b63da89ef8eec950999f92687bc
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
  - TensorFlow 2.5.0
  - cuDNN
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# cnn-inference-for-peak-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run PeakBot's trained CNN model over standardized rt x m/z areas extracted from LC-HRMS profile-mode chromatograms to classify each local signal maximum as a chromatographic peak or background, yielding peak-type, peak-center, and bounding-box predictions.

## When to use

Use when detecting chromatographic peaks in LC-HRMS profile-mode data with a trained CNN model instead of heuristics, e.g. for peak-area quantification across replicate injections.

## When NOT to use

- Input is already a peak table or feature matrix — PeakBot operates on profile-mode LC-HRMS chromatograms, not on pre-picked or centroided data
- The chromatographic peak shapes of interest differ substantially from those the CNN was trained on (fronting, tailing, etc.) and no retraining is possible
- The analysis requires de novo feature grouping only; use PeakBot's grouping functions (KNNalignFeatures, groupFeatures) rather than the classification step itself

## Inputs

- LC-HRMS profile-mode chromatograms (centroid-free profile data)
- Trained PeakBot CNN model (TensorFlow 2.5.0)
- Optional: reference feature TSV (columns RT, MZ, LeftRTBorder, RightRTBorder, MZDeviation) and wall/background TSVs if training a new model
- PeakBot settings: noiseLevel, minIntensity, minRT, maxRT, RTpeakWidth, intraScanMaxAdjacentSignalDifferencePPM, interScanMaxSimilarSignalDifferencePPM

## Outputs

- Per-feature peak-type, peak-center, and bounding-box classifications
- Tab-separated-values peak-area table (e.g. wheatear_peak_areas.tsv)
- featureML files of grouped features (viewable in TOPPView)
- Example detection images for user verification

## How to apply

Set up a PeakBot virtual environment with Anaconda (Python 3.8, TensorFlow 2.5.0, plus CUDA toolkit and cuDNN for the GPU path, e.g. via the Windows10_python38_withGPU.yml or Debian11_python38_withGPU.yml environment files) and install PeakBot with pip from its GitHub repository. Run peak detection by first extracting all local signal maxima as super-sampled standardized retention-time versus m/z areas (peakbot.cuda.preProcessChromatogram), then classify each area with the CNN (peakbot.runPeakBot), which reports peak-type (left/right isomeric compounds versus background, including 'walls'), peak-center, and bounding-box. Use a CUDA-enabled Nvidia GPU for the computationally intensive inference, adapting blockdim/griddim to the GPU's streaming-processor count (e.g. via quickFindCUDAParameters.py) and setting exportBatchSize to 2048 (~4 GB GPU memory) or reducing to 1024/512 if less memory is available. Export detections as a tab-separated-values peak-area table and as featureML files (viewable in TOPPView from the OpenMS toolbox), optionally rendering example detections to images for user verification. When a suitable trained model does not exist, train one first using at least 100 reference features (which may be isotopologs) with peak shapes covering fronting/tailing forms.

## Related tools

- **PeakBot** (Python package providing the CNN peak-picking pipeline (preProcessChromatogram, runPeakBot) and GPU-accelerated instance generation) — https://github.com/christophuv/PeakBot
- **TensorFlow 2.5.0** (Implements the CNN model used to classify standardized rt x m/z areas as peak apex, background, or wall) — https://www.tensorflow.org/
- **Anaconda** (Virtual environment manager that automatically installs CUDA dependencies for the GPU inference path)
- **CUDA toolkit / cuDNN** (Required for CUDA-enabled Nvidia GPU inference and GPU-based generation of training instances)

## Examples

```
pip install git+https://github.com/christophuv/PeakBot; then in detect.py: areas = peakbot.cuda.preProcessChromatogram(...); peakbot.runPeakBot(...)
```

## Evaluation signals

- CNN discrimination accuracy reaches ~0.99 between chromatographic peaks and background signals on training and independent validation datasets
- Exported featureML files open correctly in TOPPView (OpenMS toolbox) and example detection images show plausible peak centers and bounding boxes
- On technical replicate injections, mean peak-area RSD across features is below 10%, with at least 90% of features showing RSD < 20%
- Reported classification outputs include a peak-type (peak, left/right isomeric compounds, or background), a peak-center, and a bounding-box for each accepted local maximum
- Loss and metric values during (re)training reach satisfying levels before the model is used for untargeted detection

## Limitations

- Requires a CUDA-enabled Nvidia graphics card plus CUDA toolkit and cuDNN libraries for the GPU path; the Anaconda virtual environment is recommended so CUDA dependencies install automatically
- exportBatchSize of 2048 needs ~4 GB GPU memory; reduce to 1024 or 512 if less memory is available
- At least 100 different reference features are needed to train a high-quality model; care must be taken to cover different chromatographic peak forms and that no co-eluting compounds (isomers) are present in reference chromatograms
- Parameters intraScanMaxAdjacentSignalDifferencePPM and interScanMaxSimilarSignalDifferencePPM are difficult to evaluate; use TOPPView to estimate them or PeakBot's estimateParameters.py function
- Scans outside minRT/maxRT (e.g. waste or column reconditioning regions) are excluded and must be configured per chromatogram

## Evidence

- [readme] Detection API functions: [evidence span withheld — anchor status: external_doc]
