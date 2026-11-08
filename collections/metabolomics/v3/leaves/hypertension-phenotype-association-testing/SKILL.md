---
name: hypertension-phenotype-association-testing
description: Use when hRUV-normalised metabolite abundance data (e.g. BioHEART-CT
  plasma samples) is paired with sample-level hypertension status and you need to
  identify differentially abundant metabolites associated with that phenotype.
when_to_use_negative:
- Data is raw, un-normalised abundance — normalize first with hRUV; the article shows
  un-normalised or other normalization methods recover only subsets of known hypertension
  metabolites
- The phenotype of interest is not hypertension status and has not been shown to have
  hRUV-recoverable associations — the skill's validation criterion is recovery of
  known hypertension-associated metabolites
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: limma
  role: Fits the linear model with hypertension status as covariate, applies empirical
    Bayes moderation, and computes Benjamini-Hochberg adjusted p-values per metabolite.
- name: hRUV
  role: Produces the normalised abundance matrix that is the input to the differential
    expression test; this skill specifically validates hRUV output.
  repo: github.com/SydneyBioX/hRUV
- name: R
  role: Implementation environment for the limma model and the hRUV package.
  repo: github.com/SydneyBioX/hRUV
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2020.12.21.423723
    title: 'hRUV: Hierarchical approach to removal of unwanted variation for large-scale
      metabolomics data'
  source_content_hash: sha256:38226654d4ab9ea61f33572fec6ec01da2901d205f63a636f31916b612316370
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - hRUV
  - limma
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# hypertension-phenotype-association-testing

## Summary

Tests metabolite abundance against hypertension status using a limma linear model with empirical Bayes moderation and Benjamini-Hochberg correction on hRUV-normalised metabolomics data, to recover known hypertension-associated metabolites as significant hits.

## When to use

Use when hRUV-normalised metabolite abundance data (e.g. BioHEART-CT plasma samples) is paired with sample-level hypertension status and you need to identify differentially abundant metabolites associated with that phenotype.

## When NOT to use

- Data is raw, un-normalised abundance — normalize first with hRUV; the article shows un-normalised or other normalization methods recover only subsets of known hypertension metabolites
- The phenotype of interest is not hypertension status and has not been shown to have hRUV-recoverable associations — the skill's validation criterion is recovery of known hypertension-associated metabolites

## Inputs

- hRUV-normalised metabolite abundance matrix (e.g. from LC-MS/MS Qtrap5500 measurements integrated by MultiQuant)
- Sample-level metadata table with hypertension status per patient from the BioHEART-CT cohort

## Outputs

- Ranked differential-abundance results table with per-metabolite log2 fold change, moderated t-statistic, raw p-value, and Benjamini-Hochberg adjusted p-value
- List of significantly differentially abundant metabolites (e.g. written to hypertension_de_results.csv)

## How to apply

Load the hRUV-normalised metabolite abundance matrix together with sample-level hypertension status annotations. For each metabolite, fit a limma linear model with hypertension status as the covariate, then apply limma's empirical Bayes moderation. Correct p-values with Benjamini-Hochberg and rank metabolites by adjusted p-value to extract significant differential-abundance hits. The rationale is that differential expression testing is the article's validation criterion for whether hRUV preserves biological signal — only hRUV-normalised data recovered proline, valine, cyclic adenosine monophosphate and dimethyl guanidino valeric acid as significant across the >1,000 BioHEART-CT patients, while other normalization methods identified only subsets.

## Related tools

- **limma** (Fits the linear model with hypertension status as covariate, applies empirical Bayes moderation, and computes Benjamini-Hochberg adjusted p-values per metabolite.)
- **hRUV** (Produces the normalised abundance matrix that is the input to the differential expression test; this skill specifically validates hRUV output.) — github.com/SydneyBioX/hRUV
- **R** (Implementation environment for the limma model and the hRUV package.) — github.com/SydneyBioX/hRUV

## Examples

```
library(limma); fit <- lmFit(hruv_norm_matrix, model.matrix(~hypertension_status)); fit <- eBayes(fit); topTable(fit, number=Inf, adjust="BH")
```

## Evaluation signals

- Proline, valine, cyclic adenosine monophosphate and dimethyl guanidino valeric acid are flagged as significantly differentially abundant with hypertension status (BH-adjusted p-value below threshold)
- These four metabolites appear among significant hits ONLY when hRUV-normalised data is used; other normalization methods should identify only subsets
- The output results table contains all tested metabolites with log2FC, t-statistic, raw p-value and adjusted p-value columns
- The analysis is run over the full cohort (>1,000 patients); no subsampling or per-batch splitting should be applied

## Limitations

- The validation is specific to the hypertension phenotype; recovery of these four metabolites does not guarantee hRUV preserves signal for all phenotypes.
- hRUV expects data without missing values, which is often not possible in large-scale metabolomics data, requiring an imputation workaround before testing.
- The significance threshold and BH correction are chosen by the analyst; the article does not state an explicit FDR cutoff in the provided text.

## Evidence

- [results] Recover known hypertension-associated metabolites as differentially expressed: [evidence span withheld — anchor status: not_found]
- [abstract] Specificity of hRUV for signal retention: "We demonstrate significant improvement of hRUV over existing methods in maintaining biological signals whilst removing unwanted variation for large scale metabolomics studies"
- [discussion] Cohort description: "The samples used were from the BioHEART-CT discovery cohort, which has been described in detail previously16."
