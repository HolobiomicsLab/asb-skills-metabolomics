---
name: gpu-accelerated-chromatogram-processing
description: Use when LC-HRMS profile-mode chromatograms must be peak-picked with
  a trained PeakBot CNN across many injections (e.g. technical replicates) and the
  workload justifies a CUDA-enabled GPU.
when_to_use_negative:
- Input is already a reduced feature/peak table rather than LC-HRMS profile-mode chromatograms
  — PeakBot works on raw profile-mode data.
- No CUDA-enabled Nvidia GPU is available and the dataset is too small to justify
  GPU setup — use the noGPU Anaconda environment variants instead.
- The goal is training a new CNN model from reference features rather than detecting
  peaks in new chromatograms — use the PeakBot_Example training scripts (train.py)
  workflow instead.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PeakBot
  role: 'Python package performing GPU-accelerated peak detection: local-maxima extraction,
    CNN classification, and peak-area export'
  repo: https://github.com/christophuv/PeakBot
- name: Anaconda
  role: Virtual environment manager that auto-installs CUDA dependencies via the withGPU
    yml files
- name: CUDA toolkit
  role: Nvidia GPU computing toolkit required for PeakBot's GPU-based training-instance
    generation and inference
- name: cuDNN
  role: Nvidia deep-learning library required alongside the CUDA toolkit for the GPU
    path
- name: TensorFlow 2.5.0
  role: Package implementing the CNN model whose inference is accelerated on the CUDA
    GPU
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btac344
    title: 'PeakBot: machine-learning-based chromatographic peak picking'
  source_content_hash: sha256:15c1a4a870f9b9a06a7c1569b22e01cff19f80cf301daf3dffd9809b4dff5ef5
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
  - TensorFlow 2.5.0
  - cuDNN
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# gpu-accelerated-chromatogram-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run PeakBot's machine-learning peak picking on LC-HRMS profile-mode chromatograms using CUDA GPU acceleration, so that standardized rt x m/z areas can be classified by a TensorFlow CNN at scale instead of on CPU.

## When to use

Use when LC-HRMS profile-mode chromatograms must be peak-picked with a trained PeakBot CNN across many injections (e.g. technical replicates) and the workload justifies a CUDA-enabled GPU.

## When NOT to use

- Input is already a reduced feature/peak table rather than LC-HRMS profile-mode chromatograms — PeakBot works on raw profile-mode data.
- No CUDA-enabled Nvidia GPU is available and the dataset is too small to justify GPU setup — use the noGPU Anaconda environment variants instead.
- The goal is training a new CNN model from reference features rather than detecting peaks in new chromatograms — use the PeakBot_Example training scripts (train.py) workflow instead.

## Inputs

- LC-HRMS profile-mode chromatogram files
- Trained PeakBot CNN model
- CUDA-enabled Nvidia GPU with CUDA toolkit and cuDNN
- Anaconda environment definition yml (withGPU variant)

## Outputs

- Tab-separated-values peak table (e.g. wheatear_peak_areas.tsv) with detected peak areas
- featureML files for viewing in TOPPView
- Example detection images for user verification

## How to apply

Set up a Python 3.8 virtual environment with Anaconda using the provided yml files (e.g. Windows10_python38_withGPU.yml or Debian11_python38_withGPU.yml) so the CUDA toolkit and cuDNN dependencies are installed automatically, then pip install PeakBot from git+https://github.com/christophuv/PeakBot. PeakBot first detects all local signal maxima and extracts them as super-sampled standardized retention-time vs m/z areas; the CNN then reports peak-type, peak-center and bounding-box for each. Adapt CUDA blockdim/griddim to the GPU's streaming-processor count (e.g. via quickFindCUDAParameters.py) and set exportBatchSize to 2048 (~4 GB GPU memory), lowering to 1024 or 512 if the GPU has less memory. Export detected peaks as a tab-separated-values table and featureML files (viewable in TOPPView from the OpenMS toolbox), plus optional example detections as images for user verification.

## Related tools

- **PeakBot** (Python package performing GPU-accelerated peak detection: local-maxima extraction, CNN classification, and peak-area export) — https://github.com/christophuv/PeakBot
- **Anaconda** (Virtual environment manager that auto-installs CUDA dependencies via the withGPU yml files)
- **CUDA toolkit** (Nvidia GPU computing toolkit required for PeakBot's GPU-based training-instance generation and inference)
- **cuDNN** (Nvidia deep-learning library required alongside the CUDA toolkit for the GPU path)
- **TensorFlow 2.5.0** (Package implementing the CNN model whose inference is accelerated on the CUDA GPU)

## Examples

```
pip install git+https://github.com/christophuv/PeakBot; python quickExample_GPU.py
```

## Evaluation signals

- quickFindCUDAParameters.py reports blockdim/griddim matched to the GPU's streaming-processor count without errors
- exportBatchSize of 2048 runs within ~4 GB GPU memory without out-of-memory failures (reduce to 1024/512 otherwise)
- Exported peak table contains peak-type, peak-center and bounding-box for detected chromatographic peaks
- featureML output opens correctly in TOPPView and example detection images show plausible peak apices
- Downstream replicate analysis achieves mean peak-area RSD below 10% with 90% of features below 20% RSD

## Limitations

- A CUDA-enabled Nvidia graphics card plus CUDA toolkit and cuDNN libraries are mandatory for the GPU path; CPU-only environments use the noGPU yml variants.
- The GPU-based approach was implemented to decrease time for generating the large number of training instances; batching must be tuned to available GPU memory (~4 GB for exportBatchSize 2048).
- Minimal chromatogram settings (noiseLevel, minIntensity, minRT, maxRT, RTpeakWidth, intraScanMaxAdjacentSignalDifferencePPM, interScanMaxSimilarSignalDifferencePPM) must be specified correctly or detection quality suffers.

## Evidence
