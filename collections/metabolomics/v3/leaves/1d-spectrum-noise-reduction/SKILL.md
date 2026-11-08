---
name: 1d-spectrum-noise-reduction
description: Use when selected windows of a SMolESY-enhanced 1D 1H NMR spectrum are
  noise-dominated and low-abundance spin systems (e.g. tyrosine, phenylalanine, histidine,
  creatinine, choline) must be recovered before integration.
when_to_use_negative:
- When working with a raw, unenhanced 1D 1H NMR spectrum or a CPMG spectrum — smoothing
  is defined here specifically for SMolESY-enhanced windows, and applying it to a
  standard spectrum would erode its native resolution without the SMolESY resolution-enhancement
  trade-off.
- When the entire spectrum needs uniform noise suppression rather than selected-window
  recovery — the article's filter is applied only 'within selected NMR spectral windows',
  not globally, and window selection logic (which bins to smooth) is part of SMolESY-select,
  not this filter step.
- When the goal is baseline correction or macromolecule suppression — those are separate
  SMolESY features; the 11-point filter only denoises, it does not remove broad baseline
  components or macromolecular signals.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3174
tools:
- name: SMolESY-select
  role: Invokes the 11-point moving-average filter within the spectral windows it
    has selected for low-abundance metabolite recovery, as part of its assignment/integration
    workflow.
  repo: https://github.com/pantakis/SMolESY-select
- name: SMolESY
  role: Produces the resolution-enhanced, macromolecule-suppressed 1D 1H NMR spectrum
    that serves as the input to the smoothing filter; the filter operates on SMolESY
    output, not raw spectra.
  repo: https://github.com/pantakis/SMolESY
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.1c00113
    title: A Computationally Lightweight Algorithm for Deriving Reliable Metabolite
      Panel Measurements from 1D <sup>1</sup>H NMR
  source_content_hash: sha256:6ce08f66e5e51454ea13f8bc5ccd8737e6303074c13858064a8bdc8ab4e3d483
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pantakis/SMolESY-select
  tools:
  - SMolESY
  - SMolESY-select
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# 1d-spectrum-noise-reduction

## Summary

Applies an 11-point moving-average smoothing filter over user-selected spectral windows of a SMolESY-enhanced 1D 1H NMR spectrum, suppressing random noise to recover low-abundance metabolite signals such as tyrosine, phenylalanine, histidine, creatinine, and choline.

## When to use

Use when selected windows of a SMolESY-enhanced 1D 1H NMR spectrum are noise-dominated and low-abundance spin systems (e.g. tyrosine, phenylalanine, histidine, creatinine, choline) must be recovered before integration.

## When NOT to use

- When working with a raw, unenhanced 1D 1H NMR spectrum or a CPMG spectrum — smoothing is defined here specifically for SMolESY-enhanced windows, and applying it to a standard spectrum would erode its native resolution without the SMolESY resolution-enhancement trade-off.
- When the entire spectrum needs uniform noise suppression rather than selected-window recovery — the article's filter is applied only 'within selected NMR spectral windows', not globally, and window selection logic (which bins to smooth) is part of SMolESY-select, not this filter step.
- When the goal is baseline correction or macromolecule suppression — those are separate SMolESY features; the 11-point filter only denoises, it does not remove broad baseline components or macromolecular signals.

## Inputs

- SMolESY-enhanced 1D 1H NMR spectrum (ppm shift and intensity arrays) from a Bruker IVDr 600 MHz acquisition
- List of predefined spectral windows (bins) selected for smoothing, as used by SMolESY-select

## Outputs

- Smoothed (denoised) spectrum restricted to the selected windows, with slightly narrower Δv1/2 than the standard 1D 1H NMR spectrum
- Denoised intensity arrays per selected window, ready for direct integration by SMolESY-select

## How to apply

