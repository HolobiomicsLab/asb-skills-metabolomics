---
name: protein-nucleolar-compartment-proportion-measurement
description: Use when live-cell, two-color confocal time-lapse images (e.g., Rev-CFP/YFP-CRM1
  in HeLa cells) must be converted into time-resolved proportions of each protein
  within a subnuclear compartment, or when a drug perturbation (e.
when_to_use_negative:
- Single-color images or assays where the two probes cannot be excited/detected separately
  (no two-dimensional red/green histogram can be built).
- Situations where the physical proximity of the two proteins is expected to be <10
  nm and FRET is the appropriate readout — this skill measures compartment proportions,
  not binding distances, and the article notes FRET was negative for the large (130
  kDa) CRM1.
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: Zeiss LSM 510 confocal microscope
  role: Acquisition of live-cell CFP/YFP time-lapse confocal image series used as
    input to the proportion measurement
- name: MIPAV
  role: Software implementation of the automatic colocalization algorithm used to
    compute the compartment proportions
- name: Bitplane
  role: Commercial implementation of the same colocalization algorithm
- name: Effectene
  role: Transfection reagent for introducing pRev-CFP and pYFP-CRM1 into HeLa cells
    prior to imaging
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1529/biophysj.103.038422
    title: Automatic and Quantitative Measurement of Protein-Protein Colocalization
      in Live Cells
  source_content_hash: sha256:b7c561567b65954fcf346e70900594a606bca8daf1cd42d324f806d3f7f96b5e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bitplane
  - Effectene
  - MIPAV
  - Zeiss LSM 510 confocal microscope
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# protein-nucleolar-compartment-proportion-measurement

## Summary

Quantifies the proportion of a fluorescently tagged protein residing in a defined nuclear compartment (e.g., the nucleolus) over time in live-cell two-color confocal series, by automatic pixel colocalization classification in a two-dimensional intensity histogram after background correction and block-scrambled significance testing.

## When to use

Use when live-cell, two-color confocal time-lapse images (e.g., Rev-CFP/YFP-CRM1 in HeLa cells) must be converted into time-resolved proportions of each protein within a subnuclear compartment, or when a drug perturbation (e.g., leptomycin B) requires tracking such proportions to infer protein-protein dissociation rates.

## When NOT to use

- Single-color images or assays where the two probes cannot be excited/detected separately (no two-dimensional red/green histogram can be built).
- Situations where the physical proximity of the two proteins is expected to be <10 nm and FRET is the appropriate readout — this skill measures compartment proportions, not binding distances, and the article notes FRET was negative for the large (130 kDa) CRM1.

## Inputs

- Background-corrected two-color live-cell confocal time-lapse image series (e.g., Rev-CFP green / YFP-CRM1 red) from a Zeiss LSM 510 with 458/514 nm Argon excitation
- Compartment ROI definition (e.g., nucleolus) per cell and time point
- Perturbation time stamp (e.g., leptomycin B addition) to segment before/after analysis

## Outputs

- Per-time-point proportion of each tagged protein in the compartment (e.g., fraction of Rev in nucleolus, fraction of CRM1 in nucleolus)
- Colocalization classification map (colocalized vs. non-colocalized pixels) derived from the two-dimensional histogram
- Time-course of compartment proportions before/after perturbation, with fitted dissociation rate for the protein-protein interaction

## How to apply

Acquire background-corrected CFP/YFP confocal time-lapse images (e.g., Zeiss LSM 510, Argon lines at 458/514 nm, cells held at 37°C and 5% CO2), then for each channel define the compartment ROI (e.g., nucleolus) and compute per-time-point proportions of that protein inside the ROI. Classify colocalized pixels by building a two-dimensional histogram of red versus green intensities and selecting pixels closest to the diagonal line; assign significance via 200 randomizations in which images are divided into independent blocks approximately the size of the texture or PSF and blocks (not individual pixels) are scrambled, so the automatic threshold reflects the distribution of random overlap. Track the two compartment proportions across the series before and after perturbation; in the LMB example the proportion of Rev in the nucleolus increases while the proportion of CRM1 decreases and diffuses into the whole nucleus, and fitting the decay of colocalization yields the dissociation rate of the interaction. Expect the automatic classifier to misclassify roughly 5% of pixels as false positives/false negatives on simulated data, and validate proportions against independent assays (here, FRET on fixed HeLa cells) where possible.

## Related tools

- **Zeiss LSM 510 confocal microscope** (Acquisition of live-cell CFP/YFP time-lapse confocal image series used as input to the proportion measurement)
- **MIPAV** (Software implementation of the automatic colocalization algorithm used to compute the compartment proportions)
- **Bitplane** (Commercial implementation of the same colocalization algorithm)
- **Effectene** (Transfection reagent for introducing pRev-CFP and pYFP-CRM1 into HeLa cells prior to imaging)

## Evaluation signals

- On simulated data, ~5% of pixels should be misclassified (false positives + false negatives) — large deviations signal mis-tuned thresholding.
- After LMB addition, the expected directional result is an increase in the proportion of Rev in the nucleolus and a decrease in the proportion of CRM1 in the nucleolus (diffusing into the whole nucleus); a reversal indicates a labeling or ROI error.
- The fitted dissociation rate from the decay of colocalization should be consistent with the observed time course of the two proportions.
- Randomization-based significance must use block scrambling (~200 randomizations), not pixel scrambling — pixel-level scrambling overstates significance for textured/PSF-limited images.
- Proportions should be consistent with an independent orthogonal assay (e.g., FRET) on comparable samples where available.

## Limitations

- Assuming that all selected points in the two-dimensional histogram are entirely colocalized can lead to overprediction of the amount of colocalization.
- FRET alone would only detect interaction if the average distance between the binding proteins is <10 nm; this method measures compartment proportions and does not report binding distance.
- Roughly 5% of pixels are misclassified as false positives or false negatives even on simulated data, so absolute proportions carry this uncertainty.

## Evidence
