---
name: two-dimensional-histogram-pixel-classification
description: Use when a background-corrected two-channel (e.g., red/green CFP/YFP)
  confocal image or time series needs automatic, threshold-free classification of
  which pixels are colocalized, and when subsequent proportion-colocalized coefficients
  must be computed per region or time point.
when_to_use_negative:
- The question is direct protein-protein interaction distance (<10 nm) rather than
  colocalization — FRET is the appropriate readout, as FRET alone detects interaction
  only if the average distance between binding proteins is <10 nm
- The image is single-channel or the two channels cannot be represented as a joint
  red/green intensity histogram
- Adjacent pixel intensities are being scrambled as individual pixels without block
  scrambling when texture/PSF-scale correlations exist — this violates the method's
  assumption and biases the random-overlap distribution
tools:
- name: MIPAV
  role: Software implementation of the automatic colocalization algorithm (2D histogram
    classification, randomization thresholding, coefficient computation)
- name: Bitplane
  role: Commercialized implementation of the same pixel-classification colocalization
    algorithm
- name: Zeiss LSM 510 confocal microscope
  role: Acquisition of the two-channel CFP/YFP images that serve as input to pixel
    classification
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_004
  source_papers:
  - doi: 10.1529/biophysj.103.038422
    title: Automatic and Quantitative Measurement of Protein-Protein Colocalization
      in Live Cells
  source_content_hash: sha256:d1d5b55288164350d30187340826bbb82cd153d6b76cc7e1ea9e3036c9331fc8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bitplane
  - Effectene
  - MIPAV
  - Zeiss LSM 510 confocal microscope
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# two-dimensional-histogram-pixel-classification

## Summary

Automatic statistical classification of colocalized pixels in two-color fluorescence images by building a red/green two-dimensional histogram, setting the colocalization threshold from 200 block-scrambling randomizations, and selecting pixels near the diagonal to compute proportion-colocalized coefficients. Validated at ~5% pixel misclassification and shown to detect colocalization not obvious to the eye.

## When to use

Use when a background-corrected two-channel (e.g., red/green CFP/YFP) confocal image or time series needs automatic, threshold-free classification of which pixels are colocalized, and when subsequent proportion-colocalized coefficients must be computed per region or time point.

## When NOT to use

- The question is direct protein-protein interaction distance (<10 nm) rather than colocalization — FRET is the appropriate readout, as FRET alone detects interaction only if the average distance between binding proteins is <10 nm
- The image is single-channel or the two channels cannot be represented as a joint red/green intensity histogram
- Adjacent pixel intensities are being scrambled as individual pixels without block scrambling when texture/PSF-scale correlations exist — this violates the method's assumption and biases the random-overlap distribution

## Inputs

- Background-corrected two-color (e.g., red/green or CFP/YFP) confocal 2D or 3D image stacks (Zeiss LSM 510, Argon laser 458/514 nm excitation)
- Live-cell time-lapse series (e.g., HeLa Rev-CFP / YFP-CRM1 after leptomycin B addition)
- Simulated two-color image series spanning 100% to 0% colocalization for validation

## Outputs

- Per-region or per-time-point set of colocalized pixels selected in the 2D histogram (above threshold and near the diagonal)
- Colocalization coefficients: proportion of each protein colocalized with the other
- Counts/proportions of misclassified pixels (false positives and false negatives)
- Nucleolar proportion time courses and fitted dissociation rates (e.g., Rev-CRM1 after LMB)

## How to apply

First background-correct the two-color image and visualize colocalization in a two-dimensional histogram of red vs. green pixel intensities. To handle correlations between adjacent pixel intensities, divide the image into independent blocks approximately the size of the texture or the PSF — measured as the smaller of the widths of the autocorrelations for the two images — and scramble these blocks instead of individual pixels, performing 200 randomizations per region to build the distribution of random overlap. Set the colocalization threshold automatically as the statistical criterion separating true overlap from this random-overlap distribution. Within the selected pixels, choose those closest to the diagonal line in the 2D histogram so that low-intensity colocalized pixels are distinguished from background rather than discarded. Then compute the colocalization coefficients — the proportion of each protein colocalized with the other — rather than assuming that all selected histogram points are entirely colocalized, which overpredicts colocalization. Implemented in MIPAV and commercialized by Bitplane.

## Related tools

- **MIPAV** (Software implementation of the automatic colocalization algorithm (2D histogram classification, randomization thresholding, coefficient computation))
- **Bitplane** (Commercialized implementation of the same pixel-classification colocalization algorithm)
- **Zeiss LSM 510 confocal microscope** (Acquisition of the two-channel CFP/YFP images that serve as input to pixel classification)

## Evaluation signals

- On simulated two-color series spanning 100% to 0% colocalization, the misclassified-pixel proportion is small (~5% false positives and false negatives)
- A random-overlap negative control (e.g., Rev-CFP + free YFP) yields coefficients falling within the 200-randomization random-overlap distribution
- Known-interaction groups (e.g., Rev-Rev, Rev-Tat in fixed HeLa cells) show significant colocalization while group-level results correlate with independent FRET measurements
- The method detects and quantifies colocalization in images where colocalization was not obvious visually (subvisual detection case)
- Block size matches the smaller of the two autocorrelation widths (texture/PSF scale) before scrambling, and exactly 200 randomizations are performed per region

## Limitations

- Assuming that all selected points in the two-dimensional histogram are entirely colocalized can lead to overprediction of the amount of colocalization
- Colocalization is not proof of direct binding: FRET was negative for the Rev-CRM1 complex, presumably due to the large size of CRM1 (130 kDa), even though colocalization was detected
- Adjacent pixel intensities are correlated, so thresholding must rely on block scrambling (~PSF-sized blocks) rather than pixel-level randomization; individual-pixel scrambling distorts the random-overlap distribution

## Evidence
