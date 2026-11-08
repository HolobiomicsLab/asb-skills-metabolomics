---
name: smolesy-spectrum-smoothing
description: Use when a SMolESY-enhanced 1D ¹H NMR spectrum has been processed and
  specific spectral windows (e.g. tyrosine, phenylalanine, histidine, creatinine,
  choline) still show low s/n signals from low-abundance metabolites that need denoising
  before integration.
when_to_use_negative:
- The spectrum has not been SMolESY-enhanced (e.g. it is a raw or standard 1D ¹H NMR
  spectrum); smoothing without prior resolution enhancement would broaden Δν1/2 toward
  the standard line width and lose the s/n benefit
- The target metabolite's signals lie outside the predefined spectral windows and
  require full-spectrum processing or complex deconvolution instead
- The sample data are CPMG spectra with adequate s/n, since the smoothing filter trades
  up to ∼10% s/n loss relative to CPMG and offers no advantage there
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: SMolESY
  role: Produces the resolution-enhanced, macromolecule-suppressed 1D ¹H NMR spectrum
    whose high-resolution data within the selected windows is the input to the moving
    average filter
  repo: https://github.com/pantakis/SMolESY
- name: SMolESY-select
  role: Applies the 11-point moving average filter across the high-resolution data
    of the selected spectral windows and consumes the smoothed spectrum for direct
    integration
  repo: https://github.com/pantakis/SMolESY-select
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.1c00113
    title: A Computationally Lightweight Algorithm for Deriving Reliable Metabolite
      Panel Measurements from 1D <sup>1</sup>H NMR
  source_content_hash: sha256:85c5d8b80ccdc522d118aba902f329b0e254167eeba0c05b50d705be8c347218
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pantakis/SMolESY
  tools:
  - SMolESY
  - SMolESY-select
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# smolesy-spectrum-smoothing

## Summary

Applies an 11-point moving average smoothing filter across the high-resolution data of selected NMR spectral windows in SMolESY-enhanced 1D ¹H NMR spectra, to denoise those windows and recover signals of frequently low-abundance metabolites.

## When to use

Use when a SMolESY-enhanced 1D ¹H NMR spectrum has been processed and specific spectral windows (e.g. tyrosine, phenylalanine, histidine, creatinine, choline) still show low s/n signals from low-abundance metabolites that need denoising before integration.

## When NOT to use

- The spectrum has not been SMolESY-enhanced (e.g. it is a raw or standard 1D ¹H NMR spectrum); smoothing without prior resolution enhancement would broaden Δν1/2 toward the standard line width and lose the s/n benefit
- The target metabolite's signals lie outside the predefined spectral windows and require full-spectrum processing or complex deconvolution instead
- The sample data are CPMG spectra with adequate s/n, since the smoothing filter trades up to ∼10% s/n loss relative to CPMG and offers no advantage there

## Inputs

- SMolESY-enhanced 1D ¹H NMR spectrum (ppm shift and intensity arrays, acquired on Bruker IVDr 600 MHz spectrometer)
- Definition of the selected NMR spectral windows (bins) for the target spin systems

## Outputs

- Denoised SMolESY spectrum restricted to the selected windows (smoothed ppm and intensity arrays)
- Signals with increased s/n for tyrosine, phenylalanine, histidine, creatinine, and choline ready for direct integration

## How to apply

Load the SMolESY-enhanced 1D ¹H NMR spectrum as ppm shift and intensity arrays (calibrated to the 1H NMR anomeric proton signal of glucose at 5.233 ppm). Select the spectral windows already defined for the target spin systems' bins, then slide an 11-point moving average filter (window of 11 consecutive spectral points) across the high-resolution data within those selected windows, replacing each point with the mean of the 11-point window centered on it. The rationale is that SMolESY's resolution enhancement narrows line widths (0.0009 < SMolESY Δν1/2 < 0.0011 ppm at 600 MHz), so averaging over 11 points suppresses random noise without broadening the narrowed peaks. The filter specifically increases the s/n of signals belonging to tyrosine, phenylalanine, histidine, creatinine, and choline, and the denoised SMolESY signals maintain slightly narrower Δν1/2 than the standard 1D ¹H NMR spectrum while keeping s/n at most ∼10% less than CPMG spectra. Write the smoothed ppm/intensity arrays back out so downstream SMolESY-select integration can proceed directly without complex signal deconvolution or baseline fitting.

## Related tools

- **SMolESY** (Produces the resolution-enhanced, macromolecule-suppressed 1D ¹H NMR spectrum whose high-resolution data within the selected windows is the input to the moving average filter) — https://github.com/pantakis/SMolESY
- **SMolESY-select** (Applies the 11-point moving average filter across the high-resolution data of the selected spectral windows and consumes the smoothed spectrum for direct integration) — https://github.com/pantakis/SMolESY-select

## Examples

```
python smolesy_select.py --input smolesy_spectrum.ppm --moving-average 11 --windows selected_windows.txt --output smoothed_spectrum.ppm
```

## Evaluation signals

- The smoothed spectrum's peaks retain slightly narrower Δν1/2 than the corresponding standard 1D ¹H NMR spectrum (consistent with the 0.0009–0.0011 ppm SMolESY line-width range at 600 MHz)
- Signal-to-noise ratio of the denoised windows is at most ∼10% less than that of the corresponding CPMG spectra
- s/n visibly increases for the tyrosine, phenylalanine, histidine, creatinine, and choline signals after smoothing
- All spectra were automatically calibrated to the glucose anomeric proton signal at 5.233 ppm before filtering
- Downstream direct integration of assigned spectral components succeeds without needing complex signal deconvolution or baseline fitting

## Limitations

- The 11-point window size and the fact that it is applied only within selected spectral windows are documented in the Supporting Information rather than the main text, so exact implementation details may require consulting that material
- Denoised signals can reach at most ∼10% lower s/n than CPMG spectra, so for metabolites where maximal s/n is critical CPMG may be preferable
- The filter is only beneficial on SMolESY-enhanced spectra (with line widths of 0.0009–0.0011 ppm at 600 MHz); applying it to standard 1D spectra would smooth across the wider native line widths and lose the resolution advantage

## Evidence

- [results] 11-point moving average filter on selected windows: [evidence span withheld — anchor status: not_found]
- [results] s/n increase for named metabolites: "increases the s/n of signals belonging to tyrosine, phenylalanine, histidine, creatinine, and choline"
- [results] line-width preservation and s/n trade-off: "their s/n is a maximum of ∼10% less than that of the CPMG spectra"
- [results] calibration reference: "to the anomeric proton of glucose (Figure 2a) resonating at 5.233 ppm"
- [results] integration without deconvolution: "enables the direct integration of the assigned spectral components, precluding the need for complex signal deconvolution or baseline fitting"
