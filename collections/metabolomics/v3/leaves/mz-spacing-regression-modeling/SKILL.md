---
name: mz-spacing-regression-modeling
description: Use when you need to build reference isotope-pattern models — linear
  m/z-spacing functions or SVM training data — from theoretical formula enumerations
  instead of measured spectra.
when_to_use_negative:
- The goal is feature detection on real LC-MS data — FeatureFinderMetabo consumes
  these pre-built models; it does not fit them.
- The analyte space extends beyond the CHNOPS elements or outside the 1-1000 Da window
  the enumeration and filter cover.
- An external isotope-pattern model is already supplied and only validation/execution
  is needed — refitting mu(j)/sigma(j) would duplicate existing coefficients.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: emass
  role: Computes the theoretical isotope pattern of each sampled sum formula, providing
    the mass differences and abundance ratios the spacing models and SVM features
    are derived from.
- name: OpenMS
  role: Framework in which the fitted spacing models and SVM classifier are integrated
    into FeatureFinderMetabo's isotope-pattern scoring.
  repo: github.com/OpenMS/OpenMS
- name: FeatureFinderMetabo
  role: 'Downstream consumer: assembles features based on m/z spacing, co-elution,
    and the SVM classifier whose training instances this skill produces.'
  repo: github.com/OpenMS/OpenMS
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1074/mcp.m113.031278
    title: Automated Label-free Quantification of Metabolites from Liquid Chromatography–Mass
      Spectrometry Data
  source_content_hash: sha256:db082743d90691507a9ce0668ebeddadfa631b09af1d1327817539157618b6e0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FeatureFinderMetabo
  - OpenMS
  - emass
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# m/z-spacing regression modeling

## Summary

Generates reference metabolite isotope-pattern models by fitting linear m/z-spacing equations (mu(j), sigma(j)) against monoisotopic mass from emass-computed isotope patterns of simulated sum formulas, and derives the SVM training set used by FeatureFinderMetabo's isotope-pattern classifier.

## When to use

Use when you need to build reference isotope-pattern models — linear m/z-spacing functions or SVM training data — from theoretical formula enumerations instead of measured spectra.

## When NOT to use

- The goal is feature detection on real LC-MS data — FeatureFinderMetabo consumes these pre-built models; it does not fit them.
- The analyte space extends beyond the CHNOPS elements or outside the 1-1000 Da window the enumeration and filter cover.
- An external isotope-pattern model is already supplied and only validation/execution is needed — refitting mu(j)/sigma(j) would duplicate existing coefficients.

## Inputs

- Enumerated sum formulas over elements C, H, N, O, P, S within a 1-1000 Da mass window (HiRes-style filtered enumeration, ~24M formulas)
- Randomly sampled subset of 115,000 formula compositions
- Theoretical isotope patterns computed per composition with the emass isotope-distribution library

## Outputs

- model_parameters.csv with fitted linear mu(j)/sigma(j) spacing coefficients for isotope indices j=1..5
- svm_training_set.csv with labeled (metabolite-like vs. non-metabolite), Gaussian-noise-perturbed feature rows (first three abundance ratios + monoisotopic mass)

## How to apply

Enumerate sum formulas over C, H, N, O, P, S in the 1-1000 Da window and apply the HiRes-style heuristic element filter to keep chemically plausible compositions (~24M formulas). Randomly sample 115,000 evenly spread compositions from this filtered space and compute each one's theoretical isotope pattern with the emass isotope-distribution library. For each isotope index j=1..5, fit by linear regression the mean peak spacing mu(j) and spacing spread sigma(j) as functions of monoisotopic mass, yielding the linear m/z-spacing equations. Build SVM training instances from the feature vector (first three isotope abundance ratios plus the monoisotopic mass), label metabolite-like vs. non-metabolite patterns, and perturb features with Gaussian noise to model a 5% RMS error, matching the expected measurement error in FeatureFinderMetabo's feature assembly.

## Related tools

- **emass** (Computes the theoretical isotope pattern of each sampled sum formula, providing the mass differences and abundance ratios the spacing models and SVM features are derived from.)
- **OpenMS** (Framework in which the fitted spacing models and SVM classifier are integrated into FeatureFinderMetabo's isotope-pattern scoring.) — github.com/OpenMS/OpenMS
- **FeatureFinderMetabo** (Downstream consumer: assembles features based on m/z spacing, co-elution, and the SVM classifier whose training instances this skill produces.) — github.com/OpenMS/OpenMS

## Examples

```
FeatureFinderMetabo -in sample.mzML -out features.featureXML
```

## Evaluation signals

- model_parameters.csv contains fitted coefficients for all isotope indices j=1..5 (mu(j) and sigma(j) for each).
- Expert/graph check: fitted mu(j) and sigma(j) behave as linear functions of monoisotopic mass, as required by the model form.
- svm_training_set.csv contains the full set of labeled, noise-perturbed training rows built from the first-three-ratio + monoisotopic-mass feature vector.
- Added Gaussian noise corresponds to the intended 5% RMS error model of the abundance ratios.
- Sampled compositions (~115,000) are evenly spread across the filtered enumeration space (~24M formulas from ~1-1000 Da CHNOPS).

## Limitations

- Element scope restricted to C, H, N, O, P, S and the 1-1000 Da mass window; formulas outside this space are not represented.
- The HiRes-style heuristic element filter trades enumeration size (~24M formulas) for chemical plausibility and may exclude valid compositions.
- SVM features are perturbed with Gaussian noise assuming a 5% RMS error; real measurement error regimes differing from this may degrade classifier validity.
- Only the first five isotope spacings are modeled and only the first three abundance ratios enter the SVM feature vector.

## Evidence

- [methods] Randomly sample 115,000 compositions from the filtered enumeration space.: [evidence span withheld — anchor status: not_found]
- [methods] emass isotope computation: [evidence span withheld — anchor status: not_found]
- [methods] spacing model purpose: [evidence span withheld — anchor status: not_found]
- [abstract] SVM classifier role in article: "assembled into features based on mass-to-charge spacing, co-elution information, and a support vector machine-based classifier"
- [readme] OpenMS integration context: [evidence span withheld — anchor status: not_found]
