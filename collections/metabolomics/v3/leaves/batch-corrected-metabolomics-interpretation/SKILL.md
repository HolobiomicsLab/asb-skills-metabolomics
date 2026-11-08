---
name: batch-corrected-metabolomics-interpretation
description: Use when a batch-corrected metabolomics abundance matrix (e.g. hRUV-normalized
  from >1,000 plasma samples run across multiple batches) is available together with
  sample-level phenotype annotations, and the goal is to identify metabolites significantly
  differentially abundant with that phenotype.
when_to_use_negative:
- Input data are raw or only drift-corrected and have not been through hRUV hierarchical
  normalization — differential calls may be confounded by unwanted variation between
  batches
- Small-scale metabolomics with no embedded inter-batch sample replicates, since hRUV
  relies on replicate design (batch replicates, short replicates, pooled QC) to estimate
  unwanted variation
- The question is about batch-effect estimation itself rather than downstream phenotype
  association testing
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: hRUV
  role: Produces the batch-corrected (normalised) metabolite abundance matrix that
    the limma differential-abundance model is fit to; available as an R package and
    shiny application
  repo: https://github.com/SydneyBioX/hRUV
- name: limma
  role: Fits the linear model with hypertension status as covariate per metabolite,
    performs empirical Bayes moderation, and supplies Benjamini-Hochberg adjusted
    p-values
- name: R
  role: Analysis environment in which the limma differential expression workflow on
    hRUV-normalized data is executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2020.12.21.423723
    title: 'hRUV: Hierarchical approach to removal of unwanted variation for large-scale
      metabolomics data'
  source_content_hash: sha256:5beb0bfb94e4ec6dbb39d5cdd1be9febed333ed58f25d8984840b8446538b344
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/hRUV
  tools:
  - R
  - hRUV
  - limma
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# batch-corrected-metabolomics-interpretation

## Summary

Runs differential-abundance analysis (limma with empirical Bayes moderation and Benjamini-Hochberg correction) on hRUV-normalized large-scale metabolomics data to test whether known biology, such as hypertension-associated metabolites, is recovered only after hierarchical batch correction.

## When to use

Use when a batch-corrected metabolomics abundance matrix (e.g. hRUV-normalized from >1,000 plasma samples run across multiple batches) is available together with sample-level phenotype annotations, and the goal is to identify metabolites significantly differentially abundant with that phenotype.

## When NOT to use

- Input data are raw or only drift-corrected and have not been through hRUV hierarchical normalization — differential calls may be confounded by unwanted variation between batches
- Small-scale metabolomics with no embedded inter-batch sample replicates, since hRUV relies on replicate design (batch replicates, short replicates, pooled QC) to estimate unwanted variation
- The question is about batch-effect estimation itself rather than downstream phenotype association testing

## Inputs

- hRUV-normalised metabolite abundance matrix (samples x metabolites)
- Sample-level hypertension status annotations (phenotype covariate table)
- Optionally, abundance matrices normalized by comparison methods (13 publicly available metabolomics normalization methods) for specificity checks

## Outputs

- Ranked table of metabolites with limma test statistics and Benjamini-Hochberg adjusted p-values (hypertension_de_results.csv)
- Set of significant differential-abundance hits associated with hypertension status

## How to apply

Load the hRUV-normalised metabolite abundance matrix together with the sample-level hypertension status annotations. For each metabolite, fit a limma linear model with hypertension status as the covariate, then apply limma's empirical Bayes moderation and Benjamini-Hochberg correction to obtain adjusted p-values per metabolite. Rank metabolites by adjusted p-value and extract the significant differential-abundance hits. Interpretation is validated biologically: in this study the significant hits should include proline, valine, cyclic adenosine monophosphate and dimethyl guanidino valeric acid among over 1,000 patients, and this recovery should be compared against the same limma analysis on data normalized by other methods, where only subsets of these metabolites were detected — demonstrating that hRUV retains biological signal while removing unwanted variation.

## Related tools

- **hRUV** (Produces the batch-corrected (normalised) metabolite abundance matrix that the limma differential-abundance model is fit to; available as an R package and shiny application) — https://github.com/SydneyBioX/hRUV
- **limma** (Fits the linear model with hypertension status as covariate per metabolite, performs empirical Bayes moderation, and supplies Benjamini-Hochberg adjusted p-values)
- **R** (Analysis environment in which the limma differential expression workflow on hRUV-normalized data is executed)

## Examples

```
fit <- eBayes(lmFit(hRUV_norm_matrix, design = model.matrix(~ hypertension_status))); de <- topTable(fit, number = Inf, adjust = "BH"); write.csv(de, "hypertension_de_results.csv")
```

## Evaluation signals

- Proline, valine, cyclic adenosine monophosphate and dimethyl guanidino valeric acid are each flagged as significantly differentially abundant with hypertension status in the ranked results table
- The four known hypertension metabolites are significant in hRUV-normalized data while other normalization methods identify only subsets — a specificity check across the 13 compared normalization methods
- Every adjusted p-value derives from limma's empirical Bayes moderation followed by Benjamini-Hochberg correction across metabolites
- Only metabolites present in at least 50% of samples enter the differential-abundance analysis (downstream of QC-based outlier removal)
- Full ranked results table is written out (e.g. hypertension_de_results.csv) rather than only top hits

## Limitations

- Recovery of the expected hypertension metabolites depends on prior hRUV normalization quality; unintended clustering of batch replicates between batches (e.g. batches 6 and 7, and 13 and 14) left a slight shift in signal for selected metabolites and limited the ability to capture unwanted variability across the whole plate
- hRUV expects data without missing values, which is often not possible in large-scale metabolomics and requires an imputation workaround before this differential analysis
- Selection of appropriate negative-control metabolites is challenging because metabolite signals depend on individual chemical properties, which can influence the corrected matrix feeding this analysis

## Evidence

- [discussion] hRUV retains biological signal for downstream differential testing: "hRUV performs consistently better than them in retaining biological variation whilst at the same time removing unwanted variation within and across batches."
- [discussion] Coverage filter applied before analysis: "metabolites that were not present in at least 50% of the samples were filtered out"
- [discussion] QC-based outlier removal prior to normalization: "We examined the three consecutive QC samples embedded at the start of each plate and removed any outlying measurements."
- [intro] Batch correction step underlying the corrected matrix: "estimation of unwanted variation within and between batches with RUV-III12"
- [readme] Package provides the normalization workflow used before interpretation: "`hRUV` is a package for normalisation of multiple batches of metabolomics data in a hierarchical strategy with use of samples replicates in large-scale studies."
