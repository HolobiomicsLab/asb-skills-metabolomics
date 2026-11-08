---
name: nmr-spin-system-pattern-search
description: Use when you need to assign metabolite signals in calibrated 1D 1H NMR
  serum/plasma spectra to a curated metabolite panel by matching spin-system patterns
  inside spectral windows derived from a large cohort.
when_to_use_negative:
- The spectrum is a 2D NMR acquisition (e.g. 2D-Jres) rather than the standard one-dimensional
  1H NMR data the algorithm requires
- The spectrum has not been baseline-corrected by SMolESY or calibrated to the glucose
  anomeric proton at 5.233 ppm — the pattern search operates on SMolESY-processed
  data
- The metabolite of interest lies outside the 22-metabolite panel for which bins were
  defined using the 4023-sample cohort
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: SMolESY-select
  role: Executes the per-bin pattern search over SMolESY-processed spectra, using
    STOCSY and 2D-Jres information to assign the 22 metabolites' spin systems
  repo: https://github.com/pantakis/SMolESY-select
- name: SMolESY
  role: Produces the baseline-suppressed, resolution-enhanced spectra whose SMolESY
    Δv1/2 range (0.0009–0.0011 ppm at 600 MHz) defines the expected pattern shape
    for matching
  repo: https://github.com/pantakis/SMolESY
- name: STOCSY
  role: Used to check overlap of each spin system's SMolESY pattern with other SM
    signals during the assignment search
- name: 2D-Jres
  role: Used alongside STOCSY for pattern/overlap checks on the 22 metabolites
- name: Bruker IVDr 600 MHz spectrometer
  role: Acquires the 1D 1H NMR data that the pattern search consumes
provenance:
  source_task_ids:
  - task_007
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c00113
    title: A Computationally Lightweight Algorithm for Deriving Reliable Metabolite
      Panel Measurements from 1D <sup>1</sup>H NMR
  source_content_hash: sha256:bac6007881bb2eb52ecbe6c6023050af0e919625c1dbfd64646ab831dfd869b6
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
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# nmr-spin-system-pattern-search

## Summary

Searches SMolESY-processed 1D 1H NMR blood serum/plasma spectra for each metabolite spin system's pattern within predefined spectral windows (bins), producing per-bin assignments that feed direct integration without deconvolution or baseline fitting.

## When to use

Use when you need to assign metabolite signals in calibrated 1D 1H NMR serum/plasma spectra to a curated metabolite panel by matching spin-system patterns inside spectral windows derived from a large cohort.

## When NOT to use

- The spectrum is a 2D NMR acquisition (e.g. 2D-Jres) rather than the standard one-dimensional 1H NMR data the algorithm requires
- The spectrum has not been baseline-corrected by SMolESY or calibrated to the glucose anomeric proton at 5.233 ppm — the pattern search operates on SMolESY-processed data
- The metabolite of interest lies outside the 22-metabolite panel for which bins were defined using the 4023-sample cohort

## Inputs

- Calibrated 1D 1H NMR spectra of blood serum or plasma (e.g. the MTBLS395 serum cohort from the Metabolights repository)
- SMolESY-processed spectra with baseline suppression and resolution enhancement (SMolESY Δv1/2 range 0.0009–0.0011 ppm at 600 MHz)
- Predefined spectral-window (bin) definitions for the 24 target 1H NMR spin systems, derived from a cohort of 4023 unique plasma/serum samples (3023 plasma and 1000 serum)

## Outputs

- One row per spin-system bin (24 rows) with bin identifier, metabolite signal, assignment status, and detected δ value (e.g. assignments.csv)
- Per-metabolite integrals for the 22 target metabolites, obtained by direct integration of assigned spectral components (e.g. smolesy_select_integrals.csv)

## How to apply

First, automatically calibrate every spectrum so the glucose anomeric proton aligns at 5.233 ppm. Then, for each of the 24 predefined 1H NMR spin-system bins — e.g. bins for glycine, creatine/creatinine, choline, DMSO2, acetone, and acetic acid spanning at least 10 SMolESY Δv1/2 (>0.01 ppm), and narrower 3–5 Δv1/2 windows for singlets correlated with lactic acid/alanine — extract the corresponding segment from the SMolESY spectrum (using the Δv1/2 range 0.0009–0.0011 ppm at 600 MHz as the pattern-shape reference). Within each window, apply an 11-point moving-average smoothing filter to raise the s/n of low-abundance metabolites (tyrosine, phenylalanine, histidine, creatinine, choline), then search the smoothed data for each spin system's expected multiplet pattern, using STOCSY and 2D-Jres analyses for the 22 metabolites to resolve overlap. Record each assignment (the algorithm aligns only assigned spin systems to a random ppm value within their predefined spectral window to ease visual detection of misassignment) and emit one row per bin with bin identifier, metabolite signal, assignment status, and detected δ value; assignment accuracy greater than 99.5% is the reported benchmark across the validated cohort.

