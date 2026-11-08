---
name: two-color-histogram-colocalization-pixel-classification
description: Use when you have a background-corrected two-color (e.g., red/green)
  microscopy image pair and need an automatic, quantitative per-pixel colocalization
  decision plus M1/M2 coefficients.
when_to_use_negative:
- When the question is physical protein-protein binding at <10 nm distances — the
  paper notes FRET would only detect interaction if the average distance between the
  binding proteins is <10, so FRET rather than colocalization analysis is appropriate
- When assuming selected points in the two-dimensional histogram are entirely colocalized
  is sufficient — the paper warns this can overpredict the amount of colocalization
- When images are not background-corrected confocal two-color images; the block-scrambling
  threshold requires independent blocks approximately the size of the texture or PSF
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: MIPAV
  role: Software package in which the colocalization algorithm was made available
    to the intramural NIH research community; used to run the histogram-based pixel
    classification and M1/M2 computation
- name: Bitplane
  role: Commercial implementation of the algorithm (Bitplane AG, Zurich, Switzerland)
- name: Zeiss LSM 510 confocal microscope
  role: Acquires the two-color images that feed the two-dimensional histogram analysis
- name: Carl Zeiss Axiovert 200
  role: Microscope stand (with 40x 1.3 NA plan-NEOFLUAR oil DIC objective) used with
    the LSM 510 for image acquisition
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1529/biophysj.103.038422
    title: Automatic and Quantitative Measurement of Protein-Protein Colocalization
      in Live Cells
  source_content_hash: sha256:ac255066319fbffe3965b0180722937a10f40c8ed84e08eb0a2b587f9d147965
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bitplane
  - Carl Zeiss Axiovert 200
  - MIPAV
  - Zeiss LSM 510 confocal microscope
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# two-color-histogram-colocalization-pixel-classification

## Summary

Classifies pixels in a two-color image as colocalized or non-colocalized by analyzing the two-dimensional histogram of red-green intensities, then computes M1/M2 colocalization coefficients (proportion of each protein colocalized with the other) from the classified pixel intensities.

## When to use

Use when you have a background-corrected two-color (e.g., red/green) microscopy image pair and need an automatic, quantitative per-pixel colocalization decision plus M1/M2 coefficients.

## When NOT to use

- When the question is physical protein-protein binding at <10 nm distances — the paper notes FRET would only detect interaction if the average distance between the binding proteins is <10, so FRET rather than colocalization analysis is appropriate
- When assuming selected points in the two-dimensional histogram are entirely colocalized is sufficient — the paper warns this can overpredict the amount of colocalization
- When images are not background-corrected confocal two-color images; the block-scrambling threshold requires independent blocks approximately the size of the texture or PSF

## Inputs

- Background-corrected two-color image pair (channel 1 and channel 2 red/green intensity arrays, e.g., from a Zeiss LSM 510 confocal microscope)
- PSF/texture size estimated from the smaller of the widths of the autocorrelations of the two images

## Outputs

- Per-pixel colocalized/non-colocalized classification map
- M1/M2 colocalization coefficients (proportion of each protein colocalized with the other), written to colocalization_results.csv

## How to apply

Load the background-corrected two-channel image pair (channels 1 and 2 as red and green intensity arrays) and build the two-dimensional histogram of the number of pixels with each red-green intensity pair. Determine the colocalization threshold statistically: divide the image into independent blocks approximately the size of the texture or PSF (measured by the smaller of the widths of the autocorrelations of the two images), scramble the blocks rather than individual pixels across 200 randomizations to build the distribution of random overlap, and select the significance cutoff (e.g., the 95th percentile). Classify pixels as colocalized or non-colocalized based on the selected region of the histogram, additionally selecting pixels closest to the diagonal line in the two-dimensional histogram, which helps distinguish low-intensity colocalized signal from background. From the classified pixel intensities compute M1 and M2 — the proportion of each protein colocalized with the other — and write per-pixel classification and coefficient values to an output table. Do not assume the selected histogram points are entirely colocalized, as the paper warns this overpredicts colocalization.

## Related tools

- **MIPAV** (Software package in which the colocalization algorithm was made available to the intramural NIH research community; used to run the histogram-based pixel classification and M1/M2 computation)
- **Bitplane** (Commercial implementation of the algorithm (Bitplane AG, Zurich, Switzerland))
- **Zeiss LSM 510 confocal microscope** (Acquires the two-color images that feed the two-dimensional histogram analysis)
- **Carl Zeiss Axiovert 200** (Microscope stand (with 40x 1.3 NA plan-NEOFLUAR oil DIC objective) used with the LSM 510 for image acquisition)

## Examples

```
python colocalization_m1m2.py --ch1 red.npy --ch2 green.npy --blocks-from-autocorrelation --randomizations 200 --cutoff 95 --out colocalization_results.csv
```

## Evaluation signals

- On simulated data with known overlap, the algorithm should misclassify ~5% of pixels as false-positives and false-negatives, matching the reported performance
- The significance threshold should fall near the 95th percentile of the random-overlap distribution from the 200 block-scrambled randomizations
- M1 and M2 coefficients should lie in [0,1] and reflect the proportion of each protein colocalized with the other, consistent with the classified pixel intensities
- Validation on images of known protein-protein interactions, co-compartmentalization, or random overlap (e.g., fixed HeLa cells) should show results correlating well with FRET measurements of the same groups
- The method should detect and quantify colocalization even in images where colocalization was not obvious visually

## Limitations

- The algorithm misclassified ~5% of pixels as false-positives and false-negatives on simulated data
- Assuming selected points in the two-dimensional histogram are entirely colocalized can overpredict the amount of colocalization
- Correlation with FRET does not prove direct binding at <10 nm distances; FRET was negative for Rev-CRM1 presumably due to the large size of CRM1 (130 kDa), so colocalization and FRET measure different scales of association

## Evidence
