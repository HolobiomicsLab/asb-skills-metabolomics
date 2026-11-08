---
name: smolsesy-spectral-window-matching
description: Use when you have calibrated 1D 1H NMR serum/plasma spectra (e.g. Bruker
  IVDr 600 MHz, MTBLS395) and need to assign 24 predefined 1H NMR spin systems to
  specific spectral windows (bins) so metabolite signals can be integrated without
  deconvolution.
when_to_use_negative:
- Input is already a processed metabolite concentration table or the spectrum has
  not been SMolESY-processed — this skill operates on SMolESY data, not raw or differently
  preprocessed spectra
- The analysis requires deconvolution or baseline fitting of complex spectral mixtures
  — SMolESY-select explicitly avoids these steps
- Metabolite is outside the predefined 22-metabolite panel / 24 spin-system bins
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: SMolESY-select
  role: Performs the pattern search of spin-system windows against SMolESY data and
    emits per-bin assignments; applies the 11-point moving average filter within selected
    windows
  repo: https://github.com/pantakis/SMolESY-select
- name: SMolESY
  role: Pre-processes the 1D 1H NMR spectrum (macromolecular suppression, resolution
    enhancement) producing the data on which window matching runs
  repo: https://github.com/pantakis/SMolESY
- name: Bruker IVDr 600 MHz spectrometer
  role: Source instrument generating the calibrated 1D 1H NMR serum/plasma spectra
    that the module ingests
- name: STOCSY
  role: Used to check overlap of each spin system's SMolESY pattern with other SM
    signals and to guide bin definitions
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.1c00113
    title: A Computationally Lightweight Algorithm for Deriving Reliable Metabolite
      Panel Measurements from 1D <sup>1</sup>H NMR
  source_content_hash: sha256:28d6dcd079868c2852867a8eb48ec397d73b1bc126c99018193f9798408c4dc2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pantakis/SMolESY-select
  tools:
  - Bruker IVDr 600 MHz spectrometer
  - SMolESY
  - SMolESY-select
  - STOCSY
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# smolsesy-spectral-window-matching

## Summary

Assigns 1H NMR spin-system patterns of 22 serum/plasma metabolites to their spectral positions in SMolESY-processed spectra by searching each predefined spectral window (bin) for its expected pattern, producing per-bin assignments and direct integration without deconvolution or baseline fitting.

## When to use

Use when you have calibrated 1D 1H NMR serum/plasma spectra (e.g. Bruker IVDr 600 MHz, MTBLS395) and need to assign 24 predefined 1H NMR spin systems to specific spectral windows (bins) so metabolite signals can be integrated without deconvolution.

## When NOT to use

- Input is already a processed metabolite concentration table or the spectrum has not been SMolESY-processed — this skill operates on SMolESY data, not raw or differently preprocessed spectra
- The analysis requires deconvolution or baseline fitting of complex spectral mixtures — SMolESY-select explicitly avoids these steps
- Metabolite is outside the predefined 22-metabolite panel / 24 spin-system bins

## Inputs

- Calibrated 1D 1H NMR serum or plasma spectrum (Bruker IVDr 600 MHz format, glucose-referenced at 5.233 ppm)
- SMolESY-processed spectrum (macromolecule-suppressed, resolution-enhanced)
- Predefined spectral-window (bin) definitions for 24 1H NMR spin systems
- Spin-system pattern reference (e.g. from STOCSY / 2D-Jres)

## Outputs

- Per-bin assignment record for each of the 24 spin-system windows, including assignment status and detected ppm value
- Direct integration values for assigned spectral components
- Per-spectrum metabolite panel measurements for the 22 target metabolites

## How to apply

