---
name: nmr-library-component-matching
description: Use when a mixture 1H NMR spectrum must be deconvolved against reference
  component spectra and peaks overlap almost completely, lineshapes are distorted,
  or peaks are shifted — e.g., low-field benchtop (Magritek 43 MHz) data where peak-picking
  tools fail.
when_to_use_negative:
- 'The spectrum is mass-spectrometry data: the related MasserFront/Masserstein tooling
  targets mass spectrometry result analysis, not NMR library matching'
- Peaks are well separated and a peak-picking/deconvolution-by-integration method
  (e.g., ACD/TP, MANIQ) already gives acceptable errors
- Absolute concentrations are required but no quantitative reference spectrum can
  be added to the library (the external-standard approach requires a known-concentration
  substance spectrum algebraically added to the mixture spectrum)
edam_operation: http://edamontology.org/operation_3799
tools:
- name: Magnetstein
  role: Open-source optimal-transport quantification algorithm used to deconvolve
    the mixture spectrum against the component library and estimate relative concentrations
  repo: github.com/ElsevierSoftwareX/SOFTX-D-25-00358
- name: MANIQ
  role: Peak-based state-of-the-art tool used as a benchmark comparator for AvRE on
    the same mixture spectra
- name: ACD/Spectrus-Target-Protected Profiling
  role: Commercial peak-picking tool used as a benchmark comparator for AvRE on the
    same mixture spectra
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c03594
    title: 'Magnetstein: An Open-Source Tool for Quantitative NMR Mixture Analysis
      Robust to Low Resolution, Distorted Lineshapes, and Peak Shifts'
  source_content_hash: sha256:ec51336ffa376be7080ae84188a5cee8a89986a6ff0272b29c0e51fce8c1ba1b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ElsevierSoftwareX/SOFTX-D-25-00358
  tools:
  - ACD/Spectrus-Target-Protected Profiling
  - MANIQ
  - Magnetstein
  edam_operation: http://edamontology.org/operation_3799
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# nmr-library-component-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantify components in 1H NMR mixtures by treating mixture and reference-library spectra as probability distributions of signal intensity and running an optimal-transport deconvolution (Magnetstein), which works where peak-picking fails (e.g., low-resolution benchtop spectra with near-complete peak overlap).

## When to use

Use when a mixture 1H NMR spectrum must be deconvolved against reference component spectra and peaks overlap almost completely, lineshapes are distorted, or peaks are shifted — e.g., low-field benchtop (Magritek 43 MHz) data where peak-picking tools fail.

## When NOT to use

- The spectrum is mass-spectrometry data: the related MasserFront/Masserstein tooling targets mass spectrometry result analysis, not NMR library matching
- Peaks are well separated and a peak-picking/deconvolution-by-integration method (e.g., ACD/TP, MANIQ) already gives acceptable errors
- Absolute concentrations are required but no quantitative reference spectrum can be added to the library (the external-standard approach requires a known-concentration substance spectrum algebraically added to the mixture spectrum)

## Inputs

- Mixture 1H NMR spectrum (including low-resolution benchtop spectra, e.g., Magritek 43 MHz)
- Reference library spectra of the pure components expected in the mixture
- Gravimetrically known composition of the mixture (for computing RE/AvRE when validating)
- Magnetstein sharpness parameters κmixture and κcomponents

## Outputs

- Estimated relative concentrations (proportions πi) of each mixture component
- Per-component relative errors (RE) and average relative error (AvRE) versus known composition
- Quantification table, e.g., experiment3_quantification.csv

## How to apply

Load the mixture spectrum (e.g., the Experiment 3 fragrance mixture of α-pinene and limonene acquired on a Magritek 43 MHz benchtop spectrometer) together with the library spectra of each pure component. Treat both mixture and library spectra as probability distributions of signal intensity, as prescribed by Magnetstein's probabilistic optimal-transport formalism. Run the optimal-transport deconvolution of the mixture against the component library, setting sharpness parameters κmixture and κcomponents; for low-resolution benchtop data the best results are obtained when κcomponents is higher than κmixture, and lowering κmixture from its default can help exclude contaminating peaks. Extract the estimated relative concentrations of each component and compute per-component relative error (RE) and average relative error (AvRE) against gravimetrically known compositions, then benchmark against published values (e.g., Experiment 3: Magnetstein AvRE 9.0% versus ACD/TP 56.9% and MANIQ 28.8%).

## Related tools

- **Magnetstein** (Open-source optimal-transport quantification algorithm used to deconvolve the mixture spectrum against the component library and estimate relative concentrations) — github.com/ElsevierSoftwareX/SOFTX-D-25-00358
- **MANIQ** (Peak-based state-of-the-art tool used as a benchmark comparator for AvRE on the same mixture spectra)
- **ACD/Spectrus-Target-Protected Profiling** (Commercial peak-picking tool used as a benchmark comparator for AvRE on the same mixture spectra)

## Evaluation signals

- Reproduced AvRE matches the reported Magnetstein value of 9.0% for Experiment 3 within a reasonable tolerance
- Per-component REs match reported values: α-pinene RE = 0.0% and limonene RE = 33.3%
- Magnetstein's AvRE is far below the peak-picking benchmarks (ACD/TP 56.9%, MANIQ 28.8%) on the same benchtop spectrum
- Results are robust to κmixture settings, with κcomponents playing the bigger role (best when κcomponents > κmixture)
- Estimated proportions sum consistently with the probabilistic (distribution) interpretation of the spectra

## Limitations

- Even at best, Magnetstein showed a 33.3% RE for limonene in the Experiment 3 benchtop case, so low-resolution quantification can still be inaccurate for individual components
- For small κmixture values the method tends to put the smaller component into vortex ω, causing a large relative error
- No version number, repository URL, or Zenodo DOI for Magnetstein is given in the article, complicating provenance tracking

## Evidence
