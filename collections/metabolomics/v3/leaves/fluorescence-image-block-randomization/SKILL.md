---
name: fluorescence-image-block-randomization
description: Use when analyzing background-corrected two-channel (e.g. red/green)
  fluorescence images where adjacent pixel intensities are correlated by texture or
  the PSF, and you need a statistical threshold separating specific colocalization
  from random overlap.
when_to_use_negative:
- The question is direct protein–protein proximity rather than statistical intensity
  overlap — FRET, not this colocalization test, is the right assay, and FRET alone
  would only detect interaction if the average distance between the binding proteins
  is <10 nm
- You intend to classify colocalization by selecting pixels closest to the diagonal
  line in the two-dimensional histogram without the randomization step — the article
  warns that assuming such selected points are entirely colocalized can overpredict
  the amount of colocalization
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: Bitplane
  role: Commercial implementation of the block-scrambling colocalization algorithm
- name: MIPAV
  role: NIH software distribution in which the block-scrambling colocalization algorithm
    is made available
- name: Zeiss LSM 510 confocal microscope
  role: Acquires the two-channel confocal images to which the randomization test is
    applied
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1529/biophysj.103.038422
    title: Automatic and Quantitative Measurement of Protein-Protein Colocalization
      in Live Cells
  source_content_hash: sha256:ccf787f22bc8edd0fd39c680ddf039841909da7bff7484a2d15c9c5bd8be1ce9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bitplane
  - MIPAV
  - Zeiss LSM 510 confocal microscope
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# fluorescence-image-block-randomization

## Summary

A significance-testing method for two-color fluorescence colocalization that scrambles image blocks — sized by the texture/PSF autocorrelation width — rather than individual pixels, 200 times per region, to build the null distribution of random overlap from which the colocalization significance threshold is set.

## When to use

Use when analyzing background-corrected two-channel (e.g. red/green) fluorescence images where adjacent pixel intensities are correlated by texture or the PSF, and you need a statistical threshold separating specific colocalization from random overlap.

## When NOT to use

- The question is direct protein–protein proximity rather than statistical intensity overlap — FRET, not this colocalization test, is the right assay, and FRET alone would only detect interaction if the average distance between the binding proteins is <10 nm
- You intend to classify colocalization by selecting pixels closest to the diagonal line in the two-dimensional histogram without the randomization step — the article warns that assuming such selected points are entirely colocalized can overpredict the amount of colocalization

## Inputs

- Background-corrected two-channel (red/green, e.g. CFP/YFP) confocal image intensity arrays
- Per-channel autocorrelation width estimates (texture or PSF size)
- Optionally simulated image series spanning 0–100% colocalization for validation

## Outputs

- Observed Pearson correlation coefficient between the two channels
- Null distribution of random overlap from 200 block scrambles per region
- Significance threshold for colocalization, recorded in colocalization_threshold.csv
- Per-pixel classification into colocalized vs. random-overlap (false-positive/false-negative rates)

## How to apply

Load the two-channel (red/green) image intensity arrays from the confocal acquisition (e.g. Zeiss LSM 510 CFP/YFP pairs excited at 458/514 nm) and perform background correction. Compute the autocorrelation width for each channel and set the block size to the smaller of the two widths, so blocks approximate the size of the texture or the point spread function (PSF) — this is the key correction, because scrambling single pixels would ignore intensity correlation between adjacent pixels. Divide the image region into independent blocks of this size and compute the Pearson correlation coefficient between the two original channels. Then, instead of scrambling individual pixels, scramble the block positions 200 times per region, recomputing the overlap correlation after each randomization to build the null distribution of random overlap. Derive the significance threshold from this null distribution: an observed Pearson correlation exceeding this level indicates specific colocalization, and pixels are classified as colocalized or random accordingly. Record the threshold and null distribution statistics in colocalization_threshold.csv.

## Related tools

- **Bitplane** (Commercial implementation of the block-scrambling colocalization algorithm)
- **MIPAV** (NIH software distribution in which the block-scrambling colocalization algorithm is made available)
- **Zeiss LSM 510 confocal microscope** (Acquires the two-channel confocal images to which the randomization test is applied)

## Evaluation signals

- On simulated images spanning 0–100% colocalization, the pixel misclassification rate (false positives plus false negatives) is small, ~5%
- Exactly 200 block-scramble randomizations were performed for each region analyzed, yielding a null distribution of random-overlap correlations
- The scramble block size equals the smaller of the two channels' autocorrelation widths, approximating the texture or PSF size — not an arbitrary pixel grid
- The reported significance threshold is derived from the null distribution, and observed Pearson correlations above it are flagged as specific colocalization
- Quantified colocalization on known protein–protein interactions, co-compartmentalization, or random overlap correlates well with independent FRET measurements

## Limitations

- FRET alone would only detect interaction if the average distance between the binding proteins is <10 nm — so agreement with FRET is limited to this range and FRET-negative results (e.g. Rev-CRM1, with 130 kDa CRM1) do not necessarily contradict colocalization
- Assuming that the selected points in the two-dimensional histogram are entirely colocalized can actually lead to overprediction of the amount of colocalization; the randomization threshold avoids this but the interpretation remains statistical, not proof of physical binding

## Evidence

- [full_text] images are divided into independent blocks which are approximately the size of the texture or the PSF ... Then instead of scrambling individual pixels, the blocks are scrambled: [evidence span withheld — anchor status: not_found]
