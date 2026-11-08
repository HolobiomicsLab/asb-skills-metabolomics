---
name: nmr-spectra-ppm-shifting
description: Use when raw 1D 1H NMR spectra of serum/plasma (e.g. 600 MHz Bruker IVDr)
  show uncalibrated chemical-shift axes and a glucose-referenced, reproducible ppm
  scale is required before metabolite assignment.
when_to_use_negative:
- Spectra already referenced to a different standard or containing no detectable glucose
  anomeric proton region, so the 5.233 ppm anchor cannot be located automatically
- 2D spectra (e.g. 2D-Jres) or non-blood 1H NMR profiles where glucose is not the
  routine internal reference
- Downstream analysis steps that require SMolESY-processed (baseline-suppressed, resolution-enhanced)
  data rather than raw uncalibrated spectra
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3389
tools:
- name: SMolESY-select
  role: Performs the automatic calibration step that shifts every spectrum to the
    glucose anomeric proton signal at 5.233 ppm as its initial processing step
  repo: https://github.com/pantakis/SMolESY-select
- name: SMolESY
  role: Parent algorithm whose assignment pipeline this calibration step feeds; the
    calibrated spectrum is subsequently searched for spin-system patterns within defined
    bins
  repo: https://github.com/pantakis/SMolESY
- name: Bruker IVDr 600 MHz spectrometer
  role: Acquisition instrument producing the raw 1D 1H NMR serum/plasma spectra that
    are calibrated
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.1c00113
    title: A Computationally Lightweight Algorithm for Deriving Reliable Metabolite
      Panel Measurements from 1D <sup>1</sup>H NMR
  source_content_hash: sha256:ada68ef2771a0989cb88875efc3364593c7d614fece3984f7cd69288013aa4c3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pantakis/SMolESY-select
  tools:
  - Bruker IVDr 600 MHz spectrometer
  - MTBLS395
  - SMolESY
  - SMolESY-select
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3389
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# nmr-spectra-ppm-shifting

## Summary

Automatic spectral calibration (ppm shifting) of raw 1D 1H NMR spectra of blood products to the glucose anomeric proton signal at 5.233 ppm, the routine reference for blood 1H NMR profile analysis in the SMolESY-select workflow.

## When to use

Use when raw 1D 1H NMR spectra of serum/plasma (e.g. 600 MHz Bruker IVDr) show uncalibrated chemical-shift axes and a glucose-referenced, reproducible ppm scale is required before metabolite assignment.

## When NOT to use

- Spectra already referenced to a different standard or containing no detectable glucose anomeric proton region, so the 5.233 ppm anchor cannot be located automatically
- 2D spectra (e.g. 2D-Jres) or non-blood 1H NMR profiles where glucose is not the routine internal reference
- Downstream analysis steps that require SMolESY-processed (baseline-suppressed, resolution-enhanced) data rather than raw uncalibrated spectra

## Inputs

- Raw 1D 1H NMR spectrum of serum or plasma (600 MHz, Bruker IVDr format), e.g. from the MTBLS395 dataset

## Outputs

- Calibrated 1D 1H NMR spectrum with the glucose anomeric proton at 5.233 ppm
- Recorded ppm shift offset applied during calibration

## How to apply

Load the raw 1D 1H NMR spectrum (600 MHz, Bruker IVDr format) from a serum/plasma sample and locate the glucose anomeric proton resonance in its expected spectral region. Compute the ppm offset needed to bring that signal to 5.233 ppm, then apply the computed shift to all data points of the spectrum so the entire chemical-shift axis is calibrated. This is the first step of SMolESY-select, performed automatically before spin-system pattern search, 11-point moving-average smoothing, and direct integration. Record the applied ppm shift offset and write the calibrated spectrum to an output file. Rationale: glucose at 5.233 ppm is the routine reference for blood 1H NMR profile analysis, and calibrating all spectra to a single internal anchor makes downstream bin positions and metabolite assignments comparable across thousands of samples (the algorithm was developed on >4000 serum/plasma spectra and validated on >8800 more with >99.5% assignment accuracy for 22 metabolites).

## Related tools

- **SMolESY-select** (Performs the automatic calibration step that shifts every spectrum to the glucose anomeric proton signal at 5.233 ppm as its initial processing step) — https://github.com/pantakis/SMolESY-select
- **SMolESY** (Parent algorithm whose assignment pipeline this calibration step feeds; the calibrated spectrum is subsequently searched for spin-system patterns within defined bins) — https://github.com/pantakis/SMolESY
- **Bruker IVDr 600 MHz spectrometer** (Acquisition instrument producing the raw 1D 1H NMR serum/plasma spectra that are calibrated)

## Evaluation signals

- After calibration, the glucose anomeric proton maximum sits at 5.233 ppm (within a small ppm tolerance) in every spectrum
- The recorded ppm shift offset is non-zero only when needed and is logged alongside the output spectrum
- All data points shift by a single constant offset — relative peak spacing is unchanged
- Downstream SMolESY-select pattern search finds all 22 target metabolites' spin systems, consistent with the reported >99.5% assignment accuracy
- Identical raw spectra re-calibrated produce byte-identical outputs (fully automated, deterministic behaviour)

## Limitations

- Depends on detecting the glucose anomeric proton; samples where this resonance is obscured may fail to calibrate
- No software version is stated for SMolESY-select or SMolESY in the article
- Calibration targets blood 1H NMR profiles specifically; the 5.233 ppm glucose reference is described as the routine reference for blood 1H NMR analysis

## Evidence

- [results] Calibration of all spectra to the 1H NMR anomeric proton signal of glucose at 5.233 ppm: [evidence span withheld — anchor status: not_found]
- [results] Glucose anchor resonance location: "to the anomeric proton of glucose (Figure 2a) resonating at 5.233 ppm"
- [results] Automatic calibration of all spectra: "all spectra are automatically calibrated to the 1H NMR signals of glucose"
- [discussion] SMolESY-select availability: "it is freely available to download at https://github.com/pantakis/SMolESY-select"
- [methods] Acquisition platform: "NMR samples were acquired using Bruker IVDr 600 MHz spectrometers"
