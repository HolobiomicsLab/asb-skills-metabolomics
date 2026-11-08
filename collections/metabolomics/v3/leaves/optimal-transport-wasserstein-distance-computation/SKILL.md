---
name: optimal-transport-wasserstein-distance-computation
description: Use when quantifying components of a 1H NMR mixture spectrum against
  a spectral library whose peaks overlap, are shifted (e.g. by solvent or temperature,
  as in the 36.7 °C vs 22.27 °C Experiment 5), or have distorted lineshapes from an
  inhomogeneous field.
when_to_use_negative:
- When the goal is absolute concentration against no spectral library — the optimal-transport
  formulation yields relative proportions πi; absolute quantification instead requires
  algebraically adding a quantitative external-standard spectrum to the mixture.
- When spectra are clean, well-resolved peak lists where peak-integration or peak-based
  tools already suffice — simpler peak-based methods (MANIQ, ACD/TP) can outperform
  in cases like Experiment 2 (MANIQ AvRE 1.0% vs Magnetstein 3.0%).
- When κmixture is set very small for a heavily imbalanced mixture (e.g. ~1:10 in
  Experiment 1) without checking that the minor component has not been absorbed into
  the vortex ω.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: Magnetstein
  role: Open-source quantification algorithm rooted in optimal transport theory that
    implements the Wasserstein-distance LP with κmixture/κcomponents denoising penalties
    for NMR mixture spectra
  repo: github.com/ElsevierSoftwareX/SOFTX-D-25-00358
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c03594
    title: 'Magnetstein: An Open-Source Tool for Quantitative NMR Mixture Analysis
      Robust to Low Resolution, Distorted Lineshapes, and Peak Shifts'
  source_content_hash: sha256:8641025976050f5778e6a89974c9a63e2859ae3b41d462e73f47340fc1b5ec97
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
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# optimal-transport-wasserstein-distance-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Casts quantitative NMR mixture deconvolution as an optimal-transport problem: mixture and library spectra are normalized into discrete probability measures over the chemical-shift axis, and component proportions πi are recovered by solving a Wasserstein-distance transportation LP with denoising penalties κmixture and κcomponents.

## When to use

Use when quantifying components of a 1H NMR mixture spectrum against a spectral library whose peaks overlap, are shifted (e.g. by solvent or temperature, as in the 36.7 °C vs 22.27 °C Experiment 5), or have distorted lineshapes from an inhomogeneous field.

## When NOT to use

- When the goal is absolute concentration against no spectral library — the optimal-transport formulation yields relative proportions πi; absolute quantification instead requires algebraically adding a quantitative external-standard spectrum to the mixture.
- When spectra are clean, well-resolved peak lists where peak-integration or peak-based tools already suffice — simpler peak-based methods (MANIQ, ACD/TP) can outperform in cases like Experiment 2 (MANIQ AvRE 1.0% vs Magnetstein 3.0%).
- When κmixture is set very small for a heavily imbalanced mixture (e.g. ~1:10 in Experiment 1) without checking that the minor component has not been absorbed into the vortex ω.

## Inputs

- Mixture 1H NMR spectrum normalized as a discrete probability measure of signal intensity over the chemical-shift axis
- Library of candidate component spectra, each normalized as a discrete probability measure
- Penalty parameters κmixture and κcomponents (defaults available; tunable per experiment, optimal settings reported in Table S1)

## Outputs

- Estimated relative concentrations of the mixture components (estimated_proportions.csv)
- Average relative error (AvRE) against known ground-truth proportions when truth is available

## How to apply

Normalize the mixture spectrum and each library component spectrum into discrete probability measures of signal intensity over the chemical-shift axis. Compute the Wasserstein distance between the mixture measure and each candidate component measure by solving the underlying transportation linear program. Then solve the augmented LP that adds an auxiliary vortex point ω receiving excess unexplained signal with denoising penalty κmixture, and penalty κcomponents absorbing signals missing from the components, jointly estimating the proportions πi. Tune the penalties within the stated regime: set κcomponents higher than κmixture (as found optimal in the benchtop Experiment 3), and avoid small κmixture values, which push the smaller component into the vortex ω and cause large relative error (Experiment 1). Rationale: treating spectra as probability distributions makes quantification robust to peak shifts and lineshape distortions that break peak-based methods, which in the reported comparisons reached AvRE 34.9–39.7% under distorted lineshapes versus 3.6% for this formulation.

## Related tools

- **Magnetstein** (Open-source quantification algorithm rooted in optimal transport theory that implements the Wasserstein-distance LP with κmixture/κcomponents denoising penalties for NMR mixture spectra) — github.com/ElsevierSoftwareX/SOFTX-D-25-00358

## Evaluation signals

- Estimated relative concentrations reproduce the reference trueness: AvRE 0.0% for the two-component Experiment 1 and AvRE 3.6% under lineshape distortion (inhomogeneous-field Experiment 4), as summarized in Table S1 with optimal κmixture and κcomponents
- Penalty regime check: chosen κcomponents exceeds κmixture, and κmixture is not so small that the smaller component is assigned to vortex ω
- All mixture and library spectra are valid discrete probability measures (normalized signal intensity over the chemical-shift axis) before solving the transportation LP
- Benchmark comparison: AvRE should be in the same order as reported Magnetstein results (0.0–9.0% across Experiments 1, 3, 4, 5) rather than the 12.2–56.9% range reported for peak-based tools

## Limitations

- For small values of κmixture, the method tends to put the smaller component into vortex ω, causing a large relative error (Experiment 1, ~1:10 mixture)
- With default κmixture, contaminating/degradation peaks can inflate error (Experiment 2); performance improves by lowering the default κmixture
- Reported evaluation is on 1H NMR; generalization to higher-dimensional or other spectroscopic data via sliced Wasserstein distance or graph-theory flow representations is only proposed future work
- No version number, Zenodo DOI, or hosting location for Magnetstein is provided in the article; only the ElsevierSoftwareX/SOFTX-D-25-00358 repo (MasserFront, GPL-2.0) is referenced, and it documents a mass-spectrometry UI rather than the NMR tool itself

## Evidence
