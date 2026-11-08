---
name: two-color-image-simulation-generation
description: 'Use when validating a colocalization algorithm: you need synthetic two-color
  image pairs with known ground-truth colocalization (100% down to 0%) or a subvisual
  colocalization case to measure false-positive/false-negative misclassification rates.'
when_to_use_negative:
- Quantifying colocalization on real acquired microscopy data (e.g., Zeiss LSM 510
  CFP/YFP live-cell images) — this skill is for synthetic validation with known ground
  truth, not for measurement on experimental images
- FRET-based interaction measurements where detection requires average binding-protein
  distances <10 nm — simulation of pixel overlap cannot substitute for FRET
- Single-channel images or image pairs lacking a two-dimensional red/green intensity
  histogram representation
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: MIPAV
  role: Software in which the Costes colocalization algorithm (validated via these
    simulations) was made available to the NIH research community
- name: Bitplane
  role: Commercial implementation of the validated colocalization algorithm (Zurich,
    Switzerland)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1529/biophysj.103.038422
    title: Automatic and Quantitative Measurement of Protein-Protein Colocalization
      in Live Cells
  source_content_hash: sha256:836c21f5d1c94825c7d5d83962222cdeb30a6b07356681bd64670d56d2e35171
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bitplane
  - MIPAV
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# two-color-image-simulation-generation

## Summary

Generates simulated two-color (red/green) image pairs spanning 100% to 0% colocalization, plus subvisually obvious test cases, in order to quantitatively validate a colocalization-classification algorithm such as the Costes automatic statistical method.

## When to use

Use when validating a colocalization algorithm: you need synthetic two-color image pairs with known ground-truth colocalization (100% down to 0%) or a subvisual colocalization case to measure false-positive/false-negative misclassification rates.

## When NOT to use

- Quantifying colocalization on real acquired microscopy data (e.g., Zeiss LSM 510 CFP/YFP live-cell images) — this skill is for synthetic validation with known ground truth, not for measurement on experimental images
- FRET-based interaction measurements where detection requires average binding-protein distances <10 nm — simulation of pixel overlap cannot substitute for FRET
- Single-channel images or image pairs lacking a two-dimensional red/green intensity histogram representation

## Inputs

- Simulated two-color (red/green) image series parameterized by ground-truth colocalization fraction from 100% to 0%
- Texture or PSF width estimates derived from the smaller of the two image autocorrelation widths (to set block size)
- Ground-truth labels of which pixels are colocalized (for misclassification counting)

## Outputs

- Colocalization coefficients (proportion of each protein colocalized with the other) per simulated image
- Misclassification rate: proportion of pixels classified as false positives and false negatives across the simulation series (target ~5%)
- Subvisual-detection report: automatic colocalization detection result on a non-visually-obvious simulated image versus visual expectation

## How to apply

Generate a simulated two-color image series covering the full colocalization range, from 100% of green pixels colocalized with red pixels down to no colocalization, using the randomization-based statistical approach of the Costes method. For each simulated pair, divide the images into independent blocks approximately the size of the texture or the PSF — measured as the smaller of the widths of the autocorrelations for the two images — and scramble blocks rather than individual pixels to account for correlations in adjacent pixel intensities. Perform 200 randomizations for each region analyzed to build the distribution of random overlap, then select colocalized pixels in the two-dimensional histogram using the automatic statistical criterion, including selection of pixels closest to the diagonal line to distinguish low-intensity colocalized pixels from background. Compute the colocalization coefficients (the proportion of each protein colocalized with the other) for each simulated image. Count the proportion of pixels misclassified as false positives and false negatives across the series, targeting the reported value of ~5%. Finally, run a simulated image where colocalization was not obvious visually and record the automatic detection result versus visual expectation.

## Related tools

- **MIPAV** (Software in which the Costes colocalization algorithm (validated via these simulations) was made available to the NIH research community)
- **Bitplane** (Commercial implementation of the validated colocalization algorithm (Zurich, Switzerland))

## Evaluation signals

- Misclassification rate on the simulated series is small: ~5% of pixels as false positives and false negatives
- Colocalization coefficients vary monotonically across the series, from ~100% green-pixel colocalization with red down to no colocalization
- Each region analyzed used exactly 200 randomizations to build the distribution of random overlap
- Randomization was performed by scrambling blocks sized to the smaller autocorrelation width (texture/PSF), not individual pixels
- The automatic criterion flags the subvisual simulated image as colocalized, matching the paper's non-visually-obvious test case

## Limitations

- Block scrambling (rather than pixel scrambling) is required because adjacent pixel intensities are correlated; using pixel-level randomization on textured/PSF-broadened simulated images invalidates the statistical criterion
- Pixels closest to the diagonal line in the two-dimensional histogram must be selected separately to distinguish low-intensity colocalized pixels from background
- Assuming that the selected points in the two-dimensional histogram are entirely colocalized can overpredict the amount of colocalization

## Evidence

- [full_text] Block scrambling rationale: [evidence span withheld — anchor status: not_found]
