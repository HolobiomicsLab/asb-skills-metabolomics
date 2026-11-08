---
name: nmr-spectral-assignment
description: Use when analyzing 1D 1H NMR spectra of serum/plasma samples (e.g. IVDr
  600 MHz acquisition) to assign and quantify specific metabolite spin systems via
  spectral-window binning rather than full spectral deconvolution.
when_to_use_negative:
- When the input spectra are 2D experiments (e.g. 2D-Jres) or non-standard 1D 1H NMR
  acquisitions that do not match the IVDr serum/plasma protocol, as the predefined
  bins and SMolESY processing assumptions will not hold.
- When a full spectral deconvolution or baseline-fitting workflow is explicitly required,
  since this skill deliberately precludes those steps and cannot supply the fitted
  parameters.
- When the sample matrix is not serum/plasma (e.g. urine or tissue extracts), because
  the bin definitions were calibrated on plasma/serum cohorts and macromolecular suppression
  targets differ.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3520
tools:
- name: SMolESY-select
  role: Performs the automated assignment and integration of metabolite spin systems
    from SMolESY-processed 1D 1H NMR spectra using predefined spectral-window bins.
  repo: https://github.com/pantakis/SMolESY-select
- name: SMolESY
  role: Enhances spectral resolution and suppresses macromolecular signals in 1D 1H
    NMR data prior to assignment.
  repo: https://github.com/pantakis/SMolESY
- name: STOCSY
  role: Used to assess overlap of each spin system's SMolESY pattern with other SM
    signals during bin definition.
- name: 2D-Jres
  role: Supports validation of the 22 metabolite assignments.
- name: Bruker IVDr 600 MHz spectrometer
  role: Acquisition platform for the standard 1D 1H NMR serum/plasma spectra processed
    by this skill.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c00113
    title: A Computationally Lightweight Algorithm for Deriving Reliable Metabolite
      Panel Measurements from 1D <sup>1</sup>H NMR
  source_content_hash: sha256:5069749c25b0ace43247e58d0d90db9b46dd0895d1356a2015bf3c29cbe4f94f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pantakis/SMolESY-select
  tools:
  - 2D-Jres
  - Bruker IVDr 600 MHz spectrometer
  - SMolESY
  - SMolESY-select
  - STOCSY
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# nmr-spectral-assignment

## Summary

An automated method for assigning and integrating 1H NMR metabolite signals in serum/plasma spectra using SMolESY-derived features, achieving >99.5% assignment accuracy for a 22-metabolite panel without complex deconvolution or baseline fitting.

## When to use

Use when analyzing 1D 1H NMR spectra of serum/plasma samples (e.g. IVDr 600 MHz acquisition) to assign and quantify specific metabolite spin systems via spectral-window binning rather than full spectral deconvolution.

## When NOT to use

- When the input spectra are 2D experiments (e.g. 2D-Jres) or non-standard 1D 1H NMR acquisitions that do not match the IVDr serum/plasma protocol, as the predefined bins and SMolESY processing assumptions will not hold.
- When a full spectral deconvolution or baseline-fitting workflow is explicitly required, since this skill deliberately precludes those steps and cannot supply the fitted parameters.
- When the sample matrix is not serum/plasma (e.g. urine or tissue extracts), because the bin definitions were calibrated on plasma/serum cohorts and macromolecular suppression targets differ.

## Inputs

- 1D 1H NMR spectra of serum or plasma samples acquired on a Bruker IVDr 600 MHz spectrometer
- Predefined spectral windows (bins) for 24 1H NMR spin systems derived from a training cohort of 4023 plasma/serum samples

## Outputs

- Per-metabolite assignment and integration table for the 22 targeted metabolites (e.g. smolesy_select_assignments.csv)
- Random-ppm-aligned SMolESY spin-system peaks for visual detection of misassignment

## How to apply

Calibrate all spectra to the 1H NMR anomeric proton signal of glucose at 5.233 ppm. Apply SMolESY, which enhances resolution (Δv1/2 0.0009–0.0011 ppm at 600 MHz) and suppresses macromolecular signals, then apply a moving average filter spanning 11 points within selected spectral windows to boost s/n for tyrosine, phenylalanine, histidine, creatinine, and choline. Search the SMolESY data for each of 24 predefined spin-system patterns within spectral windows (bins) of at most 3–5 SMolESY Δv1/2 width, correlating singlet δ values with abundant metabolites (e.g. glycine, creatinine, choline, DMSO2 vs. lactic acid and alanine) to narrow the bins. Directly integrate assigned spectral components without complex deconvolution or baseline fitting; align each assigned spin system to a random ppm value within its predefined window to visually flag misassignments. Compile per-metabolite assignments into a single table (smolesy_select_assignments.csv).

## Related tools

- **SMolESY-select** (Performs the automated assignment and integration of metabolite spin systems from SMolESY-processed 1D 1H NMR spectra using predefined spectral-window bins.) — https://github.com/pantakis/SMolESY-select
- **SMolESY** (Enhances spectral resolution and suppresses macromolecular signals in 1D 1H NMR data prior to assignment.) — https://github.com/pantakis/SMolESY
- **STOCSY** (Used to assess overlap of each spin system's SMolESY pattern with other SM signals during bin definition.)
- **2D-Jres** (Supports validation of the 22 metabolite assignments.)
- **Bruker IVDr 600 MHz spectrometer** (Acquisition platform for the standard 1D 1H NMR serum/plasma spectra processed by this skill.)

## Examples

```
python smolesy_select.py --input spectra/ --calibration glucose_5.233ppm --bins predefined_24_bins.json --output smolesy_select_assignments.csv
```

## Evaluation signals

- Assignment accuracy for the 22 targeted metabolites must exceed 99.5% on the validation cohort.
- The false positive / nonassigned signal rate must remain at most 0.45% (0–40 misassigned signals per 8834 spectra).
- SMolESY Δv1/2 should lie in the range 0.0009–0.0011 ppm for a 600 MHz NMR instrument.
- Denoised SMolESY signals should retain s/n at most ∼10% less than that of CPMG spectra.
- Bins defined via singlet correlation should have a maximum width of 3–5 SMolESY Δv1/2.

## Limitations

- The validation dataset of >8800 serum/plasma samples is not named or given an accession/repository beyond MTBLS395 in the extracted context.
- No software version is stated for SMolESY-select or SMolESY in the provided sections.
- The GUI and usage video exist only as supplementary artifacts (Figure S9, Video S1) not included in this context.
- The comparison to fitting and deconvolution approaches cites only qualitative timing (several minutes per spectral set) without naming specific competing software.

## Evidence

- [results] Moving average filter: "a moving average filter spanning 11-points (see the Supporting Information) across the high resolution data"
- [abstract] Assignment accuracy criterion: "yielding an assignment accuracy greater than 99.5% for all 22"
- [results] Calibration reference: "to the anomeric proton of glucose (Figure 2a) resonating at 5.233 ppm"
- [results] Bin width specification: "with a maximum width of 3–5 SMolESY Δv1/2 (Figure 2c,d)"
- [abstract] SMolESY purpose: "Small Molecule Enhancement SpectroscopY (SMolESY) was employed to develop a unique and fully automated computational solution for the assignment and integration of 1H nuclear magnetic resonance (NMR)"
- [results] Training cohort size: "using a cohort of 4023 unique plasma/serum samples (3023 plasma and 1000 serum)"
- [results] Validation cohort and dataset: "one serum cohort from the Metabolights repository (https://www.ebi.ac.uk/metabolights) (MTBLS395)"
