---
name: confocal-image-colocalization-analysis
description: Use when analyzing two-channel (e.g., CFP/YFP) 3D confocal stacks of
  cells to decide whether two proteins truly colocalize beyond random overlap and
  to quantify it objectively without a manually chosen intensity threshold.
when_to_use_negative:
- When a direct, distance-resolved measurement of protein-protein interaction is required
  — FRET detects interaction only if the average distance between binding proteins
  is <10 nm, so FRET, not colocalization, is the right tool for that question
- When the user needs interaction status of a single protein pair in one cell rather
  than population-level colocalization statistics across groups of cells
- When images lack a measurable PSF/texture scale — the block size for scrambling
  must be approximately the PSF size, which the method requires
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_0102
tools:
- name: Bitplane
  role: Commercial implementation of the automatic colocalization thresholding and
    quantification algorithm
- name: MIPAV
  role: NIH software incorporating the algorithm so the colocalization analysis can
    be run on CFP/YFP confocal stacks
- name: Zeiss LSM 510 confocal microscope
  role: Acquisition of the two-channel 3D confocal image stacks that are the method's
    input
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1529/biophysj.103.038422
    title: Automatic and Quantitative Measurement of Protein-Protein Colocalization
      in Live Cells
  source_content_hash: sha256:2a14ec99131ade52c34194132c6904f237f0617ce9aa72bde5a26695d1fb251f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bitplane
  - MIPAV
  - Zeiss LSM 510 confocal microscope
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# confocal-image-colocalization-analysis

## Summary

An automatic, quantitative colocalization method for two-channel 3D confocal images that sets the colocalization threshold via block-scrambling randomization instead of user-defined cutoffs, then quantifies overlap with Pearson and proportion-of-protein-colocalized coefficients validated against FRET.

## When to use

Use when analyzing two-channel (e.g., CFP/YFP) 3D confocal stacks of cells to decide whether two proteins truly colocalize beyond random overlap and to quantify it objectively without a manually chosen intensity threshold.

## When NOT to use

- When a direct, distance-resolved measurement of protein-protein interaction is required — FRET detects interaction only if the average distance between binding proteins is <10 nm, so FRET, not colocalization, is the right tool for that question
- When the user needs interaction status of a single protein pair in one cell rather than population-level colocalization statistics across groups of cells
- When images lack a measurable PSF/texture scale — the block size for scrambling must be approximately the PSF size, which the method requires

## Inputs

- Two-channel 3D confocal CFP/YFP image stacks (e.g., Zeiss LSM 510 acquisitions) of fixed or live cotransfected cells
- Measured texture/PSF size (smaller autocorrelation width) to set the block size for scrambling
- Optional: independent FRET measurements of the same sample groups for validation

## Outputs

- Automatically determined colocalization threshold per region (from the 200-randomization random-overlap distribution)
- Per-cell Pearson correlation coefficient and proportion-of-protein-colocalized coefficient
- Pixel classification (colocalized vs. not) via the two-dimensional histogram diagonal selection
- Group-level colocalization summary that can be compared to FRET measurements

## How to apply

Background-correct the two-channel image stacks (e.g., CFP/YFP acquired on a Zeiss LSM 510 with Argon laser lines at 458 and 514 nm), then divide each image into independent blocks approximately the size of the texture or PSF — as measured by the smaller autocorrelation width — because adjacent pixel intensities are correlated, and scramble these blocks rather than individual pixels. Perform 200 randomizations per region to build the random-overlap distribution and automatically select the colocalization threshold by the statistical criterion. Classify pixels using the two-dimensional histogram (number of pixels with red and green intensities), selecting pixels above the threshold that are closest to the diagonal line, and compute per-cell Pearson correlation and proportion-of-protein-colocalized coefficients. Aggregate group-level results across cells (e.g., 40 fixed HeLa cells per cotransfection group: Rev-CFP + free YFP, Rev-CFP + Rev-YFP, Rev-CFP + Tat-YFP) and compare the ordering/magnitudes against independent FRET measurements of the same groups.

## Related tools

- **Bitplane** (Commercial implementation of the automatic colocalization thresholding and quantification algorithm)
- **MIPAV** (NIH software incorporating the algorithm so the colocalization analysis can be run on CFP/YFP confocal stacks)
- **Zeiss LSM 510 confocal microscope** (Acquisition of the two-channel 3D confocal image stacks that are the method's input)

## Evaluation signals

- Negative control (e.g., Rev-CFP + free YFP) coefficients must fall within the random-overlap distribution established by the 200 block-scrambling randomizations
- Group-level colocalization ordering and magnitudes must correlate with independent FRET measurements of the same cell groups
- Pixel misclassification rate should be near the ~5% (false positives and false negatives) observed on simulated data
- The method should detect and quantify colocalization in images where colocalization was not obvious visually
- Beware overprediction: treating all selected histogram points as entirely colocalized can overpredict the amount of colocalization

## Limitations

- FRET alone would only detect interaction if the average distance between the binding proteins is <10 nm, so FRET-based validation misses complexes like Rev-CRM1 where the large CRM1 size (130 kDa) yields negative FRET despite true proximity
- Assuming all selected points in the two-dimensional histogram are entirely colocalized can overpredict the amount of colocalization
- A small proportion (~5%) of pixels are misclassified as false-positives and false-negatives even on simulated data
- Scrambling must operate on blocks approximately the size of the texture or PSF, not individual pixels, because adjacent pixel intensities are correlated

## Evidence

- [methods] block scrambling rationale: [evidence span withheld — anchor status: not_found]
