---
name: spectral-moving-average-smoothing
description: Use when denoising selected windows of an SMolESY-enhanced 1D 1H NMR
  spectrum whose weak metabolite signals in noisy regions must be recovered for subsequent
  pattern assignment and direct integration.
when_to_use_negative:
- The input is a raw (non-SMolESY-enhanced) 1D 1H NMR or CPMG spectrum whose native
  line widths must be preserved without resolution enhancement or baseline suppression.
- The spectral region must retain full native resolution for pattern matching or quantitation
  that does not tolerate the ~10% s/n cost relative to CPMG.
- Data are 2D spectra (e.g. 2D-Jres) or non-NMR 1D signals where an 11-point window
  calibrated against 600 MHz SMolESY Δv1/2 line widths is not meaningful.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0041
- http://edamontology.org/topic_3520
tools:
- name: SMolESY-select
  role: Invokes the 11-point moving-average filter within selected NMR spectral windows
    during the assignment-and-integration pipeline; smoothing is a pre-processing
    stage before pattern search and direct integration.
  repo: https://github.com/pantakis/SMolESY-select
- name: SMolESY
  role: Produces the resolution-enhanced, baseline-suppressed 1D spectrum that serves
    as the input to which the 11-point moving-average filter is applied.
  repo: https://github.com/pantakis/SMolESY
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c00113
    title: A Computationally Lightweight Algorithm for Deriving Reliable Metabolite
      Panel Measurements from 1D <sup>1</sup>H NMR
  source_content_hash: sha256:21f5fc90d0f86569dc5134bb407c5a66a1023c46710a1c09aae023ad62a76d70
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pantakis/SMolESY-select
  tools:
  - 2D-Jres
  - SMolESY
  - SMolESY-select
  - STOCSY
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0041
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# spectral-moving-average-smoothing

## Summary

Applies an 11-point moving-average filter across the high-resolution SMolESY-enhanced 1D 1H NMR data within preselected spectral windows, raising s/n so that signals of low-abundance metabolites (tyrosine, phenylalanine, histidine, creatinine, choline) become recoverable while retaining the resolution-enhanced SMolESY line shape.

## When to use

Use when denoising selected windows of an SMolESY-enhanced 1D 1H NMR spectrum whose weak metabolite signals in noisy regions must be recovered for subsequent pattern assignment and direct integration.

## When NOT to use

- The input is a raw (non-SMolESY-enhanced) 1D 1H NMR or CPMG spectrum whose native line widths must be preserved without resolution enhancement or baseline suppression.
- The spectral region must retain full native resolution for pattern matching or quantitation that does not tolerate the ~10% s/n cost relative to CPMG.
- Data are 2D spectra (e.g. 2D-Jres) or non-NMR 1D signals where an 11-point window calibrated against 600 MHz SMolESY Δv1/2 line widths is not meaningful.

## Inputs

- SMolESY-enhanced 1D 1H NMR spectrum (ppm chemical shift array and intensity array) for selected spectral windows, acquired on a Bruker IVDr 600 MHz spectrometer
- Predefined spectral-window (bin) definitions for the target 1H NMR spin systems, expressed in SMolESY Δv1/2 units (0.0009–0.0011 ppm at 600 MHz)

## Outputs

- Denoised (smoothed) spectrum: ppm shift array paired with the 11-point moving-average-smoothed intensity array
- Smoothed windows in which low-abundance metabolite signals (tyrosine, phenylalanine, histidine, creatinine, choline) are detectable at higher s/n for subsequent pattern assignment

## How to apply

Load the SMolESY-enhanced 1D 1H NMR spectrum as paired ppm-shift and intensity arrays for the preselected spectral windows (bins defined by the SMolESY Δv1/2 scheme, e.g. ≥10 Δv1/2 for glycine, creatine/creatinine, choline, DMSO2, acetone, acetic acid; 3–5 Δv1/2 for correlating singlets vs lactic acid/alanine). Slide an 11-point moving-average filter centered on each point across the high-resolution data of those windows only, replacing each intensity with the mean of its 11-point neighborhood; smoothing is applied to the SMolESY-enhanced data rather than the raw spectrum so that baseline suppression and resolution enhancement are preserved. This raises the s/n of signals from tyrosine, phenylalanine, histidine, creatinine, and choline that would otherwise be buried in noise. Leave the filter width fixed at 11 points (window ≈0.011 ppm at 600 MHz given SMolESY Δv1/2 of 0.0009–0.0011 ppm); do not smooth outside the selected windows where native SMolESY resolution is needed for pattern matching. Write the smoothed spectrum (ppm, smoothed intensity) to an output file for downstream assignment and direct integration by SMolESY-select.

## Related tools

- **SMolESY-select** (Invokes the 11-point moving-average filter within selected NMR spectral windows during the assignment-and-integration pipeline; smoothing is a pre-processing stage before pattern search and direct integration.) — https://github.com/pantakis/SMolESY-select
- **SMolESY** (Produces the resolution-enhanced, baseline-suppressed 1D spectrum that serves as the input to which the 11-point moving-average filter is applied.) — https://github.com/pantakis/SMolESY

## Examples

```
from numpy.lib.stride_tricks import sliding_window_view; smoothed = sliding_window_view(y, 11).mean(axis=1); np.savetxt('smolesy_smooth.csv', np.column_stack([ppm[5:-5], smoothed]), delimiter=',')
```

## Evaluation signals

- The smoothed spectrum contains exactly the same ppm shift grid as the input (no resampling or truncation of the axis).
- Smoothed intensities equal, point-by-point, the mean of each 11-point window in the input SMolESY-enhanced intensity array (recomputable with numpy).
- The Δv1/2 of the denoised SMolESY signals remains slightly narrower than that of the standard 1D 1H NMR spectrum.
- Signal s/n of the smoothed windows is at most ∼10% less than that of the corresponding CPMG spectra.
- Signals of tyrosine, phenylalanine, histidine, creatinine, and choline become clearly detectable in the smoothed windows after denoising.

## Limitations

- The filter is applied only within selected NMR spectral windows, not across the entire spectrum, so windows outside the preselected bins remain unsmoothed and noisy.
- Denoising trades s/n for simplicity: the s/n of smoothed signals is a maximum of ∼10% less than that of the CPMG spectra.
- The 11-point window is calibrated to 600 MHz Bruker IVDr data (SMolESY Δv1/2 of 0.0009–0.0011 ppm); rescaling to other field strengths or line widths is not described in the article.

## Evidence

- [results] Moving average filter spanning 11-points across the high resolution data of selected windows: "SMolESY-select utilizes a moving average filter spanning 11-points"
- [results] Filter purpose — s/n increase for low-abundance metabolites: "increases the s/n of signals belonging to tyrosine, phenylalanine, histidine, creatinine, and choline"
- [results] Resolution retention — narrower Δv1/2 than standard 1D and s/n cost vs CPMG: "their s/n is a maximum of ∼10% less than that of the CPMG spectra"
- [results] Line-width scale for the 11-point window (600 MHz SMolESY Δv1/2): "0.0009 < SMolESY Δv1/2 < 0.0011 ppm for the 600 MHz NMR instrument"
- [results] Denosing applied only within selected windows, per the SMolESY-select design: "A smoothing filter is then applied within selected NMR spectral windows to help with the recovery of signals"
