---
name: fluorescence-microscopy-image-background-correction
description: Use when two-color confocal fluorescence images (e.g., Zeiss LSM 510
  CFP/YFP acquisitions of live cells) must be background-corrected before building
  the red-green two-dimensional histogram and computing M1/M2 colocalization coefficients.
when_to_use_negative:
- The image pair is already background-corrected — the article's workflow explicitly
  loads 'the background-corrected two-color image pair', so re-correcting would double-subtract
  and deflate M1/M2 coefficients
- The goal is absolute fluorophore quantification rather than relative colocalization,
  since the correction is designed to support overlap statistics, not intensity calibration
  across samples
- The data are single-channel or the two-dimensional red-green histogram framework
  does not apply
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3383
tools:
- name: Zeiss LSM 510 confocal microscope
  role: Acquires the two-color (CFP/YFP) image pair that is subsequently background-corrected
    before colocalization analysis
- name: MIPAV
  role: NIH software distribution in which the colocalization algorithm, including
    background-corrected inputs and the two-dimensional histogram analysis, was made
    available
- name: Bitplane
  role: Commercial implementation of the same colocalization algorithm that operates
    on background-corrected confocal images
- name: Carl Zeiss Axiovert 200
  role: Microscope platform coupled to the LSM 510 scan head for live-cell image acquisition
    prior to correction
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1529/biophysj.103.038422
    title: Automatic and Quantitative Measurement of Protein-Protein Colocalization
      in Live Cells
  source_content_hash: sha256:883f7e4b3b7fb2dcda4aca76e6cf36f0f58ff511a5f7589395243b994009cc24
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bitplane
  - Carl Zeiss Axiovert 200
  - MIPAV
  - Zeiss LSM 510 confocal microscope
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3383
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# fluorescence-microscopy-image-background-correction

## Summary

Background correction removes out-of-cell and detector background signal from confocal fluorescence image pairs (e.g., Zeiss LSM 510 CFP/YFP acquisitions) so that downstream pixel-level colocalization analysis, such as M1/M2 coefficient computation, reflects true protein overlap rather than offset or bleed-through intensity.

## When to use

Use when two-color confocal fluorescence images (e.g., Zeiss LSM 510 CFP/YFP acquisitions of live cells) must be background-corrected before building the red-green two-dimensional histogram and computing M1/M2 colocalization coefficients.

## When NOT to use

- The image pair is already background-corrected — the article's workflow explicitly loads 'the background-corrected two-color image pair', so re-correcting would double-subtract and deflate M1/M2 coefficients
- The goal is absolute fluorophore quantification rather than relative colocalization, since the correction is designed to support overlap statistics, not intensity calibration across samples
- The data are single-channel or the two-dimensional red-green histogram framework does not apply

## Inputs

- Two-channel confocal fluorescence image pair (red and green intensity arrays, e.g., CFP and YFP excited at 458 and 514 nm from a Zeiss LSM 510)
- Instrument acquisition metadata (laser lines, objective) used to keep channels comparable before correction

## Outputs

- Background-corrected red and green intensity arrays ready for two-dimensional histogram construction
- Downstream per-pixel colocalization classification and M1/M2 coefficient values (e.g., written to colocalization_results.csv)

## How to apply

Acquire the two-color image pair on a Zeiss LSM 510 confocal microscope (Axiovert 200, 40x 1.3 NA plan-NEOFLUAR objective), then apply background correction to each channel before any colocalization computation, as the paper's pipeline collects confocal images and background-corrects them prior to analysis. Corrected channels 1 and 2 (red and green intensity arrays) are then loaded with NumPy/PIL to construct the two-dimensional histogram of red-green intensity pairs. The correction matters because the subsequent automated significance threshold divides the image into blocks the size of the texture or PSF and performs 200 block-scrambled randomizations; uncorrected background would inflate the random-overlap distribution and shift the 95th-percentile cutoff. After thresholding, pixels closest to the diagonal in the histogram are selected to separate low-intensity colocalized signal from background, so residual background in the corrected images directly biases M1/M2 coefficients. Implementations are available in MIPAV and were commercialized by Bitplane AG, so the correction can be reproduced with those packages rather than ad hoc scripts.

## Related tools

- **Zeiss LSM 510 confocal microscope** (Acquires the two-color (CFP/YFP) image pair that is subsequently background-corrected before colocalization analysis)
- **MIPAV** (NIH software distribution in which the colocalization algorithm, including background-corrected inputs and the two-dimensional histogram analysis, was made available)
- **Bitplane** (Commercial implementation of the same colocalization algorithm that operates on background-corrected confocal images)
- **Carl Zeiss Axiovert 200** (Microscope platform coupled to the LSM 510 scan head for live-cell image acquisition prior to correction)

## Examples

```
from PIL import Image; import numpy as np; red, green = np.array(Image.open('ch1.tif'), float), np.array(Image.open('ch2.tif'), float)
```

## Evaluation signals

- Corrected channels show no residual intensity in cell-free (background) regions of the field before the two-dimensional histogram is built
- The automated threshold derived from 200 block-scrambled randomizations (95th-percentile cutoff) selects a stable colocalized region rather than being dragged upward by residual background
- On simulated data the full pipeline, built on corrected inputs, misclassifies only ~5% of pixels as false positives and false negatives
- Resulting M1/M2 coefficients are reproducible and agree with independent FRET measurements of the same protein groups
- Colocalization is detected and quantified even in images where it was not obvious visually, indicating background was not masking low-intensity diagonal pixels

## Limitations

- Assuming that the selected points in the two-dimensional histogram are entirely colocalized can overpredict the amount of colocalization, so corrected low-intensity pixels still require diagonal-region selection
- The correction supports relative colocalization measurement (M1/M2 proportions); the article validates against FRET only for known interactions, and FRET was negative for Rev-CRM1 presumably due to CRM1's large 130 kDa size, so cross-modal comparison has bounds
- The exact background-estimation parameters are not specified in the provided context, so the correction must be reproduced via MIPAV/Bitplane or calibrated empirically

## Evidence
