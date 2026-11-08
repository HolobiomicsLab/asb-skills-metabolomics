---
name: scaling-method-selection-multivariate-data
description: Use when fitting a multivariate discriminant model (e.g., KODAMA-based
  PLS-DA) on a lipidomic intensity matrix at the lipid molecular species (LMS) level
  and a scaling scheme among Autoscaling, Pareto, or None must be chosen before validation.
when_to_use_negative:
- The analysis is purely univariate (e.g., single-lipid comparisons) where scaling
  has no discriminant-model role
- The intensity matrix has already been scaled, so a second scaling pass would distort
  component loadings
- The target structural level is lipid classes (LCL) or lipid building blocks (LBB)
  and their tailored statistics rather than the LMS-level PLS-DA workflow
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: LipidOne 2.0
  role: Web platform that exposes the scaling choice (Autoscaling, Pareto, None) and
    runs the LMS-level PLS-DA workflow with validation bar graphs
  repo: https://lipidone.eu
- name: KODAMA
  role: R package used to fit the PLS-DA discriminant model on the scaled molecular
    species matrix and run the permutation-based cross-validation
- name: R
  role: Back-end scripting environment in which the scaled matrix is modeled and validation
    statistics are computed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/cpz1.70009
    title: 'LipidOne 2.0: A Web Tool for Discovering Biological Meanings Hidden in
      Lipidomic Data'
  source_content_hash: sha256:d6ba6e0e63f1283d30db08637ed3566590442ec132fd29b4b4ec76cf56de69a0
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
yes_rate: 1.0
n_claims: 9
---

# scaling-method-selection-multivariate-data

## Summary

Selects and applies a user-chosen data scaling scheme (Autoscaling, Pareto, or None) to a lipid molecular species intensity matrix before multivariate discriminant analysis, so that PLS-DA metrics (Accuracy, R2Y, Q2Y) computed downstream are comparable across the first five PLS components.

## When to use

Use when fitting a multivariate discriminant model (e.g., KODAMA-based PLS-DA) on a lipidomic intensity matrix at the lipid molecular species (LMS) level and a scaling scheme among Autoscaling, Pareto, or None must be chosen before validation.

## When NOT to use

- The analysis is purely univariate (e.g., single-lipid comparisons) where scaling has no discriminant-model role
- The intensity matrix has already been scaled, so a second scaling pass would distort component loadings
- The target structural level is lipid classes (LCL) or lipid building blocks (LBB) and their tailored statistics rather than the LMS-level PLS-DA workflow

## Inputs

- Lipidomic data matrix restricted to the lipid molecular species (LMS) structural level
- User-selected scaling option (Autoscaling, Pareto, or None)
- Case–control group labels for the experimental groups included in the analysis

## Outputs

- Scaled molecular species intensity matrix used for PLS-DA fitting
- PLS-DA score plot and loading plot from the fitted model
- Validation bar graph of Accuracy, R2Y, and Q2Y across the first five PLS components

## How to apply

Restrict the analysis to the lipid molecular species structural level of the data matrix, then apply the user-selected scaling method — Autoscaling, Pareto, or None — to the molecular species intensity matrix, since each structural level in LipidOne 2.0 carries tailored statistics. Fit the PLS-DA discriminant model on the scaled matrix using the KODAMA package in R. Validate with the permutation-based cross-validation scheme of 100 cross-validations with permutated samples over 10 cross-validation loops. Compute Accuracy, R2Y, and Q2Y for each of the first five PLS components and visualize them as a validation bar graph alongside the score and loading plots; scaling choice should be justified by how these metrics behave across components rather than by a fixed numeric threshold.

## Related tools

- **LipidOne 2.0** (Web platform that exposes the scaling choice (Autoscaling, Pareto, None) and runs the LMS-level PLS-DA workflow with validation bar graphs) — https://lipidone.eu
- **KODAMA** (R package used to fit the PLS-DA discriminant model on the scaled molecular species matrix and run the permutation-based cross-validation)
- **R** (Back-end scripting environment in which the scaled matrix is modeled and validation statistics are computed)

## Evaluation signals

- The validation bar graph displays Accuracy, R2Y, and Q2Y values for exactly the first five PLS components
- Cross-validation statistics come from 100 cross-validations with permutated samples over 10 cross-validation loops
- The scaling method applied matches one of the three permitted options (Autoscaling, Pareto, None) and was applied to the molecular species intensity matrix
- Score plot and loading plot render without errors from the same fitted model (qualitative acceptance check)
- Downstream metrics are computed on data restricted to the lipid molecular species level, not on class- or building-block-level matrices

## Limitations

- The section reports no numeric acceptance threshold for the validation bar graph — acceptance is a qualitative check that plots render and metrics appear for the first five components
- Scaling interacts with the structural level: analyses at each of the three lipid levels require their own tailored statistics, so a scaling choice tuned for the LMS level may not transfer to LCL or LBB analyses
- The desktop LipidOne repository README documents a standalone Perl/Windows implementation for building-block analysis, which differs from the LipidOne 2.0 web backend in R; do not assume the README's prerequisites (Windows 64-bit, Gnuplot) apply to the web tool

## Evidence

- [methods] scaling options and LMS workflow: [evidence span withheld — anchor status: not_found]
- [methods] cross-validation scheme: [evidence span withheld — anchor status: not_found]
- [abstract] three structural levels: "It facilitates the exploration of the three structural levels of lipids: classes, molecular species, and lipid building blocks (acyl, alkyl, or alkenes chains)."
- [readme] desktop tool scope (README): "LipidOne is a freely available and easy-to-use software to discover the deeper features of building blocks in a lipidomic dataset."
