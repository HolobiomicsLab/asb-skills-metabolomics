---
name: colocalization-significance-threshold-estimation
description: Use when two-channel (e.g., red/green) fluorescence images show correlated
  pixel intensities due to texture or PSF blur, and a significance threshold for Pearson
  colocalization must be derived rather than assumed.
when_to_use_negative:
- When the question is physical proximity of binding partners rather than statistical
  colocalization — FRET is the appropriate assay, though it only detects interaction
  if the average distance between the binding proteins is <10 nm
- When the two channels are unrelated by microscopy design such that pixel-intensity
  correlations from the PSF/texture are absent and a simple pixel-level randomization
  suffices
- When a pre-validated colocalization threshold already exists for identical acquisition
  conditions (same microscope, channels, and region size), so the 200-scramble null
  distribution need not be rebuilt
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: MIPAV
  role: Distributed implementation of the block-scrambling colocalization significance
    algorithm to the NIH intramural research community
- name: Bitplane
  role: Commercial implementation of the colocalization significance-threshold algorithm
- name: Zeiss LSM 510 confocal microscope
  role: Acquisition of the two-channel confocal images that serve as input to the
    block-scrambling significance test
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1529/biophysj.103.038422
    title: Automatic and Quantitative Measurement of Protein-Protein Colocalization
      in Live Cells
  source_content_hash: sha256:8388a0e747c5aaefac0a1169b428b0ab458e8e6329a210fd49dde9d10e3e4278
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bitplane
  - MIPAV
  - Zeiss LSM 510 confocal microscope
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# colocalization-significance-threshold-estimation

## Summary

Establishes a statistically grounded colocalization significance threshold by scrambling image blocks (not pixels) to build a null distribution of random red/green overlap, accounting for correlated pixel intensities from texture or the PSF.

## When to use

Use when two-channel (e.g., red/green) fluorescence images show correlated pixel intensities due to texture or PSF blur, and a significance threshold for Pearson colocalization must be derived rather than assumed.

## When NOT to use

- When the question is physical proximity of binding partners rather than statistical colocalization — FRET is the appropriate assay, though it only detects interaction if the average distance between the binding proteins is <10 nm
- When the two channels are unrelated by microscopy design such that pixel-intensity correlations from the PSF/texture are absent and a simple pixel-level randomization suffices
- When a pre-validated colocalization threshold already exists for identical acquisition conditions (same microscope, channels, and region size), so the 200-scramble null distribution need not be rebuilt

## Inputs

- Background-corrected two-channel (red/green) fluorescence image intensity arrays (e.g., 3D confocal HeLa cell images from a Zeiss LSM 510)
- Per-channel autocorrelation widths (texture or PSF size) used to set the scrambling block size
- Region of interest within the image to be tested for colocalization

## Outputs

- Null distribution of random-overlap Pearson correlation values from 200 block scrambles
- Significance threshold above which observed correlation indicates specific colocalization (recorded in colocalization_threshold.csv)
- Binary colocalization classification of pixels relative to the threshold

## How to apply

Load the two-channel intensity arrays and apply background correction, then compute the autocorrelation width of each channel and set the block size to the smaller of the two widths so blocks approximate the texture or point spread function (PSF). Divide the image region into independent blocks of this size and compute the Pearson correlation coefficient between the original channels. Instead of scrambling individual pixels, randomize the positions of the blocks 200 times per region, recomputing the overlap correlation after each randomization to build the null distribution of random overlap. Derive the significance threshold from this broadened null distribution so that an observed correlation exceeding it indicates specific colocalization; record the threshold and null-distribution statistics in colocalization_threshold.csv. Validate on simulated images spanning 0–100% colocalization, expecting only a small (~5%) pixel misclassification rate of false positives and false negatives.

## Related tools

- **MIPAV** (Distributed implementation of the block-scrambling colocalization significance algorithm to the NIH intramural research community)
- **Bitplane** (Commercial implementation of the colocalization significance-threshold algorithm)
- **Zeiss LSM 510 confocal microscope** (Acquisition of the two-channel confocal images that serve as input to the block-scrambling significance test)

## Evaluation signals

- The scrambling block size equals the smaller of the two channel autocorrelation widths and approximates the texture or PSF size
- Exactly 200 block randomizations are performed per analyzed region and each yields a recomputed overlap correlation
- On simulated images spanning 0–100% colocalization, the pixel misclassification rate is small (~5% false positives and ~5% false negatives)
- Observed correlation above the derived random-overlap threshold is classified as specific colocalization; results should corroborate independent assays such as FRET on known protein-protein interactions
- colocalization_threshold.csv contains the derived threshold and null-distribution statistics for each analyzed region

## Limitations

- Assuming that the selected points in the two-dimensional histogram are entirely colocalized can lead to overprediction of the amount of colocalization
- The null distribution must be built from block scrambling, not pixel scrambling; pixel-level randomization fails to account for correlations between adjacent pixel intensities caused by texture or the PSF
- Cross-validation against FRET is imperfect: FRET was negative for the Rev-CRM1 complex, presumably due to the large size of CRM1 (130 kDa), so agreement between methods is assay-dependent

## Evidence

- [full_text] Block size set by autocorrelation width: [evidence span withheld — anchor status: not_found]
