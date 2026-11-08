---
name: tumor-region-annotation-overlay
description: Use when raw imzML/HDF5 MSI pixel data (e.g. 100 µm FT-ICR acquisitions
  with tens of thousands of m/z variables) must be visualized as an ion image and
  compared against expert-annotated tumor regions from a consecutive H&E-stained section.
when_to_use_negative:
- No pathologist-annotated H&E section consecutive to the MSI section is available,
  so no tumor region mask can be transferred onto the MSI pixel grid.
- The goal is probabilistic pixel-level tumor classification by massNet itself, not
  spatial visualization/localization of a single m/z ion image.
- Input is already an aggregated per-peak feature or correlation table with no spatial
  pixel coordinates, so no 2D ion image can be reconstructed.
edam_operation: http://edamontology.org/operation_3552
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: massNet
  role: Python data-loading and raw-MSI utilities used to read the HDF5 dataset and
    reconstruct the ion image without peak picking
  repo: https://github.com/wabdelmoula/massNet
- name: Python
  role: Implementation environment for the ion-image reconstruction, annotation overlay,
    and intensity quantification
  repo: https://github.com/wabdelmoula/massNet
- name: h5py
  role: Reads the HDF5-converted MSI data that massNet uses as its working file format
- name: Zeiss Observer Z.1 microscope
  role: Acquires the H&E images of the consecutive section from which tumor annotations
    are derived
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btac032
    title: 'massNet: integrated processing and classification of spatially resolved
      mass spectrometry data using deep learning for rapid tumor delineation'
  source_content_hash: sha256:991617932940b24695e15409332c909170317a41a9f59d268cc23eafd0a04adf
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/wabdelmoula/massNet
  repo_url: https://github.com/wabdelmoula/massNet
  tools:
  - Python
  - Zeiss Observer Z.1 microscope
  - h5py
  - massNet
  edam_operation: http://edamontology.org/operation_3552
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# tumor-region-annotation-overlay

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs a 2D ion image for a specific m/z channel from raw MALDI FT-ICR MSI data and overlays the pathologist-derived tumor annotation transferred from a consecutive H&E section, enabling spatial localization of ion signal to tumor versus normal tissue.

## When to use

Use when raw imzML/HDF5 MSI pixel data (e.g. 100 µm FT-ICR acquisitions with tens of thousands of m/z variables) must be visualized as an ion image and compared against expert-annotated tumor regions from a consecutive H&E-stained section.

## When NOT to use

- No pathologist-annotated H&E section consecutive to the MSI section is available, so no tumor region mask can be transferred onto the MSI pixel grid.
- The goal is probabilistic pixel-level tumor classification by massNet itself, not spatial visualization/localization of a single m/z ion image.
- Input is already an aggregated per-peak feature or correlation table with no spatial pixel coordinates, so no 2D ion image can be reconstructed.

## Inputs

- imzML file exported from SCiLS lab 2020a and converted to HDF5 format (MALDI 9.4 Tesla FT-ICR MSI, 100 µm spatial resolution, 85,062 m/z variables)
- Pathologist-annotated tumor region mask transferred from H&E image of the consecutive tissue section
- Target m/z value with tolerance window (e.g. 400.9546 ± 0.01)

## Outputs

- 2D ion image reconstructed at the tissue-section pixel grid (100 µm pixels) for the selected m/z channel
- Overlay image of the ion map with the tumor annotation boundary (e.g. mz400.9546_ion_image.png)
- Mean ion intensity in tumor region versus normal region and the tumor fold-elevation of the selected m/z

## How to apply

Load the imzML/HDF5 MSI data (converted from SCiLS lab export; HDF5 is massNet's working format) using the massNet Python utilities with h5py, without prior peak picking or preprocessing, consistent with massNet's raw-data approach. Select the m/z channel of interest — e.g. m/z 400.9546, the peak most correlated with the tumor class — using a ±0.01 tolerance window applied to the 85,062 m/z variables. Reconstruct ion intensities at each 100 µm pixel into a 2D ion image matching the tissue section dimensions. Overlay the pathologist-annotated tumor region transferred from the consecutive H&E section (Randall et al., 2020), then compute mean ion intensity within tumor versus normal regions and report the fold-elevation. Render the ion image with the tumor annotation boundary (e.g. mz400.9546_ion_image.png) and accept qualitatively per Figure 5e when signal is visibly elevated in tumor.

## Related tools

- **massNet** (Python data-loading and raw-MSI utilities used to read the HDF5 dataset and reconstruct the ion image without peak picking) — https://github.com/wabdelmoula/massNet
- **Python** (Implementation environment for the ion-image reconstruction, annotation overlay, and intensity quantification) — https://github.com/wabdelmoula/massNet
- **h5py** (Reads the HDF5-converted MSI data that massNet uses as its working file format)
- **Zeiss Observer Z.1 microscope** (Acquires the H&E images of the consecutive section from which tumor annotations are derived)

## Evaluation signals

- The rendered ion image shows visibly elevated m/z 400.9546 signal within the annotated tumor region (qualitative acceptance per Figure 5e; no numeric threshold reported).
- Mean ion intensity in the annotated tumor region is higher than in the normal region (fold-elevation > 1).
- The reconstructed ion image grid matches the tissue section dimensions and 100 µm spatial pixel resolution of the 9.4 Tesla SolariX acquisition.
- The m/z extraction used a ±0.01 tolerance window against the 85,062 m/z variables of the FT-ICR dataset, with no prior peak picking applied.
- The overlaid tumor boundary corresponds to the pathologist annotation manually transferred from the consecutive H&E section.

## Limitations

- Ground-truth tumor annotation based solely on annotated histology is an approximation and may be suboptimal, especially in cases of intra-tumor heterogeneity.
- Validation of the overlay is qualitative per Figure 5e; no numeric intensity threshold for tumor elevation is reported in the text.
- The paper's classification scope is binary (tumor vs normal); tumor-margin ground truth would require further pathological investigation.

## Evidence

- [methods] H&E acquisition and MSI spatial resolution: [evidence span withheld — anchor status: not_found]
