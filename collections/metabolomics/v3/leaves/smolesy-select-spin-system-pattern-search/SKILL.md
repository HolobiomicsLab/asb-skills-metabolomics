---
name: smolesy-select-spin-system-pattern-search
description: Use when you have standard 1D 1H NMR spectra of serum/plasma from a Bruker
  IVDr 600 MHz instrument and need fast, reliable assignment and integration of a
  small targeted metabolite panel (e.g. 22 metabolites) without sample preprocessing,
  deconvolution, or baseline fitting.
when_to_use_negative:
- Input spectra are 2D NMR or not 1D 1H NMR (e.g. 2D-Jres is used only for validation,
  not as the primary input).
- The analysis requires full untargeted metabolite profiling or deconvolution of heavily
  overlapped low-abundance signals beyond the 22 targeted spin systems; SMolESY-select
  is designed for a targeted panel with predefined bins.
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3272
tools:
- name: SMolESY-select
  role: Core algorithm that searches SMolESY spectra for spin-system patterns within
    predefined bins, applies an 11-point moving average filter in selected windows,
    and integrates assigned spectral components.
  repo: https://github.com/pantakis/SMolESY-select
- name: SMolESY
  role: Preprocessing step that provides spectral resolution enhancement and macromolecular
    signal suppression to produce the SMolESY spectra searched by SMolESY-select.
  repo: https://github.com/pantakis/SMolESY
- name: STOCSY
  role: Used to characterise overlap of each spin system’s SMolESY pattern with other
    serum metabolite signals, informing bin definition.
- name: 2D-Jres
  role: Used alongside STOCSY for validation of the 22 metabolite assignments.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c00113
    title: A Computationally Lightweight Algorithm for Deriving Reliable Metabolite
      Panel Measurements from 1D <sup>1</sup>H NMR
  source_content_hash: sha256:3c847fdf91c39d7fa3a3dcf9a6dd29c8ffd84582657e577a4450c6a00458efc3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pantakis/SMolESY-select
  tools:
  - 2D-Jres
  - SMolESY
  - SMolESY-select
  - STOCSY
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# smolesy-select-spin-system-pattern-search

## Summary

SMolESY-select is a fully automated, computationally lightweight algorithm that assigns and integrates 22 targeted metabolites in 1D 1H NMR spectra of serum/plasma by searching SMolESY-enhanced spectra for each spin-system pattern within predefined spectral windows (bins), avoiding complex deconvolution or baseline fitting.

## When to use

Use when you have standard 1D 1H NMR spectra of serum/plasma from a Bruker IVDr 600 MHz instrument and need fast, reliable assignment and integration of a small targeted metabolite panel (e.g. 22 metabolites) without sample preprocessing, deconvolution, or baseline fitting.

## When NOT to use

- Input spectra are 2D NMR or not 1D 1H NMR (e.g. 2D-Jres is used only for validation, not as the primary input).
- The analysis requires full untargeted metabolite profiling or deconvolution of heavily overlapped low-abundance signals beyond the 22 targeted spin systems; SMolESY-select is designed for a targeted panel with predefined bins.

## Inputs

- 1D 1H NMR spectra of blood serum or plasma (e.g. MTBLS395 cohort from MetaboLights)
- Predefined spectral windows (bins) for 24 1H NMR spin systems
- Calibration reference: glucose anomeric proton at 5.233 ppm

## Outputs

- Per-metabolite assignment results for the 22 targeted metabolites (saved as smolesy_select_assignments.csv)
- Integrated concentration values for each assigned spectral component
- Visual alignment of assigned spin systems to random ppm values within their predefined spectral windows to highlight misassignment

## How to apply

Load the 1D 1H NMR spectra and automatically calibrate each spectrum to the glucose anomeric proton signal at 5.233 ppm. Apply the SMolESY-select algorithm: it first searches the SMolESY data for each spin-system pattern within predefined spectral windows (bins) defined for 24 1H NMR spin systems, using a maximum bin width of 3–5 SMolESY Δv1/2. To increase signal-to-noise of low-abundance metabolites such as tyrosine, phenylalanine, histidine, creatinine, and choline, a moving average filter spanning 11 points is applied across the high-resolution data of the selected windows. Correlate the δ values of singlets with signals from the most abundant and frequently occurring serum/plasma metabolites (e.g. lactic acid and alanine) to narrow the bins, then directly integrate the assigned spectral components without deconvolution or baseline fitting. To ease visual detection of misassignment, the algorithm aligns each assigned spin system to a random ppm value within its corresponding predefined spectral window.

## Related tools

- **SMolESY-select** (Core algorithm that searches SMolESY spectra for spin-system patterns within predefined bins, applies an 11-point moving average filter in selected windows, and integrates assigned spectral components.) — https://github.com/pantakis/SMolESY-select
- **SMolESY** (Preprocessing step that provides spectral resolution enhancement and macromolecular signal suppression to produce the SMolESY spectra searched by SMolESY-select.) — https://github.com/pantakis/SMolESY
- **STOCSY** (Used to characterise overlap of each spin system’s SMolESY pattern with other serum metabolite signals, informing bin definition.)
- **2D-Jres** (Used alongside STOCSY for validation of the 22 metabolite assignments.)

## Examples

```
python smolesy_select.py --input MTBLS395_spectra/ --calibration 5.233 --output smolesy_select_assignments.csv
```

## Evaluation signals

- Assignment accuracy across the 22 targeted metabolites should exceed 99.5%, with a false-positive/nonassigned signal rate of at most 0.45% (0–40 signals out of 8834 spectra).
- All spectra must be calibrated such that the glucose anomeric proton sits at 5.233 ppm before the pattern search.
- The moving average filter is only applied within the predefined selected windows for the 11-point span; applying it outside those windows is a misuse.
- Output table (smolesy_select_assignments.csv) must contain per-metabolite assignment results for all 22 targeted metabolites and align assigned spin systems within their predefined spectral windows.
- SMolESY Δv1/2 should remain in the range 0.0009–0.0011 ppm for a 600 MHz NMR instrument, confirming resolution enhancement was applied correctly.

## Limitations

- No software version is stated for SMolESY-select or SMolESY, complicating reproducibility of exact results.
- The validation dataset of >8800 serum/plasma samples is not named or given an accession/repository in the article text.
- The GUI and usage video exist only as supplementary artifacts (Figure S9, Video S1) and are not bundled in the main text; obtaining them requires accessing the SI.
- Comparison to fitting and deconvolution approaches cites only qualitative timing (several minutes) without naming the specific competing software.

## Evidence

- [results] Calibration to glucose anomeric proton: "to the anomeric proton of glucose (Figure 2a) resonating at 5.233 ppm"
- [results] Bin definition for 24 spin systems: "using a cohort of 4023 unique plasma/serum samples (3023 plasma and 1000 serum)"
- [results] 11-point moving average filter in selected windows: "a moving average filter spanning 11-points (see the Supporting Information) across the high resolution data"
- [results] Direct integration without deconvolution: "enables the direct integration of the assigned spectral components, precluding the need for complex signal deconvolution or baseline fitting"
- [abstract] Accuracy band on >8800 spectra: "yielding an assignment accuracy greater than 99.5% for all 22"
