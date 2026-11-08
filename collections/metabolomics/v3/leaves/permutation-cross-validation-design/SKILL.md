---
name: permutation-cross-validation-design
description: Use when validating a supervised PLS-DA discriminant model built on lipid
  molecular species (LMS) intensity data and you need permutation-based cross-validation
  statistics (Accuracy, R2Y, Q2Y) per PLS component to assess classification robustness.
when_to_use_negative:
- The data matrix is not at the lipid molecular species level — LipidOne 2.0 analyses
  are tailored separately to lipid class (LCL), molecular species (LMS), and building
  block (LBB) levels, and this validation scheme was surfaced at the LMS level
- You need an unsupervised or purely univariate lipidomic analysis rather than a supervised
  discriminant model validation
- You are working with the older standalone desktop Lipid One (Perl/Windows, Prima/Gnuplot
  GUI) described in the README rather than the LipidOne 2.0 web tool that implements
  this KODAMA-based validation
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: LipidOne 2.0
  role: Web platform that runs the PLS-DA analysis at the LMS level and executes the
    100-permutation / 10-loop cross-validation scheme, generating Accuracy, R2Y, Q2Y,
    score plot, loading plot, and validation bar graph
  repo: https://lipidone.eu
- name: KODAMA
  role: R package used to fit the PLS-DA discriminant model on the scaled molecular
    species data
- name: R
  role: Back-end scripting environment for LipidOne 2.0's statistical computations
  repo: github.com/matteogiulietti/LipidOne
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/cpz1.70009
    title: 'LipidOne 2.0: A Web Tool for Discovering Biological Meanings Hidden in
      Lipidomic Data'
  source_content_hash: sha256:add4d53b1fe873064a48648d0f5ed6fad3c156d8af60d809eb257ae9ca0f9ea3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KODAMA
  - LipidOne 2.0
  - R
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# permutation-cross-validation-design

## Summary

Design and execute a permutation-based cross-validation scheme for supervised PLS-DA models on lipidomic intensity data, as implemented in LipidOne 2.0 with the KODAMA R package: 100 cross-validations with permutated samples over 10 cross-validation loops, reporting Accuracy, R2Y, and Q2Y for the first five PLS components.

## When to use

Use when validating a supervised PLS-DA discriminant model built on lipid molecular species (LMS) intensity data and you need permutation-based cross-validation statistics (Accuracy, R2Y, Q2Y) per PLS component to assess classification robustness.

## When NOT to use

- The data matrix is not at the lipid molecular species level — LipidOne 2.0 analyses are tailored separately to lipid class (LCL), molecular species (LMS), and building block (LBB) levels, and this validation scheme was surfaced at the LMS level
- You need an unsupervised or purely univariate lipidomic analysis rather than a supervised discriminant model validation
- You are working with the older standalone desktop Lipid One (Perl/Windows, Prima/Gnuplot GUI) described in the README rather than the LipidOne 2.0 web tool that implements this KODAMA-based validation

## Inputs

- Lipidomic data matrix restricted to the lipid molecular species (LMS) structural level
- Experimental group labels for case-control comparisons
- User-selected scaling option (Autoscaling, Pareto, or None) for the molecular species intensity matrix

## Outputs

- Accuracy, R2Y, and Q2Y values for each of the first five PLS components
- Validation bar graph comparing Accuracy, R2Y, and Q2Y across the first five components
- PLS-DA score plot
- PLS-DA loading plot

## How to apply

Restrict the analysis to the lipid molecular species structural level of the data matrix, since LipidOne 2.0 tailors its statistics to the classes, molecular species, and building-block levels. Apply the user-selected scaling method — Autoscaling, Pareto, or None — to the molecular species intensity matrix before fitting. Fit the PLS-DA discriminant model using the KODAMA package in R, then run the LipidOne 2.0 validation scheme: 100 cross-validations with permutated samples and 10 cross-validation loops. For each of the first five PLS components, compute Accuracy, R2Y, and Q2Y, and render the PLS-DA score plot, loading plot, and a validation bar graph comparing the three metrics across components 1–5. The rationale is that permutated-sample cross-validation provides an honest measure of model predictivity that overfitting-prone PLS-DA would otherwise inflate.

## Related tools

- **LipidOne 2.0** (Web platform that runs the PLS-DA analysis at the LMS level and executes the 100-permutation / 10-loop cross-validation scheme, generating Accuracy, R2Y, Q2Y, score plot, loading plot, and validation bar graph) — https://lipidone.eu
- **KODAMA** (R package used to fit the PLS-DA discriminant model on the scaled molecular species data)
- **R** (Back-end scripting environment for LipidOne 2.0's statistical computations) — github.com/matteogiulietti/LipidOne

## Evaluation signals

- The validation bar graph displays Accuracy, R2Y, and Q2Y for exactly the first five PLS components (no more, no fewer)
- The cross-validation parameters match the protocol: 100 cross-validations with permutated samples and 10 cross-validation loops
- The PLS-DA score plot and loading plot render without errors (qualitative acceptance check; the section reports no numeric threshold)
- The fitted model was built on the molecular species matrix after one of the three allowed scaling options (Autoscaling, Pareto, None), not an unscaled or differently preprocessed matrix

## Limitations

- The acceptance criterion for the validation graphics is qualitative only — the section reports no numeric threshold on Accuracy, R2Y, or Q2Y
- The GitHub README documents the earlier standalone Lipid One written in Perl for 64-bit Microsoft Windows using Prima and Gnuplot; it does not describe the LipidOne 2.0 web tool or its KODAMA-based cross-validation, so version provenance of the code must be checked carefully
- No statement of whether the LipidOne 2.0 source code itself is publicly hosted is present in the section text, complicating independent re-implementation of the exact CV scheme

## Evidence

- [methods] Permutation CV parameters: [evidence span withheld — anchor status: not_found]
- [abstract] Structural levels: "It facilitates the exploration of the three structural levels of lipids: classes, molecular species, and lipid building blocks (acyl, alkyl, or alkenes chains)."
- [full_text] Scaling options: [evidence span withheld — anchor status: not_found]
- [discussion] R backend: "coding its back-end interac- tions with the R scripts"
- [readme] README version mismatch: "LipidOne is a stand alone software entirely written in Perl and implemented for the Microsoft Windows operating system."
