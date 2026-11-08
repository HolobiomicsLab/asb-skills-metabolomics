---
name: spectral-library-matching
description: Use when a 1H NMR mixture spectrum must be decomposed and quantified
  against a library of known component spectra, especially if peaks overlap, are unexpectedly
  shifted (e.g., different solvent or 36.7 °C vs 22.
when_to_use_negative:
- No reference spectrum for a mixture component exists in the library — the optimal
  transport matching cannot estimate concentrations of components absent from the
  library
- Higher-dimensional (e.g., 2D) spectroscopic data — the article notes graph-theory
  and sliced Wasserstein generalizations are only planned future work
- Well-resolved, high-resolution spectra with clean separated peaks where simple peak-based
  integration suffices and κ tuning offers no advantage
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: Magnetstein
  role: 'Primary algorithm: treats mixture and library spectra as probability distributions
    and solves the optimal transport problem to estimate relative concentrations pi;
    κmixture and κcomponents are tunable'
  repo: github.com/ElsevierSoftwareX/SOFTX-D-25-00358
- name: MANIQ
  role: State-of-the-art peak-based comparison baseline for benchmarking AvRE (only
    outperformed Magnetstein in Experiment 2 with default settings)
- name: ACD/Spectrus-Target-Protected Profiling
  role: Commercial peak-based baseline used to benchmark AvRE across Experiments 1–5
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c03594
    title: 'Magnetstein: An Open-Source Tool for Quantitative NMR Mixture Analysis
      Robust to Low Resolution, Distorted Lineshapes, and Peak Shifts'
  source_content_hash: sha256:e5341ccde40ac4ba07ec1ceb633f76f935af731e8ee69671caecad135d0f5f3e
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
yes_rate: 0.6923076923076923
n_claims: 13
---

# spectral-library-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantify the components of a 1H NMR mixture spectrum by treating the mixture and each library spectrum as probability distributions of signal intensity and solving an optimal transport problem, as implemented in Magnetstein. This distributional matching stays robust when peaks overlap, shift, or have distorted lineshapes, where peak-based tools fail.

## When to use

Use when a 1H NMR mixture spectrum must be decomposed and quantified against a library of known component spectra, especially if peaks overlap, are unexpectedly shifted (e.g., different solvent or 36.7 °C vs 22.27 °C library temperature), lineshapes are distorted, or resolution is low (benchtop spectrometers).

## When NOT to use

- No reference spectrum for a mixture component exists in the library — the optimal transport matching cannot estimate concentrations of components absent from the library
- Higher-dimensional (e.g., 2D) spectroscopic data — the article notes graph-theory and sliced Wasserstein generalizations are only planned future work
- Well-resolved, high-resolution spectra with clean separated peaks where simple peak-based integration suffices and κ tuning offers no advantage

## Inputs

- 1H NMR mixture spectrum (from the article's data deposit, e.g., Experiments 1–5 including low-field benchtop data)
- Library of reference spectra for each candidate mixture component
- Known gravimetric composition of the mixture (for trueness validation via AvRE)

## Outputs

- Estimated relative concentrations pi of each library component
- Per-component relative errors (RE) and average relative error (AvRE) against known composition
- Comparison table against baselines (e.g., avre_comparison.csv with MANIQ and ACD/TP values)

## How to apply

Load the mixture spectrum and matching component library spectra (e.g., from the article's data deposit) and treat each as a probability distribution of signal intensity. Solve the optimal transport problem between the mixture distribution and each component's distribution, yielding estimated relative concentrations pi. Set the tuning parameters κmixture and κcomponents (default settings unless otherwise stated); for low-field benchtop data such as Experiment 3, run with κcomponents higher than κmixture, since the article reports the best results under that configuration. For highly contaminated mixtures like Experiment 2 (limonene + α-pinene with degradation-contaminant resonances), lowering κmixture below the default removes contaminating peaks and improves trueness. Compute the average relative error (AvRE) of the estimated pi against known gravimetric compositions to judge trueness.

## Related tools

- **Magnetstein** (Primary algorithm: treats mixture and library spectra as probability distributions and solves the optimal transport problem to estimate relative concentrations pi; κmixture and κcomponents are tunable) — github.com/ElsevierSoftwareX/SOFTX-D-25-00358
- **MANIQ** (State-of-the-art peak-based comparison baseline for benchmarking AvRE (only outperformed Magnetstein in Experiment 2 with default settings))
- **ACD/Spectrus-Target-Protected Profiling** (Commercial peak-based baseline used to benchmark AvRE across Experiments 1–5)

## Evaluation signals

- Reproduced Magnetstein AvRE should match the article's reported values within numerical tolerance: 0.0% (Exp1), 3.0% (Exp2), 9.0% (Exp3), 3.6% (Exp4), 3.6% (Exp5)
- Magnetstein's AvRE should outperform MANIQ (12.2, 1.0, 28.8, 34.9, 7.2%) and ACD/TP (18.3, 6.9, 56.9, 39.7, 35.8%) in Experiments 1, 3, 4, 5
- Sanity-check parameter sensitivity: with small κmixture the smaller component may be assigned to vortex ω, producing a large relative error — flag this failure mode
- For Experiment 3, verify κcomponents > κmixture reproduces the best configuration (AvRE 9.0%; α-pinene RE 0.0%, limonene RE 33.3%)
- Cross-check the assembled comparison table against the article's results tables and Table S1 (optimal κmixture and κcomponents)

## Limitations

- With small κmixture values, the method tends to put the smaller component into vortex ω, causing a large relative error (seen in Experiment 1's ~1:10 mixture)
- With default κmixture in Experiment 2, contaminating degradation peaks degraded performance; MANIQ (AvRE 1.0%) beat Magnetstein (3.0%) unless κ was retuned
- Currently limited to 1D spectra; higher-dimensional generalizations (sliced Wasserstein distance, graph-theory flow representations) are only planned future work
- Only supports relative concentrations pi directly; absolute concentrations require acquiring a quantitative spectrum of a known standard and algebraically adding it to the mixture spectrum

## Evidence
