---
name: technical-replicate-repeatability-rsd-analysis
description: Use when assessing how repeatable a peak-detection/integration workflow
  is across technical replicate injections of the same sample, e.g. by computing per-feature
  RSD of exported peak areas and comparing against reported thresholds.
when_to_use_negative:
- Input is already a feature table from a different peak-picking tool whose integration
  behavior this skill cannot validate (the RSD here specifically evaluates PeakBot's
  peak-area integration)
- Comparing biological replicates or group-level variability rather than technical
  replicate injections of the same sample
- Only a single injection per sample is available, so no across-replicate RSD can
  be computed
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: PeakBot
  role: Detects chromatographic peaks and integrates peak areas in LC-HRMS profile-mode
    replicate chromatograms; output peak areas are the input to the RSD analysis
  repo: https://github.com/christophuv/PeakBot
- name: Python
  role: Used with pandas to load the peak-area TSV and compute per-feature RSD across
    replicate injections
  repo: https://github.com/christophuv/PeakBot
- name: Anaconda
  role: Recommended virtual environment for installing PeakBot with its CUDA dependencies
  repo: https://github.com/christophuv/PeakBot
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btac344
    title: 'PeakBot: machine-learning-based chromatographic peak picking'
  source_content_hash: sha256:c67df07fd330fe90782fd3eb2e4c3bec026555561e2ef4d18c644d90b39f245c
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
  - cuDNN
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# technical-replicate-repeatability-rsd-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies the repeatability of peak-area integration across technical replicate injections by computing per-feature relative standard deviation (RSD) on PeakBot peak-picking output, as demonstrated on the WheatEar LC-HRMS study.

## When to use

Use when assessing how repeatable a peak-detection/integration workflow is across technical replicate injections of the same sample, e.g. by computing per-feature RSD of exported peak areas and comparing against reported thresholds.

## When NOT to use

- Input is already a feature table from a different peak-picking tool whose integration behavior this skill cannot validate (the RSD here specifically evaluates PeakBot's peak-area integration)
- Comparing biological replicates or group-level variability rather than technical replicate injections of the same sample
- Only a single injection per sample is available, so no across-replicate RSD can be computed

## Inputs

- LC-HRMS profile-mode chromatogram files of technical replicate injections (e.g. WheatEar study replicates)
- Trained PeakBot CNN model
- Exported peak-area table (TSV, e.g. wheatear_peak_areas.tsv) with per-feature peak areas per injection

## Outputs

- Per-feature RSD values of peak areas across technical replicate injections
- Repeatability summary CSV (wheatear_repeatability_rsd.csv) with mean replicate RSD and fraction of features with RSD < 20%

## How to apply

First run peak detection on the profile-mode LC-HRMS chromatograms of the technical replicate injections (e.g. WheatEar replicates) with PeakBot using a trained CNN model, exporting detected peaks as a tab-separated-values peak-area table (wheatear_peak_areas.tsv) and optionally featureML files. Then load the peak-area table with pandas and compute, per feature, the relative standard deviation (RSD = standard deviation / mean) of peak areas across the technical replicate injections. Aggregate the per-feature RSDs into a repeatability summary (wheatear_repeatability_rsd.csv) reporting the mean replicate RSD and the fraction of features with RSD below 20%. Validation follows the article's reported repeatability: accept the reproduction when the mean replicate RSD of peak areas is below 10% and at least 90% of features show an RSD below 20%.

## Related tools

- **PeakBot** (Detects chromatographic peaks and integrates peak areas in LC-HRMS profile-mode replicate chromatograms; output peak areas are the input to the RSD analysis) — https://github.com/christophuv/PeakBot
- **Python** (Used with pandas to load the peak-area TSV and compute per-feature RSD across replicate injections) — https://github.com/christophuv/PeakBot
- **Anaconda** (Recommended virtual environment for installing PeakBot with its CUDA dependencies) — https://github.com/christophuv/PeakBot

## Examples

```
conda env create -f Windows10_python38_withGPU.yml; pip install git+https://github.com/christophuv/PeakBot; python -c "import pandas as pd; df=pd.read_csv('wheatear_peak_areas.tsv',sep='\t'); rsd=df.groupby('feature').peakArea.apply(lambda x: x.std()/x.mean()*100); rsd.to_csv('wheatear_repeatability_rsd.csv')"
```

## Evaluation signals

- Mean replicate RSD of PeakBot peak areas is below 10% across features
- At least 90% of features show an RSD below 20%
- Repeatability summary CSV is produced (wheatear_repeatability_rsd.csv) with mean RSD and fraction of features with RSD < 20%
- Detected peaks export successfully as TSV and featureML files (featureML viewable in TOPPView from the OpenMS toolbox)

## Limitations

- PeakBot requires a trained CNN model; a minimum of 100 reference features is needed to train the model to achieve high-quality peak-picking results
- GPU inference is memory-hungry: exportBatchSize of 2048 uses ~4 GB GPU memory, so batch size must be reduced to 1024/512 if less memory is available
- PeakBot expects LC-HRMS profile-mode data with correctly set noiseLevel, minRT/maxRT, and ppm parameters (intraScanMaxAdjacentSignalDifferencePPM, interScanMaxSimilarSignalDifferencePPM); wrong settings degrade detections and hence repeatability

## Evidence

- [results] PeakBot's peak-area integration of the WheatEar technical replicate injections showed on average less than 10% RSD across features, with 90% of features having RSD values lower than 20%.: [evidence span withheld — anchor status: not_found]
- [methods] Export the detected chromatographic peaks as a tab-separated-values file (wheatear_peak_areas.tsv) and as featureML files (viewable in TOPPView from the OpenMS toolbox): [evidence span withheld — anchor status: not_found]
