---
name: theoretical-isotope-pattern-simulation
description: Use when reproducing or retraining the metabolite isotope-pattern reference
  models behind FeatureFinderMetabo's SVM classifier, e.g. to regenerate linear m/z-spacing
  fits or noise-perturbed training data for metabolite feature validation.
when_to_use_negative:
- When the goal is detecting features directly in LC-MS spectra — the reconstructed
  models are consumed downstream by FeatureFinderMetabo's SVM classifier, not applied
  to raw data
- When a validated set of model_parameters.csv / svm_training_set.csv already exists
  and only feature-hypothesis validation is required
- When the analyte class is outside the metabolite/small-molecule scope (e.g. proteomic
  peptide isotopes), since the simulated pattern space was filtered for metabolite-like
  compositions
tools:
- name: OpenMS
  role: Framework in which FeatureFinderMetabo — the downstream consumer of the simulated
    isotope-pattern models and SVM classifier — is implemented (version 1.10)
  repo: github.com/OpenMS/OpenMS
- name: FeatureFinderMetabo
  role: Metabolite feature-detection tool whose isotope-pattern scoring consumes the
    reconstructed mu(j)/sigma(j) spacing models and SVM training set
  repo: github.com/OpenMS/OpenMS
- name: HiRes
  role: Formula enumeration engine whose heuristic element filter defines the chemically
    plausible composition space (~24M formulas over C, H, O, N, P, S, 1–1000 Da) being
    reconstructed
- name: emass
  role: Isotope-distribution library used to compute the theoretical isotope patterns
    from which mass differences and abundance ratios are extracted
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1074/mcp.m113.031278
    title: Automated Label-free Quantification of Metabolites from Liquid Chromatography–Mass
      Spectrometry Data
  source_content_hash: sha256:381f0d9c6c97e72ad88736dbca35dc0366d71d78c9ea3777b3d35e32dac95458
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FeatureFinderMetabo
  - HiRes
  - OpenMS
  - emass
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# theoretical-isotope-pattern-simulation

## Summary

Reconstructs the metabolite isotope-pattern reference models used by FeatureFinderMetabo's SVM classifier: theoretical isotope patterns are simulated for enumerated sum formulas, from which linear m/z-spacing models (mu(j)/sigma(j)) and a labeled SVM training set are derived.

## When to use

Use when reproducing or retraining the metabolite isotope-pattern reference models behind FeatureFinderMetabo's SVM classifier, e.g. to regenerate linear m/z-spacing fits or noise-perturbed training data for metabolite feature validation.

## When NOT to use

- When the goal is detecting features directly in LC-MS spectra — the reconstructed models are consumed downstream by FeatureFinderMetabo's SVM classifier, not applied to raw data
- When a validated set of model_parameters.csv / svm_training_set.csv already exists and only feature-hypothesis validation is required
- When the analyte class is outside the metabolite/small-molecule scope (e.g. proteomic peptide isotopes), since the simulated pattern space was filtered for metabolite-like compositions

## Inputs

- Element set definition (C, H, N, O, P, S) with mass window 1–1000 Da
- HiRes-style heuristic element-filter configuration
- emass isotope-distribution library parameters
- Sample size specification (115,000 compositions)

## Outputs

- model_parameters.csv — fitted mu(j)/sigma(j) linear m/z-spacing coefficients for isotope indices j=1..5
- svm_training_set.csv — labeled, noise-perturbed training rows (first-three-peak-ratio plus monoisotopic-mass features)
- Simulated theoretical isotope patterns (mass differences and abundance ratios) for each sampled composition

## How to apply

Enumerate sum formulas over the elements C, H, N, O, P, S in the 1–1000 Da mass window and apply the HiRes-style heuristic element filter, yielding roughly 24 million chemically plausible compositions. Randomly sample 115,000 evenly spread compositions from this filtered space and compute each one's theoretical isotope pattern with the emass isotope-distribution library. For each isotope index j=1..5, fit the linear m/z-spacing models mu(j) (mean peak spacing vs. monoisotopic mass) and sigma(j) (spacing spread vs. monoisotopic mass) by linear regression, and emit the coefficients as model_parameters.csv. Build the SVM training set from the four-feature vector of the first three abundance ratios plus the monoisotopic mass, label metabolite-like vs. non-metabolite patterns, and perturb features with Gaussian noise modeling a 5% RMS error before writing svm_training_set.csv. Rationale: these simulated models provide the ground-truth spacing and abundance behavior that the feature-hypothesis scoring in FeatureFinderMetabo uses to distinguish true metabolite isotope patterns from noise.

## Related tools

- **OpenMS** (Framework in which FeatureFinderMetabo — the downstream consumer of the simulated isotope-pattern models and SVM classifier — is implemented (version 1.10)) — github.com/OpenMS/OpenMS
- **FeatureFinderMetabo** (Metabolite feature-detection tool whose isotope-pattern scoring consumes the reconstructed mu(j)/sigma(j) spacing models and SVM training set) — github.com/OpenMS/OpenMS
- **HiRes** (Formula enumeration engine whose heuristic element filter defines the chemically plausible composition space (~24M formulas over C, H, O, N, P, S, 1–1000 Da) being reconstructed)
- **emass** (Isotope-distribution library used to compute the theoretical isotope patterns from which mass differences and abundance ratios are extracted)

## Evaluation signals

- model_parameters.csv contains fitted mu(j)/sigma(j) coefficients for all isotope indices j=1..5
- Expert/plausibility check confirms the fitted spacing functions behave as linear functions of monoisotopic mass
- svm_training_set.csv contains the full set of labeled, noise-perturbed patterns with the four-feature vector (first three abundance ratios plus monoisotopic mass) at 5% RMS Gaussian noise
- Formula enumeration over C, H, N, O, P, S in the 1–1000 Da window with the HiRes heuristic filter reproduces the expected ~24 million composition scale before random sampling of 115,000 compositions
- Extracted mass differences and abundance ratios from emass patterns are consistent with the linear regression inputs for the first five isotope spacings

## Limitations

- Enumeration is restricted to the elements C, H, O, N, P, S within a 1–1000 Da mass window, so isotope models do not cover compositions outside this element/mass space
- The heuristic element filter is only approximately chemically plausible and may exclude or include compositions accordingly
- The SVM training set's realism depends on the assumed 5% RMS Gaussian-noise error model, which may not match actual instrument noise
- OpenMS implementation details such as version-specific parameter handling (e.g. via INIFileEditor) are not documented in the source discussion section, per the article's own missing-signal notes

## Evidence

- [abstract] assembled into features based on mass-to-charge spacing, co-elution information, and a support vector machine-based classifier able to identify potential metabolite isotope patterns: "assembled into features based on mass-to-charge spacing, co-elution information, and a support vector machine-based classifier able to identify potential metabolite isotope patterns"
- [other] Randomly sample 115,000 compositions from the filtered enumeration space. Compute the theoretical isotope pattern of each sampled composition using the emass isotope-distribution library.: [evidence span withheld — anchor status: not_found]
- [other] For each isotope index j=1..5, fit the linear m/z-spacing models mu(j) (mean peak spacing vs. monoisotopic mass) and sigma(j) (spacing spread vs. monoisotopic mass) by linear regression on the computed patterns.: [evidence span withheld — anchor status: not_found]
- [readme] It comes with a vast variety of pre-built and ready-to-use tools for proteomics
and metabolomics data analysis (TOPPTools): "It comes with a vast variety of pre-built and ready-to-use tools for proteomics
and metabolomics data analysis (TOPPTools)"
