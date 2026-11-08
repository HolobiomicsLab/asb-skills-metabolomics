---
name: inst-mfa-output-processing
description: Use when INCA INST-MFA 'fit' objects (flux upper/lower CI bounds) exist
  for two or more isotope labeling designs and the goal is to compare flux confidence-interval
  widths and measurement counts across designs.
when_to_use_negative:
- Only a single labeling design's fit is available — no baseline exists for computing
  percentage CI-width reductions
- Fit results are from steady-state MFA or from nominal-resolution (non-HRMS) data,
  where dual-labeled 13C15N isotopologues cannot be distinguished
edam_operation: http://edamontology.org/operation_0306
tools:
- name: INCA
  role: Produces the fitted INST-MFA flux confidence intervals that are post-processed
    and compared across labeling designs
- name: SIMPEL
  role: R package whose interfaces (e.g. getClustersAndPlots(fit=...)) consume INCA
    fit results and whose exported isotopologue tables feed the INST-MFA runs
  repo: github.com/SIMPELmetabolism/SIMPEL
- name: R
  role: Environment in which CI widths, percentage reductions, and the summary CSV
    are computed (SIMPEL is an R package)
  repo: github.com/SIMPELmetabolism/SIMPEL
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-024-05844-z
    title: 'SIMPEL: using stable isotopes to elucidate dynamics of context specific
      metabolism'
  source_content_hash: sha256:9d27797b780bb1a283a482aa5a1c35af07c8080c621b24af2594fc876fdec833
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - INCA
  - R
  - SIMPEL
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# inst-mfa-output-processing

## Summary

Post-process INCA INST-MFA flux estimation results (fitted flux confidence intervals) in R to quantify how labeling design — single 13C vs parallel 13C/15N vs dual 13C15N — affects flux precision, reporting percentage CI-width reductions and isotopologue measurement counts.

## When to use

Use when INCA INST-MFA 'fit' objects (flux upper/lower CI bounds) exist for two or more isotope labeling designs and the goal is to compare flux confidence-interval widths and measurement counts across designs.

## When NOT to use

- Only a single labeling design's fit is available — no baseline exists for computing percentage CI-width reductions
- Fit results are from steady-state MFA or from nominal-resolution (non-HRMS) data, where dual-labeled 13C15N isotopologues cannot be distinguished

## Inputs

- INCA INST-MFA fit results with flux confidence-interval upper and lower bounds for a single 13C labeling design
- INCA INST-MFA fit results for the parallel 13C/15N labeling design
- INCA INST-MFA fit results for the dual 13C15N labeling design
- Isotopologue distribution tables exported from SIMPEL (from the [13C5,15N2] glutamine labeling experiment)

## Outputs

- flux_ci_width_reduction_summary.csv containing average CI width per design, % reduction for parallel (target ~71%) and dual (target ~84%) designs, and the glutamine measurement count (18)
- Per-flux CI width estimates for each labeling design

## How to apply

Load the fitted flux confidence intervals (upper and lower bounds) produced by INCA INST-MFA for each labeling design (single 13C, parallel 13C/15N, dual 13C15N), e.g. the 'fit' results consumed by SIMPEL's getClustersAndPlots(fit=...) interface. For every flux estimate, compute the CI width as upper bound minus lower bound, average widths per design, and calculate the percentage width reduction of the parallel 13C/15N and dual 13C15N designs relative to the single 13C design. Additionally, compute the multivariate isotopologue count for tracers such as [13C5,15N2] glutamine as I = product of (Ne+1) over labeled elements, giving I = (5+1)*(2+1) = 18 for the dual design — HRMS is required because nominal-resolution MS cannot resolve such dual-labeled isotopologues. Write the summary table (average CI width per design, % reduction, glutamine measurement count) to flux_ci_width_reduction_summary.csv.

## Related tools

- **INCA** (Produces the fitted INST-MFA flux confidence intervals that are post-processed and compared across labeling designs)
- **SIMPEL** (R package whose interfaces (e.g. getClustersAndPlots(fit=...)) consume INCA fit results and whose exported isotopologue tables feed the INST-MFA runs) — github.com/SIMPELmetabolism/SIMPEL
- **R** (Environment in which CI widths, percentage reductions, and the summary CSV are computed (SIMPEL is an R package)) — github.com/SIMPELmetabolism/SIMPEL

## Examples

```
widths <- fit$upperBound - fit$lowerBound; write.csv(data.frame(design, meanWidth, pctReduction), "flux_ci_width_reduction_summary.csv")
```

## Evaluation signals

- The parallel 13C/15N design's average CI-width reduction must reproduce the reported ~71% decrease relative to the single 13C design
- The dual 13C15N design's average CI-width reduction must reproduce the reported ~84% decrease relative to the single 13C design
- The glutamine multivariate isotopologue count must equal I = (5+1)*(2+1) = 18
- Every flux estimate must have lower bound ≤ upper bound so CI widths are non-negative
- The summary table must contain all three designs (single 13C, parallel 13C/15N, dual 13C15N) with consistent flux identifiers

## Limitations

- Reduction values depend on the specific tracer and time-course (Arabidopsis roots labeled with [13C5,15N2] glutamine, 0–8 h); they are not universal across experiments
- Dual-labeled isotopologue counts are only resolvable with HRMS, since nominal m/z resolution cannot separate nominally identical dual-isotope mass shifts
- No software versions for INCA or SIMPEL are documented, so numerical reproducibility across versions is unverified

## Evidence

- [intro] Data tables exported from SIMPEL were used for INST-MFA with INCA.: "Data tables exported from SIMPEL were used for INST-MFA with INCA."
- [intro] Isotopologue count formula: "the number of multivariate isotopologues35 of a given compound (I) increases exponentially with the number of labeled atoms (N) for each element (e) of the molecular formula"
- [results] Dual-dataset derivation: "all three datasets could be derived from a single 13C15N-glutamine labeling experiment by summing isotopologue abundances contained in the comprehensive HRMS dataset"
- [intro] HRMS necessity: "High resolution mass spectrometry (HRMS) can distinguish metabolites (and isotopologues of heteroatoms) through accurate measurement of m/z values that differ by <0.01 Da"
- [results] Labeling experiment: "ten-day-old Arabidopsis roots were grown on a predefined carbon and nitrogen rich medium68 supplemented with [13C515N2] glutamine (Fig. 1a) and sampled over time (0–8 h)"
