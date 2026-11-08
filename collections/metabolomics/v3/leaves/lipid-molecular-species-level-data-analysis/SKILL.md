---
name: lipid-molecular-species-level-data-analysis
description: Use when a lipidomic intensity matrix must be analyzed at the lipid molecular
  species level and a validated supervised discrimination model (PLS-DA) with score,
  loading, and validation outputs is required.
when_to_use_negative:
- The analysis target is the lipid class (LCL) or lipid building block (LBB) structural
  level rather than lipid molecular species
- The goal is unsupervised exploration or pathway enrichment rather than validated
  supervised discrimination of case-control groups
- No experimental group labels exist, so permutation-based cross-validation of a discriminant
  model cannot be performed
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3306
tools:
- name: LipidOne 2.0
  role: Web platform that restricts the data matrix to the lipid molecular species
    level, applies the chosen scaling, and drives the PLS-DA analysis with validation
    plots
  repo: https://lipidone.eu
- name: KODAMA
  role: R package used by the PLS-DA function to fit the discriminant model and run
    the permutation-based cross-validation scheme
- name: R
  role: Back-end scripting environment in which the KODAMA-based PLS-DA cross-validation
    executes
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/cpz1.70009
    title: 'LipidOne 2.0: A Web Tool for Discovering Biological Meanings Hidden in
      Lipidomic Data'
  source_content_hash: sha256:09688ccc1f79c1dbaa52fc4effadd6cff795ac432cf06564a74661613da3eb8f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KODAMA
  - LipidOne 2.0
  - R
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3306
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# lipid-molecular-species-level-data-analysis

## Summary

Performs supervised multivariate analysis (PLS-DA) on a lipidomic data matrix restricted to the lipid molecular species (LMS) structural level, using KODAMA-based permutation cross-validation (100 cross-validations over 10 loops) to validate Accuracy, R2Y, and Q2Y across the first five PLS components.

## When to use

Use when a lipidomic intensity matrix must be analyzed at the lipid molecular species level and a validated supervised discrimination model (PLS-DA) with score, loading, and validation outputs is required.

## When NOT to use

- The analysis target is the lipid class (LCL) or lipid building block (LBB) structural level rather than lipid molecular species
- The goal is unsupervised exploration or pathway enrichment rather than validated supervised discrimination of case-control groups
- No experimental group labels exist, so permutation-based cross-validation of a discriminant model cannot be performed

## Inputs

- Lipidomic data matrix containing lipid molecular species intensities (e.g., uploaded via LipidOne 2.0 'Load Data Matrix')
- Experimental group/case-control labels for supervised PLS-DA discrimination
- User-selected data scaling method choice (Autoscaling, Pareto, or None)

## Outputs

- PLS-DA score plot for the molecular species level
- PLS-DA loading plot
- Validation statistics table: Accuracy, R2Y, and Q2Y for each of the first five PLS components
- Validation bar graph comparing Accuracy, R2Y, and Q2Y across the first five components

## How to apply

Restrict the analysis to the lipid molecular species structural level of the data matrix, excluding lipid class (LCL) and lipid building block (LBB) views. Apply one of the user-selected scaling methods — Autoscaling, Pareto, or None — to the molecular species intensity matrix. Fit a PLS-DA discriminant model on the scaled matrix using the KODAMA package in R via LipidOne 2.0. Run the permutation-based cross-validation scheme with 100 cross-validations over 10 cross-validation loops. Compute Accuracy, R2Y, and Q2Y for each of the first five PLS components, then generate the PLS-DA score plot, loading plot, and the validation bar graph comparing Accuracy, R2Y, and Q2Y across those five components.

## Related tools

- **LipidOne 2.0** (Web platform that restricts the data matrix to the lipid molecular species level, applies the chosen scaling, and drives the PLS-DA analysis with validation plots) — https://lipidone.eu
- **KODAMA** (R package used by the PLS-DA function to fit the discriminant model and run the permutation-based cross-validation scheme)
- **R** (Back-end scripting environment in which the KODAMA-based PLS-DA cross-validation executes)

## Evaluation signals

- The validation bar graph displays Accuracy, R2Y, and Q2Y for exactly the first five PLS components
- Cross-validation used exactly 100 cross-validations with permutated samples over 10 cross-validation loops
- PLS-DA score plot and loading plot render without errors (qualitative acceptance check)
- The molecular species intensity matrix was scaled with one of the three allowed methods (Autoscaling, Pareto, or None) before model fitting
- No numeric threshold is reported for validation in this section, so acceptance is qualitative rather than cutoff-based

## Limitations

- The section reports no numeric validation threshold; acceptance of the model output is only a qualitative check that plots render and the bar graph covers five components
- The desktop LipidOne README describes a Perl/Windows stand-alone predecessor (GPL-era FreeBSD-licensed, Gnuplot-based), whereas the validated PLS-DA workflow runs in the LipidOne 2.0 web/R back-end — do not assume the desktop binary reproduces it
- No installability/deployment information (Docker, conda, source install) for LipidOne 2.0 is provided, and no software license for LipidOne 2.0 is stated

## Evidence

- [other] PLS-DA cross-validation scheme with 100 permutations over 10 loops: [evidence span withheld — anchor status: not_found]
- [abstract] Three structural levels including molecular species: "It facilitates the exploration of the three structural levels of lipids: classes, molecular species, and lipid building blocks (acyl, alkyl, or alkenes chains)."
- [abstract] Mono- and multivariate analyses tailored to each structural level: "LipidOne 2.0 offers a range of mono- and multivariate statistical analyses, specifically tailored to each structural level."
- [discussion] Web-based back-end implemented with R scripts: "coding its back-end interac- tions with the R scripts"
