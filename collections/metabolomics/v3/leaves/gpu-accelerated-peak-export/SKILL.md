---
name: gpu-accelerated-peak-export
description: Use when raw LC-HRMS profile-mode data (mzXML/mzML) must be converted
  into CNN-ready candidate peak areas — i.e., before running PeakBot's classification
  or when regenerating/verifying the local-maxima export file.
when_to_use_negative:
- Input is already a peak-picked/centroided or feature-table dataset — this skill
  requires raw profile-mode signal matrices
- Goal is training-instance generation or CNN classification, not candidate extraction
  — use peakbot.train.cuda.generateTestInstances or peakbot.runPeakBot instead
- Detection should be limited to known reference features rather than an untargeted
  sweep of all local maxima
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PeakBot
  role: Provides peakbot.cuda.preProcessChromatogram for extracting/exporting the
    standardized areas from profile-mode chromatograms
  repo: https://github.com/christophuv/PeakBot
- name: CUDA toolkit
  role: GPU dependency enabling the batched CUDA export of standardized areas; install
    cuDNN as well; batch size lowered on GPUs with < 4 GB memory
- name: PeakBot_Example
  role: detect.py script demonstrating the export/detection workflow end to end
  repo: https://github.com/christophuv/PeakBot_Example
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btac344
    title: 'PeakBot: machine-learning-based chromatographic peak picking'
  source_content_hash: sha256:abf92fce20d1da9c2bfd4ce4cb8cd76aea173c839e33e53989222eccb91b88b1
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
  - PeakBot_Example
  - Python
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# gpu-accelerated-peak-export

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Exports every 8-neighborhood local maximum of a profile-mode LC-HRMS chromatogram (mzXML/mzML) as a super-sampled, intensity-normalized 32-scan x 128-m/z standardized area on the GPU, producing the candidate pool that PeakBot's CNN later classifies.

## When to use

Use when raw LC-HRMS profile-mode data (mzXML/mzML) must be converted into CNN-ready candidate peak areas — i.e., before running PeakBot's classification or when regenerating/verifying the local-maxima export file.

## When NOT to use

- Input is already a peak-picked/centroided or feature-table dataset — this skill requires raw profile-mode signal matrices
- Goal is training-instance generation or CNN classification, not candidate extraction — use peakbot.train.cuda.generateTestInstances or peakbot.runPeakBot instead
- Detection should be limited to known reference features rather than an untargeted sweep of all local maxima

## Inputs

- Profile-mode LC-HRMS chromatogram file (mzXML or mzML)
- PeakBot chromatogram properties (noiseLevel, minIntensity, minRT, maxRT, RTpeakWidth, intraScanMaxAdjacentSignalDifferencePPM, interScanMaxSimilarSignalDifferencePPM)

## Outputs

- Single export file of standardized areas, each 32 HRMS scans x 128 m/z signals, indexed by local-maximum coordinates, max abundance scaled to 1

## How to apply

Load the profile-mode chromatogram with PeakBot's chromatogram reader after setting the minimal parameters (noiseLevel, minIntensity, minRT/maxRT to exclude waste/column-reconditioning scans, and the intra/inter-scan ppm difference parameters). Scan the retention-time x m/z matrix for all 8-neighborhood local maxima using minIntensity as the seed threshold. Extract each maximum's neighborhood and super-sample it onto the standardized 32 scans x 128 m/z grid, then scale every area so its maximum abundance equals 1. Export the normalized areas to a single file indexed by local-maximum coordinates using the CUDA/GPU export; if the GPU has less than 4 GB memory, reduce exportBatchSize from 2048 to 1024 or 512.

## Related tools

- **PeakBot** (Provides peakbot.cuda.preProcessChromatogram for extracting/exporting the standardized areas from profile-mode chromatograms) — https://github.com/christophuv/PeakBot
- **CUDA toolkit** (GPU dependency enabling the batched CUDA export of standardized areas; install cuDNN as well; batch size lowered on GPUs with < 4 GB memory)
- **PeakBot_Example** (detect.py script demonstrating the export/detection workflow end to end) — https://github.com/christophuv/PeakBot_Example

## Examples

```
import peakbot; cores = peakbot.cuda.preProcessChromatogram(chromFile)  # extract standardized areas per detect.py from PeakBot_Example
```

## Evaluation signals

- Exported file exists and every entry is a 32x128 standardized area with maximum abundance scaled to 1
- Entries are indexed by their local-maximum (rt, m/z) coordinates from the chromatogram
- All 8-neighborhood local maxima above minIntensity within the minRT–maxRT window appear in the export (no candidate missed)
- exportBatchSize set appropriately for GPU memory (2048, or 1024/512 when < 4 GB VRAM) with no CUDA out-of-memory errors

## Limitations

- Requires a CUDA-enabled Nvidia graphics card plus the CUDA toolkit and cuDNN libraries; CPU-mode environments cannot use the GPU-based export
- Export is untargeted over all local maxima — downstream CNN classification, not the export itself, separates true peaks from background signals (e.g., walls)
- Grid dimensions and normalization are fixed defaults (32 scans x 128 m/z, max abundance 1); deviating chromatogram properties require adjusting minRT/maxRT and ppm-difference parameters

## Evidence

- [methods] Every detected local maximum is exported as a standardized LC-HRMS area with a pre-set size of 32 HRMS scans (retention-time dimension) by 128 m/z signals by default.: [evidence span withheld — anchor status: not_found]
- [readme] peakbot.cuda.preProcessChromatogram for extracting the standardized areas from the chromatogram and peakbot.runPeakBot for testing the standardized area for chromatographic peaks or backgrounds.: [evidence span withheld — anchor status: external_doc]
