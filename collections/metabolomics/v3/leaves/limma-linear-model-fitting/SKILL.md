---
name: limma-linear-model-fitting
description: Use when a normalised metabolite abundance matrix (metabolites × samples)
  is paired with a sample-level binary phenotype such as hypertension status, and
  per-metabolite differential-abundance hits with adjusted p-values are required.
when_to_use_negative:
- Input is raw or un-normalised metabolite abundance data — the hypertension associations
  were recoverable only from hRUV-normalised data, so normalise first (e.g. with the
  hRUV package)
- The analysis goal is batch/drift correction itself rather than downstream phenotype
  testing — use the hRUV hierarchical normalisation workflow instead of a limma model
- The input has missing values and no completed imputation/normalisation step has
  been applied, since the upstream hRUV algorithm expects data without missing values
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: limma
  role: Fits the per-metabolite linear model with hypertension status as covariate,
    performs empirical Bayes moderation, and supplies Benjamini-Hochberg adjusted
    p-values
- name: R
  role: Environment in which the limma model fitting and result-table export are executed
- name: hRUV
  role: Produces the normalised metabolite abundance matrix that serves as the input
    to the limma differential-abundance model
  repo: github.com/SydneyBioX/hRUV
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2020.12.21.423723
    title: 'hRUV: Hierarchical approach to removal of unwanted variation for large-scale
      metabolomics data'
  source_content_hash: sha256:aa2137d9e45bb9a993433b77b6bec53df58239a6169e75ce34678e867338e688
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - hRUV
  - limma
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# limma-linear-model-fitting

## Summary

Differential-abundance testing of metabolomics data by fitting a per-metabolite limma linear model (with empirical Bayes moderation and Benjamini-Hochberg correction) against a clinical covariate such as hypertension status, applied to hRUV-normalised metabolite abundance matrices from large-scale plasma studies.

## When to use

Use when a normalised metabolite abundance matrix (metabolites × samples) is paired with a sample-level binary phenotype such as hypertension status, and per-metabolite differential-abundance hits with adjusted p-values are required.

## When NOT to use

- Input is raw or un-normalised metabolite abundance data — the hypertension associations were recoverable only from hRUV-normalised data, so normalise first (e.g. with the hRUV package)
- The analysis goal is batch/drift correction itself rather than downstream phenotype testing — use the hRUV hierarchical normalisation workflow instead of a limma model
- The input has missing values and no completed imputation/normalisation step has been applied, since the upstream hRUV algorithm expects data without missing values

## Inputs

- hRUV-normalised metabolite abundance matrix (metabolites × samples) from the targeted LC-MS/MS plasma assay
- Sample-level hypertension status annotations for the BioHEART-CT discovery cohort patients

## Outputs

- Ranked differential-abundance results table (limma logFC, empirical-Bayes moderated statistics, Benjamini-Hochberg adjusted p-values per metabolite), written to hypertension_de_results.csv
- Set of significant differential-abundance hits for the hypertension response variable

## How to apply

Load the hRUV-normalised metabolite abundance matrix together with the sample-level hypertension status annotations. For each metabolite, fit a limma linear model with hypertension status as the covariate, then apply limma's empirical Bayes moderation and Benjamini-Hochberg correction to obtain adjusted p-values per metabolite. Rank metabolites by adjusted p-value and extract the significant differential-abundance hits. As a biological sanity check on the BioHEART-CT discovery cohort data (over 1,000 human plasma samples run over 44 days), verify that proline, valine, cyclic adenosine monophosphate and dimethyl guanidino valeric acid appear among the significant hits, and write the full results table to hypertension_de_results.csv. The rationale for running this only on hRUV-normalised data is that hRUV preserves biological signal while removing unwanted variation, which is what makes the hypertension associations recoverable.

## Related tools

- **limma** (Fits the per-metabolite linear model with hypertension status as covariate, performs empirical Bayes moderation, and supplies Benjamini-Hochberg adjusted p-values)
- **R** (Environment in which the limma model fitting and result-table export are executed)
- **hRUV** (Produces the normalised metabolite abundance matrix that serves as the input to the limma differential-abundance model) — github.com/SydneyBioX/hRUV

## Examples

```
Rscript -e 'fit <- limma::eBayes(limma::lmFit(hruv_norm_mat, design=model.matrix(~hypertension_status))); res <- limma::topTable(fit, number=Inf, adjust.method="BH"); write.csv(res, "hypertension_de_results.csv")'
```

## Evaluation signals

- Proline, valine, cyclic adenosine monophosphate and dimethyl guanidino valeric acid are each flagged as significantly differentially abundant with hypertension status
- All four reported metabolites are recovered on hRUV-normalised data — comparisons against other normalisation methods should recover only subsets, so a match to the hRUV result is expected
- Every metabolite row carries a Benjamini-Hochberg adjusted p-value derived from limma's empirical Bayes moderation (no missing statistics)
- Metabolites are ranked by adjusted p-value and the full table is exported as hypertension_de_results.csv

## Limitations

- Recovery of the known hypertension-associated metabolites was specific to hRUV-normalised data — other normalisation methods identified only subsets, so the limma result is conditional on the normalisation quality
- Selection of appropriate negative-control metabolites for the upstream batch correction is challenging because metabolite signals depend on their individual chemical properties
- The upstream hRUV algorithm expects data without missing values, which is often not possible in large-scale metabolomics and requires an imputation workaround

## Evidence

- [methods] limma covariate fitting: [evidence span withheld — anchor status: not_found]
- [results] expected significant hits: [evidence span withheld — anchor status: not_found]
- [abstract] why hRUV input matters: "We demonstrate significant improvement of hRUV over existing methods in maintaining biological signals whilst removing unwanted variation for large scale metabolomics studies"
- [discussion] p-value ranking per batch: "ranking non-differentially expressed metabolites by p-values per batch for the hypertension response variable"
- [readme] R runtime tested: [evidence span withheld — anchor status: external_doc]
