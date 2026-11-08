---
name: kappa-sharpness-parameter-tuning
description: Use when running Magnetstein on a 1H NMR mixture where default parameters
  underperform — e.g., a ~1:10 two-component mixture, spectra with contaminant peaks,
  or low-field benchtop spectra — and per-component RE/AvRE must be optimized.
when_to_use_negative:
- When default settings already yield acceptable AvRE — the paper ran all programs
  with defaults unless tuning was needed
- When the failure is not parameter-related, e.g., peak-picking tools (ACD/TP, MANIQ)
  failing on almost completely overlapped low-resolution peaks — there the choice
  of method, not κ tuning, is the issue
tools:
- name: Magnetstein
  role: Optimal-transport quantification tool whose κmixture and κcomponents sharpness
    parameters are tuned by this skill
  repo: github.com/ElsevierSoftwareX/SOFTX-D-25-00358
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c03594
    title: 'Magnetstein: An Open-Source Tool for Quantitative NMR Mixture Analysis
      Robust to Low Resolution, Distorted Lineshapes, and Peak Shifts'
  source_content_hash: sha256:98c6ea278d75dadeb603cf8911f382da77f1eb49f1f619cf2317aedf5ff8aaa1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ElsevierSoftwareX/SOFTX-D-25-00358
  tools:
  - Magnetstein
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# kappa-sharpness-parameter-tuning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Tunes Magnetstein's two sharpness parameters (κmixture and κcomponents) when quantifying 1H NMR mixture spectra by optimal transport, since default settings can fail in specific regimes such as strongly imbalanced mixtures or contamination by extra peaks.

## When to use

Use when running Magnetstein on a 1H NMR mixture where default parameters underperform — e.g., a ~1:10 two-component mixture, spectra with contaminant peaks, or low-field benchtop spectra — and per-component RE/AvRE must be optimized.

## When NOT to use

- When default settings already yield acceptable AvRE — the paper ran all programs with defaults unless tuning was needed
- When the failure is not parameter-related, e.g., peak-picking tools (ACD/TP, MANIQ) failing on almost completely overlapped low-resolution peaks — there the choice of method, not κ tuning, is the issue

## Inputs

- Mixture 1H NMR spectrum (e.g., Magritek 43 MHz benchtop spectrum of α-pinene and limonene)
- Reference library spectra of the mixture components
- Gravimetrically known relative concentrations for computing RE/AvRE

## Outputs

- Estimated relative concentrations of each component
- Per-component relative errors (RE) and average relative error (AvRE) at default and tuned κ settings
- Tuned κmixture / κcomponents values (as compiled in Table S1)

## How to apply

Treat the mixture and library spectra as probability distributions of signal intensity and run Magnetstein's optimal-transport deconvolution with explicit κmixture and κcomponents settings rather than defaults. For a two-component mixture with a large concentration ratio (Experiment 1, ~1:10), avoid small κmixture values, which push the smaller component into the auxiliary vortex ω and inflate its relative error; raise κmixture there. When the spectrum contains peaks not present in the library (Experiment 2 degradation contaminants), lower κmixture from its default so contaminating peaks are not forced onto library components. For low-resolution benchtop spectra (Magritek 43 MHz, Experiment 3), set κcomponents higher than κmixture, which the paper reports as optimal. Report results both with defaults and with the tuned optimum (as in Table S1), and quantify improvement via per-component RE and AvRE against gravimetrically known composition.

## Related tools

- **Magnetstein** (Optimal-transport quantification tool whose κmixture and κcomponents sharpness parameters are tuned by this skill) — github.com/ElsevierSoftwareX/SOFTX-D-25-00358

## Evaluation signals

- Experiment 3 (benchtop, α-pinene/limonene): tuned run should give AvRE ≈ 9.0% with RE α-pinene = 0.0% and RE limonene = 33.3%
- Experiment 1 (~1:10 mixture): tuned κmixture should yield AvRE ≈ 0.0% rather than the large error from assigning the smaller component to vortex ω
- Experiment 2: lowering κmixture from default should reduce or remove the contribution of contaminant peaks and improve AvRE toward the reported 3.0%
- Optimal κ settings satisfy κcomponents > κmixture for the low-resolution Experiment 3 case
- Reported tuned results should be reproducible/traceable to Table S1

## Limitations

- Small κmixture values cause Magnetstein to assign the smaller component to vortex ω, producing a large relative error in imbalanced two-component mixtures
- There is no single universal κ setting: Experiment 2 required lowering the default κmixture (contaminant peaks) while Experiment 3 required κcomponents > κmixture
- In Experiment 3, results were robust to κmixture but sensitive mainly to κcomponents, so tuning effort should target the dominant parameter per regime

## Evidence
