---
name: metabolite-signal-assignment-nmr
description: Use when assigning metabolite signals in calibrated 1D 1H NMR spectra
  of serum/plasma (e.g. Bruker IVDr, 600 MHz) to the 22-metabolite panel via the 24
  predefined spin-system bins from SMolESY-select.
when_to_use_negative:
- The spectrum is not a calibrated 1D 1H NMR serum/plasma profile from the 22-metabolite
  panel (e.g. LC–MS, 2D NMR, or tissue extracts) — the bins and spin-system patterns
  are specific to that panel and acquisition format.
- The spectrum has not been automatically calibrated to the glucose anomeric proton
  at 5.233 ppm; uncalibrated chemical-shift scales break the bin-pattern matching.
- You need quantitation of metabolites outside the 22-metabolite panel (the algorithm
  was built and validated only for those 22 targets).
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3292
tools:
- name: SMolESY-select
  role: Runs the bin-pattern search that assigns each of the 24 spin systems to its
    metabolite signal within the predefined spectral windows; also applies the 11-point
    moving average smoothing in selected windows.
  repo: https://github.com/pantakis/SMolESY-select
- name: SMolESY
  role: Produces the baseline-suppressed, resolution-enhanced 1D 1H NMR data whose
    narrow Δv1/2 patterns the assignment search matches.
  repo: https://github.com/pantakis/SMolESY
- name: Bruker IVDr 600 MHz spectrometer
  role: Source instrument for the standard 1D 1H NMR serum/plasma spectra that SMolESY-select
    consumes.
- name: STOCSY
  role: Used to check the overlap of each spin system's SMolESY pattern with other
    SM signals during bin design.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.1c00113
    title: A Computationally Lightweight Algorithm for Deriving Reliable Metabolite
      Panel Measurements from 1D <sup>1</sup>H NMR
  source_content_hash: sha256:dcc4ce2d94308122c94763e6b4d1271781c0cf9e6e20c1c6869d575c89b95ec0
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
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# metabolite-signal-assignment-nmr

## Summary

SMolESY-select assigns the 24 predefined 1H NMR spin systems of the 22-metabolite serum/plasma panel by pattern-matching each metabolite's SMolESY-enhanced multiplet within its predefined spectral bin, enabling direct integration of calibrated 1D 1H NMR spectra without deconvolution or baseline fitting.

## When to use

Use when assigning metabolite signals in calibrated 1D 1H NMR spectra of serum/plasma (e.g. Bruker IVDr, 600 MHz) to the 22-metabolite panel via the 24 predefined spin-system bins from SMolESY-select.

## When NOT to use

- The spectrum is not a calibrated 1D 1H NMR serum/plasma profile from the 22-metabolite panel (e.g. LC–MS, 2D NMR, or tissue extracts) — the bins and spin-system patterns are specific to that panel and acquisition format.
- The spectrum has not been automatically calibrated to the glucose anomeric proton at 5.233 ppm; uncalibrated chemical-shift scales break the bin-pattern matching.
- You need quantitation of metabolites outside the 22-metabolite panel (the algorithm was built and validated only for those 22 targets).

## Inputs

- Calibrated 1D 1H NMR spectrum of human serum or plasma (e.g. Bruker IVDr 600 MHz acquisition, MTBLS395-style)
- SMolESY-processed version of the same spectrum (baseline-suppressed, resolution-enhanced)
- Predefined spectral-window (bin) definitions for the 24 1H NMR spin systems, with per-bin window boundaries in ppm

## Outputs

- Per-bin assignment table (one row per spin-system bin) with bin identifier, metabolite signal, assignment status, and detected δ (ppm) position — e.g. assignments.csv
- Aligned visualization in which each assigned spin system is placed at a random ppm value inside its bin to expose misassignment

## How to apply

