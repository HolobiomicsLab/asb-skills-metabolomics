---
name: synthetic-fluorescence-image-simulation
description: Use when validating or stress-testing a colocalization coefficient or
  thresholding algorithm, and you need simulated red/green images with known colocalized
  fractions to compare computed coefficients against ground truth.
when_to_use_negative:
- The input is already real experimental imagery with no reliable ground-truth colocalization
  fractions — simulation benchmarks require known mixing fractions to quantify overprediction
- The analysis goal is biological interpretation of a specific interaction (e.g.,
  Rev-Rev vs Rev-CRM1 FRET comparisons) rather than algorithm validation
- The algorithm under test does not operate on two-color pixel-intensity histograms,
  since the simulation's ground truth is defined against red/green 2D-histogram classification
edam_operation: http://edamontology.org/operation_3552
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_2269
tools:
- name: MIPAV
  role: Software implementing the Costes automatic threshold/colocalization algorithm
    that the synthetic images are used to validate
- name: Bitplane
  role: Commercial implementation of the colocalization algorithm under test against
    the simulated ground-truth images
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1529/biophysj.103.038422
    title: Automatic and Quantitative Measurement of Protein-Protein Colocalization
      in Live Cells
  source_content_hash: sha256:dd8f056924fa2dcd7da29257b0365f967944915315529d3606e7f537e5133271
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bitplane
  - MIPAV
  edam_operation: http://edamontology.org/operation_3552
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# synthetic-fluorescence-image-simulation

## Summary

Generates synthetic two-channel (red/green) fluorescence image pairs spanning the full colocalization range — from 100% colocalized green pixels down to none, plus mixed-signal pixels whose intensities are sums of colocalized and non-colocalized components with known fractions — to serve as ground-truth benchmarks for colocalization algorithms such as the Costes automatic threshold.

## When to use

Use when validating or stress-testing a colocalization coefficient or thresholding algorithm, and you need simulated red/green images with known colocalized fractions to compare computed coefficients against ground truth.

## When NOT to use

- The input is already real experimental imagery with no reliable ground-truth colocalization fractions — simulation benchmarks require known mixing fractions to quantify overprediction
- The analysis goal is biological interpretation of a specific interaction (e.g., Rev-Rev vs Rev-CRM1 FRET comparisons) rather than algorithm validation
- The algorithm under test does not operate on two-color pixel-intensity histograms, since the simulation's ground truth is defined against red/green 2D-histogram classification

## Inputs

- Ground-truth mixing fractions (colocalized vs non-colocalized component fractions per pixel)
- Simulation parameters: colocalization range (100% to 0% colocalized green pixels), image dimensions, PSF/texture block size
- Optionally, real confocal image statistics (e.g., Zeiss LSM 510 acquisitions of HeLa cells expressing Rev-CFP/YFP-CRM1) to make simulations realistic

## Outputs

- Synthetic two-channel (red/green) image pairs for each colocalization condition
- Computed colocalization coefficients per condition (proportion of each protein colocalized with the other)
- Overprediction table (computed coefficient minus true colocalized fraction), e.g., overprediction_analysis.csv

## How to apply

Generate synthetic two-channel image pairs covering the full colocalization range described in the paper, from 100% of green pixels colocalized with red pixels down to no colocalized pixels, plus mixed-signal variants where each pixel's intensity is the sum of a colocalized and a non-colocalized component with known fractions. Structure each image so colocalization can be visualized in a two-dimensional red/green intensity histogram, with pixels near the diagonal representing low-intensity colocalized signal distinct from background. Then run the algorithm under test (e.g., Costes automatic threshold: block scrambling at approximately PSF/texture size with 200 randomizations per region, followed by pixel classification in the 2D histogram) and compute the proportion of each protein colocalized with the other. Compare computed coefficients against the known mixed fractions; for the pure-colocalization simulations the article reports ~5% pixel misclassification as the expected error floor. Tabulate the overprediction (computed coefficient minus true colocalized fraction) across the mixing series — because real pixel intensities are sums of colocalized and non-colocalized signals, assuming selected 2D-histogram points are entirely colocalized can overpredict colocalization.

## Related tools

- **MIPAV** (Software implementing the Costes automatic threshold/colocalization algorithm that the synthetic images are used to validate)
- **Bitplane** (Commercial implementation of the colocalization algorithm under test against the simulated ground-truth images)

## Examples

```
python run_overprediction_analysis.py --mixing-series 1.0,0.75,0.5,0.25,0.0 --blocks psf_size --randomizations 200 --output overprediction_analysis.csv
```

## Evaluation signals

- Pure-colocalization simulations reproduce the paper's validation regime of ~5% misclassified pixels (false positives and false negatives)
- Binary pixel-classification assumption yields non-negative overprediction (computed coefficient ≥ true colocalized fraction) across all mixed-signal conditions
- Computed coefficients approach the ground-truth mixed fractions as the simulation approaches the pure regimes (100% or 0% colocalized)
- Overprediction values tabulated across the full mixing series and written to overprediction_analysis.csv with one row per condition
- Synthetic images render sensible 2D red/green intensity histograms, with colocalized pixels closest to the diagonal and separable from background

## Limitations

- Real pixel intensities are sums of colocalized and non-colocalized signals, so binary classification of pixels as entirely colocalized or non-colocalized can overpredict the amount of colocalization
- The paper reports the binary-classification assumption's overprediction as 'data not shown', so the mixing-series magnitudes are not quantified in the article itself
- Simulations must match the paper's randomization scheme (blocks sized to texture/PSF, 200 randomizations per region) or the ~5% misclassification benchmark does not apply

## Evidence
