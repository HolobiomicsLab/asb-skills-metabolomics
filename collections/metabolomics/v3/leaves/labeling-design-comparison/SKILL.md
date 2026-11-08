---
name: labeling-design-comparison
description: Use when fitted flux estimates with confidence-interval bounds exist
  for two or more isotope labeling designs (single 13C, parallel 13C/15N, dual 13C15N)
  and the goal is to quantify which design reduces flux uncertainty.
when_to_use_negative:
- The dataset contains only one labeling design (e.g. a single 13C tracer experiment),
  so no cross-design CI comparison is possible
- Data come from nominal-resolution MS where dual 13C15N isotopologues cannot be resolved,
  making the dual-design CI gains unmeasurable
- The analysis goal is isotopologue distribution generation or natural abundance correction
  rather than flux confidence-interval assessment
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: INCA
  role: Performs INST-MFA and produces the fitted flux estimates with confidence-interval
    bounds that are compared across labeling designs
- name: SIMPEL
  role: R package whose data tables exported from HRMS isotopologue distributions
    feed INCA INST-MFA; its getClustersAndPlots(fit=...) interface consumes the fit
    results
  repo: github.com/SIMPELmetabolism/SIMPEL
- name: R
  role: Language used to compute CI widths, percentage reductions, and multivariate
    isotopologue counts and to write the summary CSV
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-024-05844-z
    title: 'SIMPEL: using stable isotopes to elucidate dynamics of context specific
      metabolism'
  source_content_hash: sha256:5db6336941d4ded1a3bdcc07e755e1d273dfc5225e66749bcff5b1d2da86dca4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - INCA
  - R
  - SIMPEL
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# labeling-design-comparison

## Summary

Quantifies how INST-MFA flux confidence intervals shrink under parallel 13C/15N and dual 13C15N labeling designs relative to single-13C MFA, using fitted INCA flux bounds exported from SIMPEL. It motivates HRMS dual-isotope designs by showing average CI-width reductions of ~71% and ~84%.

## When to use

Use when fitted flux estimates with confidence-interval bounds exist for two or more isotope labeling designs (single 13C, parallel 13C/15N, dual 13C15N) and the goal is to quantify which design reduces flux uncertainty.

## When NOT to use

- The dataset contains only one labeling design (e.g. a single 13C tracer experiment), so no cross-design CI comparison is possible
- Data come from nominal-resolution MS where dual 13C15N isotopologues cannot be resolved, making the dual-design CI gains unmeasurable
- The analysis goal is isotopologue distribution generation or natural abundance correction rather than flux confidence-interval assessment

## Inputs

- Fitted INST-MFA flux results from INCA for the single 13C labeling design (upper and lower CI bounds per flux)
- Fitted INST-MFA flux results from INCA for the parallel 13C/15N labeling design (upper and lower CI bounds per flux)
- Fitted INST-MFA flux results from INCA for the dual 13C15N labeling design (upper and lower CI bounds per flux)
- Molecular formula / labeled-atom counts (Ne per element) of metabolites such as [13C5,15N2] glutamine for isotopologue-count calculations

## Outputs

- flux_ci_width_reduction_summary.csv containing average CI width per labeling design, % CI-width reduction for the parallel 13C/15N and dual 13C15N designs vs single 13C, and glutamine multivariate isotopologue count
- Per-flux CI width comparison across the three designs

## How to apply

Load the fitted flux confidence intervals (upper and lower bounds) produced by INCA INST-MFA for each labeling design, e.g. the 'fit' results consumed through SIMPEL's getClustersAndPlots(fit=...) interface. For every flux estimate, compute the CI width as upper bound minus lower bound, then average widths per design. Calculate the percentage width reduction of the parallel 13C/15N design and the dual 13C15N design relative to the single 13C design; the expected results are ~71% (parallel) and ~84% (dual). As an additional design metric, compute the multivariate isotopologue count for a target metabolite (e.g. glutamine) via I = product of (Ne+1) over labeled elements, giving I = (5+1)*(2+1) = 18 for the dual 13C15N glutamine design. Write the summary table of average CI widths, % reductions, and measurement counts to flux_ci_width_reduction_summary.csv.

## Related tools

- **INCA** (Performs INST-MFA and produces the fitted flux estimates with confidence-interval bounds that are compared across labeling designs)
- **SIMPEL** (R package whose data tables exported from HRMS isotopologue distributions feed INCA INST-MFA; its getClustersAndPlots(fit=...) interface consumes the fit results) — github.com/SIMPELmetabolism/SIMPEL
- **R** (Language used to compute CI widths, percentage reductions, and multivariate isotopologue counts and to write the summary CSV)

## Examples

```
Rscript -e 'fit <- getClustersAndPlots(fit = inca_fit); ci <- fit$ci_upper - fit$ci_lower; write.csv(data.frame(design, mean_ci = tapply(ci, design, mean)), "flux_ci_width_reduction_summary.csv")'
```

## Evaluation signals

- Computed % CI-width reduction for the parallel 13C/15N design relative to single-13C MFA reproduces the reported ~71% decrease
- Computed % CI-width reduction for the dual 13C15N design relative to single-13C MFA reproduces the reported ~84% decrease
- Glutamine multivariate isotopologue count via I = product of (Ne+1) equals (5+1)*(2+1) = 18 for the dual 13C15N design
- Every flux CI width is non-negative (upper bound >= lower bound) within each design before averaging
- Summary table contains one average-CI-width row per labeling design and is written to flux_ci_width_reduction_summary.csv

## Limitations

- The CI reductions (71% / 84%) are averages across fluxes; individual flux estimates may improve more or less depending on identifiability under each design
- Dual 13C15N isotopologues must be resolved by HRMS; with nominal m/z resolution, dual-labeled tracers produce nominally identical mass shifts and the multi-isotope information is lost
- Comparisons assume the three designs' fits come from the same underlying model and labeling time-course (e.g. the 0-8 h [13C5,15N2] glutamine Arabidopsis root data)

## Evidence

- [results] On average, parallel 13C/15N labeling-based MFA and dual 13C15N labeling-based MFA reduced flux confidence intervals by 71% and 84%, respectively, compared to a single 13C-based MFA.: [evidence span withheld — anchor status: not_found]
- [intro] Data tables exported from SIMPEL were used for INST-MFA with INCA.: "Data tables exported from SIMPEL were used for INST-MFA with INCA."
- [intro] the number of multivariate isotopologues35 of a given compound (I) increases exponentially with the number of labeled atoms (N) for each element (e): "the number of multivariate isotopologues35 of a given compound (I) increases exponentially with the number of labeled atoms (N) for each element (e)"
- [results] all three datasets could be derived from a single 13C15N-glutamine labeling experiment by summing isotopologue abundances contained in the comprehensive HRMS dataset: "all three datasets could be derived from a single 13C15N-glutamine labeling experiment by summing isotopologue abundances contained in the comprehensive HRMS dataset"
