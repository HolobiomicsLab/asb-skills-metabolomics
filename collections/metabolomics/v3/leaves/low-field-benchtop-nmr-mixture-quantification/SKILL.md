---
name: low-field-benchtop-nmr-mixture-quantification
description: Use when a mixture spectrum comes from a low-field benchtop 1H NMR spectrometer
  (e.g. Magritek 43 MHz) with nearly complete peak overlap, or when peak-picking quantification
  has failed on overlapping signals.
when_to_use_negative:
- Spectra where peaks are well resolved and peak-picking tools (ACD/TP, MANIQ) suffice
  — Magnetstein's transport-based deconvolution is aimed at overlapped, low-resolution,
  distorted, or shifted spectra.
- Mass spectrometry data — the sibling tool Masserstein/MasserFront covers mass-spectrometry
  result analysis, not NMR mixture quantification.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: Magnetstein
  role: Open-source optimal-transport quantification algorithm that treats spectra
    as probability distributions of signal intensity and deconvolves the mixture spectrum
    against component library spectra
  repo: github.com/ElsevierSoftwareX/SOFTX-D-25-00358
- name: ACD/Spectrus-Target-Protected Profiling
  role: Peak-picking commercial comparator used to benchmark AvRE/RE on the same spectra
    (fails in this regime)
- name: MANIQ
  role: Peak-picking comparator used to benchmark AvRE/RE on the same spectra (fails
    in this regime)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c03594
    title: 'Magnetstein: An Open-Source Tool for Quantitative NMR Mixture Analysis
      Robust to Low Resolution, Distorted Lineshapes, and Peak Shifts'
  source_content_hash: sha256:4b6c8f786508abbcb9a4cecf25d9eae697bd93e2738cf656916dba3f3bc35d6a
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
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# low-field-benchtop-nmr-mixture-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies mixture-component proportions from low-resolution benchtop 1H NMR spectra (e.g. Magritek 43 MHz) by applying Magnetstein's optimal-transport deconvolution, a regime where peak-picking tools like ACD/TP and MANIQ fail because peaks overlap almost completely.

## When to use

Use when a mixture spectrum comes from a low-field benchtop 1H NMR spectrometer (e.g. Magritek 43 MHz) with nearly complete peak overlap, or when peak-picking quantification has failed on overlapping signals.

## When NOT to use

- Spectra where peaks are well resolved and peak-picking tools (ACD/TP, MANIQ) suffice — Magnetstein's transport-based deconvolution is aimed at overlapped, low-resolution, distorted, or shifted spectra.
- Mass spectrometry data — the sibling tool Masserstein/MasserFront covers mass-spectrometry result analysis, not NMR mixture quantification.

## Inputs

- Low-resolution benchtop 1H NMR mixture spectrum (e.g. Magritek 43 MHz) of compounds such as fragrance α-pinene and limonene
- Reference library spectra of each mixture component (1H NMR)

## Outputs

- Estimated relative concentrations (proportions pi) of each mixture component
- Per-component relative error (RE) and average relative error (AvRE) table (e.g. experiment3_quantification.csv)

## How to apply

Load the benchtop 1H NMR mixture spectrum together with reference library spectra of the components, and treat both the mixture and library spectra as probability distributions of signal intensity per Magnetstein's probabilistic formalism. Run Magnetstein's optimal-transport deconvolution of the mixture spectrum against the component library. For low-resolution benchtop data, the paper reports optimal results when the sharpness parameter κcomponents is set higher than κmixture (which can otherwise be left near default; Magnetstein was robust to κmixture settings in this regime). Extract the estimated relative concentrations of each component (e.g. α-pinene and limonene) and compute per-component relative error (RE) and average relative error (AvRE) against the gravimetrically known composition. Validate against reported Experiment 3 benchmarks: Magnetstein AvRE 9.0% (RE α-pinene 0.0%, limonene 33.3%) versus AvRE 56.9% for ACD/TP and 28.8% for MANIQ.

## Related tools

- **Magnetstein** (Open-source optimal-transport quantification algorithm that treats spectra as probability distributions of signal intensity and deconvolves the mixture spectrum against component library spectra) — github.com/ElsevierSoftwareX/SOFTX-D-25-00358
- **ACD/Spectrus-Target-Protected Profiling** (Peak-picking commercial comparator used to benchmark AvRE/RE on the same spectra (fails in this regime))
- **MANIQ** (Peak-picking comparator used to benchmark AvRE/RE on the same spectra (fails in this regime))

## Examples

```
python magnetstein.py --mixture experiment3_benchtop_43MHz.csv --library alpha_pinene.csv limonene.csv --kappa_mixture 0.01 --kappa_components 1.0 --out experiment3_quantification.csv
```

## Evaluation signals

- Reproduced AvRE matches the reported Experiment 3 value of 9.0% within a reasonable tolerance
- RE for α-pinene equals 0.0% and RE for limonene equals 33.3%, as reported for Experiment 3
- Benchmarked peak-picking AvRE falls in the failure regime reported for this data (ACD/TP 56.9%, MANIQ 28.8%)
- Parameterization check: κcomponents set higher than κmixture, the condition reported as optimal for Experiment 3

## Limitations

- For small κmixture values, Magnetstein tends to put the smaller component into vortex ω, causing a large relative error — tune sharpness parameters rather than using arbitrary defaults.
- Results are reported for two-component fragrance mixtures; behavior on more components or other low-field datasets is not demonstrated in Experiment 3.
- No Magnetstein version number or dedicated repository URL/Zenodo DOI is provided in the article's discussion.

## Evidence
