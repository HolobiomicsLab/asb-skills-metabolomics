---
name: pearson-correlation-computation
description: Use when you need to verify that a targeted-lipidomics assay produces
  a linear response — e.g., after importing Skyline peak-area results for a transition
  list and fitting response-versus-amount regressions per lipid class.
when_to_use_negative:
- Data are already normalized or the relationship is nonlinear (e.g., saturating signal
  at high amounts) — a plain Pearson r over the full range would understate fit quality.
- The transition list has not been generated or peak areas have not been exported
  from Skyline; no response values exist to correlate.
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: flipR
  role: Nonlinear-regression framework used to fit fragment-specific response profiles
    and derive the linear correlation coefficients
- name: minpack.lm
  role: Levenberg-Marquardt nonlinear least-squares solver backing the regression
    fits
  repo: https://rdrr.io/cran/minpack.lm/
- name: nls.multstart
  role: Bounded grid-search optimization to find starting values for the regression
    fits
  repo: https://rdrr.io/cran/nls.multstart/
- name: LipidCreator
  role: Generates the transition list whose MS response linearity is being validated
  repo: github.com/lifs-tools/lipidcreator
- name: Skyline
  role: Produces the peak-area response data that serve as the correlated input
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-15960-z
    title: LipidCreator workbench to probe the lipidomic landscape
  source_content_hash: sha256:fd37217d66d2fd5320bcbb280d89e54ac61ae5842416cc7479ce4bb25ca916c5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidCreator
  - Skyline
  - flipR
  - minpack.lm
  - nls.multstart
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# pearson-correlation-computation

## Summary

Computes Pearson correlation coefficients to quantify the linear relationship between analyte peak-area response and amount, used here to validate that LipidCreator-computed SRM transitions for PUFA-PL species (18:0–20:4 in PA, PC, PE, PG, PI, PS) yield a linear MS response (reported average r = 0.99).

## When to use

Use when you need to verify that a targeted-lipidomics assay produces a linear response — e.g., after importing Skyline peak-area results for a transition list and fitting response-versus-amount regressions per lipid class.

## When NOT to use

- Data are already normalized or the relationship is nonlinear (e.g., saturating signal at high amounts) — a plain Pearson r over the full range would understate fit quality.
- The transition list has not been generated or peak areas have not been exported from Skyline; no response values exist to correlate.

## Inputs

- Skyline peak-area response data for LipidCreator-generated transitions (18:0–20:4 species in PA, PC, PE, PG, PI, PS) measured by LC/ESI-SRM in human platelet matrix
- LipidCreator transition list defining the targeted fragment entries

## Outputs

- Per-lipid-class Pearson correlation coefficients (platelet_response_correlations.csv)
- Average correlation coefficient across the six classes (benchmark ≈ 0.99)
- Fitted linear regression models of response versus amount per headgroup

## How to apply

Import Skyline transition results into R and organize fragment-specific peak-area response values by lipid class (PA, PC, PE, PG, PI, PS). Fit a linear regression of peak-area response versus amount for each headgroup using the flipR nonlinear-regression framework with the minpack.lm Levenberg-Marquardt solver and nls.multstart bounded grid-search optimization, restricting to the linear response region. Compute the Pearson correlation coefficient (r) for each class and average the six coefficients; compare against the reported benchmark of 0.99 for the 18:0–20:4 PUFA-PL series. Persist per-class coefficients (e.g., in platelet_response_correlations.csv) so the linearity evidence can be audited per transition.

## Related tools

- **flipR** (Nonlinear-regression framework used to fit fragment-specific response profiles and derive the linear correlation coefficients)
- **minpack.lm** (Levenberg-Marquardt nonlinear least-squares solver backing the regression fits) — https://rdrr.io/cran/minpack.lm/
- **nls.multstart** (Bounded grid-search optimization to find starting values for the regression fits) — https://rdrr.io/cran/nls.multstart/
- **LipidCreator** (Generates the transition list whose MS response linearity is being validated) — github.com/lifs-tools/lipidcreator
- **Skyline** (Produces the peak-area response data that serve as the correlated input)

## Examples

```
Rscript -e 'library(minpack.lm); library(nls.multstart); fit <- nlsLM(response ~ a * amount + b, data = platelet_transitions, start = list(a = 1, b = 0)); cor(platelet_transitions$response, predict(fit))'
```

## Evaluation signals

- Average Pearson r across the six lipid classes ≈ 0.99 (reported value).
- Each per-class r is close to 1 (linear response region); any class with r notably below the average signals a non-linear or poorly fitting transition.
- platelet_response_correlations.csv contains one coefficient per lipid class and the fitted regression parameters are reproducible.
- Regression residuals show no systematic curvature within the linear response region.

## Limitations

- Linear response is assumed; LipidCreator CE-dependent dissociation behavior is modeled only for lipid mediators, so the linear-response claim rests on the measured platelet data rather than on a universally predictive model.
- Correlation alone does not address coeluting/isobaric lipids, peak-area integration errors, or retention-time mis-annotation — those must be handled upstream.
- Isotope labeled standards and retention-time/elution-order information are needed to avoid mis-annotations before computing r.

## Evidence

- [results] Linear MS response for PUFA-PL species: [evidence span withheld — anchor status: not_found]
- [discussion] flipR regression framework: "Nonlinear Regression Model: flipR uses nonlinear regression with an extended log-normal kernel to fit fragment-specific dissociation profiles"
- [discussion] minpack.lm solver: "We use the R-packages minpack.lm (https://rdrr.io/cran/minpack.lm/), for nonlinear regression based on the Levenberg-Marquardt nonlinear least squares algorithm)"
- [discussion] nls.multstart optimization: "and nls.multstart (https://rdrr.io/cran/nls.multstart/) to perform a bounded grid search optimization"
- [discussion] LipidCreator integration with Skyline: "LipidCreator is well integrated into Skyline for small molecules, which makes it a vendor-independent software for fast assay development in targeted lipidomics."
- [discussion] Caveat on mis-annotation controls: "additional information such as retention time, the elution order of lipids, as well as isotope labeled standards, if available, always need to be considered in order to avoid mis-annotations."
