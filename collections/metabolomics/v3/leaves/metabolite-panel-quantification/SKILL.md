---
name: metabolite-panel-quantification
description: Use when the input is standard 1D 1H NMR serum/plasma spectra (e.g.,
  Bruker IVDr 600 MHz) and a targeted panel of metabolite concentrations (22 spin
  systems) is needed without manual peak fitting.
when_to_use_negative:
- Input spectra are 2D (e.g., 2D-Jres) or non-IVDr acquisitions requiring spectral
  fitting — the method targets standard 1D 1H NMR only
- The analysis requires discovery of metabolites outside the 22 predefined spin systems,
  since bins are fixed from the 4023-sample cohort
- Baseline fitting or complex deconvolution of overlapping multiplets is explicitly
  desired — SMolESY-select deliberately precludes these steps
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3292
tools:
- name: SMolESY-select
  role: Core algorithm that assigns 22 metabolite spin systems to predefined bins
    and directly integrates them from 1D 1H NMR spectra
  repo: https://github.com/pantakis/SMolESY-select
- name: SMolESY
  role: Upstream signal processing providing baseline (macromolecular) suppression
    and spectral resolution enhancement before bin search
  repo: https://github.com/pantakis/SMolESY
- name: STOCSY
  role: Used to assess overlap of each spin system's SMolESY pattern with other SM
    signals during validation of assignments
- name: Bruker IVDr 600 MHz spectrometer
  role: Instrument/standard for acquiring the 1D 1H NMR serum/plasma spectra the algorithm
    expects
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c00113
    title: A Computationally Lightweight Algorithm for Deriving Reliable Metabolite
      Panel Measurements from 1D <sup>1</sup>H NMR
  source_content_hash: sha256:63e9147cc9d22cf7d8b0bd2290809df1d3df3bb7c57d7d9a316f639bbdb2b212
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pantakis/SMolESY-select
  tools:
  - Bruker IVDr 600 MHz spectrometer
  - SMolESY
  - SMolESY-select
  - STOCSY
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7222222222222222
n_claims: 18
---

# metabolite-panel-quantification

## Summary

Quantifies a panel of 22 serum/plasma metabolites from standard 1D 1H NMR spectra using SMolESY-select, a fully automated algorithm that assigns spin systems to predefined bins and integrates them directly, avoiding deconvolution and baseline fitting.

## When to use

Use when the input is standard 1D 1H NMR serum/plasma spectra (e.g., Bruker IVDr 600 MHz) and a targeted panel of metabolite concentrations (22 spin systems) is needed without manual peak fitting.

## When NOT to use

- Input spectra are 2D (e.g., 2D-Jres) or non-IVDr acquisitions requiring spectral fitting — the method targets standard 1D 1H NMR only
- The analysis requires discovery of metabolites outside the 22 predefined spin systems, since bins are fixed from the 4023-sample cohort
- Baseline fitting or complex deconvolution of overlapping multiplets is explicitly desired — SMolESY-select deliberately precludes these steps

## Inputs

- Standard 1D 1H NMR serum or plasma spectra (Bruker IVDr, 600 MHz)
- Metabolights repository cohort, e.g., MTBLS395 serum spectra
- Predefined spectral window (bin) definitions for 24 1H NMR spin systems

## Outputs

- Relative concentration values for the 22 targeted metabolite spin systems per spectrum
- Assigned spin-system positions within predefined spectral windows (randomized within-bin alignment for QC)
- Per-spectrum runtime suitable for benchmarking (<15 s per spectrum)

## How to apply

