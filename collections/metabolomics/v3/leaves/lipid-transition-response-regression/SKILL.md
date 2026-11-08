---
name: lipid-transition-response-regression
description: Use when targeted SRM/PRM peak areas exported from Skyline for LipidCreator-computed
  transitions must be checked for linearity against analyte amount, e.g. verifying
  PUFA-PL ion signatures in a human platelet lipid matrix before releasing a transition
  list as an assay.
when_to_use_negative:
- The goal is generating or curating the transition list itself (fragment masses,
  collision energies, isotope labeling) rather than validating response linearity
  — use LipidCreator's assay-design workflow instead
- The response data are fragment-specific dissociation profiles requiring nonlinear
  (log-normal kernel) modeling rather than a linear response-versus-amount check
- Assay development is untargeted lipid screening rather than verification of a specific
  LipidCreator-targeted transition set in a defined matrix
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: LipidCreator
  role: Computes the targeted transition list (ion signatures) whose MS response is
    being regression-verified; fully integrated with Skyline for PRM and MRM experiments
  repo: github.com/lifs-tools/lipidcreator
- name: Skyline
  role: Source of the peak-area response data for the transitions; transition results
    are exported from Skyline for import into R
  repo: https://skyline.ms/project/home/software/Skyline/begin.view
- name: flipR
  role: Nonlinear regression framework used to fit fragment-specific response models
    (extended log-normal kernel) per headgroup
- name: minpack.lm
  role: Provides the Levenberg-Marquardt nonlinear least-squares solver underlying
    the regression fits
  repo: https://rdrr.io/cran/minpack.lm/
- name: nls.multstart
  role: Performs bounded grid-search optimization of the regression start parameters
  repo: https://rdrr.io/cran/nls.multstart/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-15960-z
    title: LipidCreator workbench to probe the lipidomic landscape
  source_content_hash: sha256:0f3b4cb0cef08b6a60a1f677797cd28fd420669871757ce34e1b696dd07a832a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidCreator
  - Skyline
  - flipR
  - minpack.lm
  - nls.multstart
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# lipid-transition-response-regression

## Summary

Regresses Skyline peak-area response versus amount for LipidCreator-computed lipid transitions (e.g. the 18:0–20:4 fatty acyl combination across PA, PC, PE, PG, PI, and PS) to verify that the computed ion signatures yield a linear MS response, fitting per-headgroup models with flipR/minpack.lm/nls.multstart and averaging Pearson correlation coefficients (expected ≈ 0.99).

## When to use

Use when targeted SRM/PRM peak areas exported from Skyline for LipidCreator-computed transitions must be checked for linearity against analyte amount, e.g. verifying PUFA-PL ion signatures in a human platelet lipid matrix before releasing a transition list as an assay.

## When NOT to use

- The goal is generating or curating the transition list itself (fragment masses, collision energies, isotope labeling) rather than validating response linearity — use LipidCreator's assay-design workflow instead
- The response data are fragment-specific dissociation profiles requiring nonlinear (log-normal kernel) modeling rather than a linear response-versus-amount check
- Assay development is untargeted lipid screening rather than verification of a specific LipidCreator-targeted transition set in a defined matrix

## Inputs

- LipidCreator-generated transition list (PRM/MRM targets and fragments, optionally exported as .blib spectral library)
- Skyline peak-area response data for the transitions, acquired e.g. by LC/ESI SRM on a QTRAP 6500 or PRM on a Q-Exactive HF
- Per-lipid-class amount/level annotations (calibration series) in the biological matrix

## Outputs

- Per-lipid-class linear regression fits of peak-area response versus amount
- Pearson correlation coefficients for the linear response region of each lipid class (e.g. platelet_response_correlations.csv)
- Average correlation coefficient across classes compared against the reported reference value of 0.99

## How to apply

Load the LipidCreator-generated transition list for the fatty acyl combination of interest (e.g. 18:0–20:4 across PA, PC, PE, PG, PI, PS) together with the Skyline peak-area response data from the matrix (e.g. human platelet lipid extract). Import the Skyline transition results into R and organize fragment-specific response values by lipid class/headgroup. For each headgroup, fit a linear model of peak-area response versus amount using the flipR nonlinear-regression framework with the minpack.lm Levenberg-Marquardt solver and nls.multstart bounded grid-search optimization, restricting to the linear response region. Compute the Pearson correlation coefficient per lipid class, average the coefficients (six classes → reported average 0.99), and record per-class coefficients (e.g. in platelet_response_correlations.csv).

## Related tools

- **LipidCreator** (Computes the targeted transition list (ion signatures) whose MS response is being regression-verified; fully integrated with Skyline for PRM and MRM experiments) — github.com/lifs-tools/lipidcreator
- **Skyline** (Source of the peak-area response data for the transitions; transition results are exported from Skyline for import into R) — https://skyline.ms/project/home/software/Skyline/begin.view
- **flipR** (Nonlinear regression framework used to fit fragment-specific response models (extended log-normal kernel) per headgroup)
- **minpack.lm** (Provides the Levenberg-Marquardt nonlinear least-squares solver underlying the regression fits) — https://rdrr.io/cran/minpack.lm/
- **nls.multstart** (Performs bounded grid-search optimization of the regression start parameters) — https://rdrr.io/cran/nls.multstart/

## Examples

```
Rscript -e 'library(minpack.lm); fit <- nlsLM(area ~ a*amount, data=pc_response, start=list(a=1)); cor(pc_response$amount, fitted(fit))'
```

## Evaluation signals

- Average Pearson correlation coefficient across the six lipid classes (PA, PC, PE, PG, PI, PS) reproduces the reported value of 0.99
- Every analyzed lipid class individually shows a linear response for peak area versus amount
- Per-class coefficients are recorded in platelet_response_correlations.csv with values consistent with the Skyline peak-area data
- Residuals of the per-headgroup linear fits in the linear response region show no systematic curvature (justifying use of the linear region only)

## Limitations

- LipidCreator's collision energy prediction models are currently exclusive to lipid mediators; extending them to other lipid classes requires more structurally diverse lipid standards and CE-dependent fragmentation measurements
- LipidCreator does not provide a dedicated scoring algorithm for false discovery rates and intentionally does not address coeluting/isobaric lipids and peak area integration
- Retention time, elution order, and isotope-labeled standards must be considered to avoid mis-annotations; no large-scale method exists to handle stereoisomers due to the lack of isotope-coded standards
- Linearity was demonstrated for the 18:0–20:4 fatty acyl combination in human platelet matrix; other fatty acyl combinations or matrices require revalidation

## Evidence

- [discussion] flipR regression framework: "Nonlinear Regression Model: flipR uses nonlinear regression with an extended log-normal kernel to fit fragment-specific dissociation profiles"
- [discussion] minpack.lm Levenberg-Marquardt: "We use the R-packages minpack.lm (https://rdrr.io/cran/minpack.lm/), for nonlinear regression based on the Levenberg-Marquardt nonlinear least squares algorithm)"
- [discussion] nls.multstart grid search: "and nls.multstart (https://rdrr.io/cran/nls.multstart/) to perform a bounded grid search optimization"
- [readme] LipidCreator is a Skyline plugin for targeted lipidomics workflows: "LipidCreator is a plugin for [Skyline](https://skyline.ms/project/home/software/Skyline/begin.view) supporting targeted workflow development in lipidomics."
- [discussion] Integration with Skyline makes assay development vendor-independent: "LipidCreator is well integrated into Skyline for small molecules, which makes it a vendor-independent software for fast assay development in targeted lipidomics."
