---
name: bonferroni-multiple-testing-correction
description: Use when many correlated biomarkers (e.g., 226 Nightingale NMR metabolomics
  measures) are simultaneously tested and an explicit family-wise alpha split (0.05
  divided by the number of tests) is needed to declare significance or prune a model.
when_to_use_negative:
- The biomarker panel or hypothesis count differs from 226 — the threshold must be
  recomputed as 0.05 divided by the actual number of tests, not copied as 2.21 x 10^-4
- False discovery rate control (e.g., FDR/q-values) is the desired error metric rather
  than family-wise error control, since Bonferroni is conservative for correlated
  biomarkers
- Testing a small pre-specified set of hypotheses where no multiplicity correction
  was planned
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: R-package meta
  role: Performs the fixed-effect meta-analyses whose pooled P values are compared
    against the 2.21 x 10^-4 Bonferroni threshold in each backward-removal round
- name: R
  role: Environment in which the threshold computation, stepwise selection loop, and
    significance filtering are implemented
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-11311-9
    title: A metabolic profile of all-cause mortality risk identified in an observational
      study of 44,168 individuals
  source_content_hash: sha256:23314edb7c9a95c9daf2496a888c546bb24c87dc44d75d008d1b332d4068bfff
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - R-package meta
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# bonferroni-multiple-testing-correction

## Summary

Applies Bonferroni family-wise error control across the 226 NMR metabolic biomarkers tested for association with all-cause mortality, yielding a significance threshold of 2.21 x 10^-4 (0.05/226) that governs both initial significance calls and the backward-removal step of the forward-backward biomarker selection.

## When to use

Use when many correlated biomarkers (e.g., 226 Nightingale NMR metabolomics measures) are simultaneously tested and an explicit family-wise alpha split (0.05 divided by the number of tests) is needed to declare significance or prune a model.

## When NOT to use

- The biomarker panel or hypothesis count differs from 226 — the threshold must be recomputed as 0.05 divided by the actual number of tests, not copied as 2.21 x 10^-4
- False discovery rate control (e.g., FDR/q-values) is the desired error metric rather than family-wise error control, since Bonferroni is conservative for correlated biomarkers
- Testing a small pre-specified set of hypotheses where no multiplicity correction was planned

## Inputs

- Cohort-level Cox proportional hazards summary statistics for 226 metabolic biomarkers (deposited in the BBMRI-omics atlas)
- Per-biomarker P values from fixed-effect meta-analyses at each round of the forward-backward procedure
- Number of tested biomarkers (226) used to derive 0.05/226 = 2.21 x 10^-4

## Outputs

- Bonferroni-adjusted significance threshold (2.21 x 10^-4)
- Set of biomarkers with P < 2.21 x 10^-4 (basis of the 63-biomarker candidate subset)
- Final set of 14 independently associated biomarkers retained after backward removal at the threshold, with HR, 95% CI, P, I2 and P(het)

## How to apply

First count the total number of hypotheses tested — here the 226 metabolic biomarkers profiled on the Nightingale Health NMR platform — and compute the threshold as 0.05/226 = 2.21 x 10^-4. Apply this threshold in two places: (1) to flag biomarkers as significant in the basic Cox survival meta-analysis (age at blood sampling as time scale, adjusted for sex and study-specific covariates, combined via fixed-effect meta-analysis in the R-package meta), and (2) during the backward step of the stepwise forward-backward procedure, where after each forward addition any biomarker in the working model whose P value rises above 2.21 x 10^-4 is removed. Candidate additions come only from the prespecified 63-biomarker subset (not all 226) to reduce overfitting risk. Iterate forward/backward rounds of meta-analysis until every unselected biomarker has P above the threshold, converging on 14 independently associated biomarkers. The rationale is strict family-wise control because the 226 biomarkers are highly correlated and hundreds of simultaneous tests would otherwise yield many false positives.

## Related tools

- **R-package meta** (Performs the fixed-effect meta-analyses whose pooled P values are compared against the 2.21 x 10^-4 Bonferroni threshold in each backward-removal round)
- **R** (Environment in which the threshold computation, stepwise selection loop, and significance filtering are implemented)

## Examples

```
threshold <- 0.05 / 226  # 2.21e-4 Bonferroni threshold; keep <- subset(meta_results, p.value < threshold)
```

## Evaluation signals

- Computed threshold equals exactly 0.05 divided by the number of tested biomarkers (2.21 x 10^-4 for 226 biomarkers)
- Backward step removes exactly those biomarkers whose meta-analysis P value rose above 2.21 x 10^-4 after the preceding forward addition (citrate drops out due to limited additional contribution)
- Procedure converges to exactly 14 independently associated biomarkers
- Retained biomarkers' HRs and 95% CIs match published Table 2 values (e.g., GlycA HR 1.32, 95% CI 1.27-1.38; XXL-VLDL-L HR 0.80, 95% CI 0.75-0.85)
- No biomarker remaining in the final model has P above the threshold, and every unselected candidate has P above the threshold at termination

## Limitations

- The threshold is tied to the 226-biomarker panel; it is not transferable to panels with a different number of measures without recomputation
- Bonferroni control is conservative for the highly correlated NMR biomarkers, and selection was deliberately restricted to a 63-biomarker subset to prevent overfitting, so some significant biomarkers are excluded from selection
- Per-cohort scaling and log-transformation of biomarkers (x+1 for zero-containing biomarkers) must precede testing or P values will not be comparable to published results

## Evidence

- [discussion] Biomarkers were considered significant when the P value was below the Bonferroni-adjusted threshold of 2.21 x 10^-4 (0.05/226): "Biomarkers were considered significant when the P value was below the Bonferroni-adjusted threshold of 2.21 × 10−4 (0.05/226)"
- [discussion] 63-biomarker candidate subset rationale: "the selection of independently associated biomarkers was based on a subset of 63 biomarkers to prevent overfitting"
- [discussion] Stepwise procedure applying the threshold in successive meta-analysis rounds: "we performed a stepwise (forward-backward) procedure based on successive rounds of meta-analyses"
- [abstract] 14 biomarkers identified as outcome of thresholded selection: "We apply a stepwise (forward-backward) procedure based on meta-analysis results and identify 14 circulating biomarkers independently associating with all-cause mortality."
- [discussion] Preprocessing required before thresholded testing: [evidence span withheld — anchor status: not_found]
