---
name: multiassayexperiment-metadata-extraction
description: Use when a MetaboDiff analysis is stored in a MultiAssayExperiment object
  and you need colData group labels or the t-test results stored in the metadata slot
  to identify differentially abundant metabolites.
when_to_use_negative:
- When more than two groups are compared — MetaboDiff then applies ANOVA instead of
  the Student's T-Test, so the two-group metadata extraction pattern does not apply
- When the metabolite measurement table has not yet been imputed (knn_impute, cutoff
  0.4) and normalized (vsn) into a MultiAssayExperiment object
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetaboDiff
  role: Provides create_mae, diff_test and the metadata slot where t-test results
    (pval, adj_pval, dm) are stored for extraction
  repo: http://github.com/andreasmock/MetaboDiff/
- name: MultiAssayExperiment
  role: Container whose colData holds per-sample group annotations and whose metadata
    slot holds differential test results
- name: R
  role: Runtime environment for MetaboDiff; requires R version 4.0.2 or higher
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/bty344
    title: 'MetaboDiff: an R package for differential metabolomic analysis'
  source_content_hash: sha256:dfcb40e3abe52eb08a37bd487babc5580cb83829f63c196a26759bfd36710f96
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/andreasmock/MetaboDiff
  tools:
  - MetaboDiff
  - MultiAssayExperiment
  - R
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# multiassayexperiment-metadata-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract per-sample metadata (e.g. 'group', 'tumor_groups') and per-metabolite statistical results (pval, adj_pval, dm) from a MetaboDiff MultiAssayExperiment object so differential-abundance calls can be audited and exported.

## When to use

Use when a MetaboDiff analysis is stored in a MultiAssayExperiment object and you need colData group labels or the t-test results stored in the metadata slot to identify differentially abundant metabolites.

## When NOT to use

- When more than two groups are compared — MetaboDiff then applies ANOVA instead of the Student's T-Test, so the two-group metadata extraction pattern does not apply
- When the metabolite measurement table has not yet been imputed (knn_impute, cutoff 0.4) and normalized (vsn) into a MultiAssayExperiment object

## Inputs

- MetaboDiff MultiAssayExperiment object with assays, colData sample annotations (e.g. 'group', 'tumor_groups') and imputed/normalized data (norm_imputed slot)
- Group factor names for the comparison (e.g. c('group','tumor_groups'))
- Comparison label such as 'MYC-high_vs_AKT1-high'

## Outputs

- Per-metabolite table of pval, adj_pval and dm (difference in means) from the metadata slot entry 'ttest_<factor>_<comparison>'
- Exported CSV of extracted statistics (e.g. diff_test_human_tumor_groups.csv)

## How to apply

After building the MultiAssayExperiment with create_mae and preprocessing (knn_impute with cutoff=0.4, then variance stabilizing normalization via normalize_met), read sample groupings from the colData of the MAE to subset conditions such as 'Control', 'MYC-high' and 'AKT1-high'. Run diff_test on the MAE with group_factors=c('group','tumor_groups'); for two-group comparisons the function applies Student's T-Test and corrects p-values with the Benjamini-Hochberg procedure, writing results into the metadata slot under a name like 'ttest_tumor_groups_MYC-high_vs_AKT1-high'. Extract the pval, adj_pval and dm (difference in means) columns per metabolite from that metadata entry, then export them (e.g. to CSV) for comparison against expected significance patterns.

## Related tools

- **MetaboDiff** (Provides create_mae, diff_test and the metadata slot where t-test results (pval, adj_pval, dm) are stored for extraction) — http://github.com/andreasmock/MetaboDiff/
- **MultiAssayExperiment** (Container whose colData holds per-sample group annotations and whose metadata slot holds differential test results)
- **R** (Runtime environment for MetaboDiff; requires R version 4.0.2 or higher)

## Examples

```
diff_test(norm_imputed_mae, group_factors = c('group','tumor_groups'))
```

## Evaluation signals

- The metadata slot contains an entry named 'ttest_tumor_groups_MYC-high_vs_AKT1-high' after diff_test with group_factors=c('group','tumor_groups')
- Extracted table includes pval, adj_pval and dm columns for named metabolites such as arachidonic acid, docosahexaenoic acid and oleic acid
- Significance stars match the case study: arachidonic acid and docosahexaenoic acid at p<0.05 (*) and oleic acid at p<0.01 (**) in the MYC-high vs AKT1-high comparison
- adj_pval values correspond to Benjamini-Hochberg correction of the raw pvals
- Exported CSV (e.g. diff_test_human_tumor_groups.csv) reproduces the extracted statistics without dropped rows

## Limitations

- Extraction of two-group t-test results is only valid after knn_impute (cutoff 0.4) and variance stabilizing normalization; unnormalized data yields different statistics
- diff_test stores results in the metadata slot keyed by group factor and comparison label, so renaming factors or groups changes the access path
- The example significance pattern is specific to the Priolo et al. (2014) human prostate tissue dataset (case1_human) and will not transfer to other datasets

## Evidence
