---
name: nmr-moving-average-smoothing-filter
description: Use when processing 1D 1H NMR serum/plasma spectra (e.g. Bruker IVDr
  600 MHz acquisitions) where targeted low-concentration metabolite spin systems are
  obscured by noise and need a standardized 11-point moving-average denoising before
  pattern-based assignment.
when_to_use_negative:
- When the input is already a 2D spectrum (e.g. 2D-Jres) or a CPMG spectrum — the
  filter is specified only for high-resolution 1D 1H NMR data within selected windows
- When the analysis goal is quantitative deconvolution of overlapped peaks rather
  than s/n enhancement for pattern assignment — SMolESY-select explicitly avoids complex
  signal deconvolution or baseline fitting
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3292
tools:
- name: SMolESY-select
  role: Hosts the 11-point moving average filter as a step within its automated assignment
    workflow for 1D 1H NMR metabolite panels
  repo: https://github.com/pantakis/SMolESY-select
- name: SMolESY
  role: Provides the baseline suppression and resolution enhancement that precedes/accompanies
    the smoothing filter
  repo: https://github.com/pantakis/SMolESY
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c00113
    title: A Computationally Lightweight Algorithm for Deriving Reliable Metabolite
      Panel Measurements from 1D <sup>1</sup>H NMR
  source_content_hash: sha256:abd48311dc054e876b590fb53f87f2fdf7edb56789f28ebce4e5928e27493ff8
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
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# nmr-moving-average-smoothing-filter

## Summary

Applies a fixed 11-point moving average filter across high-resolution 1D 1H NMR spectral windows to boost signal-to-noise of low-abundance metabolite signals (tyrosine, phenylalanine, histidine, creatinine, choline) while preserving near-CMPG line widths — the smoothing step inside SMolESY-select.

## When to use

Use when processing 1D 1H NMR serum/plasma spectra (e.g. Bruker IVDr 600 MHz acquisitions) where targeted low-concentration metabolite spin systems are obscured by noise and need a standardized 11-point moving-average denoising before pattern-based assignment.

## When NOT to use

- When the input is already a 2D spectrum (e.g. 2D-Jres) or a CPMG spectrum — the filter is specified only for high-resolution 1D 1H NMR data within selected windows
- When the analysis goal is quantitative deconvolution of overlapped peaks rather than s/n enhancement for pattern assignment — SMolESY-select explicitly avoids complex signal deconvolution or baseline fitting

## Inputs

- 1D 1H NMR spectrum of serum or plasma (e.g. Bruker IVDr 600 MHz acquisition), already calibrated to glucose anomeric proton at 5.233 ppm
- Selected spectral windows (bins) for the 24 target spin systems

## Outputs

- Smoothed high-resolution spectral segments with enhanced s/n for tyrosine, phenylalanine, histidine, creatinine, and choline
- Denoised signal ready for spin-system pattern search within predefined SMolESY bins

## How to apply

First, calibrate the 1D 1H NMR spectrum to the anomeric proton of glucose at 5.233 ppm. Then apply a moving average filter spanning exactly 11 points across the high-resolution data of the selected spectral windows — not globally — because the goal is to raise the s/n of signals belonging to tyrosine, phenylalanine, histidine, creatinine, and choline without destroying resolution. The article reports the resulting denoised SMolESY signals retain slightly narrower Δv1/2 than standard 1D 1H NMR and lose at most ∼10% s/n compared with CPMG spectra, so the filter's effect should be checked against these line-width and s/n bounds. Within SMolESY-select, this smoothed data is then searched for each of 24 predefined spin-system bins (widths of 3–5 SMolESY Δv1/2 for singlets correlated with lactic acid/alanine, or ≥10 Δv1/2 for glycine, creatine/creatinine, choline, DMSO2, acetone, acetic acid).

## Related tools

- **SMolESY-select** (Hosts the 11-point moving average filter as a step within its automated assignment workflow for 1D 1H NMR metabolite panels) — https://github.com/pantakis/SMolESY-select
- **SMolESY** (Provides the baseline suppression and resolution enhancement that precedes/accompanies the smoothing filter) — https://github.com/pantakis/SMolESY

## Examples

```
python smolesy_select.py --input spectrum.pz --calibrate 5.233 --moving-average 11 --window tyrosine,phenylalanine,histidine,creatinine,choline
```

## Evaluation signals

- Line width check: denoised SMolESY signals should have slightly narrower Δv1/2 than the unsmoothed 1D 1H NMR spectrum
- Signal-to-noise check: s/n of the smoothed signals should be no more than ∼10% lower than the corresponding CPMG spectra
- Target metabolites check: s/n improvement should be evident specifically for tyrosine, phenylalanine, histidine, creatinine, and choline
- Window check: the filter must be applied only within the selected spectral windows, not across the full spectrum

## Limitations

- The exact implementation details of the 11-point filter are deferred to the Supporting Information (see the Supporting Information), so reproducibility depends on accessing that document
- The filter trades a modest s/n loss (up to ∼10% relative to CPMG) for resolution, so it is not appropriate when maximal s/n is the overriding priority
- No software version is stated for SMolESY-select or SMolESY in the article

## Evidence

- [results] 11-point moving average filter: "Specifically, SMolESY-select utilizes a moving average filter spanning 11-points"
- [results] target metabolites: "increases the s/n of signals belonging to tyrosine, phenylalanine, histidine, creatinine, and choline"
- [results] line-width and s/n bounds: "their s/n is a maximum of ∼10% less than that of the CPMG spectra"
- [results] scope: "A smoothing filter is then applied within selected NMR spectral windows to help with the recovery of signals"
