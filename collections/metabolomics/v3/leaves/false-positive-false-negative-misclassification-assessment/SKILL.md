---
name: false-positive-false-negative-misclassification-assessment
description: Use when validating a colocalization pixel-classification method against
  ground-truth simulated two-color images spanning full to no colocalization, to estimate
  misclassification (target ~5%) and subvisual detection ability.
when_to_use_negative:
- Input is real experimental microscopy data without ground-truth pixel classification
  labels — this skill quantifies error against simulated truth, not against FRET or
  other experimental measurements
- The goal is to measure interaction distances or FRET efficiency rather than pixel
  classification accuracy
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_2269
tools:
- name: MIPAV
  role: Software in which the colocalization algorithm was made available for performing
    the randomization-based analysis on simulated images
- name: Bitplane
  role: Commercial implementation of the colocalization algorithm usable to classify
    pixels in simulated two-color images
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1529/biophysj.103.038422
    title: Automatic and Quantitative Measurement of Protein-Protein Colocalization
      in Live Cells
  source_content_hash: sha256:b8e7e27b43edf09059cc60b59ae9d6a3395671f3a0f89c62d7ff6cad43940735
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bitplane
  - MIPAV
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# false-positive-false-negative-misclassification-assessment

## Summary

Quantifies pixel-level classification error of the Costes colocalization algorithm by measuring the proportion of pixels labeled as false-positives and false-negatives across a simulated two-color image series spanning 100% to 0% colocalization, and demonstrates detection of subvisually obvious colocalization.

## When to use

Use when validating a colocalization pixel-classification method against ground-truth simulated two-color images spanning full to no colocalization, to estimate misclassification (target ~5%) and subvisual detection ability.

## When NOT to use

- Input is real experimental microscopy data without ground-truth pixel classification labels — this skill quantifies error against simulated truth, not against FRET or other experimental measurements
- The goal is to measure interaction distances or FRET efficiency rather than pixel classification accuracy

## Inputs

- Simulated two-color (red/green) image series spanning 100% to 0% colocalization with pixel-level ground truth
- Estimates of texture or PSF size (autocorrelation widths) for each image pair

## Outputs

- Proportion of pixels misclassified as false-positives and false-negatives per simulated image (target ~5%)
- Colocalization coefficients (proportion of each protein colocalized with the other) per simulated image
- Automatic detection result versus visual expectation for the subvisually obvious colocalization case

## How to apply

Generate simulated two-color image series covering the full range of colocalization from 100% of green pixels colocalized with red pixels down to no colocalization, using the randomization-based statistical approach. Divide each image into independent blocks approximately the size of the texture or PSF, measured by the smaller of the widths of the autocorrelations for the two images, and scramble blocks rather than individual pixels to account for correlations in adjacent pixel intensities. Perform 200 randomizations per analyzed region to build the distribution of random overlap, then select colocalized pixels in the two-dimensional histogram using the automatic statistical criterion, including pixels closest to the diagonal line to distinguish low-intensity colocalized pixels from background. Compute the colocalization coefficients (proportion of each protein colocalized with the other) for each simulated image pair, then compare classified pixels against ground truth to count false-positives and false-negatives. Validate by confirming the misclassified proportion is small (~5%) and that colocalization is automatically detected in the subvisual simulated case.

## Related tools

- **MIPAV** (Software in which the colocalization algorithm was made available for performing the randomization-based analysis on simulated images)
- **Bitplane** (Commercial implementation of the colocalization algorithm usable to classify pixels in simulated two-color images)

## Evaluation signals

- Misclassification rate across the full simulation series is small (~5% false-positives and false-negatives combined)
- Colocalization coefficients correctly span from full (100% green colocalized with red) to zero colocalization across the simulated series
- The automatic criterion detects colocalization in the subvisual simulated image, consistent with but not reliant on visual expectation
- The distribution of random overlap is built from exactly 200 randomizations per region, using block scrambling at the PSF/texture block size
- Low-intensity colocalized pixels are separated from background via selection of pixels closest to the diagonal line in the two-dimensional histogram

## Limitations

- Assuming selected points in the two-dimensional histogram are entirely colocalized can lead to overprediction of the amount of colocalization
- Validation error rates are established only on simulated images; experimental correlation was assessed separately against FRET measurements on fixed HeLa cells

## Evidence

- [methods] Block scrambling rationale: [evidence span withheld — anchor status: not_found]
