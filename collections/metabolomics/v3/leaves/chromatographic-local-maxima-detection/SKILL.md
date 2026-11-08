---
name: chromatographic-local-maxima-detection
description: Use when processing raw profile-mode LC-HRMS chromatograms (mzXML/mzML)
  and you need to enumerate all candidate peak-apex signals before machine-learning
  peak/background classification.
when_to_use_negative:
- Input is already centroided/peak-picked feature data rather than profile-mode LC-HRMS
  chromatograms
- You need finalized peak groups or a feature matrix — this skill only enumerates
  candidate apexes; use PeakBot's CNN classification and grouping steps (peakbot.runPeakBot,
  groupFeatures) instead
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: PeakBot
  role: Provides the chromatogram reader, 8-neighborhood local-maximum scan, super-sampling
    to 32x128 standardized areas, normalization, and CUDA/GPU-based export
  repo: https://github.com/christophuv/PeakBot
- name: CUDA toolkit
  role: Required dependency for the GPU-accelerated standardized-area export (with
    configurable exportBatchSize)
- name: cuDNN
  role: Required Nvidia library dependency for running PeakBot's GPU-based preprocessing
    on a CUDA-enabled graphics card
- name: TensorFlow
  role: Consumes the exported standardized areas downstream as CNN model input (TensorFlow
    2.5.0)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btac344
    title: 'PeakBot: machine-learning-based chromatographic peak picking'
  source_content_hash: sha256:24130011b09bf9a6b4f8ea408687b59d7decc65efdc3b708106ddf5b178cc5c9
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
  - CUDA toolkit
  - PeakBot
  - Python
  - TensorFlow
  - cuDNN
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# chromatographic-local-maxima-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Candidate-peak-apex detection for LC-HRMS profile-mode data: scan the retention-time x m/z matrix for 8-neighborhood local maxima and export each neighborhood as a super-sampled standardized 32 scans x 128 m/z area for downstream CNN classification in PeakBot.

## When to use

Use when processing raw profile-mode LC-HRMS chromatograms (mzXML/mzML) and you need to enumerate all candidate peak-apex signals before machine-learning peak/background classification.

## When NOT to use

- Input is already centroided/peak-picked feature data rather than profile-mode LC-HRMS chromatograms
- You need finalized peak groups or a feature matrix — this skill only enumerates candidate apexes; use PeakBot's CNN classification and grouping steps (peakbot.runPeakBot, groupFeatures) instead

## Inputs

- Profile-mode LC-HRMS chromatogram file (mzXML or mzML)
- PeakBot chromatogram settings: noiseLevel, minIntensity, minRT, maxRT, RTpeakWidth, intraScanMaxAdjacentSignalDifferencePPM, interScanMaxSimilarSignalDifferencePPM
- CUDA-enabled Nvidia GPU with CUDA toolkit and cuDNN (or reduced exportBatchSize on <4 GB GPUs)

## Outputs

- Single export file of standardized areas, each 32 scans x 128 m/z signals, indexed by local-maximum coordinates (retention time, m/z)
- Maximum abundance of every standardized area scaled to 1

## How to apply

Load the profile-mode chromatogram with PeakBot's chromatogram reader and scan the retention-time versus m/z signal matrix for all 8-neighborhood local maxima (optionally restricted by settings such as minIntensity, minRT, and maxRT to exclude noise and non-useful scans). Extract the neighborhood of each local maximum and super-sample it to a standardized grid of 32 HRMS scans x 128 m/z signals, then scale each standardized area to a maximum abundance of 1. Export all standardized areas to a single file indexed by local-maximum coordinates using PeakBot's CUDA/GPU-based exporter, e.g. peakbot.cuda.preProcessChromatogram, reducing exportBatchSize from 2048 to 1024 or 512 if less than 4 GB GPU memory is available. The rationale is that a fixed-size, normalized grid makes every candidate directly consumable by the PeakBot-CNN model, which reports peak apex/left-right isomeric compound vs background plus bounding-box and peak-center. Correctness is judged by the exported file containing one 32x128 normalized area per detected local maximum, keyed by its coordinates.

## Related tools

- **PeakBot** (Provides the chromatogram reader, 8-neighborhood local-maximum scan, super-sampling to 32x128 standardized areas, normalization, and CUDA/GPU-based export) — https://github.com/christophuv/PeakBot
- **CUDA toolkit** (Required dependency for the GPU-accelerated standardized-area export (with configurable exportBatchSize))
- **cuDNN** (Required Nvidia library dependency for running PeakBot's GPU-based preprocessing on a CUDA-enabled graphics card)
- **TensorFlow** (Consumes the exported standardized areas downstream as CNN model input (TensorFlow 2.5.0))

## Examples

```
peakbot.cuda.preProcessChromatogram(chromatogram, exportFileName, settings)  # scans rt x mz matrix for local maxima, exports 32x128 areas (set settings.exportBatchSize=512 if GPU < 4 GB)
```

## Evaluation signals

- Exported file exists and contains an entry for every detected local maximum, indexed by its (retention time, m/z) coordinates
- Every entry is a standardized area of exactly 32 HRMS scans x 128 m/z signals
- Each standardized area has maximum abundance scaled to 1
- If GPU memory is under 4 GB, exportBatchSize was reduced from 2048 to 1024 or 512 without export failure
- Detected candidate coordinates fall within the user-specified minRT/maxRT scan windows and exceed minIntensity

## Limitations

- Requires a CUDA-enabled Nvidia graphics card with the CUDA toolkit and cuDNN libraries installed; GPUs with less than 4 GB memory need a reduced exportBatchSize (1024 or 512)
- Only signals above minIntensity and within the minRT/maxRT windows are used as local-maximum seeds; scans outside these ranges (e.g. waste or column reconditioning) are excluded
- This step produces raw candidates only — every local maximum including background signals is exported, so classification of peak vs background is deferred to the CNN model

## Evidence

- [methods] Every detected local maximum is exported as a standardized LC-HRMS area with a pre-set size of 32 HRMS scans (retention-time dimension) by 128 m/z signals by default.: [evidence span withheld — anchor status: not_found]
- [abstract] It first detects all local signal maxima in a chromatogram, which are then extracted as super-sampled standardized areas (retention-time versus m/z).: "It first detects all local signal maxima in a chromatogram, which are then extracted as super-sampled standardized areas (retention-time versus m/z)."
- [methods] it requires a CUDA-enabled graphics card from Nvidia as well as the CUDA tookit and the cuDNN libraries to be installed: "it requires a CUDA-enabled graphics card from Nvidia as well as the CUDA tookit and the cuDNN libraries to be installed"
- [readme] The main functions to detect chromatographic peaks in a LC-HRMS chromatogram with a PeakBot model are peakbot.cuda.preProcessChromatogram for extracting the standardized areas: [evidence span withheld — anchor status: not_found]
- [readme] nosieLevel: signal intensity below which a signal is considered to represent noise. minIntensity: minimum intensity of signals to be used as a local maximum seed.: [evidence span withheld — anchor status: external_doc]
