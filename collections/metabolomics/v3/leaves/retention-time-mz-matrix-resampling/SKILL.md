---
name: retention-time-mz-matrix-resampling
description: Use when raw profile-mode mzXML/mzML LC-HRMS chromatograms must be converted
  into fixed-size (rt x mz) candidate-peak areas before CNN-based peak-apex classification,
  e.g. as the preprocessing step of PeakBot peak picking.
when_to_use_negative:
- Input data are already peak-picked/centroided or a processed feature table — PeakBot
  expects profile-mode chromatograms and re-detects local maxima from raw signals
- The goal is targeted quantification with a supplied peak list rather than untargeted
  candidate-signal enumeration before CNN classification
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: PeakBot
  role: Provides the chromatogram reader, local-maxima detection, GPU-based super-sampled
    area export (peakbot.cuda.preProcessChromatogram), and abundance normalization
    to 1
  repo: https://github.com/christophuv/PeakBot
- name: CUDA toolkit
  role: GPU acceleration for generating/exporting standardized areas; CUDA dependencies
    can be installed automatically via the Anaconda environment files
  repo: https://github.com/christophuv/PeakBot
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btac344
    title: 'PeakBot: machine-learning-based chromatographic peak picking'
  source_content_hash: sha256:c78037d2122b3aa2454c4e33545c1b2d427ddd6aad714df2fa47239eaf12d638
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# retention-time-mz-matrix-resampling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Standardizes raw LC-HRMS profile-mode candidate signals by detecting every 8-neighborhood local maximum in the retention-time vs m/z signal matrix and resampling its neighborhood onto a fixed 32 scans x 128 m/z grid, normalized to maximum abundance 1, so downstream CNN classification receives uniformly sized inputs.

## When to use

Use when raw profile-mode mzXML/mzML LC-HRMS chromatograms must be converted into fixed-size (rt x mz) candidate-peak areas before CNN-based peak-apex classification, e.g. as the preprocessing step of PeakBot peak picking.

## When NOT to use

- Input data are already peak-picked/centroided or a processed feature table — PeakBot expects profile-mode chromatograms and re-detects local maxima from raw signals
- The goal is targeted quantification with a supplied peak list rather than untargeted candidate-signal enumeration before CNN classification

## Inputs

- Raw profile-mode LC-HRMS chromatogram file (mzXML or mzML)
- PeakBot runtime settings (noiseLevel, minIntensity, minRT, maxRT, RTpeakWidth, intraScanMaxAdjacentSignalDifferencePPM, interScanMaxSimilarSignalDifferencePPM)
- CUDA-enabled Nvidia GPU with CUDA toolkit and cuDNN libraries installed

## Outputs

- A single export file of standardized 32 scans x 128 m/z areas, each indexed by its local-maximum coordinates, with maximum abundance scaled to 1
- Local-maximum coordinate list (rt, m/z) identifying each candidate signal

## How to apply

Load the profile-mode LC-HRMS chromatogram (mzXML/mzML) with PeakBot's chromatogram reader, applying user settings such as noiseLevel, minIntensity, minRT/maxRT, and intra/inter-scan ppm differences. Scan the retention-time versus m/z matrix for all 8-neighborhood local maxima (respecting minIntensity as the seed threshold). For each local maximum, extract the surrounding signal neighborhood and super-sample it onto the standardized grid of 32 HRMS scans (retention-time dimension) by 128 m/z signals. Scale each standardized area so its maximum abundance equals 1, making intensities comparable across samples. Export all areas to a single file indexed by their local-maximum coordinates using PeakBot's CUDA/GPU export; reduce exportBatchSize from the default 2048 to 1024 or 512 when less than 4 GB GPU memory is available. This normalization is intentional: identical grid size and scale let the subsequent TensorFlow CNN classify peak apexes (plus peak-center and bounding-box) versus background uniformly.

## Related tools

- **PeakBot** (Provides the chromatogram reader, local-maxima detection, GPU-based super-sampled area export (peakbot.cuda.preProcessChromatogram), and abundance normalization to 1) — https://github.com/christophuv/PeakBot
- **CUDA toolkit** (GPU acceleration for generating/exporting standardized areas; CUDA dependencies can be installed automatically via the Anaconda environment files) — https://github.com/christophuv/PeakBot

## Examples

```
python detect.py  # peakbot.cuda.preProcessChromatogram extracts 32x128 standardized areas; peakbot.runPeakBot then classifies them
```

## Evaluation signals

- The exported file exists and every entry is a 32 scans x 128 m/z standardized area
- Each standardized area has its maximum abundance scaled exactly to 1
- Every local maximum in the rt-m/z signal matrix (above minIntensity, within minRT/maxRT) has a corresponding entry indexed by its local-maximum coordinates
- PeakBot offers a function to export detection examples to images for user-verification of the resampled areas

## Limitations

- The GPU/CUDA export can exhaust memory: reduce exportBatchSize from the default 2048 to 1024 or 512 on GPUs with less than 4 GB memory
- Requires a CUDA-enabled Nvidia graphics card plus CUDA toolkit and cuDNN libraries (though CPU-only Anaconda environment files are provided)
- Settings intraScanMaxAdjacentSignalDifferencePPM and interScanMaxSimilarSignalDifferencePPM are difficult to determine manually; PeakBot's estimateParameters.py can estimate them automatically
- Scans outside minRT/maxRT (e.g. waste, column reconditioning) and signals below noiseLevel/minIntensity are excluded from local-maximum seeds

## Evidence

- [methods] Every detected local maximum is exported as a standardized LC-HRMS area with a pre-set size of 32 HRMS scans ... by 128 m/z signals by default.: [evidence span withheld — anchor status: not_found]
- [readme] the main functions to detect chromatographic peaks in a LC-HRMS chromatogram with a PeakBot model are peakbot.cuda.preProcessChromatogram for extracting the standardized areas: [evidence span withheld — anchor status: external_doc]
