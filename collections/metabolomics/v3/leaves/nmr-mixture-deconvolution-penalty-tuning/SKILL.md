---
name: nmr-mixture-deconvolution-penalty-tuning
description: Use when a 1H NMR mixture spectrum must be quantified against library
  component spectra despite overlapping peaks, unexpected shifts (e.g. different solvents
  or temperature), low-field benchtop acquisition, or distorted lineshapes from poor
  shimming.
when_to_use_negative:
- The spectrum contains a large amount of signal absent from the library that is genuine
  sample content rather than contamination — a small κmixture will route true minor
  components into vortex ω and destroy the estimate
- The goal is absolute concentration without an external standard; the OT formulation
  yields proportions πi, and absolute values require algebraically adding a quantitative
  reference spectrum of a known-concentration substance to the mixture spectrum
- Spectra are not 1D 1H NMR profiles representable as probability measures over chemical
  shift; the article's planned extensions (sliced Wasserstein distance, graph-theory
  flow representations) for higher-dimensional data are future work, not implemented
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: Magnetstein
  role: Open-source optimal-transport tool that solves the augmented Wasserstein-distance
    LP to estimate mixture component proportions and accepts κmixture/κcomponents
    tuning
  repo: https://github.com/ElsevierSoftwareX/SOFTX-D-25-00358
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c03594
    title: 'Magnetstein: An Open-Source Tool for Quantitative NMR Mixture Analysis
      Robust to Low Resolution, Distorted Lineshapes, and Peak Shifts'
  source_content_hash: sha256:78abf3d03bf39bc7f7d006422d8802f636140bb75ab53b7b4be0159acd25b510
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ElsevierSoftwareX/SOFTX-D-25-00358
  repo_url: https://github.com/ElsevierSoftwareX/SOFTX-D-25-00358
  tools:
  - Magnetstein
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# nmr-mixture-deconvolution-penalty-tuning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Casts 1H NMR mixture quantification as an optimal-transport (Wasserstein distance) deconvolution against library spectra using Magnetstein, adding an auxiliary vortex ω that absorbs excess signal under a denoising penalty κmixture, and tunes κmixture/κcomponents to control robustness to peak shifts, overlaps, and lineshape distortion.

## When to use

Use when a 1H NMR mixture spectrum must be quantified against library component spectra despite overlapping peaks, unexpected shifts (e.g. different solvents or temperature), low-field benchtop acquisition, or distorted lineshapes from poor shimming.

## When NOT to use

- The spectrum contains a large amount of signal absent from the library that is genuine sample content rather than contamination — a small κmixture will route true minor components into vortex ω and destroy the estimate
- The goal is absolute concentration without an external standard; the OT formulation yields proportions πi, and absolute values require algebraically adding a quantitative reference spectrum of a known-concentration substance to the mixture spectrum
- Spectra are not 1D 1H NMR profiles representable as probability measures over chemical shift; the article's planned extensions (sliced Wasserstein distance, graph-theory flow representations) for higher-dimensional data are future work, not implemented

## Inputs

- Mixture 1H NMR spectrum normalized as a discrete probability measure of signal intensity over the chemical-shift axis
- Library of component spectra, each normalized the same way
- Tuning parameters κmixture and κcomponents (with denoising penalty regime, κcomponents > κmixture)

## Outputs

- Estimated relative concentrations of mixture components (estimated_proportions.csv)
- Average relative error (AvRE) of estimated proportions against ground truth
- Optimal κmixture and κcomponents values (as reported in Table S1)

## How to apply

Normalize the mixture spectrum and each library component spectrum as discrete probability measures of signal intensity over the chemical-shift axis. Solve the transportation linear program that computes the Wasserstein distance between the mixture measure and candidate component measures, using Magnetstein's augmented formulation in which an auxiliary point ω (vortex) receives excess signal under penalty κmixture while signal missing from components is absorbed under penalty κcomponents, and solve the joint program to estimate proportions πi. When tuning, set κcomponents higher than κmixture (best results in Experiment 3), and avoid small κmixture values, which push the smaller component into vortex ω and cause large relative error (observed in Experiment 1, a two-component ~1:10 mixture); lowering κmixture from the default can instead help when contaminant/degradation peaks must be removed (Experiment 2). Run comparisons at default settings first, then tune κmixture and κcomponents and report optimal values as in Table S1. Output estimated relative concentrations and score trueness by average relative error (AvRE) against known ground-truth proportions.

## Related tools

- **Magnetstein** (Open-source optimal-transport tool that solves the augmented Wasserstein-distance LP to estimate mixture component proportions and accepts κmixture/κcomponents tuning) — https://github.com/ElsevierSoftwareX/SOFTX-D-25-00358

## Evaluation signals

- Two-component Experiment 1 (~1:10 concentration difference): estimated proportions reproduce ground truth with AvRE 0.0%
- Lineshape-distortion Experiment 4: AvRE ≈ 3.6%, far below peak-based comparators (39.7% ACD/TP, 34.9% MANIQ)
- Peak-shift Experiment 5 (mixture at 36.7 °C vs library at 22.27 °C): AvRE ≈ 3.6%, beating ACD/TP (35.8%) and MANIQ (7.2%)
- Sanity check on penalty regime: best results obtained when κcomponents is higher than κmixture; verify small κmixture does not drive the smaller component into vortex ω
- Results with tuned penalties are comparable to the reference values in Table S1

## Limitations

- Small κmixture values cause Magnetstein to assign the smaller component to vortex ω, producing large relative error
- On low-field benchtop spectra (Experiment 3) even Magnetstein errs on the weaker component (limonene RE 33.3%, AvRE 9.0%), though it remains far more robust than ACD/TP (AvRE 56.9%) and MANIQ (28.8%)
- Default κmixture can be suboptimal when contaminant peaks are present — Experiment 2 required lowering κmixture from the default to remove contaminating peaks and improve performance
- No version number or changelog is provided for Magnetstein, complicating reproducibility

## Evidence
