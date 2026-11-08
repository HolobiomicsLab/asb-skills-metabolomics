---
name: score-loading-plot-visualization
description: Use when a case-control lipidomic dataset has been scaled and fitted
  with a PLS-DA model and you need to visualize sample separation (score plot), variable
  contribution (loading plot), and cross-validated Accuracy/R2Y/Q2Y for the first
  five components.
when_to_use_negative:
- The analysis goal is univariate testing or lipid biomarker identification rather
  than supervised multivariate discrimination — LipidOne 2.0 offers these as separate
  analyses tailored per structural level
- The data matrix has not yet been restricted to the LMS level and scaled, so no fitted
  PLS-DA model exists to plot from
- You are using the standalone desktop Lipid One (Perl/Windows build described in
  the README) rather than LipidOne 2.0's web back-end, whose plot and cross-validation
  behavior is what this skill documents
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3353
tools:
- name: LipidOne 2.0
  role: Web platform that fits the PLS-DA model on the scaled LMS matrix and renders
    the score plot, loading plot, and validation bar graph
  repo: https://lipidone.eu
- name: KODAMA
  role: R package used inside LipidOne 2.0's back-end to fit the PLS-DA discriminant
    model and run the permutation-based cross-validation
- name: R
  role: Back-end scripting language encoding LipidOne 2.0's statistical analyses
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/cpz1.70009
    title: 'LipidOne 2.0: A Web Tool for Discovering Biological Meanings Hidden in
      Lipidomic Data'
  source_content_hash: sha256:c3148ab2be6a0cb12ae426b18bb7c2ca1d262d5327839e80c96b94d7bf3c2621
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KODAMA
  - LipidOne 2.0
  - R
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3353
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# score-loading-plot-visualization

## Summary

Renders and interprets the PLS-DA score plot and loading plot from a fitted lipidomics discriminant model in LipidOne 2.0, pairing them with a validation bar graph of Accuracy, R2Y, and Q2Y across the first five PLS components so model separation and predictive quality can be judged together.

## When to use

Use when a case-control lipidomic dataset has been scaled and fitted with a PLS-DA model and you need to visualize sample separation (score plot), variable contribution (loading plot), and cross-validated Accuracy/R2Y/Q2Y for the first five components.

## When NOT to use

- The analysis goal is univariate testing or lipid biomarker identification rather than supervised multivariate discrimination — LipidOne 2.0 offers these as separate analyses tailored per structural level
- The data matrix has not yet been restricted to the LMS level and scaled, so no fitted PLS-DA model exists to plot from
- You are using the standalone desktop Lipid One (Perl/Windows build described in the README) rather than LipidOne 2.0's web back-end, whose plot and cross-validation behavior is what this skill documents

## Inputs

- Lipid molecular species (LMS) level intensity matrix from the uploaded lipidomic dataset
- Experimental group/case-control labels for samples
- User-selected scaling method (Autoscaling, Pareto, or None)
- Fitted PLS-DA model from the KODAMA package in R

## Outputs

- PLS-DA score plot showing sample separation on latent components
- PLS-DA loading plot showing lipid molecular species contributions
- Validation bar graph comparing Accuracy, R2Y, and Q2Y across the first five PLS components

## How to apply

First restrict the analysis to the lipid molecular species structural level (LMS) of the data matrix and apply the user-selected scaling method — Autoscaling, Pareto, or None — to the molecular species intensity matrix. Fit the PLS-DA discriminant model using the KODAMA package in R as implemented by LipidOne 2.0's back-end. The model's validation is not a single split: LipidOne 2.0 runs 100 cross-validations with permutated samples over 10 cross-validation loops. From the fitted model, generate the score plot (sample separation) and loading plot (which lipid molecular species drive the separation). Finally, draw the validation bar graph comparing Accuracy, R2Y, and Q2Y for each of the first five PLS components; comparing Q2Y against R2Y across components is how overfitting is assessed, since validation is permutation-based.

## Related tools

- **LipidOne 2.0** (Web platform that fits the PLS-DA model on the scaled LMS matrix and renders the score plot, loading plot, and validation bar graph) — https://lipidone.eu
- **KODAMA** (R package used inside LipidOne 2.0's back-end to fit the PLS-DA discriminant model and run the permutation-based cross-validation)
- **R** (Back-end scripting language encoding LipidOne 2.0's statistical analyses)

## Evaluation signals

- The bar graph displays Accuracy, R2Y, and Q2Y values for exactly the first five PLS components — no more, no fewer
- Score plot and loading plot render without errors from the fitted KODAMA PLS-DA model (qualitative acceptance check; the section reports no numeric threshold)
- Cross-validation statistics reflect 100 permutations over 10 loops, matching LipidOne 2.0's scheme
- The scaling method actually applied (Autoscaling, Pareto, or None) matches the user's selection before model fitting
- Q2Y is compared against R2Y across components in the bar graph so overfitting is visible

## Limitations

- The section reports only a qualitative acceptance check (plots render, bar graph shows the first five components) with no numeric threshold for Accuracy, R2Y, or Q2Y
- The GitHub repository README describes the standalone desktop Lipid One written in Perl for Microsoft Windows — a different artifact from the LipidOne 2.0 web back-end used here
- Example datasets (ERYTHROCYTES, FUNGUS, SCIATIC NERVE) are only accessible through the web platform, not a persistent repository accession

## Evidence

- [results] PLS-DA fit and CV scheme: [evidence span withheld — anchor status: not_found]
- [methods] Model fitting via KODAMA: [evidence span withheld — anchor status: not_found]
- [methods] Plots to generate: [evidence span withheld — anchor status: not_found]
- [methods] Validation bar graph check: [evidence span withheld — anchor status: not_found]
- [abstract] Level-tailored analyses: "LipidOne 2.0 offers a range of mono- and multivariate statistical analyses, specifically tailored to each structural level."
- [readme] Desktop artifact caveat: "LipidOne is a stand alone software entirely written in Perl and implemented for the Microsoft Windows operating system."