Load the SMolESY-enhanced 1D 1H NMR spectrum as paired ppm-shift and intensity arrays. Identify the predefined spectral windows (bins) that SMolESY-select has selected for smoothing — these are the windows where low-abundance spin systems need signal-to-noise recovery. Within each selected window, slide an 11-point moving average across the high-resolution data, replacing each point with the mean of the 11 consecutive points centered on it; the article specifies the filter spans 11 points per the Supporting Information. The rationale is that the macromolecule-suppressed, resolution-enhanced SMolESY spectrum still carries random noise in low-population windows, and window-local smoothing raises s/n without resorting to CPMG acquisition. Do not smooth outside the selected windows, since smoothing outside them would blur the high-resolution features that SMolESY's resolution enhancement (Δv1/2 ≈ 0.0009–0.0011 ppm at 600 MHz) was designed to preserve. The smoothed output feeds directly into SMolESY-select's assignment/integration pipeline, where denoised signals maintain slightly narrower Δv1/2 than the standard 1D 1H NMR spectrum and at most ~10% lower s/n than the corresponding CPMG spectra.

## Related tools

- **SMolESY-select** (Invokes the 11-point moving-average filter within the spectral windows it has selected for low-abundance metabolite recovery, as part of its assignment/integration workflow.) — https://github.com/pantakis/SMolESY-select
- **SMolESY** (Produces the resolution-enhanced, macromolecule-suppressed 1D 1H NMR spectrum that serves as the input to the smoothing filter; the filter operates on SMolESY output, not raw spectra.) — https://github.com/pantakis/SMolESY

## Examples

```
python -c "import numpy as np; p,i = np.loadtxt('smolesy.tsv', unpack=True); k=11; smooth=np.convolve(i, np.ones(k)/k, mode='same'); np.savetxt('smolesy_smoothed.tsv', np.c_[p, smooth], fmt='%.6f')"
```

## Evaluation signals

- The smoothed spectrum's Δv1/2 remains slightly narrower than the standard 1D 1H NMR spectrum's Δv1/2 for the same windows — check that linewidth is not broadened beyond the pre-filter value.
- Signal-to-noise of tyrosine, phenylalanine, histidine, creatinine, and choline peaks in the selected windows increases relative to the unsmoothed SMolESY spectrum, and stays at most ~10% below the s/n of the corresponding CPMG spectra.
- Only points inside the predefined selected windows are altered; ppm values outside those windows are byte-identical to the input SMolESY spectrum.
- The moving-average window size is exactly 11 points; verify that each output point equals the mean of 11 consecutive centered input points.

## Limitations

- Filter width is fixed at 11 points per the article/Supporting Information; the article does not state a user-adjustable window size.
- The article does not state a software version for SMolESY-select or SMolESY, and the GUI/video usage guidance (Figure S9, Video S1) is only available as supplementary artifacts, not in the main text.
- Smoothing outside the selected windows would degrade the high-resolution features that SMolESY was designed to preserve, so window selection must be correct before applying the filter.

## Evidence

- [results] SMolESY-select utilizes a moving average filter spanning 11-points: "SMolESY-select utilizes a moving average filter spanning 11-points"
- [results] Moving average filter increases s/n of signals for tyrosine, phenylalanine, histidine, creatinine, and choline: "increases the s/n of signals belonging to tyrosine, phenylalanine, histidine, creatinine, and choline"
- [results] Denoised SMolESY signals maintain slightly narrower Δv1/2 than standard 1D 1H NMR, with s/n at most ∼10% less than CPMG spectra: "their s/n is a maximum of ∼10% less than that of the CPMG spectra"
- [results] 0.0009 < SMolESY Δv1/2 < 0.0011 ppm for the 600 MHz NMR instrument: "0.0009 < SMolESY Δv1/2 < 0.0011 ppm for the 600 MHz NMR instrument"
- [results] A smoothing filter is then applied within selected NMR spectral windows to help with the recovery of signals: "A smoothing filter is then applied within selected NMR spectral windows to help with the recovery of signals"
