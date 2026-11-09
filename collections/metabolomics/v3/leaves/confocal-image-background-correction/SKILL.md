---
name: confocal-image-background-correction
description: Use when raw two-color confocal images (e.g., CFP/YFP channels excited
  at 458/514 nm on a Zeiss LSM 510) must be background-corrected before automatic
  colocalization thresholding, block-scrambling randomization, or 2D-histogram proportion
  analysis.
when_to_use_negative:
- The series is already background-corrected — the task workflow starts by loading
  a background-corrected Rev-CFP/YFP-CRM1 image series, and re-correcting would alter
  the intensities feeding the automatic threshold determination.
- The requested task is the colocalization measurement itself (block scrambling, 200
  randomizations, diagonal selection in the 2D histogram); this skill is only the
  prerequisite intensity-correction step, not the measurement.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: Zeiss LSM 510 confocal microscope
  role: Instrument that acquires the raw two-color Rev-CFP/YFP-CRM1 confocal images
    that require background correction before colocalization analysis
- name: Carl Zeiss Axiovert 200
  role: Microscope platform housing the LSM 510 (40x 1.3 NA plan-NEOFLUAR oil DIC
    objective) during live-cell image acquisition
- name: MIPAV
  role: NIH software distribution in which the automated colocalization algorithm
    consuming background-corrected images was made available
- name: Bitplane
  role: Commercial implementation of the automated colocalization algorithm applied
    to background-corrected images
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1529/biophysj.103.038422
    title: Automatic and Quantitative Measurement of Protein-Protein Colocalization
      in Live Cells
  source_content_hash: sha256:03928c049b69a6cbfec0fde03c45c5ce3b2d3e0b3881309938c1357a82f3c99e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bitplane
  - Carl Zeiss Axiovert 200
  - Effectene
  - MIPAV
  - Zeiss LSM 510 confocal microscope
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# confocal-image-background-correction

## Summary

Preprocessing skill that background-corrects raw two-color confocal fluorescence images (e.g., Rev-CFP/YFP-CRM1 HeLa time series from a Zeiss LSM 510, CFP/YFP excited with Argon laser lines at 458 and 514 nm) before automatic colocalization measurement, so that 2D-histogram thresholds and nucleolar proportion estimates reflect true signal rather than instrument background.

## When to use

Use when raw two-color confocal images (e.g., CFP/YFP channels excited at 458/514 nm on a Zeiss LSM 510) must be background-corrected before automatic colocalization thresholding, block-scrambling randomization, or 2D-histogram proportion analysis.

## When NOT to use

- The series is already background-corrected — the task workflow starts by loading a background-corrected Rev-CFP/YFP-CRM1 image series, and re-correcting would alter the intensities feeding the automatic threshold determination.
- The requested task is the colocalization measurement itself (block scrambling, 200 randomizations, diagonal selection in the 2D histogram); this skill is only the prerequisite intensity-correction step, not the measurement.

## Inputs

- Raw (uncorrected) two-color confocal fluorescence image series, e.g., Rev-CFP/YFP-CRM1 HeLa cell time-lapse from a Zeiss LSM 510 (CFP excited at 458 nm, YFP at 514 nm)
- Optionally, simulated two-color image pairs with known colocalization used to benchmark downstream threshold accuracy (~5% misclassification tolerance)

## Outputs

- Background-corrected two-channel image series ready for automatic colocalization threshold determination
- Per-timepoint corrected frames suitable for computing nucleolar proportions of Rev and CRM1 across the LMB time course

## How to apply

Background-correct each channel of the two-color confocal series (e.g., Rev-CFP green and YFP-CRM1 red acquired on a Zeiss LSM 510 with Argon laser lines at 458 and 514 nm, cells held at 37°C and 5% CO2) before any colocalization computation, applying the correction consistently across every frame of the time-lapse. The rationale is that the downstream automatic threshold algorithm operates on absolute pixel intensities: it visualizes colocalization in a two-dimensional histogram of red and green pixel intensities, scrambles image blocks approximately the size of the texture or the PSF (200 randomizations per region) to build the distribution of random overlap, and classifies colocalized pixels by selecting those closest to the diagonal line — uncorrected background offsets would shift pixels relative to the diagonal and bias both the automatic threshold and the computed nucleolar proportions. After correction, the proportion of Rev in the nucleolus and the proportion of CRM1 in the nucleolus can be computed per time point (e.g., before and after LMB addition). Judge correctness by benchmarking on simulated data, where the algorithm misclassifies only ~5% of pixels (false positives plus false negatives), and by confirming the colocalization results correlate well with independent FRET measurements on fixed HeLa cells.

## Related tools

- **Zeiss LSM 510 confocal microscope** (Instrument that acquires the raw two-color Rev-CFP/YFP-CRM1 confocal images that require background correction before colocalization analysis)
- **Carl Zeiss Axiovert 200** (Microscope platform housing the LSM 510 (40x 1.3 NA plan-NEOFLUAR oil DIC objective) during live-cell image acquisition)
- **MIPAV** (NIH software distribution in which the automated colocalization algorithm consuming background-corrected images was made available)
- **Bitplane** (Commercial implementation of the automated colocalization algorithm applied to background-corrected images)

## Evaluation signals

- After correction, low-intensity background pixels sit near the origin of the red/green two-dimensional histogram so that diagonal-nearest-neighbor selection is driven by true signal, not offset.
- On simulated data with known ground truth, the downstream classification misclassifies only ~5% of pixels (false positives and false negatives).
- Colocalization results on corrected fixed HeLa cell series correlate well with independent FRET measurements of the same groups.
- Across the LMB time course, corrected data reproduce the directional result: the proportion of Rev in the nucleolus increases while the proportion of CRM1 decreases and diffuses into the whole nucleus.
- Correction is applied uniformly to all frames before block scrambling (blocks approximately texture/PSF size, 200 randomizations per region), keeping automatic thresholds comparable across time points.

## Limitations

- The article states the images were background-corrected but the available text does not specify the correction algorithm or its parameters, so the exact implementation must be established locally.
- Intensity-based colocalization does not establish physical proximity: FRET was negative for the Rev-CRM1 complex, presumably due to the large size of CRM1 (130 kDa).
- Even after correction, assuming that the selected points in the two-dimensional histogram are entirely colocalized can lead to overprediction of the amount of colocalization.

## Evidence

- [full_text] Downstream randomization operates on blocks sized to the texture or PSF: [evidence span withheld — anchor status: not_found]
