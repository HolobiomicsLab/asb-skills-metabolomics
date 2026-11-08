---
name: direct-peak-integration-assigned-components
description: Use when raw 1D 1H NMR serum/plasma spectra (e.g. Bruker IVDr 600 MHz)
  must be converted into a per-metabolite concentration/integral table without fitting
  or deconvolution, relying on SMolESY preprocessing and predefined spin-system bins.
when_to_use_negative:
- Input is already a binned/integrated feature table or quantified metabolite panel
  — no spectral assignment or integration is needed.
- The spectra are 2D (2D-Jres) or non-standard 1D acquisitions; SMolESY-select requires
  standard one-dimensional 1H NMR spectra.
- A problem demands full spectral deconvolution or baseline fitting of severely overlapped
  regions that fall outside the predefined 24 spin-system bins.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: SMolESY-select
  role: Performs the automated assignment of spin-system patterns within bins and
    direct integration of the assigned components into the per-metabolite integral
    table; applies the 11-point moving-average smoothing filter within selected windows.
  repo: https://github.com/pantakis/SMolESY-select
- name: SMolESY
  role: Upstream preprocessing providing baseline suppression and resolution enhancement
    so assigned components can be integrated directly without deconvolution or baseline
    fitting.
  repo: https://github.com/pantakis/SMolESY
- name: STOCSY
  role: Used to check overlap of each spin system's SMolESY pattern with other signals
    during the assignment step preceding integration.
- name: 2D-Jres
  role: Auxiliary analysis used together with STOCSY to verify spin-system assignments
    for the 22 metabolites before integration.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c00113
    title: A Computationally Lightweight Algorithm for Deriving Reliable Metabolite
      Panel Measurements from 1D <sup>1</sup>H NMR
  source_content_hash: sha256:6d734d6e09caad592a44919c75b589b6b9d0c2619b500e6f7639e21298db5775
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pantakis/SMolESY-select
  tools:
  - 2D-Jres
  - SMolESY
  - SMolESY-select
  - STOCSY
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# Direct peak integration of assigned spectral components

## Summary

Quantitative metabolite measurement from 1D 1H NMR spectra by assigning spin-system patterns inside predefined SMolESY bins and then directly integrating the assigned components, precluding complex deconvolution and spectral baseline fitting (the SMolESY-select quantitation step, targeting 22 plasma/serum metabolites).

## When to use

Use when raw 1D 1H NMR serum/plasma spectra (e.g. Bruker IVDr 600 MHz) must be converted into a per-metabolite concentration/integral table without fitting or deconvolution, relying on SMolESY preprocessing and predefined spin-system bins.

## When NOT to use

- Input is already a binned/integrated feature table or quantified metabolite panel — no spectral assignment or integration is needed.
- The spectra are 2D (2D-Jres) or non-standard 1D acquisitions; SMolESY-select requires standard one-dimensional 1H NMR spectra.
- A problem demands full spectral deconvolution or baseline fitting of severely overlapped regions that fall outside the predefined 24 spin-system bins.

## Inputs

- Raw 1D 1H NMR spectra of serum/plasma acquired on a Bruker IVDr 600 MHz spectrometer (e.g. MTBLS395 serum cohort from the MetaboLights repository)
- Predefined spectral windows (bins) for 24 1H NMR spin systems expressed in SMolESY Δv1/2 units (Δv1/2 ≈ 0.0009–0.0011 ppm at 600 MHz)
- Auxiliary STOCSY and 2D-Jres analyses for pattern/overlap verification during assignment

## Outputs

- CSV table of direct-integral values per metabolite and per sample for the 22 targeted metabolites (e.g. smolesy_select_integrals.csv)
- Assignment record mapping each of the 24 spin systems to a metabolite per spectrum (with random-ppm alignment of assigned systems to ease visual detection of misassignment)

## How to apply

