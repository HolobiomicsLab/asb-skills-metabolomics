---
name: ump-peak-localization-in-tissue
description: Use when a specific m/z peak identified from MSI analysis (e.g. a tumor-correlated
  peak from msiPL) must be spatially localized — reconstructing its ion image at 100
  µm pixels and checking elevation within a histology-annotated region.
when_to_use_negative:
- The goal is pixel-based tissue classification rather than visualizing/localizing
  a single known m/z peak — use the massNet classification workflow instead
- The data has already been peak-picked/preprocessed into a feature table, since massNet
  and this localization approach operate directly on raw spectra without prior preprocessing
- Multi-class tissue delineation is required — the published massNet model was only
  demonstrated for binary classification (normal vs tumor)
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: massNet
  role: Python data-loading utilities for imzML/HDF5 MSI data and the deep learning
    framework that identified the m/z channel without prior peak picking
  repo: github.com/wabdelmoula/massNet
- name: msiPL
  role: Identifies candidate peaks and spatial patterns whose Pearson correlation
    (≥ 0.7) with the tumor class determines which m/z to localize
- name: Keras
  role: Implementation platform (with tensorflow backend) for the deep learning architecture
    used to classify spectra and surface the localized peak
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btac032
    title: 'massNet: integrated processing and classification of spatially resolved
      mass spectrometry data using deep learning for rapid tumor delineation'
  source_content_hash: sha256:7fc260f7fc963bd3e4345ea48fed71b78ba4c8d4920eba5bf884462e3b01ac6c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/wabdelmoula/massNet
  tools:
  - Keras
  - Python
  - massNet
  - msiPL
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# ump-peak-localization-in-tissue

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Spatially localizes a specific m/z peak from raw MALDI FT-ICR mass spectrometry imaging data by reconstructing its 2D ion image at native pixel resolution and comparing it to pathologist-annotated tissue regions, as done in the massNet GBM PDX study for m/z 400.9546.

## When to use

Use when a specific m/z peak identified from MSI analysis (e.g. a tumor-correlated peak from msiPL) must be spatially localized — reconstructing its ion image at 100 µm pixels and checking elevation within a histology-annotated region.

## When NOT to use

- The goal is pixel-based tissue classification rather than visualizing/localizing a single known m/z peak — use the massNet classification workflow instead
- The data has already been peak-picked/preprocessed into a feature table, since massNet and this localization approach operate directly on raw spectra without prior preprocessing
- Multi-class tissue delineation is required — the published massNet model was only demonstrated for binary classification (normal vs tumor)

## Inputs

- imzML or HDF5 MALDI FT-ICR MSI data (raw, unprocessed; e.g. 85,062 m/z variables at 100 µm spatial resolution)
- Target m/z value with tolerance window (e.g. m/z 400.9546 ± 0.01), typically a co-localized peak from msiPL (Pearson correlation ≥ 0.7)
- Pathologist-annotated tumor region transferred from a consecutive H&E-stained tissue section

## Outputs

- 2D ion image of the selected m/z peak with tumor annotation boundary overlay (e.g. mz400.9546_ion_image.png)
- Mean ion intensity in tumor region versus normal region and reported fold-elevation
- Spatial co-localization assessment of the peak's Pearson-correlated pattern with the tumor class

## How to apply

Load the imzML/HDF5 MALDI FT-ICR MSI data (e.g. Metabolomics Workbench PR001292, 85,062 m/z variables) using the massNet data-loading utilities in Python, without prior peak picking or preprocessing. Select the m/z channel of interest with a ±0.01 tolerance window (e.g. m/z 400.9546 ± 0.01) directly from the raw spectra. Reconstruct ion intensities at each 100 µm pixel into a 2D ion image matching the tissue section dimensions, then overlay or compare against the pathologist-annotated tumor region transferred from the consecutive H&E section. Compute mean ion intensity inside the annotated tumor region versus normal region to report fold-elevation. Prioritize candidate peaks using msiPL spatial patterns with Pearson correlation ≥ 0.7 for co-localized m/z peaks; render the final ion image with the annotation boundary and save it (e.g. mz400.9546_ion_image.png).

## Related tools

- **massNet** (Python data-loading utilities for imzML/HDF5 MSI data and the deep learning framework that identified the m/z channel without prior peak picking) — github.com/wabdelmoula/massNet
- **msiPL** (Identifies candidate peaks and spatial patterns whose Pearson correlation (≥ 0.7) with the tumor class determines which m/z to localize)
- **Keras** (Implementation platform (with tensorflow backend) for the deep learning architecture used to classify spectra and surface the localized peak)

## Examples

```
import h5py; f = h5py.File('msi_data.h5','r'); mz = f['mz'][:]; mask = (mz >= 400.9446) & (mz <= 400.9646); ion_image = f['spectra'][:, mask].mean(axis=1)
```

## Evaluation signals

- The rendered ion image shows visibly elevated m/z 400.9546 signal within the annotated tumor region (qualitative acceptance per Figure 5e)
- Mean tumor-region intensity exceeds mean normal-region intensity, reported as a fold-elevation
- The selected peak passes the co-localization threshold: Pearson correlation coefficient T ≥ 0.7 with the tumor-class spatial pattern
- The extracted m/z window (±0.01) matches a plausible metabolite assignment, e.g. C9H13N2O9P [M+2K-H]+ at 400.9554553 (HMDB, -2.1 ppm)

## Limitations

- Acceptance is qualitative: no numeric fold-elevation threshold is reported in the paper text for judging tumor-region elevation
- Ground truth annotation based solely on annotated histology is an approximation and may not be optimal, especially for intra-tumor heterogeneity
- No software version numbers or hyperparameter values for the trained massNet model are reported in the section text, limiting exact reproducibility

## Evidence
