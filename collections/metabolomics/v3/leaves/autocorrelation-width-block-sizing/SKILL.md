---
name: autocorrelation-width-block-sizing
description: Use when a two-channel fluorescence image shows correlated pixel intensities
  from texture or the microscope PSF, so that per-pixel scrambling would produce an
  unrealistically narrow random-overlap distribution and a significance threshold
  that falsely flags colocalization.
when_to_use_negative:
- Images whose pixel intensities are spatially uncorrelated (no texture or PSF-driven
  neighborhood correlation) — per-pixel scrambling suffices and block sizing would
  add unnecessary coarse-graining
- Single-channel images or comparisons where no two-channel overlap distribution is
  being built — the block-scrambling scheme is defined for red/green colocalization
  testing
- Very small regions where the number of independent blocks is too low for a meaningful
  200-randomization null distribution
edam_operation: http://edamontology.org/operation_3436
edam_topics:
- http://edamontology.org/topic_3385
- http://edamontology.org/topic_3474
tools:
- name: Bitplane
  role: Commercial implementation of the block-scrambling colocalization significance
    algorithm
- name: MIPAV
  role: NIH software distribution that carries the algorithm for intramural research
    use
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1529/biophysj.103.038422
    title: Automatic and Quantitative Measurement of Protein-Protein Colocalization
      in Live Cells
  source_content_hash: sha256:e1d483752c553cf100cf2b17be06b0b2228c27d2b9798bd6fbfd572d5b2d7d82
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bitplane
  - MIPAV
  edam_operation: http://edamontology.org/operation_3436
  edam_topics:
  - http://edamontology.org/topic_3385
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# autocorrelation-width-block-sizing

## Summary

Sizes randomization blocks for the colocalization significance test to the texture or PSF autocorrelation width, so the null distribution of random overlap accounts for intensity correlations between adjacent pixels rather than treating pixels as independent.

## When to use

Use when a two-channel fluorescence image shows correlated pixel intensities from texture or the microscope PSF, so that per-pixel scrambling would produce an unrealistically narrow random-overlap distribution and a significance threshold that falsely flags colocalization.

## When NOT to use

- Images whose pixel intensities are spatially uncorrelated (no texture or PSF-driven neighborhood correlation) — per-pixel scrambling suffices and block sizing would add unnecessary coarse-graining
- Single-channel images or comparisons where no two-channel overlap distribution is being built — the block-scrambling scheme is defined for red/green colocalization testing
- Very small regions where the number of independent blocks is too low for a meaningful 200-randomization null distribution

## Inputs

- Background-corrected two-channel (red/green) confocal image intensity arrays from a Zeiss LSM 510
- Per-channel texture/PSF autocorrelation width measurements

## Outputs

- Block-sized null distribution of random-overlap correlations (200 randomizations per region)
- Significance threshold for colocalization (recorded in colocalization_threshold.csv per the source card)

## How to apply

Load the two-channel (red/green) intensity arrays and compute the autocorrelation width for each channel; set the block size to the smaller of the two widths so blocks approximate the size of the texture or the point spread function. Divide the image region into independent blocks of this size and, instead of scrambling individual pixels, scramble the block positions 200 times per region, recomputing the overlap correlation after each randomization to build the null distribution of random overlap. Derive the significance threshold from this broadened null distribution; an observed Pearson correlation above this random-overlap level indicates specific colocalization. Validation on simulated images spanning 0–100% colocalization showed the resulting pixel misclassification rate is small (~5% false positives and false negatives).

## Related tools

- **Bitplane** (Commercial implementation of the block-scrambling colocalization significance algorithm)
- **MIPAV** (NIH software distribution that carries the algorithm for intramural research use)

## Evaluation signals

- The number of randomizations per analyzed region equals 200 (matches the article's stated figure)
- Block size equals the smaller of the two per-channel autocorrelation widths, not the larger or a fixed pixel count
- Randomization scrambles block positions rather than individual pixels — verify the null distribution is broader than a per-pixel null
- Simulated images spanning 0–100% colocalization yield ~5% misclassified pixels (false positives and false negatives)
- Significance threshold is recorded alongside the null distribution statistics in colocalization_threshold.csv

## Limitations

- Requires accurate autocorrelation-width estimation for both channels; errors in texture/PSF sizing propagate to the null distribution
- Breadth of the null distribution depends on block size — undersized blocks mimic the per-pixel regime and can inflate false colocalization calls
- 200 randomizations give a finite sampling limit on the precision of the derived significance threshold

## Evidence

- [full_text] images are divided into independent blocks which are approximately the size of the texture or the PSF: "images are divided into independent blocks which are approximately the size of the texture or the PSF"
- [full_text] Then instead of scrambling individual pixels, the blocks are scrambled.: "Then instead of scrambling individual pixels, the blocks are scrambled."
- [full_text] We performed 200 randomizations for each region being analyzed: "We performed 200 randomizations for each region being analyzed"
- [full_text] Our algorithm resulted in a small proportion (∼5%) of pixels being misclassified as false-positives and false-negatives: "Our algorithm resulted in a small proportion (∼5%) of pixels being misclassified as false-positives and false-negatives"
