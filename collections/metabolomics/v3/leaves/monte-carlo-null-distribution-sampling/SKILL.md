---
name: monte-carlo-null-distribution-sampling
description: Use when testing whether the Pearson correlation between two fluorescence
  channels (e.g., CFP/YFP or red/green from a Zeiss LSM 510) exceeds random overlap,
  and adjacent pixel intensities are spatially correlated by texture or the PSF so
  pixel-level scrambling would be invalid.
when_to_use_negative:
- Input is a single-channel image or lacks a paired second-channel image, so no two-color
  overlap null distribution can be constructed.
- The goal is measuring physical interaction distance between proteins (e.g., FRET,
  which detects interaction only if average binding distance is <10 nm) rather than
  colocalization significance.
- The question is colocalization extent from the 2D histogram's diagonal-selection
  approach, where assuming selected pixels are entirely colocalized can overpredict
  the amount of colocalization.
edam_operation: http://edamontology.org/operation_0306
tools:
- name: Bitplane
  role: Commercial implementation of the block-scrambling colocalization algorithm
- name: MIPAV
  role: Software distribution through which the colocalization significance algorithm
    was made available to the NIH research community
- name: Zeiss LSM 510 confocal microscope
  role: Acquires the two-channel (CFP 458 nm / YFP 514 nm Argon laser excitation)
    images whose intensities feed the null-distribution test
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1529/biophysj.103.038422
    title: Automatic and Quantitative Measurement of Protein-Protein Colocalization
      in Live Cells
  source_content_hash: sha256:5ac14291bf4201f1aefbf8a963e1f70828f726aa7489f6671dfed18dd492d6dc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bitplane
  - MIPAV
  - Zeiss LSM 510 confocal microscope
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# monte-carlo-null-distribution-sampling

## Summary

Monte Carlo block-scrambling significance test for two-color colocalization: after background correction, blocks of image (sized by the texture/PSF autocorrelation width rather than single pixels) are randomized 200 times per region to build a null distribution of random overlap from which the colocalization significance threshold is derived.

## When to use

Use when testing whether the Pearson correlation between two fluorescence channels (e.g., CFP/YFP or red/green from a Zeiss LSM 510) exceeds random overlap, and adjacent pixel intensities are spatially correlated by texture or the PSF so pixel-level scrambling would be invalid.

## When NOT to use

- Input is a single-channel image or lacks a paired second-channel image, so no two-color overlap null distribution can be constructed.
- The goal is measuring physical interaction distance between proteins (e.g., FRET, which detects interaction only if average binding distance is <10 nm) rather than colocalization significance.
- The question is colocalization extent from the 2D histogram's diagonal-selection approach, where assuming selected pixels are entirely colocalized can overpredict the amount of colocalization.

## Inputs

- Background-corrected two-channel (red/green, e.g., CFP/YFP) confocal image intensity arrays from a Zeiss LSM 510 confocal microscope
- Per-channel autocorrelation widths (texture or PSF size) used to set the scrambling block size

## Outputs

- Null distribution of random-overlap correlation values (200 randomizations per region)
- Significance threshold for colocalization derived from the null distribution, recorded in colocalization_threshold.csv
- Observed Pearson correlation coefficient and its significance verdict (specific colocalization vs. random overlap)

## How to apply

Load the two-channel image intensity arrays and background-correct them, then compute the autocorrelation width for each channel and set the block size to the smaller of the two widths so blocks approximate the size of the texture or the point spread function. Divide the image region into independent blocks of this size and compute the Pearson correlation coefficient between the two original channels. Instead of scrambling individual pixels, scramble block positions 200 times per region, recomputing the overlap correlation after each randomization to build the null distribution of random overlap. Derive the significance threshold from this null distribution such that an observed correlation in excess of this random-overlap level indicates specific colocalization, and record the threshold and null-distribution statistics in colocalization_threshold.csv. The rationale is that preserving within-block spatial correlation avoids an artificially narrow null distribution that would overcall colocalization in textured or PSF-blurred images.

## Related tools

- **Bitplane** (Commercial implementation of the block-scrambling colocalization algorithm)
- **MIPAV** (Software distribution through which the colocalization significance algorithm was made available to the NIH research community)
- **Zeiss LSM 510 confocal microscope** (Acquires the two-channel (CFP 458 nm / YFP 514 nm Argon laser excitation) images whose intensities feed the null-distribution test)

## Evaluation signals

- Exactly 200 randomizations are performed for each region analyzed, producing a null distribution of that size.
- Block size equals the smaller of the two channels' autocorrelation widths (texture or PSF size), not the raw pixel size.
- Validation on simulated images spanning 0–100% colocalization yields a small pixel misclassification rate of ~5% false positives and false negatives.
- The significance threshold and null-distribution statistics are written to colocalization_threshold.csv.
- An observed Pearson correlation is declared colocalized only if it exceeds the random-overlap threshold from the null distribution.

## Limitations

- Assuming selected points in the two-dimensional histogram are entirely colocalized can lead to overprediction of the amount of colocalization.
- The algorithm's validity was established on simulated images and fixed HeLa cells; results correlated well with FRET measurements but FRET and colocalization probe different scales of proximity (FRET needs <10 nm average distance).

## Evidence

- [full_text] Block scrambling preserves spatial correlation structure: [evidence span withheld — anchor status: not_found]
