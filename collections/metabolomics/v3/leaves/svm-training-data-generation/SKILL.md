---
name: svm-training-data-generation
description: Use when you need to build or regenerate training data for a classifier
  that recognizes metabolite isotope patterns from mass-to-charge spacing and abundance
  ratios, and only theoretical formula simulations are available.
when_to_use_negative:
- The trained SVM model already exists and you only need to detect features in LC-MS
  maps — run FeatureFinderMetabo directly instead of regenerating training data.
- You want to learn isotope-pattern characteristics from measured raw spectra rather
  than from simulated theoretical isotope distributions.
- Your target analytes require elements beyond C, H, N, O, P, S or masses above 1000
  Da, since the enumeration window and heuristic element filter are fixed to that
  space.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: OpenMS
  role: Framework in which the SVM-based metabolite isotope-pattern classifier is
    implemented; the generated training data underlies this classifier component
  repo: https://github.com/OpenMS/OpenMS
- name: FeatureFinderMetabo
  role: Consumer of the generated SVM training set — its scoring function uses the
    classifier to identify potential metabolite isotope patterns during feature assembly
  repo: https://github.com/OpenMS/OpenMS
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1074/mcp.m113.031278
    title: Automated Label-free Quantification of Metabolites from Liquid Chromatography–Mass
      Spectrometry Data
  source_content_hash: sha256:ec6c9b8331480f2b8934ce593c84a874008c9c252959eb86695d9b75c2583a7c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OpenMS/OpenMS
  tools:
  - FeatureFinderMetabo
  - OpenMS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# svm-training-data-generation

## Summary

Generates the labeled SVM training set and linear isotope-spacing reference models that FeatureFinderMetabo uses to identify metabolite isotope patterns in LC-MS data, by simulating sum formulas and their theoretical isotope distributions rather than training on measured peaks.

## When to use

Use when you need to build or regenerate training data for a classifier that recognizes metabolite isotope patterns from mass-to-charge spacing and abundance ratios, and only theoretical formula simulations are available.

## When NOT to use

- The trained SVM model already exists and you only need to detect features in LC-MS maps — run FeatureFinderMetabo directly instead of regenerating training data.
- You want to learn isotope-pattern characteristics from measured raw spectra rather than from simulated theoretical isotope distributions.
- Your target analytes require elements beyond C, H, N, O, P, S or masses above 1000 Da, since the enumeration window and heuristic element filter are fixed to that space.

## Inputs

- Sum-formula enumeration space over the elements C, H, N, O, P, S restricted to the 1-1000 Da mass window, filtered by the HiRes-style heuristic element filter
- Isotope abundance data as used by the emass isotope-distribution library
- Random sample of 115,000 evenly spread compositions from the filtered formula enumeration

## Outputs

- model_parameters.csv containing fitted mu(j)/sigma(j) linear m/z-spacing coefficients for isotope indices j=1..5
- svm_training_set.csv containing labeled, Gaussian-noise-perturbed training rows with the 4-feature vector (monoisotopic mass plus first three isotope-abundance ratios)

## How to apply

Enumerate sum formulas over the elements C, H, N, O, P, S in the 1-1000 Da mass window and apply the HiRes-style heuristic element filter, which keeps chemically plausible compositions (~24M formulas). Randomly sample 115,000 evenly spread compositions from this filtered enumeration space and compute each one's theoretical isotope pattern with the emass isotope-distribution library. For each isotope index j=1..5, fit linear models mu(j) (mean peak m/z-spacing vs. monoisotopic mass) and sigma(j) (spacing spread vs. monoisotopic mass) by linear regression on the computed patterns. Build SVM training instances from the 4-feature vector of monoisotopic mass plus the first three abundance ratios, label metabolite-like vs. non-metabolite patterns, and perturb features with Gaussian noise modeling a 5% RMS error. Emit the fitted mu(j)/sigma(j) coefficients as model_parameters.csv and the labeled training rows as svm_training_set.csv; these feed the SVM-based classifier inside FeatureFinderMetabo's feature-assembly scoring function.

## Related tools

- **OpenMS** (Framework in which the SVM-based metabolite isotope-pattern classifier is implemented; the generated training data underlies this classifier component) — https://github.com/OpenMS/OpenMS
- **FeatureFinderMetabo** (Consumer of the generated SVM training set — its scoring function uses the classifier to identify potential metabolite isotope patterns during feature assembly) — https://github.com/OpenMS/OpenMS

## Evaluation signals

- model_parameters.csv contains coefficients for all isotope indices j=1..5 (both mu(j) and sigma(j) per index)
- Expert review confirms the fitted spacing functions behave as linear functions of monoisotopic mass
- svm_training_set.csv contains the full set of labeled, noise-perturbed patterns drawn from the 115,000 sampled compositions
- Every training row carries exactly 4 features: monoisotopic mass plus the first three abundance ratios
- Downstream check: when embedded in FeatureFinderMetabo, the classifier identifies metabolite isotope patterns with sensitivity and specificity above 95% on typical data

## Limitations

- Training patterns are purely simulated (theoretical isotope distributions from emass), with Gaussian noise only approximating a 5% RMS measurement error — real instrument-specific deviation structure is not captured.
- The enumeration space is restricted to C, H, N, O, P, S in the 1-1000 Da window and further pruned by the HiRes heuristic element filter (~24M formulas), so chemically excluded compositions are unrepresented.
- Reference models cover only the first five isotope spacings and the first three abundance ratios; higher isotope indices or unusual charge regimes fall outside the modeled space.

## Evidence

- [abstract] assembled into features based on mass-to-charge spacing, co-elution information, and a support vector machine-based classifier able to identify potential metabolite isotope patterns: "assembled into features based on mass-to-charge spacing, co-elution information, and a support vector machine-based classifier able to identify potential metabolite isotope patterns"
- [discussion] An important component of the scoring function is the detection of likely metabolite profiles by means of statistical learning methods.: "An important component of the scoring function is the detection of likely metabolite profiles by means of statistical learning methods."
- [readme] It comes with a vast variety of pre-built and ready-to-use tools for proteomics and metabolomics data analysis (TOPPTools): [evidence span withheld — anchor status: external_doc]
