---
name: nmr-signal-calibration-glucose-anomeric
description: Use when 1D 1H NMR spectra of serum or plasma are about to enter a spin-system
  assignment workflow and must first be referenced to a common chemical-shift axis,
  e.g. before running SMolESY-select or comparing spectral windows across samples.
when_to_use_negative:
- When spectra already carry a trusted digital reference (e.g. published calibration
  curves or added TSP/DSS internal standard) that the downstream pipeline explicitly
  expects
- When spectra contain strong glucose resonances that are distorted by severe baseline
  problems or poor water suppression, since a misidentified glucose peak would propagate
  a wrong ppm offset
- When the analyte matrix is not serum/plasma-like and lacks a reliable glucose anomeric
  signal to reference to
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: SMolESY-select
  role: consumes the calibrated spectra; calibration is described as part of its automated
    assignment workflow
  repo: https://github.com/pantakis/SMolESY-select
- name: SMolESY
  role: transformation applied after calibration to produce the resolution-enhanced,
    macromolecule-suppressed spectra in which bins are defined
  repo: https://github.com/pantakis/SMolESY
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c00113
    title: A Computationally Lightweight Algorithm for Deriving Reliable Metabolite
      Panel Measurements from 1D <sup>1</sup>H NMR
  source_content_hash: sha256:f17d109e879af99e693cc11dc142009a529ad1d77b7ce875d1662e61befc1e0a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pantakis/SMolESY-select
  tools:
  - 2D-Jres
  - SMolESY
  - SMolESY-select
  - STOCSY
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# nmr-signal-calibration-glucose-anomeric

## Summary

Automated ppm-alignment of 1D 1H NMR spectra to the anomeric proton signal of glucose at 5.233 ppm, the first step of the SMolESY-select metabolite-assignment pipeline for serum/plasma spectra.

## When to use

Use when 1D 1H NMR spectra of serum or plasma are about to enter a spin-system assignment workflow and must first be referenced to a common chemical-shift axis, e.g. before running SMolESY-select or comparing spectral windows across samples.

## When NOT to use

- When spectra already carry a trusted digital reference (e.g. published calibration curves or added TSP/DSS internal standard) that the downstream pipeline explicitly expects
- When spectra contain strong glucose resonances that are distorted by severe baseline problems or poor water suppression, since a misidentified glucose peak would propagate a wrong ppm offset
- When the analyte matrix is not serum/plasma-like and lacks a reliable glucose anomeric signal to reference to

## Inputs

- Raw 1D 1H NMR spectra of serum or plasma samples (e.g. Bruker IVDr format from a 600 MHz spectrometer)
- A set of spectra to be calibrated in batch (e.g. a MetaboLights cohort such as MTBLS395)

## Outputs

- Chemical-shift-calibrated 1D 1H NMR spectra with the glucose anomeric proton fixed at 5.233 ppm, ready for SMolESY transformation and window-based assignment

## How to apply

Apply this calibration to every 1D 1H NMR spectrum (acquired e.g. on Bruker IVDr 600 MHz spectrometers) prior to SMolESY transformation and assignment, automatically referencing the spectrum so that the glucose anomeric proton resonance sits at 5.233 ppm. The rationale is that downstream SMolESY-select searches for spin-system patterns within predefined spectral windows (bins, with widths expressed as multiples of SMolESY Δv1/2 ≈ 0.0009–0.0011 ppm at 600 MHz); even small inter-sample chemical-shift drift would move signals out of their bins and inflate false-positive/nonassigned counts. Because glucose is abundant and its anomeric proton is well resolved in serum/plasma spectra, its 5.233 ppm signal is a robust single-point reference that avoids manual phase/offset tuning. After calibration, verify the glucose resonance reads 5.233 ppm and that neighbouring singlet positions (e.g. lactate, alanine, glycine, creatinine) fall inside the expected bins before proceeding to the moving-average (11-point) smoothing and assignment steps.

## Related tools

- **SMolESY-select** (consumes the calibrated spectra; calibration is described as part of its automated assignment workflow) — https://github.com/pantakis/SMolESY-select
- **SMolESY** (transformation applied after calibration to produce the resolution-enhanced, macromolecule-suppressed spectra in which bins are defined) — https://github.com/pantakis/SMolESY

## Evaluation signals

- After calibration, the 1H NMR glucose anomeric proton resonance reads 5.233 ppm in every processed spectrum
- Residual chemical-shift drift across the cohort is below the bin tolerance (≤3–5 SMolESY Δv1/2, i.e. ≲0.005 ppm for narrow bins)
- Downstream SMolESY-select assignment of the 22 metabolites achieves >99.5% accuracy with a false-positive/nonassigned rate of at most 0.45% (0–40 misassigned signals out of 8834 serum/plasma spectra)
- In SMolESY space, denoised signals keep Δv1/2 in the 0.0009–0.0011 ppm window at 600 MHz, confirming that calibration did not broaden or shift lines

## Limitations

- Calibration relies on the presence of a well-resolved glucose anomeric signal; spectra where this resonance is obscured may calibrate incorrectly
- The article does not specify a fallback reference or software version for the calibration routine, and the GUI (Figure S9, Video S1) is available only as a supplementary artifact not included here
- The calibration step alone does not guarantee accurate assignments — it must be followed by SMolESY transformation, 11-point moving-average smoothing, and window-based pattern matching for the 24 spin systems

## Evidence

- [results] Calibration of all spectra to the 1H NMR anomeric proton signal of glucose at 5.233 ppm: "to the anomeric proton of glucose (Figure 2a) resonating at 5.233 ppm"
- [results] The SMolESY-select pipeline performs automatic calibration of the spectra: "all spectra are automatically calibrated to the 1H NMR signals of glucose"
- [results] SMolESY-select aligns only each assigned spin system to a random ppm value within their corresponding predefined spectral window: "aligns only each assigned spin system to a random ppm value within their corresponding predefined spectral window"
- [results] SMolESY Δv1/2 range at 600 MHz for the bins used downstream: "0.0009 < SMolESY Δv1/2 < 0.0011 ppm for the 600 MHz NMR instrument"
- [discussion] SMolESY-select availability as a reproducible implementation: "it is freely available to download at https://github.com/pantakis/SMolESY-select"
