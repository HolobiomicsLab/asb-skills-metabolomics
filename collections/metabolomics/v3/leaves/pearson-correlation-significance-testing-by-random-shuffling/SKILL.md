---
name: pearson-correlation-significance-testing-by-random-shuffling
description: Use when two co-registered LA-ICP-MS elemental images (identical pixel
  dimensions) require quantification of spatial colocalisation, and a significance
  probability for the observed Pearson R is needed beyond the raw correlation coefficient.
when_to_use_negative:
- The two channels have different pixel dimensions or are not spatially co-registered
  (e.g. spot-wise data without a shared grid) — Pearson R and shuffling-based ρ are
  undefined on misaligned arrays.
- Only a qualitative visual check of RGB overlay magenta overlap is required and no
  statistical significance is needed — a simple RGB overlay of 55Mn/56Fe/66Zn suffices.
- Data are time-series or spectra rather than 2D spatial images; the shuffling-based
  null assumes pixel-permutation exchangeability within an image.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Pew2
  role: GUI implementing the colocalisation dialog that computes Pearson R, Li ICQ,
    Manders coefficients, and triggers the shuffle-based ρ calculation
  repo: https://github.com/djdt/pewpew
- name: pewlib
  role: Python library that imports the LA-ICP-MS data into NumPy arrays feeding the
    colocalisation calculations
  repo: https://github.com/djdt/pewlib
- name: NumPy
  role: Vectorized array arithmetic used to compute Pearson R and to perform random
    pixel shuffles for the permutation test
- name: PySide2
  role: Qt5 GUI framework hosting the colocalisation dialog, context menus, and the
    ρ trigger button
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.1c02138
    title: 'Pew <sup>2</sup> : Open-Source Imaging Software for Laser Ablation–Inductively
      Coupled Plasma–Mass Spectrometry'
  source_content_hash: sha256:286454d2e8ad4822d71f9c0343a61d792180328a7a303fb34168f505a4b1fb7a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/djdt/pewpew
  tools:
  - NumPy
  - Pew2
  - PySide2
  - Python
  - pewlib
  - pewpew
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# pearson-correlation-significance-testing-by-random-shuffling

## Summary

Computes Pearson's R between two co-registered LA-ICP-MS elemental images (e.g. 55Mn and 66Zn) and derives an empirical significance probability ρ by comparing the observed R against R values from randomly shuffled pixel permutations of the two channels, as implemented in Pew2's colocalisation dialog.

## When to use

Use when two co-registered LA-ICP-MS elemental images (identical pixel dimensions) require quantification of spatial colocalisation, and a significance probability for the observed Pearson R is needed beyond the raw correlation coefficient.

## When NOT to use

- The two channels have different pixel dimensions or are not spatially co-registered (e.g. spot-wise data without a shared grid) — Pearson R and shuffling-based ρ are undefined on misaligned arrays.
- Only a qualitative visual check of RGB overlay magenta overlap is required and no statistical significance is needed — a simple RGB overlay of 55Mn/56Fe/66Zn suffices.
- Data are time-series or spectra rather than 2D spatial images; the shuffling-based null assumes pixel-permutation exchangeability within an image.

## Inputs

- Two co-registered LA-ICP-MS elemental images (NumPy arrays of identical pixel dimensions, e.g. 55Mn and 66Zn from Agilent MassHunter imports)
- Optional user-drawn lasso or rectangle selection restricting the coefficient inputs to a sub-region

## Outputs

- Pearson R correlation coefficient over the full image or selected region
- Significance probability ρ obtained by comparing measured R to R values from randomly shuffled pairs of the two channels
- Supporting colocalisation coefficients (Li ICQ, Manders overlap via Costes method) displayed in the same dialog with copyable results

## How to apply

Load both elemental channels (e.g. 55Mn and 66Zn) into Pew2's NumPy-array backend via pewlib, ensuring identical pixel dimensions; compute Pearson R over the full image, or restrict to a lasso/rectangle selection via the image or selection context menu. Then trigger the ρ (probability) button: the software repeatedly and randomly shuffles the pixel values of the two channels, recalculates R for each shuffle, and estimates the probability of observing an R at least as extreme as the measured one by comparing the measured R to this shuffled null distribution. This empirical permutation p-value complements Li ICQ and Manders/Costes-overlap coefficients computed in the same dialog. The random-shuffle approach avoids parametric assumptions about the joint pixel-value distribution, which is appropriate for LA-ICP-MS data where signal may be sparse or non-Gaussian.

## Related tools

- **Pew2** (GUI implementing the colocalisation dialog that computes Pearson R, Li ICQ, Manders coefficients, and triggers the shuffle-based ρ calculation) — https://github.com/djdt/pewpew
- **pewlib** (Python library that imports the LA-ICP-MS data into NumPy arrays feeding the colocalisation calculations) — https://github.com/djdt/pewlib
- **NumPy** (Vectorized array arithmetic used to compute Pearson R and to perform random pixel shuffles for the permutation test)
- **PySide2** (Qt5 GUI framework hosting the colocalisation dialog, context menus, and the ρ trigger button)

## Examples

```
pewpew  # launch GUI, open 55Mn & 66Zn images from Agilent .b batch, select both channels, open colocalisation dialog, click ρ button to run shuffle-based significance test
```

## Evaluation signals

- Computed Pearson R and Li ICQ for the 55Mn/66Zn channel pair should show a positive correlative spatial relationship consistent with the magenta regions visible in the RGB overlay (Figure 6).
- The ρ value must be derived from Monte Carlo comparison of measured R against R values from random shuffles of the two channels, not from an analytic t-distribution approximation.
- Restricting inputs to a lasso/rectangle selection must change the reported R (and ρ) relative to the full-image value, confirming the region restriction is honoured.
- Coefficients must be reproducible across repeated runs of the same channel pair with fixed random seeds, and the dialog's results output must be copyable.
- Calibration of the underlying elemental images (e.g. 55Mn, 66Zn against gelatin standards, r² ≈ 0.99) must be complete before colocalisation so that noise from unquantified channels does not inflate R.

## Limitations

- The permutation ρ is computed by random shuffling and is a Monte Carlo estimate, so it carries sampling uncertainty and depends on the number of shuffles performed.
- Spot-wise LA-ICP-MS data requires at least one element to be present across the entire sample with similar peak positions for each element for correct peak integration; if peak integration is unreliable, downstream R and ρ are compromised.
- Drift, spikes, and low/high-signal instrument artefacts should be corrected (polynomial drift renormalisation, local mean/median spike filtering) before computing colocalisation, otherwise the null distribution and R are biased.
- The shuffle-based null assumes pixel exchangeability within each channel; spatial autocorrelation in the images can inflate the apparent significance of R.

## Evidence

- [results] Pew2 quantifies the spatial overlap of two elemental images by calculating Pearson's R correlation coefficient from the pixel values of the two images: [evidence span withheld — anchor status: not_found]
- [results] Implement the Pearson R coefficient over the entire image with its associated probability (ρ), computed by comparing the R value to those of random shuffles of the two channels: [evidence span withheld — anchor status: not_found]
