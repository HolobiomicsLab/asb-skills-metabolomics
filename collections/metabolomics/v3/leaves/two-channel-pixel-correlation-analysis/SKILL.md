---
name: two-channel-pixel-correlation-analysis
description: Use when you have background-corrected two-channel (CFP/YFP or red/green)
  confocal microscopy images from a Zeiss LSM 510 or similar and need to determine
  whether, and in what proportion of pixels, the two fluorescent signals colocalize
  beyond random overlap.
when_to_use_negative:
- When FRET alone is expected to detect proximity, since FRET only detects interaction
  if the average distance between binding proteins is <10 nm and can be negative for
  large complexes (e.g., CRM1 at 130 kDa).
- When images are not background-corrected or when individual pixels are scrambled
  instead of blocks sized to the autocorrelation width, since correlated adjacent
  pixels would inflate the apparent overlap and produce false positives.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: Bitplane
  role: Commercial implementation of the colocalization algorithm for two-channel
    image analysis
- name: MIPAV
  role: NIH intramural software distribution of the colocalization algorithm used
    to compute thresholds and overlap statistics
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1529/biophysj.103.038422
    title: Automatic and Quantitative Measurement of Protein-Protein Colocalization
      in Live Cells
  source_content_hash: sha256:63baa91f8ae137541c2d0a6efee025427432aefe8ee1a4f1131b67c044169a25
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bitplane
  - MIPAV
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# two-channel-pixel-correlation-analysis

## Summary

Quantifies protein-protein colocalization in two-channel (red/green) fluorescence images by computing a Pearson correlation coefficient between pixel intensities and comparing it to a significance threshold derived from a random-overlap null distribution built by scrambling image blocks sized to the autocorrelation width, implemented in MIPAV/Bitplane.

## When to use

Use when you have background-corrected two-channel (CFP/YFP or red/green) confocal microscopy images from a Zeiss LSM 510 or similar and need to determine whether, and in what proportion of pixels, the two fluorescent signals colocalize beyond random overlap.

## When NOT to use

- When FRET alone is expected to detect proximity, since FRET only detects interaction if the average distance between binding proteins is <10 nm and can be negative for large complexes (e.g., CRM1 at 130 kDa).
- When images are not background-corrected or when individual pixels are scrambled instead of blocks sized to the autocorrelation width, since correlated adjacent pixels would inflate the apparent overlap and produce false positives.

## Inputs

- Background-corrected two-channel (e.g., CFP and YFP) confocal microscopy image acquired with a Zeiss LSM 510, ideally a three-dimensional image stack
- Two-dimensional intensity histogram of red and green channel pixel values

## Outputs

- colocalization_threshold.csv containing the significance threshold and null distribution statistics
- Proportion of pixels classified as colocalized per region
- Two-dimensional histogram visualization of pixel overlap between channels

## How to apply

Load the two-channel intensity arrays and perform background correction, then compute the autocorrelation width for each channel to determine the block size (the smaller of the two widths), which approximates the texture or point spread function (PSF). Divide the image region into independent blocks of this size rather than treating individual pixels, because adjacent pixels are correlated by the PSF and would otherwise be falsely counted as independent observations. Compute the Pearson correlation coefficient between the original two channels, then scramble the block positions 200 times to build a null distribution of random overlap from which the significance threshold is derived; only correlations exceeding this threshold indicate specific colocalization. Select pixels closest to the diagonal line in the two-dimensional intensity histogram as fully colocalized, and record the threshold and null distribution statistics in colocalization_threshold.csv.

## Related tools

- **Bitplane** (Commercial implementation of the colocalization algorithm for two-channel image analysis)
- **MIPAV** (NIH intramural software distribution of the colocalization algorithm used to compute thresholds and overlap statistics)

## Examples

```
python mipav_colocalization.py --image stack.tif --channel1 CFP --channel2 YFP --blocks 200 --out colocalization_threshold.csv
```

## Evaluation signals

- Applied to simulated images covering 0–100% colocalization, the pixel misclassification rate should be small (~5% false positives and false negatives)
- The derived significance threshold must come from the null distribution of the 200 block-scrambled randomizations, not from an arbitrary correlation cutoff
- Block size must equal the smaller of the two channels' autocorrelation widths, not a fixed pixel count
- Results should correlate well with FRET measurements of the same protein-protein interaction groups in fixed HeLa cells

## Limitations

- Assuming selected points in the two-dimensional histogram are entirely colocalized can lead to overprediction of the amount of colocalization
- Algorithm performance was validated on simulated images and fixed HeLa cells; results may differ for live-cell imaging with different noise properties
- Requires knowledge of the PSF or texture autocorrelation width to set block size; misestimating this can break the block independence assumption

## Evidence