First automatically calibrate every spectrum so the anomeric proton of glucose resonates at 5.233 ppm. Run SMolESY preprocessing — its baseline suppression and resolution enhancement give SMolESY Δv1/2 of 0.0009–0.0011 ppm on a 600 MHz instrument — then search the SMolESY data for each spin system's pattern inside its predefined bin (bins for glycine, creatine/creatinine, choline, DMSO2, acetone and acetic acid span at least 10 Δv1/2 (>0.01 ppm); narrow bins for correlating singlets of glycine, creatinine, choline and DMSO2 vs lactic acid/alanine are 3–5 Δv1/2 wide), cross-checking overlap with STOCSY and 2D-Jres analyses. Within selected spectral windows, apply an 11-point moving-average smoothing filter across the high-resolution data to raise s/n for low-abundance metabolites (tyrosine, phenylalanine, histidine, creatinine, choline) before assignment. For every assigned spin system, perform direct integration of the spectral component — no complex signal deconvolution and no baseline fitting — because SMolESY's macromolecular suppression and resolution enhancement leave the assigned components cleanly separated enough to integrate as-is. Record each integral per sample and assemble the per-metabolite integrals for the 22 target metabolites into a table (e.g. smolesy_select_integrals.csv).

## Related tools

- **SMolESY-select** (Performs the automated assignment of spin-system patterns within bins and direct integration of the assigned components into the per-metabolite integral table; applies the 11-point moving-average smoothing filter within selected windows.) — https://github.com/pantakis/SMolESY-select
- **SMolESY** (Upstream preprocessing providing baseline suppression and resolution enhancement so assigned components can be integrated directly without deconvolution or baseline fitting.) — https://github.com/pantakis/SMolESY
- **STOCSY** (Used to check overlap of each spin system's SMolESY pattern with other signals during the assignment step preceding integration.)
- **2D-Jres** (Auxiliary analysis used together with STOCSY to verify spin-system assignments for the 22 metabolites before integration.)

## Evaluation signals

- Output table contains exactly 22 metabolite rows × all input samples, with finite positive integral values.
- All spectra were auto-calibrated so the glucose anomeric proton sits at 5.233 ppm before assignment/integration.
- Each assigned spin system falls inside its predefined bin width (≥10 SMolESY Δv1/2 for wide bins; 3–5 Δv1/2 for narrow correlating-singlet bins).
- Spot-check assignments against the paper's stated benchmark: assignment accuracy >99.5% for all 22 metabolites on the validation cohort (>8800 serum/plasma spectra).
- Low-abundance metabolite signals (tyrosine, phenylalanine, histidine, creatinine, choline) are recoverable after the 11-point moving-average filter, with s/n no more than ~10% below CPMG spectra.

## Limitations

- No software version is stated for SMolESY-select or SMolESY, and the >8800-sample validation dataset is not given an accession/repository.
- SMolESY signal s/n is at most ~10% lower than CPMG spectra, so very low-abundance signals may remain marginal even after 11-point smoothing.
- Quantitation relies on bins predefined for 24 spin systems and 22 metabolites; signals outside these windows are not handled.
- Operation is supported via a GUI (Figure S9, Video S1) provided only as supplementary artifacts, and comparison to fitting/deconvolution software is only qualitative (several minutes of computation per spectral set).

## Evidence

- [results] Direct integration of assigned spectral components without complex signal deconvolution or baseline fitting: "enables the direct integration of the assigned spectral components, precluding the need for complex signal deconvolution or baseline fitting"
- [abstract] 1D 1H NMR only, no preprocessing/deconvolution/baseline fitting: "requiring only standard one-dimensional 1H NMR spectra and avoiding the need for sample preprocessing, complex deconvolution, and spectral baseline fitting"
- [results] Calibration to glucose anomeric proton at 5.233 ppm: "to the anomeric proton of glucose (Figure 2a) resonating at 5.233 ppm"
- [results] 11-point moving-average filter and s/n recovery for low-abundance metabolites: "increases the s/n of signals belonging to tyrosine, phenylalanine, histidine, creatinine, and choline"
- [abstract] >99.5% assignment accuracy benchmark on >8800 validation spectra: "yielding an assignment accuracy greater than 99.5% for all 22"
