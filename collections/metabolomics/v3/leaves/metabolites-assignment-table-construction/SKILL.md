---
name: metabolites-assignment-table-construction
description: Use when the goal is to convert a cohort of raw 1D 1H NMR serum/plasma
  spectra (e.g. Bruker IVDr 600 MHz) into an automated, per-sample quantitative table
  for a predefined panel of ~22 abundant metabolites.
when_to_use_negative:
- You need quantitation of metabolites outside the 22-metabolite panel targeted by
  the algorithm's defined bins
- Input spectra are not standard 1D 1H NMR (e.g. only 2D-Jres or CPMG-only data, or
  spectra lacking the glucose anomeric proton needed for 5.233 ppm calibration)
- The task requires deconvolving heavily overlapped regions or identifying unknown/unassigned
  signals rather than quantifying the predefined spin-system panel
edam_operation: http://edamontology.org/operation_3799
tools:
- name: SMolESY-select
  role: Runs the automated assignment of the 24 spin-system patterns and direct integration
    of the 22 metabolites; has a GUI (Figure S9, Video S1) requiring minimal prerequisite
    knowledge
  repo: https://github.com/pantakis/SMolESY-select
- name: SMolESY
  role: Transforms the raw 1D 1H NMR data with baseline suppression and resolution
    enhancement so the assigned components can be directly integrated without deconvolution
  repo: https://github.com/pantakis/SMolESY
- name: STOCSY
  role: Used to check overlap of each spin system's SMolESY pattern with other signals
    during assignment
- name: 2D-Jres
  role: Used alongside STOCSY for pattern/overlap checks for the 22 metabolites
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c00113
    title: A Computationally Lightweight Algorithm for Deriving Reliable Metabolite
      Panel Measurements from 1D <sup>1</sup>H NMR
  source_content_hash: sha256:9cca0168ee6c3c5227e9a5c2b4050e0bdf3bfe8d0e65fa8450ef6a8847664028
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pantakis/SMolESY-select
  tools:
  - 2D-Jres
  - SMolESY
  - SMolESY-select
  - STOCSY
  edam_operation: http://edamontology.org/operation_3799
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# metabolites-assignment-table-construction

## Summary

Builds a quantitative 22-metabolite × samples table from standard 1D 1H NMR blood serum/plasma spectra by automatically assigning 24 spin-system patterns in SMolESY-transformed data and directly integrating each assigned component, with no deconvolution or baseline fitting.

## When to use

Use when the goal is to convert a cohort of raw 1D 1H NMR serum/plasma spectra (e.g. Bruker IVDr 600 MHz) into an automated, per-sample quantitative table for a predefined panel of ~22 abundant metabolites.

## When NOT to use

- You need quantitation of metabolites outside the 22-metabolite panel targeted by the algorithm's defined bins
- Input spectra are not standard 1D 1H NMR (e.g. only 2D-Jres or CPMG-only data, or spectra lacking the glucose anomeric proton needed for 5.233 ppm calibration)
- The task requires deconvolving heavily overlapped regions or identifying unknown/unassigned signals rather than quantifying the predefined spin-system panel

## Inputs

- Raw 1D 1H NMR blood serum/plasma spectra (e.g. MTBLS395 serum cohort from the Metabolights repository, acquired on Bruker IVDr 600 MHz spectrometers)
- Predefined bin definitions for the 24 1H NMR spin systems (SMolESY Δv1/2 window widths)

## Outputs

- smolesy_select_integrals.csv — per-metabolite direct-integration values for all 22 target metabolites × all samples
- Per-spectrum spin-system assignments (each assigned spin system aligned to a random ppm value within its predefined spectral window to ease visual detection of misassignment)

## How to apply

First automatically calibrate every spectrum so the glucose anomeric proton aligns at 5.233 ppm. Transform the spectra with SMolESY (resolution enhancement plus macromolecular/baseline suppression) and, within the predefined bins for the 24 1H NMR spin systems (≥10 Δv1/2 wide, >0.01 ppm, for glycine, creatine/creatinine, choline, DMSO2, acetone, acetic acid; 3–5 Δv1/2 for singlet bins of glycine, creatinine, choline, DMSO2 narrowed by correlation with lactic acid and alanine), apply an 11-point moving average filter to raise s/n of low-abundance signals (tyrosine, phenylalanine, histidine, creatinine, choline). Search the smoothed SMolESY data for each metabolite's spin-system pattern inside its bin, using STOCSY and 2D-Jres to check overlap and confirm assignments. For every assigned spin system, perform direct integration — SMolESY's baseline suppression and resolution enhancement make deconvolution and baseline fitting unnecessary — and record the integral per sample. Assemble the per-metabolite integrals into a table (e.g. smolesy_select_integrals.csv) for all 22 target metabolites across the cohort.

## Related tools

- **SMolESY-select** (Runs the automated assignment of the 24 spin-system patterns and direct integration of the 22 metabolites; has a GUI (Figure S9, Video S1) requiring minimal prerequisite knowledge) — https://github.com/pantakis/SMolESY-select
- **SMolESY** (Transforms the raw 1D 1H NMR data with baseline suppression and resolution enhancement so the assigned components can be directly integrated without deconvolution) — https://github.com/pantakis/SMolESY
- **STOCSY** (Used to check overlap of each spin system's SMolESY pattern with other signals during assignment)
- **2D-Jres** (Used alongside STOCSY for pattern/overlap checks for the 22 metabolites)

## Evaluation signals

- The pipeline runs end-to-end on all spectra with no manual intervention
- Output table contains exactly 22 metabolite rows × all samples
- Every spectrum is calibrated so the glucose anomeric proton sits at 5.233 ppm
- Spot-checked assignments are consistent with the paper's design (≥99.5% assignment accuracy on its own validation cohort is the reference benchmark)
- Assignments are visually inspectable because each assigned spin system is aligned to a random ppm value within its predefined spectral window

## Limitations

- No software version is stated for SMolESY-select or SMolESY
- The >8800-sample validation dataset is not named or given an accession/repository
- The GUI and usage video exist only as supplementary artifacts (Figure S9, Video S1)
- Bins were calibrated on 4023 plasma/serum samples (3023 plasma, 1000 serum); moving-average smoothing raises s/n at a cost — denoised SMolESY signals have s/n at most ~10% less than CPMG spectra

## Evidence

- [abstract] The approach is highly efficient, requiring only standard one-dimensional 1H NMR spectra and avoiding the need for sample preprocessing, complex deconvolution, and spectral baseline fitting.: "The approach is highly efficient, requiring only standard one-dimensional 1H NMR spectra and avoiding the need for sample preprocessing, complex deconvolution, and spectral baseline fitting."
- [results] Calibration of all spectra to the 1H NMR anomeric proton signal of glucose at 5.233 ppm: [evidence span withheld — anchor status: not_found]
- [results] Bins were defined for 24 1H NMR spin systems using a cohort of 4023 unique plasma/serum samples (3023 plasma and 1000 serum): [evidence span withheld — anchor status: not_found]
- [results] enables the direct integration of the assigned spectral components, precluding the need for complex signal deconvolution or baseline fitting: "enables the direct integration of the assigned spectral components, precluding the need for complex signal deconvolution or baseline fitting"
- [abstract] yielding an assignment accuracy greater than 99.5% for all: "yielding an assignment accuracy greater than 99.5% for all"
