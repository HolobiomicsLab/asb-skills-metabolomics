---
name: fluorescent-protein-cotransfection-image-interpretation
description: Use when interpreting CFP/YFP two-channel 3D confocal stacks from cotransfected
  fixed or live HeLa cells where the goal is to classify pixels as colocalized versus
  random overlap and rank interaction groups against FRET ground truth.
when_to_use_negative:
- When the question is direct molecular interaction at <10 nm distances — FRET alone
  (not colocalization) detects this, and the article shows FRET can be negative for
  large complexes like Rev-CRM1 (130 kDa) even when colocalization is real
- When images lack a second channel or a negative control condition (e.g., free YFP)
  — the statistical threshold and random-overlap distribution cannot be built
- When individual adjacent pixels can be assumed independent — scrambling single pixels
  instead of PSF-sized blocks violates the method's correlation assumption
edam_operation: http://edamontology.org/operation_3443
tools:
- name: Zeiss LSM 510 confocal microscope
  role: Acquisition of two-channel 3D CFP/YFP image stacks from cotransfected HeLa
    cells (458 nm and 514 nm Argon excitation)
- name: Bitplane
  role: Commercial implementation of the automatic colocalization threshold/classification
    algorithm
- name: MIPAV
  role: NIH intramural software distribution carrying the colocalization algorithm
    for block-scrambled thresholding and 2D histogram classification
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1529/biophysj.103.038422
    title: Automatic and Quantitative Measurement of Protein-Protein Colocalization
      in Live Cells
  source_content_hash: sha256:7ed1cae6e1a7c03586132d1d769fb4306224eb226fe47ae5306514c72212d283
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bitplane
  - MIPAV
  - Zeiss LSM 510 confocal microscope
  edam_operation: http://edamontology.org/operation_3443
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# fluorescent-protein-cotransfection-image-interpretation

## Summary

Interpretation of two-channel CFP/YFP confocal images of cotransfected HeLa cells (e.g., Rev-CFP with free YFP, Rev-YFP, or Tat-YFP) to quantify protein-protein colocalization and cross-validate group-level results against independent FRET measurements.

## When to use

Use when interpreting CFP/YFP two-channel 3D confocal stacks from cotransfected fixed or live HeLa cells where the goal is to classify pixels as colocalized versus random overlap and rank interaction groups against FRET ground truth.

## When NOT to use

- When the question is direct molecular interaction at <10 nm distances — FRET alone (not colocalization) detects this, and the article shows FRET can be negative for large complexes like Rev-CRM1 (130 kDa) even when colocalization is real
- When images lack a second channel or a negative control condition (e.g., free YFP) — the statistical threshold and random-overlap distribution cannot be built
- When individual adjacent pixels can be assumed independent — scrambling single pixels instead of PSF-sized blocks violates the method's correlation assumption

## Inputs

- Two-channel 3D confocal CFP/YFP image stacks (Zeiss LSM 510, Argon laser lines 458 and 514 nm) from fixed HeLa cotransfection groups (40 cells per group)
- Cotransfection construct pairs: pRev-CFP with pYFP (free YFP control), pRev-YFP, or pTat-YFP, transfected with Effectene
- Measured texture/PSF size from autocorrelation width to set block size for scrambling

## Outputs

- Automatic colocalization threshold per region derived from the 200-randomization random-overlap distribution
- Per-cell Pearson correlation and proportion-of-protein-colocalized coefficients for each cotransfection group
- Group-level colocalization ordering and magnitudes comparable to independent FRET measurements
- Two-dimensional histogram pixel classification (colocalized vs. random-overlap pixels)

## How to apply

Acquire two-channel 3D CFP/YFP stacks on a Zeiss LSM 510 confocal microscope (CFP excited at 458 nm, YFP at 514 nm) from cotransfection groups such as Rev-CFP + free YFP (negative control), Rev-CFP + Rev-YFP, and Rev-CFP + Tat-YFP, maintaining cells at 37°C and 5% CO2. Apply background correction, then divide images into independent blocks approximately the size of the texture or PSF (measured by the smaller autocorrelation width) because adjacent pixel intensities are correlated, and scramble blocks rather than individual pixels. Run 200 randomizations per region to build the random-overlap distribution and automatically select the colocalization threshold by the statistical criterion. Classify pixels via the two-dimensional histogram (pixels above threshold and near the diagonal) and compute per-cell Pearson and proportion-of-protein-colocalized coefficients. Confirm the free-YFP negative control falls within the random-overlap distribution while Rev-Rev and Rev-Tat groups show significant colocalization, then compare group-level ordering and magnitudes with FRET measurements of the same groups.

## Related tools

- **Zeiss LSM 510 confocal microscope** (Acquisition of two-channel 3D CFP/YFP image stacks from cotransfected HeLa cells (458 nm and 514 nm Argon excitation))
- **Bitplane** (Commercial implementation of the automatic colocalization threshold/classification algorithm)
- **MIPAV** (NIH intramural software distribution carrying the colocalization algorithm for block-scrambled thresholding and 2D histogram classification)

## Evaluation signals

- Negative control (Rev-CFP + free YFP) coefficients must fall within the random-overlap distribution built from 200 block-scrambling randomizations
- Pixel misclassification (false positives plus false negatives) should be near the ~5% rate observed in simulations
- Group-level ordering and magnitudes must correlate with independent FRET measurements of the same three 40-cell groups
- 200 randomizations must have been performed per analyzed region (not per image)
- Selected histogram pixels must be both above threshold and near the diagonal; assuming they are entirely colocalized indicates overprediction

## Limitations

- FRET alone only detects interaction if the average distance between binding proteins is <10 nm, so FRET-negative results (e.g., Rev-CRM1 with 130 kDa CRM1) do not rule out colocalization
- Assuming selected points in the two-dimensional histogram are entirely colocalized can overpredict the amount of colocalization
- Approximately 5% of pixels are expected to be misclassified as false positives or false negatives

## Evidence
