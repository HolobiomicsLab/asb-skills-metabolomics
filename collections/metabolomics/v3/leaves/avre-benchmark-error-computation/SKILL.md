---
name: avre-benchmark-error-computation
description: Use when a 1H NMR mixture spectrum has a known gravimetric composition
  and you need to quantify how accurately an optimal-transport quantifier (Magnetstein)
  recovers relative concentrations, especially for overlapping/shifted peaks, distorted
  lineshapes, or low-field benchtop data.
when_to_use_negative:
- The input is a mass-spectrometry dataset handled by Masserstein/MasserFront (the
  linked repository is a mass-spec UI application, not the NMR pipeline)
- There is no known ground-truth (gravimetric) composition, so relative error against
  true concentrations cannot be computed
- The goal is absolute concentration via an external standard rather than benchmarking
  relative-concentration trueness
tools:
- name: Magnetstein
  role: 'Primary quantifier: treats spectra as probability distributions and solves
    the optimal transport problem to estimate relative concentrations pi'
  repo: github.com/ElsevierSoftwareX/SOFTX-D-25-00358
- name: MANIQ
  role: Peak-based benchmark quantifier whose AvRE is tabulated for comparison (only
    leads in Experiment 2)
- name: ACD/Spectrus-Target-Protected Profiling
  role: Commercial peak-based benchmark quantifier whose AvRE is tabulated for comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c03594
    title: 'Magnetstein: An Open-Source Tool for Quantitative NMR Mixture Analysis
      Robust to Low Resolution, Distorted Lineshapes, and Peak Shifts'
  source_content_hash: sha256:d9df79282fa205d05f877932c2d1766f84c8e9ff7f8ff66db37cc61fe727bb63
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# avre-benchmark-error-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce and benchmark quantification trueness for 1H NMR mixture spectra by computing average relative error (AvRE) of Magnetstein's estimated relative concentrations against known gravimetric compositions, alongside MANIQ and ACD/Spectrus-TP baselines, across Experiments 1–5 of the Magnetstein study.

## When to use

Use when a 1H NMR mixture spectrum has a known gravimetric composition and you need to quantify how accurately an optimal-transport quantifier (Magnetstein) recovers relative concentrations, especially for overlapping/shifted peaks, distorted lineshapes, or low-field benchtop data.

## When NOT to use

- The input is a mass-spectrometry dataset handled by Masserstein/MasserFront (the linked repository is a mass-spec UI application, not the NMR pipeline)
- There is no known ground-truth (gravimetric) composition, so relative error against true concentrations cannot be computed
- The goal is absolute concentration via an external standard rather than benchmarking relative-concentration trueness

## Inputs

- Deposited 1H NMR mixture spectra for Experiments 1–5 (from the article's data deposit)
- Component library spectra matching each mixture (e.g., cholesterol/linoleic-acid-type components for Exp1; limonene and α-pinene for Exps 2–3)
- Known gravimetric compositions per experiment (ground-truth relative concentrations)

## Outputs

- Per-experiment AvRE values (Magnetstein ≈ 0.0, 3.0, 9.0, 3.6, 3.6 %) with estimated relative concentrations pi
- avre_comparison.csv — table comparing Magnetstein, MANIQ, and ACD/TP AvRE across Experiments 1–5

## How to apply

Load the deposited 1H NMR mixture spectra and matching component library spectra for Experiments 1–5, then quantify each mixture with Magnetstein by treating spectra as probability distributions of signal intensity and solving the optimal transport problem with default κmixture and κcomponents. For Experiment 3 (low-field benchtop fragrance-mixture data), rerun with κcomponents set higher than κmixture, since the article reports best results under that configuration; for Experiment 2 (limonene/α-pinene with degradation-contaminant peaks), consider lowering the default κmixture to remove contaminating peaks. Extract the estimated relative concentrations pi and compute average relative error (AvRE) per experiment against the known gravimetric compositions. Assemble Magnetstein's AvRE (expected ≈ 0.0, 3.0, 9.0, 3.6, 3.6 % for Experiments 1–5) alongside MANIQ (12.2, 1.0, 28.8, 34.9, 7.2 %) and ACD/TP (18.3, 6.9, 56.9, 39.7, 35.8 %) values into a single comparison table written to avre_comparison.csv.

## Related tools

- **Magnetstein** (Primary quantifier: treats spectra as probability distributions and solves the optimal transport problem to estimate relative concentrations pi) — github.com/ElsevierSoftwareX/SOFTX-D-25-00358
- **MANIQ** (Peak-based benchmark quantifier whose AvRE is tabulated for comparison (only leads in Experiment 2))
- **ACD/Spectrus-Target-Protected Profiling** (Commercial peak-based benchmark quantifier whose AvRE is tabulated for comparison)

## Evaluation signals

- Reproduced Magnetstein AvRE per experiment matches the article's reported values (Exp1 0.0%, Exp2 3.0%, Exp3 9.0%, Exp4 3.6%, Exp5 3.6%) within reasonable numerical tolerance
- Baseline rows in avre_comparison.csv match MANIQ AvRE of 12.2, 1.0, 28.8, 34.9, 7.2 % and ACD/TP AvRE of 18.3, 6.9, 56.9, 39.7, 35.8 % for Experiments 1–5
- Experiment 3 rerun uses κcomponents higher than κmixture and reproduces per-component RE (α-pinene 0.0%, limonene 33.3%) rather than the default-setting failure modes (ACD/TP α-pinene RE 125.0%)
- Magnetstein outperforms MANIQ and ACD/TP in Experiments 1, 3, 4, and 5, with MANIQ leading only in Experiment 2
- All experiments except Experiment 3 use default κmixture/κcomponents settings, matching the article's protocol (defaults unless otherwise stated)

## Limitations

- For small values of κmixture, Magnetstein tends to put the smaller component into vortex ω, causing a large relative error (observed in Experiment 1's ~1:10 concentration difference)
- In Experiment 2, contaminating degradation-product peaks degrade results under the default κmixture; performance improves only after lowering it
- Experiment 5's mixture spectrum was acquired at 36.7 °C versus library spectra at 22.27 °C, causing significant peak shifts that challenge peak-based baselines
- The article's discussion omits the Magnetstein version number and hosting/Zenodo DOI for the tool, complicating exact reproduction

## Evidence

- [results] Magnetstein estimated the relative concentrations with a very high trueness (AvRE 0.0%), an order of magnitude better than that of the other two programs (AvRE 18.3% for ACD/TP, 12.2% for MANIQ): "Magnetstein estimated the relative concentrations with a very high trueness (AvRE 0.0%), an order of magnitude better than that of the other two programs (AvRE 18.3% for ACD/TP, 12.2% for MANIQ)"
- [discussion] we consider spectra as probability distributions of signal intensity and use the mathematical tools of the optimal transport theory: "we consider spectra as probability distributions of signal intensity and use the mathematical tools of the optimal transport theory"
- [results] Note that the best results are obtained when κcomponents is higher than κmixture.: "Note that the best results are obtained when κcomponents is higher than κmixture."
- [results] Magnetstein make it very robust to lineshape distortions (AvRE 3.6%), while peak-based methods struggle (AvRE 39.7% for ACD/TP, 34.9% for MANIQ): "Magnetstein make it very robust to lineshape distortions (AvRE 3.6%), while peak-based methods struggle (AvRE 39.7% for ACD/TP, 34.9% for MANIQ)"
