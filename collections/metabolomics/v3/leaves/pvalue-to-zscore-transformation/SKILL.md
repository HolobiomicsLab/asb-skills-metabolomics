---
name: pvalue-to-zscore-transformation
description: Use when a KO/compound abundance table has already been tested per feature
  (P-values and log fold changes available) and per-feature Z-scores are needed as
  input to reporter-score pathway aggregation.
when_to_use_negative:
- The input is already a Z-score or effect-size matrix rather than raw P-values from
  a differential test
- The feature abundance table has been prefiltered to significant features, breaking
  the threshold-free background GRSA needs
- The goal is thresholded over-representation (hypergeometric-style KO_enrich/KO_fisher)
  rather than the continuous reporter-score statistic
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3697
tools:
- name: ReporterScore
  role: R package implementing pvalue2zs (mode='mixed'), ko.test with p.adjust.method1,
    and the reporter_score/get_reporter_score pipeline that consumes the transformed
    Z-scores
  repo: https://github.com/Asa12138/ReporterScore
- name: R
  role: Runtime environment in which pvalue2zs and the GRSA pipeline are executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae116
    title: Generalized reporter score-based enrichment analysis for omics data
  source_content_hash: sha256:ccd725c5128be9b6a2aa94e311369483ad5d8220d730a53a2a6b34d3c32f60d6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Asa12138/ReporterScore
  tools:
  - R
  - ReporterScore
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# pvalue-to-zscore-transformation

## Summary

Converts per-feature differential P-values into signed or unsigned Z-scores (pvalue2zs, mode='mixed') as the first computational step of the GRSA reporter-score pipeline, before features are aggregated into pathway-level reporter scores.

## When to use

Use when a KO/compound abundance table has already been tested per feature (P-values and log fold changes available) and per-feature Z-scores are needed as input to reporter-score pathway aggregation.

## When NOT to use

- The input is already a Z-score or effect-size matrix rather than raw P-values from a differential test
- The feature abundance table has been prefiltered to significant features, breaking the threshold-free background GRSA needs
- The goal is thresholded over-representation (hypergeometric-style KO_enrich/KO_fisher) rather than the continuous reporter-score statistic

## Inputs

- KO-level differential statistics table (feature IDs such as K numbers as rownames; P-values and log fold changes per feature), e.g. from KO_abundance_test
- Sample metadata matching abundance-table colnames (rownames must match/partial-match)
- Choice of transformation mode ('mixed' for signed Z-scores) and p.adjust.method1

## Outputs

- Per-feature signed or unsigned Z-score vector/matrix feeding into pathway aggregation
- Downstream pathway reporter score table with permutation P-values and adjusted P-values (reporter_scores.csv)

## How to apply

Load the KO-level differential statistics (P-values and log fold changes) — e.g. from the packaged KO_abundance_test dataset with its metadata — and apply the pvalue2zs transformation, choosing mode='mixed' for signed Z-scores that preserve direction of change, with optional P-value adjustment via p.adjust.method1 before conversion. The signed/unsigned choice is the key decision point: mixed mode encodes up- vs down-regulation in the sign of the Z, which downstream aggregation into pathway reporter scores then inherits. Do not prefilter the abundance table first, since GRSA is explicitly a threshold-free method that uses the full background feature set for the permutation null. The transformed Z-scores are then mapped to pathways (Module_htable, filtered by min_exist_KO/max_exist_KO) and standardized against a label-permuted null (perm) to yield permutation P-values, corrected with p.adjust.method2.

## Related tools

- **ReporterScore** (R package implementing pvalue2zs (mode='mixed'), ko.test with p.adjust.method1, and the reporter_score/get_reporter_score pipeline that consumes the transformed Z-scores) — https://github.com/Asa12138/ReporterScore
- **R** (Runtime environment in which pvalue2zs and the GRSA pipeline are executed)

## Examples

```
library(ReporterScore); data("KO_abundance_test"); res <- reporter_score(KO_abundance, group="Group", metadata=metadata, mode="mixed", perm=1000)
```

## Evaluation signals

- reporter_scores.csv exists and contains a reporter-score column and a corrected-P column for every pathway
- All adjusted pathway P-values fall within [0,1]
- Signs of transformed Z-scores match the direction of log fold changes in mixed mode (up-regulated features positive, down-regulated negative)
- Every input feature P-value maps to a finite Z-score with no NAs for features passing coverage filters (min_exist_KO/max_exist_KO)
- Permutation-derived P-values are stable across reruns with the same perm seed

## Limitations

- GRSA exhibits slightly lower specificity than thresholded methods, so enriched pathways may need additional experimental validation
- Using signed (mode='mixed') vs unsigned Z-scores changes the interpretation of reporter-score signs, a known source of misuse of the original RSA
- No version number is stated for the ReporterScore package in the article, limiting reproducibility of the transformation

## Evidence

- [methods] Convert each feature's P-value to a signed (mode='mixed') or unsigned Z-score using the pvalue2zs transformation: [evidence span withheld — anchor status: not_found]
- [readme] threshold-free method: [evidence span withheld — anchor status: external_doc]
- [methods] reporter_score signature: "reporter_score(kodf, group, metadata, method, pattern, p.adjust.method1, mode, verbose, feature, type, p.adjust.method2, modulelist, threads, perm, min_exist_KO, max_exist_KO)"
- [results] A pathway's Z-score is standardized against a background null: [evidence span withheld — anchor status: not_found]
