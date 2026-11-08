---
name: subvisual-colocalization-detection-verification
description: Use when reproducing the Costes colocalization validation on simulated
  red/green image pairs, or when verifying that an automated colocalization method
  detects subvisually obvious overlap and quantifies false positives/negatives.
when_to_use_negative:
- Images whose pixel-intensity correlations are handled by scrambling individual pixels
  rather than PSF-sized blocks — adjacent-pixel intensity correlations will inflate
  the random-overlap distribution.
- Scenarios that assume all points selected in the two-dimensional histogram are entirely
  colocalized — the article warns this can overpredict the amount of colocalization.
- Protein-protein interaction questions requiring sub-10 nm distance resolution, where
  FRET (requiring average distance <10 nm between binding proteins) is the appropriate
  readout rather than pixel colocalization.
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: MIPAV
  role: Platform in which the automatic colocalization algorithm was made available
    to the intramural NIH research community for performing the randomization-based
    colocalization analysis
- name: Bitplane
  role: Commercial implementation of the algorithm, usable as an alternative environment
    for running the automated colocalization selection criterion
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1529/biophysj.103.038422
    title: Automatic and Quantitative Measurement of Protein-Protein Colocalization
      in Live Cells
  source_content_hash: sha256:f333e7986821373e50186c9d8ded96a4a4f87cd509705187928cf8a0d38d9848
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bitplane
  - MIPAV
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# subvisual-colocalization-detection-verification

## Summary

Validates the Costes automatic colocalization algorithm on simulated two-color image series spanning 100% to 0% colocalization, confirming ~5% pixel misclassification and the ability to detect colocalization not obvious to the eye.

## When to use

Use when reproducing the Costes colocalization validation on simulated red/green image pairs, or when verifying that an automated colocalization method detects subvisually obvious overlap and quantifies false positives/negatives.

## When NOT to use

- Images whose pixel-intensity correlations are handled by scrambling individual pixels rather than PSF-sized blocks — adjacent-pixel intensity correlations will inflate the random-overlap distribution.
- Scenarios that assume all points selected in the two-dimensional histogram are entirely colocalized — the article warns this can overpredict the amount of colocalization.
- Protein-protein interaction questions requiring sub-10 nm distance resolution, where FRET (requiring average distance <10 nm between binding proteins) is the appropriate readout rather than pixel colocalization.

## Inputs

- Simulated two-color (red/green) image series spanning 100% to 0% pixel colocalization
- Per-image PSF/texture size estimates (smaller width of the two autocorrelations) used to set block size

## Outputs

- Proportion of pixels misclassified as false-positives and false-negatives per simulated image (target ~5%)
- Colocalization coefficients (proportion of each protein colocalized with the other) per image
- Automatic detection verdict (colocalized vs. random) for the subvisual colocalization case

## How to apply

Generate simulated two-color image series covering the full colocalization range from 100% of green pixels colocalized with red down to no colocalization. Before randomizing, divide each image into independent blocks approximately the size of the texture or the point spread function (PSF), sized by the smaller of the widths of the two images' autocorrelations, and scramble blocks rather than individual pixels to account for correlations in adjacent pixel intensities. Perform 200 randomizations per analyzed region to build the distribution of random overlap, then apply the automatic statistical criterion to select colocalized pixels in the two-dimensional histogram, choosing pixels closest to the diagonal line to distinguish low-intensity colocalized pixels from background. Compute the colocalization coefficients (the proportion of each protein colocalized with the other) for each simulated image. Count the proportion of pixels misclassified as false-positives and false-negatives across the series and record automatic detection on the simulated image where colocalization was not visually obvious.

## Related tools

- **MIPAV** (Platform in which the automatic colocalization algorithm was made available to the intramural NIH research community for performing the randomization-based colocalization analysis)
- **Bitplane** (Commercial implementation of the algorithm, usable as an alternative environment for running the automated colocalization selection criterion)

## Evaluation signals

- Across the full simulated series (100% to 0% colocalization), the fraction of misclassified pixels (false positives + false negatives) is small, approximately 5%.
- Randomizations are performed 200 times per analyzed region, and scrambling is done on PSF/texture-sized blocks, not individual pixels.
- Colocalized pixels are selected in the two-dimensional histogram with the automatic statistical criterion, including pixels closest to the diagonal line (low-intensity colocalized pixels distinguished from background).
- On the simulated image where colocalization is not visually obvious, the automatic detection flags colocalization, matching the reported subvisual detection result.
- Computed colocalization coefficients report the proportion of each protein colocalized with the other, not just a single global overlap metric.

## Limitations

- Assuming the selected points in the two-dimensional histogram are entirely colocalized can overpredict the amount of colocalization.
- Background must be corrected before analysis; images were collected with a confocal microscope and background-corrected.
- The automatic detection can find colocalization that is not obvious to the eye, so visual inspection alone is insufficient to confirm or refute the algorithm's verdict.

## Evidence

- [full_text] Block-based scrambling rationale: [evidence span withheld — anchor status: not_found]
