---
name: maldi-fticr-ion-image-extraction
description: Use when raw imzML/HDF5 FT-ICR MSI data (e.g., tens of thousands of m/z
  variables) contains a specific candidate m/z (e.g., 400.9546) whose spatial pattern
  must be rendered as an ion image and compared against histology-annotated tumor
  regions.
when_to_use_negative:
- The data have already been peak-picked or reduced to a feature table, since this
  skill depends on extracting directly from raw spectral channels without preprocessing
- The goal is pixel-level tissue classification rather than visualizing the spatial
  distribution of a single candidate m/z peak
- The dataset is not spatially resolved MSI (no pixel coordinates), so no ion image
  can be reconstructed
edam_operation: http://edamontology.org/operation_0570
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: massNet
  role: Provides Python data-loading utilities for reading the raw imzML/HDF5 FT-ICR
    MSI data and reconstructing the ion image without prior peak picking
  repo: github.com/wabdelmoula/massNet
- name: Python
  role: Implementation environment (with h5py for HDF5 access and matplotlib-style
    rendering) used to extract the m/z channel and generate the ion image PNG
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btac032
    title: 'massNet: integrated processing and classification of spatially resolved
      mass spectrometry data using deep learning for rapid tumor delineation'
  source_content_hash: sha256:cb359ae0627a4c865b84e5c4ffb8583c2a25f5e5702087960ea5a410eefefe96
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/wabdelmoula/massNet
  tools:
  - Python
  - massNet
  edam_operation: http://edamontology.org/operation_0570
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# maldi-fticr-ion-image-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts and spatially reconstructs a single m/z channel from raw high-resolution MALDI FT-ICR mass spectrometry imaging data into a 2D ion image, without peak picking, to visualize the spatial distribution of a candidate tumor-associated peak.

## When to use

Use when raw imzML/HDF5 FT-ICR MSI data (e.g., tens of thousands of m/z variables) contains a specific candidate m/z (e.g., 400.9546) whose spatial pattern must be rendered as an ion image and compared against histology-annotated tumor regions.

## When NOT to use

- The data have already been peak-picked or reduced to a feature table, since this skill depends on extracting directly from raw spectral channels without preprocessing
- The goal is pixel-level tissue classification rather than visualizing the spatial distribution of a single candidate m/z peak
- The dataset is not spatially resolved MSI (no pixel coordinates), so no ion image can be reconstructed

## Inputs

- Raw imzML or HDF5 MALDI FT-ICR MSI dataset (e.g., 85,062 m/z variables, 100 µm spatial resolution, from Metabolomics Workbench PR001292)
- Target m/z value with tolerance window (e.g., 400.9546 ± 0.01)
- Pathologist-annotated tumor region mask transferred from a consecutive H&E-stained section

## Outputs

- 2D ion image of the selected m/z channel matching the tissue section dimensions
- Mean ion intensity within tumor region versus normal region and fold-elevation estimate
- Rendered PNG ion image with tumor annotation boundary overlay (e.g., mz400.9546_ion_image.png)

## How to apply

Load the raw imzML/HDF5 MSI data (e.g., Metabolomics Workbench PR001292, 9.4 Tesla SolariX FT-ICR, positive ion mode, 100 µm pixel size) using the massNet Python data-loading utilities, deliberately skipping peak picking or spectral preprocessing to stay consistent with massNet's raw-data approach. Select the m/z channel of interest from the 85,062 m/z variables using a ±0.01 tolerance window around the target m/z (e.g., 400.9546). Reconstruct the ion intensities at each 100 µm pixel into a 2D ion image matching the tissue section dimensions. Compare or overlay the ion image with the pathologist-annotated tumor region transferred from a consecutive H&E-stained section to localize the signal. Quantify elevation by computing mean ion intensity inside the annotated tumor region versus the normal region and report the fold-elevation. Render the ion image with the tumor annotation boundary overlaid and save it (e.g., as mz400.9546_ion_image.png).

## Related tools

- **massNet** (Provides Python data-loading utilities for reading the raw imzML/HDF5 FT-ICR MSI data and reconstructing the ion image without prior peak picking) — github.com/wabdelmoula/massNet
- **Python** (Implementation environment (with h5py for HDF5 access and matplotlib-style rendering) used to extract the m/z channel and generate the ion image PNG)

## Evaluation signals

- The reconstructed ion image dimensions match the original tissue section dimensions at the 100 µm pixel grid
- Mean ion intensity within the pathologist-annotated tumor region is visibly and numerically higher than in the normal region (qualitative acceptance per Figure 5e; no numeric threshold is reported)
- The extraction used the ±0.01 m/z tolerance window (e.g., 400.9546 ± 0.01) directly on raw spectral channels with no prior peak picking or preprocessing
- The rendered output PNG includes the tumor annotation boundary transferred from the consecutive H&E section
- The fold-elevation of tumor-region versus normal-region mean intensity is reported alongside the image

## Limitations

- Validation is qualitative (visual elevation of signal in the tumor region per Figure 5e); no numeric intensity threshold is reported in the paper
- The ground truth tumor annotation is based solely on annotated histology, an approximation that may not be optimal, especially for intra-tumor heterogeneity
- The m/z peak must have been previously identified as tumor-correlated (e.g., via msiPL with Pearson correlation ≥ 0.7); this skill does not itself discover candidate peaks

## Evidence

- [methods] FT-ICR acquisition parameters: [evidence span withheld — anchor status: not_found]
- [other] task-specific finding on m/z 400.9546: [evidence span withheld — anchor status: not_found]
