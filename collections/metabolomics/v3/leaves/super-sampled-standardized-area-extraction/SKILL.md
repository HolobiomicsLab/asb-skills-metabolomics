---
name: super-sampled-standardized-area-extraction
description: Use when raw profile-mode LC-HRMS chromatograms (mzXML/mzML) must be
  converted into fixed-size rt x m/z candidate areas around local maxima before CNN
  peak classification, or when reproducing PeakBot's candidate-generation stage.
when_to_use_negative:
- Input is already a picked/centroided peak list or feature table rather than profile-mode
  LC-HRMS data — PeakBot's candidate extraction requires raw profile signals
- The goal is CNN training-instance generation rather than detection candidate export
  — use the GPU-based generateTestInstances augmentation workflow instead
- No CUDA-capable GPU is available and the export dataset is too large for a CPU-bound
  fallback; the documented CUDA export path assumes GPU memory of at least ~4 GB (or
  reduced exportBatchSize)
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3382
tools:
- name: PeakBot
  role: Provides the chromatogram reader, 8-neighborhood local-maximum scan, CUDA-based
    standardized-area export (preProcessChromatogram), and downstream runPeakBot CNN
    classification
  repo: https://github.com/christophuv/PeakBot
- name: Python
  role: Implementation language of the PeakBot package used to run the extraction
    pipeline
  repo: https://github.com/christophuv/PeakBot
- name: CUDA toolkit
  role: Accelerates the GPU-based export of standardized training/detection areas;
    exportBatchSize tuning (2048/1024/512) depends on available GPU memory
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btac344
    title: 'PeakBot: machine-learning-based chromatographic peak picking'
  source_content_hash: sha256:06f1efd70528d0c09d03ef8d2bcf85fbde2d94ab2bfde235f14d211a14c851ba
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
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# super-sampled-standardized-area-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Pre-CNN candidate peak extraction for PeakBot: every 8-neighborhood local maximum in raw profile-mode LC-HRMS data is exported as a standardized, super-sampled area of 32 HRMS scans by 128 m/z signals, normalized to maximum abundance 1, so a fixed-size input grid can feed the CNN classifier.

## When to use

Use when raw profile-mode LC-HRMS chromatograms (mzXML/mzML) must be converted into fixed-size rt x m/z candidate areas around local maxima before CNN peak classification, or when reproducing PeakBot's candidate-generation stage.

## When NOT to use

- Input is already a picked/centroided peak list or feature table rather than profile-mode LC-HRMS data — PeakBot's candidate extraction requires raw profile signals
- The goal is CNN training-instance generation rather than detection candidate export — use the GPU-based generateTestInstances augmentation workflow instead
- No CUDA-capable GPU is available and the export dataset is too large for a CPU-bound fallback; the documented CUDA export path assumes GPU memory of at least ~4 GB (or reduced exportBatchSize)

## Inputs

- Raw profile-mode LC-HRMS chromatogram file (mzXML or mzML)
- PeakBot chromatogram settings (noiseLevel, minIntensity, minRT, maxRT, RTpeakWidth, intraScanMaxAdjacentSignalDifferencePPM, interScanMaxSimilarSignalDifferencePPM)
- CUDA-enabled Nvidia GPU with CUDA toolkit and cuDNN libraries installed

## Outputs

- Single export file containing one standardized 32 scans x 128 m/z signals area per local maximum, indexed by the maximum's rt/m/z coordinates, each scaled to maximum abundance 1

## How to apply

Load the profile-mode chromatogram (mzXML/mzML) with PeakBot's chromatogram reader, applying user-set bounds such as minRT/maxRT and a noise level / minIntensity floor for seeds. Scan the retention-time versus m/z signal matrix for all 8-neighborhood local maxima; each maximum becomes a candidate peak apex. Extract the neighborhood of each maximum and super-sample it onto a standardized grid of 32 HRMS scans (retention-time dimension) x 128 m/z signals by default, then scale each standardized area so its maximum abundance equals 1. Export all standardized areas to a single file indexed by their local-maximum coordinates using the CUDA/GPU-based export; if less than 4 GB of GPU memory is available, reduce exportBatchSize from 2048 to 1024 or 512. These exported areas are then the input to peakbot.runPeakBot / the TensorFlow CNN, which classifies each area as peak apex, wall, or other background.

## Related tools

- **PeakBot** (Provides the chromatogram reader, 8-neighborhood local-maximum scan, CUDA-based standardized-area export (preProcessChromatogram), and downstream runPeakBot CNN classification) — https://github.com/christophuv/PeakBot
- **Python** (Implementation language of the PeakBot package used to run the extraction pipeline) — https://github.com/christophuv/PeakBot
- **CUDA toolkit** (Accelerates the GPU-based export of standardized training/detection areas; exportBatchSize tuning (2048/1024/512) depends on available GPU memory)

## Examples

```
python detect.py  # uses peakbot.cuda.preProcessChromatogram(chromFile, settings) then peakbot.runPeakBot(areas, model)
```

## Evaluation signals

- The exported output file exists after the export step completes
- Every entry in the exported file is a 32 x 128 standardized area (retention-time dimension x m/z dimension)
- Each standardized area has its maximum abundance scaled to exactly 1
- Each exported area is indexed by the coordinates of its originating local maximum
- Detection output downstream reports a peak-type, -center, and bounding-box per classified area, confirming the areas fed the CNN correctly

## Limitations

- The CUDA-based export assumes a CUDA-enabled Nvidia graphics card with the CUDA toolkit and cuDNN libraries installed; with less than 4 GB GPU memory, exportBatchSize must be reduced from 2048 to 1024 or 512
- Only signals above the user-specified minIntensity seed threshold are considered, and scans outside minRT/maxRT are excluded (e.g. waste or column reconditioning regions)
- Default grid size is fixed at 32 HRMS scans x 128 m/z signals; extremely wide/narrow chromatographic peaks may not fit this standardized rt window
- Correct estimation of intraScanMaxAdjacentSignalDifferencePPM and interScanMaxSimilarSignalDifferencePPM is difficult and influences which profile signals are linked; TOPPView or PeakBot's estimateParameters.py may be needed

## Evidence

- [other] Every detected local maximum is exported as a standardized LC-HRMS area with a pre-set size of 32 HRMS scans (retention-time dimension) by 128 m/z signals by default.: [evidence span withheld — anchor status: not_found]
- [readme] peakbot.cuda.preProcessChromatogram for extracting the standardized areas from the chromatogram and peakbot.runPeakBot for testing the standardized area for chromatographic peaks or backgrounds.: [evidence span withheld — anchor status: external_doc]
- [other] reduce exportBatchSize from 2048 to 1024 or 512 if less than 4 GB GPU memory is available: [evidence span withheld — anchor status: not_found]