First calibrate the 1D 1H NMR spectrum to the glucose anomeric proton at 5.233 ppm, then run SMolESY to suppress macromolecular signal and enhance resolution (SMolESY Δv1/2 ≈ 0.0009–0.0011 ppm at 600 MHz). For each of the 24 predefined spectral windows (bins), derived from a 4023-sample cohort and sized from at least 10 SMolESY Δv1/2 (>0.01 ppm) for broad singlets or 3–5 Δv1/2 for correlated singlets (glycine, creatinine, choline, DMSO2 vs. lactic acid/alanine), search within that window for the expected spin-system multiplet pattern. Apply a 11-point moving average filter inside selected windows to raise s/n of low-abundance signals (tyrosine, phenylalanine, histidine, creatinine, choline). Where a spin-system pattern matches, record the bin's assignment and its detected ppm value; the pattern search uses STOCSY and 2D-Jres as reference for overlap handling, and SMolESY-select aligns only assigned spin systems to a random ppm within their window to ease visual inspection.

## Related tools

- **SMolESY-select** (Performs the pattern search of spin-system windows against SMolESY data and emits per-bin assignments; applies the 11-point moving average filter within selected windows) — https://github.com/pantakis/SMolESY-select
- **SMolESY** (Pre-processes the 1D 1H NMR spectrum (macromolecular suppression, resolution enhancement) producing the data on which window matching runs) — https://github.com/pantakis/SMolESY
- **Bruker IVDr 600 MHz spectrometer** (Source instrument generating the calibrated 1D 1H NMR serum/plasma spectra that the module ingests)
- **STOCSY** (Used to check overlap of each spin system's SMolESY pattern with other SM signals and to guide bin definitions)

## Examples

```
from smolsesy_select import assign
result = assign(spectrum, bins='24_spin_systems.json', ppm_ref=5.233)
```

## Evaluation signals

- Every output row corresponds to one of the 24 predefined spin-system bins
- Assignment accuracy exceeds 99.5% across the 22 targeted metabolites (as reported on >8800 validation spectra)
- Detected ppm values fall inside the expected bin window width (at least 10 SMolESY Δv1/2 (>0.01 ppm) for broad bins, or 3–5 Δv1/2 for narrow singlet bins)
- SMolESY Δv1/2 of assigned signals lies within the 600 MHz reference range 0.0009–0.0011 ppm
- Denoised SMolESY signal-to-noise is at most ∼10% below CPMG spectra for the same windows

## Limitations

- Requires that the spectrum be automatically calibrated to the glucose anomeric proton signal at 5.233 ppm before pattern search
- Narrow singlet bins (glycine, creatinine, choline, DMSO2) are only 3–5 SMolESY Δv1/2 wide, making them vulnerable to misassignment when overlapping signals are strong
- Performance depends on the spin-system pattern library; metabolites outside the 22-target panel are not assigned
- The >8800-sample validation set and the specific software version are not named in the article

## Evidence

- [results] pattern search over the SMolESY-processed spectrum, matching each spin system pattern within predefined spectral windows (bins) to produce per-bin metabolite assignments: "the algorithm searches the SMolESY data for each spin system pattern within defined spectral windows (bins)"
- [results] moving average filter spanning 11-points applied within selected NMR spectral windows to help recover low abundance metabolite signals: "a moving average filter spanning 11-points (see the Supporting Information) across the high resolution data"
- [results] SMolESY Δv1/2 window widths for bin construction (broad vs. narrow singlet bins): [evidence span withheld — anchor status: not_found]
- [results] Narrow bins for correlated singlets sized 3–5 SMolESY Δv1/2: "with a maximum width of 3–5 SMolESY Δv1/2 (Figure 2c,d)"
- [abstract] assignment accuracy greater than 99.5% for all 22 metabolites: "yielding an assignment accuracy greater than 99.5% for all 22"
- [results] Calibration to glucose anomeric proton at 5.233 ppm: "to the anomeric proton of glucose (Figure 2a) resonating at 5.233 ppm"
- [results] Direct integration without deconvolution or baseline fitting: "enables the direct integration of the assigned spectral components, precluding the need for complex signal deconvolution or baseline fitting"
- [results] 600 MHz SMolESY linewidth reference range used as the pattern-shape reference: "0.0009 < SMolESY Δv1/2 < 0.0011 ppm for the 600 MHz NMR instrument"
- [results] Random ppm alignment within window aids visual misassignment detection: "aligns only each assigned spin system to a random ppm value within their corresponding predefined spectral window"
- [results] Cohort size for bin construction: "using a cohort of 4023 unique plasma/serum samples (3023 plasma and 1000 serum)"