First automatically calibrate the 1D 1H NMR spectrum to the glucose anomeric proton at 5.233 ppm. Next, apply the SMolESY transform (resolution enhancement plus macromolecular/baseline suppression) so that each spin system's pattern is narrow (Δv1/2 ≈ 0.0009–0.0011 ppm at 600 MHz). Then, for each of the 24 predefined bins (window boundaries derived from a 4023-sample plasma/serum cohort; singlets like glycine, creatine/creatinine, choline, DMSO2, acetone, acetic acid get bins ≥10 SMolESY Δv1/2, while narrow singlet bins of 3–5 SMolESY Δv1/2 are used where correlated with lactic acid/alanine), search the SMolESY data within that bin for the expected spin-system multiplet pattern, recording the assignment and the detected δ value. A moving average filter spanning 11 points is applied within selected windows (e.g. tyrosine, phenylalanine, histidine, creatinine, choline) to boost s/n for low-abundance signals, trading a maximum ~10% s/n loss versus CPMG spectra for pattern clarity. Each assigned spin system is aligned to a random ppm value inside its bin so misassignment is visually obvious, and the assigned components are then integrated directly — no deconvolution or baseline fitting.

## Related tools

- **SMolESY-select** (Runs the bin-pattern search that assigns each of the 24 spin systems to its metabolite signal within the predefined spectral windows; also applies the 11-point moving average smoothing in selected windows.) — https://github.com/pantakis/SMolESY-select
- **SMolESY** (Produces the baseline-suppressed, resolution-enhanced 1D 1H NMR data whose narrow Δv1/2 patterns the assignment search matches.) — https://github.com/pantakis/SMolESY
- **Bruker IVDr 600 MHz spectrometer** (Source instrument for the standard 1D 1H NMR serum/plasma spectra that SMolESY-select consumes.)
- **STOCSY** (Used to check the overlap of each spin system's SMolESY pattern with other SM signals during bin design.)

## Evaluation signals

- Every output row must correspond to exactly one of the 24 predefined spin-system bins; row count equals 24.
- Assignment decisions should reproduce the reported >99.5% assignment accuracy across the validated cohort for all 22 metabolites.
- SMolESY Δv1/2 of assigned patterns should fall in the expected range 0.0009–0.0011 ppm at 600 MHz; deviations flag calibration or processing errors.
- Spot-check a subset of spectra against the 22-metabolite target panel to confirm each assigned δ value lies inside the correct predefined bin.
- Any assigned spin system whose plotted position falls outside its bin (the random-ppm-in-bin alignment) indicates misassignment.

## Limitations

- Bins were defined for exactly 24 spin systems; the panel is limited to the 22 metabolites covered by those bins.
- The moving-average filter trades s/n: assigned signals have at most ~10% less s/n than CPMG spectra.
- Narrow singlet bins (3–5 SMolESY Δv1/2) for glycine, creatine/creatinine, choline, DMSO2, acetone, acetic acid rely on correlation with lactic acid/alanine δ values; wide shifts in those reference metabolites could shift the bin boundaries.
- The >8800-sample validation set is not named or accessioned in the article text, so independent reproduction of the accuracy figure is not directly possible from this source alone.

## Evidence

- [results] the algorithm searches the SMolESY data for each spin system pattern within defined spectral windows (bins): "the algorithm searches the SMolESY data for each spin system pattern within defined spectral windows (bins)"
- [results] Bins were defined for 24 1H NMR spin systems using a cohort of 4023 unique plasma/serum samples (3023 plasma and 1000 serum): [evidence span withheld — anchor status: not_found]
- [results] 0.0009 < SMolESY Δv1/2 < 0.0011 ppm for the 600 MHz NMR instrument: "0.0009 < SMolESY Δv1/2 < 0.0011 ppm for the 600 MHz NMR instrument"
- [results] all spectra are automatically calibrated to the 1H NMR signals of glucose: "all spectra are automatically calibrated to the 1H NMR signals of glucose"
- [abstract] yielding an assignment accuracy greater than 99.5% for all
22: "yielding an assignment accuracy greater than 99.5% for all
22"
- [results] aligns only each assigned spin system to a random ppm value within their corresponding predefined spectral window: "aligns only each assigned spin system to a random ppm value within their corresponding predefined spectral window"