Acquire or obtain 1D 1H NMR spectra recorded under Bruker IVDr conditions at 600 MHz, then automatically calibrate every spectrum to the glucose anomeric proton signal at 5.233 ppm. Run SMolESY-select, which first enhances resolution and suppresses macromolecular signals via SMolESY, then searches the SMolESY data for each of 22 spin-system patterns within predefined spectral windows (bins) defined from a 4023-sample plasma/serum cohort. For singlets such as glycine, creatinine, choline, and DMSO2, correlate their δ values with abundant metabolite signals (e.g., lactic acid, alanine) to narrow bins to 3–5 SMolESY Δv1/2; apply an 11-point moving average filter within selected windows to recover low-abundance signals (tyrosine, phenylalanine, histidine, creatinine, choline). The algorithm aligns each assigned spin system to a random ppm value inside its window so misassignments are visually detectable, then directly integrates assigned components — no complex deconvolution or baseline fitting is required. Judge success by assignment accuracy (>99.5% for all 22 metabolites was achieved) and computational speed (<15 s per spectrum on a conventional laptop).

## Related tools

- **SMolESY-select** (Core algorithm that assigns 22 metabolite spin systems to predefined bins and directly integrates them from 1D 1H NMR spectra) — https://github.com/pantakis/SMolESY-select
- **SMolESY** (Upstream signal processing providing baseline (macromolecular) suppression and spectral resolution enhancement before bin search) — https://github.com/pantakis/SMolESY
- **STOCSY** (Used to assess overlap of each spin system's SMolESY pattern with other SM signals during validation of assignments)
- **Bruker IVDr 600 MHz spectrometer** (Instrument/standard for acquiring the 1D 1H NMR serum/plasma spectra the algorithm expects)

## Evaluation signals

- Assignment accuracy across the 22 metabolites exceeds 99.5%, matching the article's benchmark
- Mean per-spectrum runtime is below 15 s on a conventional laptop (validated on the external MTBLS395 serum cohort)
- All spectra are automatically calibrated to the glucose anomeric proton at 5.233 ppm before assignment
- Each assigned spin system lies within its predefined spectral window; the random within-window alignment makes misassignments visually detectable
- Signal-to-noise of denoised SMolESY signals is at most ~10% less than CPMG spectra, with Δv1/2 of 0.0009–0.0011 ppm at 600 MHz

## Limitations

- No software version is stated for SMolESY-select or SMolESY, complicating reproducibility
- The >8800-spectrum validation cohort is not named or given an accession/repository; only MTBLS395 is identified
- The GUI and usage video (Figure S9, Video S1) exist only as supplementary artifacts, and the comparison to fitting/deconvolution approaches cites only qualitative timing without naming competing software
- Bins for some metabolites (glycine, creatine/creatinine, choline, DMSO2, acetone, acetic acid) span ≥10 SMolESY Δv1/2 (>0.01 ppm), so overlap-driven narrowing relies on correlating singlets with abundant metabolites

## Evidence

- [abstract] The approach is highly efficient, requiring only standard one-dimensional 1H NMR spectra and avoiding the need for sample preprocessing, complex deconvolution, and spectral baseline fitting.: "The approach is highly efficient, requiring only standard one-dimensional 1H NMR spectra and avoiding the need for sample preprocessing, complex deconvolution, and spectral baseline fitting."
- [abstract] developed using >4000 NMR serum and plasma spectra, was evaluated using an additional >8800 spectra, yielding an assignment accuracy greater than 99.5% for all 22: "developed using >4000 NMR serum and plasma spectra, was evaluated using an additional >8800 spectra, yielding an assignment accuracy greater than 99.5% for all 22"
- [results] a moving average filter spanning 11-points (see the Supporting Information) across the high resolution data of the selected windows: "a moving average filter spanning 11-points (see the Supporting Information) across the high resolution data of the selected windows"
- [results] enables the direct integration of the assigned spectral components, precluding the need for complex signal deconvolution or baseline fitting: "enables the direct integration of the assigned spectral components, precluding the need for complex signal deconvolution or baseline fitting"
- [results] aligns only each assigned spin system to a random ppm value within their corresponding predefined spectral window: "aligns only each assigned spin system to a random ppm value within their corresponding predefined spectral window"
- [discussion] It is freely available to download at https://github.com/pantakis/SMolESY-select: "it is freely available to download at https://github.com/pantakis/SMolESY-select"
