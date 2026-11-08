---
name: multiple-testing-fdr-correction
description: Use when a limma linear model has been fit per metabolite across a large-scale
  metabolomics cohort (e.g. BioHEART-CT plasma samples) and you need adjusted p-values
  to call significant hypertension-associated metabolites.
when_to_use_negative:
- The hypothesis concerns a single pre-specified metabolite, so there is no multiple-testing
  family to correct
- The input is a raw or un-normalised feature matrix and no per-metabolite p-values
  exist yet (normalisation, e.g. hRUV, must precede testing)
- The goal is adaptive negative-control selection by ranking non-differentially expressed
  metabolites by p-values per batch, not FDR-controlled significance calling
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0091
tools:
- name: limma
  role: Fits the per-metabolite linear model with hypertension status as covariate,
    applies empirical Bayes moderation, and produces the raw p-values that are then
    BH-adjusted
- name: hRUV
  role: Produces the normalised metabolite matrix whose differential-abundance testing
    the FDR correction is applied to; the paper reports recovery of known hypertension
    metabolites only in hRUV-normalized data
  repo: github.com/SydneyBioX/hRUV
- name: R
  role: Language in which limma model fitting, empirical Bayes moderation, and p.adjust/BH
    correction are executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2020.12.21.423723
    title: 'hRUV: Hierarchical approach to removal of unwanted variation for large-scale
      metabolomics data'
  source_content_hash: sha256:666ce9919eba02f9db697d1306d99f290f5298acfae7e0c238d7ecb754b3ca2c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - hRUV
  - limma
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# multiple-testing-fdr-correction

## Summary

Benjamini-Hochberg false-discovery-rate correction of per-metabolite p-values from limma differential-abundance testing on hRUV-normalised metabolomics data, controlling the expected fraction of false positives when hundreds of metabolites are tested simultaneously against a phenotype such as hypertension status.

## When to use

Use when a limma linear model has been fit per metabolite across a large-scale metabolomics cohort (e.g. BioHEART-CT plasma samples) and you need adjusted p-values to call significant hypertension-associated metabolites.

## When NOT to use

- The hypothesis concerns a single pre-specified metabolite, so there is no multiple-testing family to correct
- The input is a raw or un-normalised feature matrix and no per-metabolite p-values exist yet (normalisation, e.g. hRUV, must precede testing)
- The goal is adaptive negative-control selection by ranking non-differentially expressed metabolites by p-values per batch, not FDR-controlled significance calling

## Inputs

- hRUV-normalised metabolite abundance matrix (samples x metabolites)
- Sample-level hypertension status annotations
- Per-metabolite raw p-values from limma linear models with hypertension status as covariate

## Outputs

- Benjamini-Hochberg adjusted p-values (q-values) per metabolite
- Ranked differential-abundance results table (hypertension_de_results.csv)
- Set of significant differential-abundance hits at the chosen FDR threshold

## How to apply

For each metabolite in the hRUV-normalised abundance matrix, fit a limma linear model with hypertension status as the covariate, apply limma's empirical Bayes moderation, then compute Benjamini-Hochberg adjusted p-values across all metabolites to control the FDR over the family of tests. Rank metabolites by adjusted p-value and extract the significant differential-abundance hits rather than thresholding raw p-values, since the paper's validation accepts metabolites only when flagged significant after this correction. Before testing, ensure metabolites absent from at least 50% of samples were filtered out so the tested family matches the paper's design. The corrected results table is written out (hypertension_de_results.csv) and used to check recovery of known hypertension-associated metabolites.

## Related tools

- **limma** (Fits the per-metabolite linear model with hypertension status as covariate, applies empirical Bayes moderation, and produces the raw p-values that are then BH-adjusted)
- **hRUV** (Produces the normalised metabolite matrix whose differential-abundance testing the FDR correction is applied to; the paper reports recovery of known hypertension metabolites only in hRUV-normalized data) — github.com/SydneyBioX/hRUV
- **R** (Language in which limma model fitting, empirical Bayes moderation, and p.adjust/BH correction are executed)

## Examples

```
fit <- eBayes(lmFit(hruv_normalized_matrix, model.matrix(~ hypertension_status))); adj.p <- p.adjust(fit$p.value, method="BH")
```

## Evaluation signals

- All four reported metabolites (proline, valine, cyclic adenosine monophosphate, dimethyl guanidino valeric acid) appear among the significant hits after BH correction, consistent with the paper's Results
- Adjusted p-values are in [0,1] and are monotone (non-decreasing) relative to the raw limma p-values
- The number of significant metabolites at BH FDR threshold does not exceed the number significant by raw p-value threshold (correction can only shrink the hit list)
- Recovery of the four metabolites is specific to hRUV-normalized data: other normalization methods identified only subsets of the four
- The full ranked results table (hypertension_de_results.csv) contains adjusted p-values for every metabolite retained after the 50%-presence filter

## Limitations

- Metabolites not present in at least 50% of the samples are filtered out before testing, so the multiple-testing family is limited to detectable metabolites
- Selection of appropriate negative-control metabolites for batch correction is challenging because metabolite signals depend on their individual chemical properties, affecting which tests enter the family
- hRUV expects data without missing values, often requiring imputation before the downstream limma testing and FDR correction are valid

## Evidence

- [other] The workflow specifies limma testing and BH correction on hRUV-normalised data.: [evidence span withheld — anchor status: not_found]
- [abstract] Only hRUV-normalised data recovers all four known hypertension metabolites as significant, defining the expected validation outcome of the corrected test.: "We demonstrate significant improvement of hRUV over existing methods in maintaining biological signals whilst removing unwanted variation for large scale metabolomics studies"
- [discussion] Pre-filtering defines the family of metabolites tested before FDR correction.: "metabolites that were not present in at least 50% of the samples were filtered out"
- [discussion] Per-metabolite p-values are the input to the multiple-testing correction and are also used per batch for negative-control ranking.: "ranking non-differentially expressed metabolites by p-values per batch for the hypertension response variable"
