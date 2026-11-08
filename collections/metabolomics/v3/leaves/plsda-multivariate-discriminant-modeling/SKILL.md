---
name: plsda-multivariate-discriminant-modeling
description: Use when a lipidomic intensity matrix at the lipid molecular species
  (LMS) level must be discriminated between case-control experimental groups with
  supervised multivariate modeling, and permutated cross-validation statistics (Accuracy,
  R2Y, Q2Y) are needed.
when_to_use_negative:
- The analysis targets the lipid class (LCL) or lipid building block (LBB) structural
  level rather than lipid molecular species — LipidOne 2.0 tailors analyses to each
  structural level separately.
- An unsupervised exploration is needed (no case-control group labels), since PLS-DA
  is a supervised discriminant method.
- The tool's flexibility allows including or excluding experimental groups and lipid
  classes, so exclude groups/classes outside the contrast of interest before fitting
  rather than passing unfiltered data.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: LipidOne 2.0
  role: Web platform whose PLS-DA module performs supervised discriminant analysis
    of lipid molecular species with permutation-based cross-validation and renders
    score, loading, and validation-bar plots
  repo: https://lipidone.eu
- name: R
  role: Back-end language in which the PLS-DA discriminant model is coded and executed
- name: KODAMA
  role: R package used to fit the PLS-DA model and run the 100-permutation / 10-loop
    cross-validation scheme
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/cpz1.70009
    title: 'LipidOne 2.0: A Web Tool for Discovering Biological Meanings Hidden in
      Lipidomic Data'
  source_content_hash: sha256:4abb6147288da3021bf9058929a1ea718c78c63a95ef4f4b41d03db2eccb5bf5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KODAMA
  - LipidOne 2.0
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# plsda-multivariate-discriminant-modeling

## Summary

Supervised PLS-DA discriminant modeling of lipidomic data as implemented in LipidOne 2.0: a PLS-DA model fitted with the R KODAMA package on scaled lipid molecular species data, validated by permutation-based cross-validation (100 cross-validations with permutated samples over 10 loops) and reported as Accuracy, R2Y, and Q2Y for the first five PLS components.

## When to use

Use when a lipidomic intensity matrix at the lipid molecular species (LMS) level must be discriminated between case-control experimental groups with supervised multivariate modeling, and permutated cross-validation statistics (Accuracy, R2Y, Q2Y) are needed.

## When NOT to use

- The analysis targets the lipid class (LCL) or lipid building block (LBB) structural level rather than lipid molecular species — LipidOne 2.0 tailors analyses to each structural level separately.
- An unsupervised exploration is needed (no case-control group labels), since PLS-DA is a supervised discriminant method.
- The tool's flexibility allows including or excluding experimental groups and lipid classes, so exclude groups/classes outside the contrast of interest before fitting rather than passing unfiltered data.

## Inputs

- Lipidomic data matrix restricted to the lipid molecular species (LMS) structural level
- Case-control experimental group labels (user-selectable include/exclude groups)
- User-selected scaling choice: Autoscaling, Pareto, or None

## Outputs

- PLS-DA score plot
- PLS-DA loading plot
- Validation bar graph of Accuracy, R2Y, and Q2Y for the first five PLS components
- Permutation-based cross-validation statistics (100 cross-validations, 10 loops)

## How to apply

Restrict the analysis to the lipid molecular species structural level of the data matrix, then apply the user-selected scaling method — Autoscaling, Pareto, or None — to the molecular species intensity matrix before fitting. Fit the PLS-DA discriminant model on the scaled data using the KODAMA package in R, as LipidOne 2.0's back-end does. Run the permutation-based cross-validation scheme with 100 cross-validations with permutated samples over 10 cross-validation loops to obtain validation statistics. Compute Accuracy, R2Y, and Q2Y for each of the first five PLS components, then generate the PLS-DA score plot and loading plot from the fitted model and a validation bar graph comparing the three statistics across those five components. Acceptance is qualitative: the bar graph must display Accuracy, R2Y, and Q2Y for exactly the first five components and the score/loading plots must render without errors — the section reports no numeric threshold.

## Related tools

- **LipidOne 2.0** (Web platform whose PLS-DA module performs supervised discriminant analysis of lipid molecular species with permutation-based cross-validation and renders score, loading, and validation-bar plots) — https://lipidone.eu
- **R** (Back-end language in which the PLS-DA discriminant model is coded and executed)
- **KODAMA** (R package used to fit the PLS-DA model and run the 100-permutation / 10-loop cross-validation scheme)

## Evaluation signals

- The validation bar graph displays Accuracy, R2Y, and Q2Y for exactly the first five PLS components
- Cross-validation statistics come from 100 cross-validations with permutated samples over 10 cross-validation loops
- The score plot and loading plot render without errors after fitting
- The molecular species intensity matrix was scaled with the selected method (Autoscaling, Pareto, or None) before model fitting
- No numeric validation threshold is defined in the source section — acceptance is a qualitative render/content check only

## Limitations

- Validation acceptance is qualitative (plots render, bar graph shows the three metrics for five components); the source reports no numeric threshold for model quality.
- Cross-validation results depend on the user-chosen scaling (Autoscaling, Pareto, or None), which changes R2Y/Q2Y/Accuracy values.
- The desktop LipidOne predecessor is Windows-only Perl software; LipidOne 2.0 itself is a web tool accessed via https://lipidone.eu.

## Evidence

- [methods] PLS-DA cross-validation scheme: [evidence span withheld — anchor status: not_found]
- [abstract] Structural-level tailoring: "LipidOne 2.0 offers a range of mono- and multivariate statistical analyses, specifically tailored to each structural level."
- [readme] Tool availability: "LipidOne is a freely available and easy-to-use software to discover the deeper features of building blocks in a lipidomic dataset."
