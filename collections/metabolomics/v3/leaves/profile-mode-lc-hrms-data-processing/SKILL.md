---
name: profile-mode-lc-hrms-data-processing
description: Use when raw mzXML/mzML LC-HRMS profile-mode data must be converted into
  fixed-size (32 scans x 128 m/z) standardized areas around every local maximum, prior
  to CNN peak classification.
when_to_use_negative:
- Input is already a centroided peak list or a feature table (e.g. RT/m/z/intensity
  matrix) — PeakBot requires profile-mode data to extract local-maxima neighborhoods.
- The goal is feature alignment/grouping of already-detected peaks (use peakbot.cuda.KNNalignFeatures
  / groupFeatures instead).
- No CUDA-enabled GPU is available and very large batches must be exported quickly,
  or GPU memory is below 4 GB without lowering exportBatchSize.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: PeakBot
  role: Python package whose chromatogram reader, 8-neighborhood local-maxima detection,
    and CUDA export (peakbot.cuda.preProcessChromatogram) produce the standardized
    areas
  repo: https://github.com/christophuv/PeakBot
- name: Python 3.8
  role: Runtime environment in which PeakBot is implemented and preprocessing scripts
    are executed
  repo: https://github.com/christophuv/PeakBot
- name: TensorFlow 2.5.0
  role: Machine-learning backend consuming the exported standardized areas for CNN
    classification in the subsequent step
- name: CUDA toolkit
  role: Enables GPU-based export of standardized areas and generation of training
    instances; requires cuDNN libraries
- name: Anaconda
  role: Recommended virtual-environment manager that automatically installs CUDA dependencies
    for PeakBot
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btac344
    title: 'PeakBot: machine-learning-based chromatographic peak picking'
  source_content_hash: sha256:472a2b3b2a445772f6460631547d1a7866e95aed4acd0cd99443b99ef0685812
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
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# profile-mode-lc-hrms-data-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocess raw profile-mode LC-HRMS chromatograms by detecting all local signal maxima and exporting each as a super-sampled, intensity-scaled standardized area ready for CNN-based peak-apex classification with PeakBot.

## When to use

Use when raw mzXML/mzML LC-HRMS profile-mode data must be converted into fixed-size (32 scans x 128 m/z) standardized areas around every local maximum, prior to CNN peak classification.

## When NOT to use

- Input is already a centroided peak list or a feature table (e.g. RT/m/z/intensity matrix) — PeakBot requires profile-mode data to extract local-maxima neighborhoods.
- The goal is feature alignment/grouping of already-detected peaks (use peakbot.cuda.KNNalignFeatures / groupFeatures instead).
- No CUDA-enabled GPU is available and very large batches must be exported quickly, or GPU memory is below 4 GB without lowering exportBatchSize.

## Inputs

- Raw profile-mode LC-HRMS chromatogram file (mzXML or mzML)
- User-specified LC-HRMS settings: nosieLevel, minIntensity, minRT, maxRT, RTpeakWidth, intraScanMaxAdjacentSignalDifferencePPM, interScanMaxSimilarSignalDifferencePPM
- CUDA-enabled Nvidia GPU with CUDA toolkit and cuDNN installed (CPU-only export possible)

## Outputs

- Single export file of standardized 32 scans x 128 m/z areas, each scaled to maximum abundance 1 and indexed by local-maximum coordinates (retention time, m/z)
- Standardized areas ready as CNN input for peak-apex/background classification via peakbot.runPeakBot

## How to apply

Load the profile-mode chromatogram (mzXML/mzML) with PeakBot's chromatogram reader, first setting chromatogram-level parameters such as nosieLevel, minIntensity, minRT and maxRT to exclude noise and scans outside the analytical range. Scan the retention-time versus m/z signal matrix for all 8-neighborhood local maxima (seeding constrained by minIntensity), then extract each maximum's neighborhood and super-sample it to a standardized grid of 32 HRMS scans (retention-time dimension) by 128 m/z signals. Scale every standardized area to a maximum abundance of 1 so that peak shape, not raw intensity, drives downstream CNN classification. Export all areas to a single file indexed by local-maximum coordinates using PeakBot's CUDA/GPU-based export with exportBatchSize=2048, reducing it to 1024 or 512 if less than 4 GB of GPU memory is available. Run preprocessing via peakbot.cuda.preProcessChromatogram and pass the exported areas to peakbot.runPeakBot for classification.

## Related tools

- **PeakBot** (Python package whose chromatogram reader, 8-neighborhood local-maxima detection, and CUDA export (peakbot.cuda.preProcessChromatogram) produce the standardized areas) — https://github.com/christophuv/PeakBot
- **Python 3.8** (Runtime environment in which PeakBot is implemented and preprocessing scripts are executed) — https://github.com/christophuv/PeakBot
- **TensorFlow 2.5.0** (Machine-learning backend consuming the exported standardized areas for CNN classification in the subsequent step)
- **CUDA toolkit** (Enables GPU-based export of standardized areas and generation of training instances; requires cuDNN libraries)
- **Anaconda** (Recommended virtual-environment manager that automatically installs CUDA dependencies for PeakBot)

## Examples

```
peakbot.cuda.preProcessChromatogram(chromatogram, standardizedAreaSizeRT=32, standardizedAreaSizeMZ=128, exportBatchSize=2048); peaks = peakbot.runPeakBot(standardizedAreas, model)
```

## Evaluation signals

- The exported file exists and every entry is a 32x128 standardized area with maximum abundance scaled to 1.
- Every detected 8-neighborhood local maximum above minIntensity has exactly one corresponding entry in the export file, indexed by its (RT, m/z) coordinates.
- exportBatchSize was kept at 2048 only when >=4 GB GPU memory was available; otherwise reduced to 1024 or 512 without export errors.
- Subsequent peakbot.runPeakBot classification on the exported areas returns a peak-type decision (peak vs. background/wall) plus bounding-box and peak-center for each area.

## Limitations

- The export is CUDA/GPU-based; with less than 4 GB GPU memory exportBatchSize must be reduced from 2048 to 1024 or 512.
- Only signals above nosieLevel and minIntensity, and scans between minRT and maxRT, are considered — regions flagged as waste or column reconditioning are excluded.
- Preprocessing operates on raw profile-mode data only; centroided inputs cannot be used.
- Parameters intraScanMaxAdjacentSignalDifferencePPM and interScanMaxSimilarSignalDifferencePPM are difficult to estimate manually (TOPPView or estimateParameters.py are recommended).

## Evidence

- [methods] Every detected local maximum is exported as a standardized LC-HRMS area with a pre-set size of 32 HRMS scans: [evidence span withheld — anchor status: not_found]
- [readme] reduce exportBatchSize from 2048 to 1024 or 512 if less than 4 GB GPU memory is available: [evidence span withheld — anchor status: not_found]
- [readme] peakbot.cuda.preProcessChromatogram for extracting the standardized areas from the chromatogram: [evidence span withheld — anchor status: external_doc]
- [methods] Scale each standardized area to a maximum abundance of 1: [evidence span withheld — anchor status: not_found]
