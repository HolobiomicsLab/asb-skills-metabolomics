---
name: fluorescence-colocalization-quantification
description: Use when analyzing background-corrected two-color live-cell confocal
  images (e.g. CFP/YFP pairs) to quantify what fraction of a protein resides in a
  region of interest, or to measure interaction/dissociation kinetics such as Rev-CRM1
  after LMB addition.
when_to_use_negative:
- 'When FRET is required to confirm direct molecular contact: FRET was negative for
  the Rev-CRM1 complex presumably due to CRM1''s large size (130 kDa), and FRET alone
  would only detect interaction if the average distance between binding proteins is
  <10 nm — colocalization quantification does not substitute for FRET in that regime.'
- When images are too noisy or the texture/PSF scale cannot be estimated, since block
  scrambling depends on dividing the image into blocks approximately the size of the
  texture or PSF.
- When only a single channel is available, since the method fundamentally relies on
  the two-dimensional histogram of paired red/green intensities.
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_3385
tools:
- name: Zeiss LSM 510 confocal microscope
  role: Acquires the two-color live-cell confocal image series (CFP/YFP excited at
    458 and 514 nm) that is the input to the colocalization algorithm.
- name: MIPAV
  role: Software in which the colocalization algorithm was made available for automatic
    and quantitative measurement of colocalization.
- name: Bitplane
  role: Commercial implementation of the same colocalization algorithm.
- name: Effectene
  role: Transfection reagent used to cotransfect HeLa cells with pRev-CFP and pYFP-CRM1
    prior to live-cell imaging.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1529/biophysj.103.038422
    title: Automatic and Quantitative Measurement of Protein-Protein Colocalization
      in Live Cells
  source_content_hash: sha256:657b7d6eda7b5c16d4f3c20d9494a9e54844aabec78a819dbc761f1e25c79383
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bitplane
  - Effectene
  - MIPAV
  - Zeiss LSM 510 confocal microscope
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_3385
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# fluorescence-colocalization-quantification

## Summary

An automatic, threshold-free method to quantify protein-protein colocalization in two-color fluorescence images by block-scrambling pixels to build a random-overlap distribution, classifying pixels near the diagonal of the two-dimensional intensity histogram, and computing per-region colocalization proportions.

## When to use

Use when analyzing background-corrected two-color live-cell confocal images (e.g. CFP/YFP pairs) to quantify what fraction of a protein resides in a region of interest, or to measure interaction/dissociation kinetics such as Rev-CRM1 after LMB addition.

## When NOT to use

- When FRET is required to confirm direct molecular contact: FRET was negative for the Rev-CRM1 complex presumably due to CRM1's large size (130 kDa), and FRET alone would only detect interaction if the average distance between binding proteins is <10 nm — colocalization quantification does not substitute for FRET in that regime.
- When images are too noisy or the texture/PSF scale cannot be estimated, since block scrambling depends on dividing the image into blocks approximately the size of the texture or PSF.
- When only a single channel is available, since the method fundamentally relies on the two-dimensional histogram of paired red/green intensities.

## Inputs

- Background-corrected two-color (e.g. green/red or CFP/YFP) confocal image series from a Zeiss LSM 510
- Region-of-interest annotations (e.g. nucleolus) per time point
- Time-lapse series spanning pre- and post-perturbation (e.g. LMB addition) acquisition

## Outputs

- Automatic colocalization threshold per region derived from the 200-randomization distribution
- Proportion of each fluorophore (e.g. Rev and CRM1) colocalized within each region of interest per time point
- Time-course of colocalization proportions and a fitted dissociation rate of the protein-protein interaction

## How to apply

Acquire two-color confocal images (e.g. Zeiss LSM 510 with CFP and YFP excited by Argon laser lines at 458 and 514 nm) and background-correct them. Divide each image into independent blocks approximately the size of the texture or PSF and scramble blocks (not individual pixels), performing 200 randomizations per region to build the distribution of random overlap used for automatic threshold determination. Visualize the red/green intensity pairs in a two-dimensional histogram and select colocalized pixels as those closest to the diagonal line, avoiding overprediction that results from assuming all histogram-selected points are truly colocalized. Compute the proportion of each fluorophore in the region of interest (e.g. the nucleolus) per time point; for dissociation kinetics, track the two proportions across the time series before and after perturbation (e.g. leptomycin B addition) and fit the decay of colocalization to obtain the dissociation rate.

## Related tools

- **Zeiss LSM 510 confocal microscope** (Acquires the two-color live-cell confocal image series (CFP/YFP excited at 458 and 514 nm) that is the input to the colocalization algorithm.)
- **MIPAV** (Software in which the colocalization algorithm was made available for automatic and quantitative measurement of colocalization.)
- **Bitplane** (Commercial implementation of the same colocalization algorithm.)
- **Effectene** (Transfection reagent used to cotransfect HeLa cells with pRev-CFP and pYFP-CRM1 prior to live-cell imaging.)

## Evaluation signals

- On simulated data the algorithm should misclassify only a small proportion (~5%) of pixels as false-positives and false-negatives.
- Results should correlate well with FRET measurements on the same groups of known protein-protein interactions, co-compartmentalization, or random overlap.
- After LMB addition, the expected directional result is that the proportion of Rev in the nucleolus increases while the proportion of CRM1 decreases in the nucleolus and diffuses into the whole nucleus.
- The fitted dissociation rate should be consistent with the observed time-course of the two nucleolar proportions.
- The algorithm should detect and quantify colocalization in images where colocalization was not obvious visually.

## Limitations

- Assuming that selected points in the two-dimensional histogram are entirely colocalized can lead to overprediction of the amount of colocalization.
- FRET (the cross-validation method) was negative for the Rev-CRM1 complex, presumably due to the large size of CRM1 (130 kDa), limiting FRET-based validation of large complexes.
- A small proportion (~5%) of pixels are still misclassified as false-positives and false-negatives on simulated data.

## Evidence
