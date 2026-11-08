---
name: classification-bias-quantification
description: Decision support for whether reported colocalization coefficients overestimate
  true colocalization when pixel intensities are sums of colocalized and non-colocalized
  signals.
when_to_use_negative:
- When true colocalization fractions are unknown and cannot be simulated — overprediction
  cannot be tabulated without mixed-signal ground truth
- For FRET-based interaction detection, where only average distances <10 nm are detected
  and large proteins like CRM1 (130 kDa) give negative FRET
edam_operation: http://edamontology.org/operation_3741
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_3315
tools:
- name: MIPAV
  role: Implementation platform hosting the Costes colocalization algorithm for threshold
    selection and coefficient computation
- name: Bitplane
  role: Commercial implementation of the algorithm used for automatic threshold and
    colocalization classification
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1529/biophysj.103.038422
    title: Automatic and Quantitative Measurement of Protein-Protein Colocalization
      in Live Cells
  source_content_hash: sha256:8b8fe667323bdbb7d88e434f5aac298cd186993666beda85b64b759a2d9c2c73
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bitplane
  - MIPAV
  edam_operation: http://edamontology.org/operation_3741
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# classification-bias-quantification

## Summary

Quantifies the overprediction bias incurred when every pixel in a two-color image is forced into a binary colocalized/non-colocalized decision, by comparing per-pixel Costes coefficients against known mixed-signal ground truth.

## When to use

Decision support for whether reported colocalization coefficients overestimate true colocalization when pixel intensities are sums of colocalized and non-colocalized signals.

## When NOT to use

- When true colocalization fractions are unknown and cannot be simulated — overprediction cannot be tabulated without mixed-signal ground truth
- For FRET-based interaction detection, where only average distances <10 nm are detected and large proteins like CRM1 (130 kDa) give negative FRET

## Inputs

- Synthetic two-channel (red/green) image pairs covering the full colocalization range from 100% colocalized to none
- Mixed-signal variants where pixel intensities are sums of colocalized and non-colocalized components with known fractions
- Background-corrected confocal images from a Zeiss LSM 510 (CFP/YFP or equivalent red/green channels)

## Outputs

- overprediction_analysis.csv — computed colocalization coefficients vs. ground-truth mixed fractions per condition
- Per-condition binary colocalized/non-colocalized pixel masks from the thresholded two-dimensional red/green intensity histogram
- Misclassification rate (false positives + false negatives) for validation against the ~5% benchmark

## How to apply

Generate synthetic red/green two-channel image pairs spanning the full colocalization range (100% of green pixels colocalized with red pixels down to zero), including mixed-signal variants where each pixel's intensity is the sum of a colocalized and a non-colocalized component with known fractions. Apply the Costes automatic threshold — divide each image into blocks approximately the size of the texture or PSF (based on the narrower autocorrelation width), scramble the blocks rather than individual pixels, perform 200 randomizations per region, and select the threshold from the resulting random-overlap distribution. Classify pixels using the two-dimensional red/green intensity histogram, selecting the thresholded region and pixels closest to the diagonal to separate low-intensity colocalized pixels from background. Compute the colocalization coefficients (proportion of each protein colocalized with the other) and tabulate overprediction (computed coefficient minus true mixed fraction) across the mixing series; on the pure-colocalization simulations the algorithm misclassified ~5% of pixels as false positives and false negatives.

## Related tools

- **MIPAV** (Implementation platform hosting the Costes colocalization algorithm for threshold selection and coefficient computation)
- **Bitplane** (Commercial implementation of the algorithm used for automatic threshold and colocalization classification)

## Examples

```
python run_colocalization.py --images synthetic_pairs/ --blocks PSF_sized --randomizations 200 --out overprediction_analysis.csv
```

## Evaluation signals

- Misclassified pixel proportion on pure-colocalization simulations falls at ~5% (false positives + false negatives)
- Overprediction (computed coefficient − true colocalized fraction) is non-negative across the mixed-signal mixing series
- Colocalization coefficients reproduce the ground-truth range from 100% of green pixels colocalized down to zero
- Two-dimensional histogram classification places pixels closest to the diagonal within the thresholded colocalized region, separating them from background

## Limitations

- Assuming the selected points in the two-dimensional histogram are entirely colocalized can lead to overprediction of the amount of colocalization
- The binary per-pixel classification ignores that real pixel intensities are sums of colocalized and non-colocalized signals
- FRET alone would only detect interaction if the average distance between binding proteins is <10 nm; the algorithm and FRET are complementary, not interchangeable

## Evidence