## Related tools

- **SMolESY-select** (Executes the per-bin pattern search over SMolESY-processed spectra, using STOCSY and 2D-Jres information to assign the 22 metabolites' spin systems) — https://github.com/pantakis/SMolESY-select
- **SMolESY** (Produces the baseline-suppressed, resolution-enhanced spectra whose SMolESY Δv1/2 range (0.0009–0.0011 ppm at 600 MHz) defines the expected pattern shape for matching) — https://github.com/pantakis/SMolESY
- **STOCSY** (Used to check overlap of each spin system's SMolESY pattern with other SM signals during the assignment search)
- **2D-Jres** (Used alongside STOCSY for pattern/overlap checks on the 22 metabolites)
- **Bruker IVDr 600 MHz spectrometer** (Acquires the 1D 1H NMR data that the pattern search consumes)

## Examples

```
Rscript smolesy_select.R --input mtbls395_spectra/ --bins bins_24.csv --metabolites 22 --output assignments.csv
```

## Evaluation signals

- Every output row corresponds to one of the 24 predefined spin-system bins — no bin is missing and no unlisted bin appears
- Assignment decisions reproduce the >99.5% assignment accuracy reported across the validated cohort; spot-check a subset against the 22-metabolite target panel
- Detected δ values fall inside the bin's predefined spectral window (e.g. ≥10 SMolESY Δv1/2 for glycine, creatine/creatinine, choline, DMSO2, acetone, acetic acid; 3–5 Δv1/2 for correlated singlets)
- The pattern shapes matched lie within the SMolESY Δv1/2 range of 0.0009–0.0011 ppm for the 600 MHz instrument
- The pipeline runs end-to-end on all spectra with no manual intervention and yields exactly 22 metabolite rows × all samples

## Limitations

- The algorithm requires standard one-dimensional 1H NMR spectra and avoids sample preprocessing, complex deconvolution, and spectral baseline fitting — it will not replace 2D or deconvolution-based workflows
- The validation dataset of >8800 serum/plasma samples is not named or given an accession/repository; the >99.5% accuracy figure is the paper's own benchmark on its own validation cohort
- No software version is stated for SMolESY-select or SMolESY; the GUI and usage video exist only as supplementary artifacts (Figure S9, Video S1)
- Assignment accuracy depends on bin widths defined from a specific cohort (4023 samples); singlet bins are deliberately narrow (3–5 Δv1/2), so small calibration errors to the 5.233 ppm glucose reference can cause misassignment

## Evidence

- [results] The algorithm searches the SMolESY data for each spin system pattern within defined spectral windows (bins): "the algorithm searches the SMolESY data for each spin system pattern within defined spectral windows (bins)"
- [results] Calibration to the glucose anomeric proton at 5.233 ppm: "all spectra are automatically calibrated to the 1H NMR signals of glucose"
- [results] Bins derived from a 4023-sample cohort: "using a cohort of 4023 unique plasma/serum samples (3023 plasma and 1000 serum)"
- [results] 11-point moving average filter to raise s/n for low-abundance metabolites: "increases the s/n of signals belonging to tyrosine, phenylalanine, histidine, creatinine, and choline"
- [results] Direct integration of assigned components, precluding deconvolution or baseline fitting: "enables the direct integration of the assigned spectral components, precluding the need for complex signal deconvolution or baseline fitting"
- [abstract] >99.5% assignment accuracy for all 22 metabolites: "yielding an assignment accuracy greater than 99.5% for all"
- [abstract] Requires only standard one-dimensional 1H NMR spectra: "requiring only standard one-dimensional 1H NMR spectra"
- [results] STOCSY and 2D-Jres used for overlap checks: "overlap of each spin system’s SMolESY pattern with other SM signals using STOCSY"
- [results] Binned window widths: "consisted of at least 10 SMolESY Δv1/2 (>0.01 ppm"
- [results] Random ppm alignment of assigned spin systems eases visual detection of misassignment: "aligns only each assigned spin system to a random ppm value within their corresponding predefined spectral window"
