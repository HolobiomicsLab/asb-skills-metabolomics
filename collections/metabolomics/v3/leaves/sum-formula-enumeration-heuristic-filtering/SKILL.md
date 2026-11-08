---
name: sum-formula-enumeration-heuristic-filtering
description: Use when building reference metabolite isotope-pattern models for feature-hypothesis
  validation in LC-MS data (e.g. FeatureFinderMetabo), requiring enumerated sum formulas
  and theoretical isotope patterns as training material.
when_to_use_negative:
- Input is already a set of observed isotope patterns or feature intensities — this
  skill builds the reference model space from simulated sum formulas, not from measured
  data.
- You need compound identification against a spectral library rather than generating
  theoretical isotope-pattern models from enumeration.
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: OpenMS
  role: framework hosting FeatureFinderMetabo and supporting pipeline integration
    for the metabolite isotope-pattern models
  repo: https://github.com/OpenMS/OpenMS
- name: FeatureFinderMetabo
  role: consumes the reference isotope-pattern models when assembling mass traces
    into metabolite features
  repo: https://github.com/OpenMS/OpenMS
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1074/mcp.m113.031278
    title: Automated Label-free Quantification of Metabolites from Liquid Chromatography–Mass
      Spectrometry Data
  source_content_hash: sha256:d4477926e8949c0f3b553d02a0216f68598d78d7e7eade7978258fc648dcad66
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OpenMS/OpenMS
  tools:
  - FeatureFinderMetabo
  - OpenMS
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# sum-formula-enumeration-heuristic-filtering

## Summary

Generates a reference space of chemically plausible metabolite sum formulas over C, H, N, O, P, S in the 1–1000 Da window by exhaustive enumeration followed by a HiRes-style heuristic element filter, then computes theoretical isotope patterns (emass) to derive linear m/z-spacing models and an SVM training set for metabolite feature validation.

## When to use

Use when building reference metabolite isotope-pattern models for feature-hypothesis validation in LC-MS data (e.g. FeatureFinderMetabo), requiring enumerated sum formulas and theoretical isotope patterns as training material.

## When NOT to use

- Input is already a set of observed isotope patterns or feature intensities — this skill builds the reference model space from simulated sum formulas, not from measured data.
- You need compound identification against a spectral library rather than generating theoretical isotope-pattern models from enumeration.

## Inputs

- enumeration space parameters (element set C,H,N,O,P,S; mass window 1–1000 Da)
- emass isotope-distribution library
- HiRes-style heuristic element filter
- Gaussian noise of 5% RMS

## Outputs

- model_parameters.csv (fitted μ(j)/σ(j) coefficients for j=1..5)
- svm_training_set.csv (labeled, noise-perturbed feature vectors)

## How to apply

Enumerate all sum formulas composed of C, H, N, O, P, S within the 1–1000 Da mass window (≈24 M candidates) and apply a HiRes-style heuristic element filter to keep chemically plausible compositions. Randomly sample 115,000 evenly spread compositions from the filtered space and compute each one's theoretical isotope distribution with the emass library. From these patterns extract mass differences and abundance ratios, then fit linear models μ(j) and σ(j) for the first five isotope spacings as functions of monoisotopic mass via linear regression. Build the SVM training set using the first-three-peak-ratio plus monoisotopic-mass as a four-feature vector, label metabolite-like against non-metabolite patterns, and perturb features with Gaussian noise of 5% RMS to mimic measurement error. Emit fitted μ(j)/σ(j) coefficients as model_parameters.csv and labeled, noise-perturbed rows as svm_training_set.csv.

## Related tools

- **OpenMS** (framework hosting FeatureFinderMetabo and supporting pipeline integration for the metabolite isotope-pattern models) — https://github.com/OpenMS/OpenMS
- **FeatureFinderMetabo** (consumes the reference isotope-pattern models when assembling mass traces into metabolite features) — https://github.com/OpenMS/OpenMS

## Examples

```
python build_reference_models.py --elements C,H,N,O,P,S --mass-window 1-1000 --sample-size 115000 --noise-rms 0.05
```

## Evaluation signals

- model_parameters.csv contains coefficients for all isotope indices j=1..5
- svm_training_set.csv contains the full set of labeled, noise-perturbed patterns
- fitted spacing functions behave as linear functions of monoisotopic mass
- sampled composition count equals 115,000 and is evenly spread
- Gaussian noise of 5% RMS is applied to the feature vector

## Limitations

- Heuristic element filter reduces ~24 M enumerated formulas to a chemically plausible subset; formulas outside the filter's scope are excluded.
- The SVM training set uses only the first three abundance ratios plus monoisotopic mass — higher-peak information is not used.
- The noise model assumes 5% RMS Gaussian error; real mass spectrometry errors may deviate from this assumption.

## Evidence

- [methods] Heuristic element filter: [evidence span withheld — anchor status: not_found]
- [methods] emass isotope computation: [evidence span withheld — anchor status: not_found]
- [discussion] Feature assembly scoring: "Model scoring with a hybrid mass deviation and isotope profile scoring results in accurately assembled features."
- [abstract] Isotope pattern classifier: "assembled into features based on mass-to-charge spacing, co-elution information, and a support vector machine-based classifier able to identify potential metabolite isotope patterns"
