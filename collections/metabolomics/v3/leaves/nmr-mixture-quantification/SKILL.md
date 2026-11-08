---
name: nmr-mixture-quantification
description: Use when 1H NMR mixture spectra contain overlapping peaks, shifted resonances
  (e.g., 36.7 °C sample vs 22.27 °C library), lineshape distortions from inhomogeneous
  fields, or low-field benchtop resolution, and relative molar proportions of components
  are needed.
when_to_use_negative:
- The data are mass spectrometry spectra — that domain is handled by the separate
  Masserstein/MasserFront tools, not this 1H NMR quantification workflow
- Absolute concentrations are required but no quantitative spectrum of a known-concentration
  standard has been acquired and added to the mixture spectrum
- The task is peak assignment/annotation rather than concentration quantification,
  or the spectrum is a single pure compound where simple integration suffices
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: Magnetstein
  role: 'Primary quantification engine: solves the optimal-transport-based quantification
    of 1H NMR mixture spectra given component library spectra and tunable κmixture/κcomponents'
  repo: github.com/ElsevierSoftwareX/SOFTX-D-25-00358
- name: MANIQ
  role: Commercial peak-picking baseline used to benchmark Magnetstein's AvRE on the
    same mixture spectra
- name: ACD/Spectrus-Target-Protected Profiling
  role: Commercial peak-picking baseline (ACD/TP) used to benchmark Magnetstein's
    AvRE and demonstrate order-of-magnitude trueness gains
- name: MasserFront
  role: Related React and Flask UI application from the same group for Masserstein
    mass spectrometry result analysis; distinct from the 1H NMR Magnetstein workflow
  repo: https://gitlab.mimuw.edu.pl/ag181044/masserfront
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c03594
    title: 'Magnetstein: An Open-Source Tool for Quantitative NMR Mixture Analysis
      Robust to Low Resolution, Distorted Lineshapes, and Peak Shifts'
  source_content_hash: sha256:6795e25dd4d548061fbfa30c7c7cd7cc1be1cf1b6e7123bd41cc01e81da20411
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
  - MasserFront
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# nmr-mixture-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantify components of 1H NMR mixture spectra using Magnetstein, an optimal-transport-based open-source algorithm that treats spectra as probability distributions of signal intensity and is robust to low resolution, distorted lineshapes, contaminant peaks, and temperature-induced peak shifts.

## When to use

Use when 1H NMR mixture spectra contain overlapping peaks, shifted resonances (e.g., 36.7 °C sample vs 22.27 °C library), lineshape distortions from inhomogeneous fields, or low-field benchtop resolution, and relative molar proportions of components are needed.

## When NOT to use

- The data are mass spectrometry spectra — that domain is handled by the separate Masserstein/MasserFront tools, not this 1H NMR quantification workflow
- Absolute concentrations are required but no quantitative spectrum of a known-concentration standard has been acquired and added to the mixture spectrum
- The task is peak assignment/annotation rather than concentration quantification, or the spectrum is a single pure compound where simple integration suffices

## Inputs

- Deposited 1H NMR mixture spectrum (e.g., from Experiments 1–5 or the BCAA supplement of Experiment 11)
- Reference library 1H NMR spectra of the mixture components (e.g., leucine, isoleucine, valine, α-pinene, limonene)
- Known gravimetric composition of the mixture (for computing AvRE validation)
- Magnetstein parameters κmixture and κcomponents (default, or κcomponents > κmixture for low-field data)
- Optional: quantitative spectrum of an external standard of known concentration

## Outputs

- Estimated relative molar proportions pi per component (e.g., Ile 0.25, Leu 0.25, Val 0.25 for the BCAA supplement)
- Average relative error (AvRE) per experiment against known composition
- Comparison table of AvRE values for Magnetstein vs MANIQ vs ACD/TP (e.g., avre_comparison.csv)
- Absolute concentrations when an external standard spectrum is included in the library

## How to apply

Load the deposited 1H NMR mixture spectrum plus the component library spectra, and run Magnetstein with default tunable parameters κmixture and κcomponents; the algorithm treats each spectrum as a probability distribution of signal intensity and solves an optimal transport problem to estimate relative concentrations pi. For low-field benchtop data (as in Experiment 3 with α-pinene and limonene), set κcomponents higher than κmixture, since the article reports the best results under that configuration; for spectra with contaminating peaks (Experiment 2, limonene/α-pinene), lowering the default κmixture removes contaminant resonances and improves accuracy. Validate estimates against gravimetric compositions by computing the average relative error (AvRE); the article reports Magnetstein AvREs of 0.0%, 3.0%, 9.0%, 3.6%, and 3.6% for Experiments 1–5, versus commercial peak-picking tools MANIQ and ACD/Spectrus-Target-Protected Profiling. For absolute (not relative) concentrations, acquire a quantitative spectrum of a known substance and algebraically add it to the mixture spectrum so proportions pi can be converted to concentrations. Compare optimal-parameter results against Table S1 in the Supporting Information.

## Related tools

- **Magnetstein** (Primary quantification engine: solves the optimal-transport-based quantification of 1H NMR mixture spectra given component library spectra and tunable κmixture/κcomponents) — github.com/ElsevierSoftwareX/SOFTX-D-25-00358
- **MANIQ** (Commercial peak-picking baseline used to benchmark Magnetstein's AvRE on the same mixture spectra)
- **ACD/Spectrus-Target-Protected Profiling** (Commercial peak-picking baseline (ACD/TP) used to benchmark Magnetstein's AvRE and demonstrate order-of-magnitude trueness gains)
- **MasserFront** (Related React and Flask UI application from the same group for Masserstein mass spectrometry result analysis; distinct from the 1H NMR Magnetstein workflow) — https://gitlab.mimuw.edu.pl/ag181044/masserfront

## Evaluation signals

- Reproduced Magnetstein AvRE matches reported values within numerical tolerance: 0.0% (Exp1), 3.0% (Exp2), 9.0% (Exp3), 3.6% (Exp4), 3.6% (Exp5)
- Per-experiment gap vs ACD/TP remains near or exceeds an order of magnitude in Experiments 1, 3, 4, and 5 (e.g., ACD/TP − Magnetstein = 36.1 pp in Experiment 4)
- The single reversal is correctly captured: MANIQ (1.0%) outperforms Magnetstein (3.0%) in Experiment 2
- BCAA supplement output contains three proportions matching 0.25 (Ile), 0.25 (Leu), 0.25 (Val) within reported relative errors of 4.2%, 4.2%, and 10.7%
- Optimal-parameter results are consistent with the article's Table S1 entries obtained with optimal κmixture and κcomponents

## Limitations

- No version number, repository URL, Zenodo DOI, or hosting location is provided for Magnetstein in the article's discussion; the listed GitHub repo and the cloned gitlab repository correspond to MasserFront/Masserstein (mass spectrometry), not a documented Magnetstein download
- For small κmixture values the method tends to put the smaller component into vortex ω, causing a large relative error (Experiment 1, ~1:10 concentration difference)
- Results depend on parameter tuning: Experiment 3 requires κcomponents > κmixture, and Experiment 2 requires lowering the default κmixture to remove contaminating peaks; all other comparisons used default settings
- A future-work item: generalization to higher dimensions via sliced Wasserstein distance or graph-theory flow/network representations is not yet implemented

## Evidence
