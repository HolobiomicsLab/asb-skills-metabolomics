---
name: multi-dataset-median-conformity-aggregation
description: Use when you have multiple annotated lipidomics datasets with retention
  times and need to quantify how consistently a relative retention-time ordering rule
  holds across all of them, rather than within a single dataset.
when_to_use_negative:
- You have only a single dataset and want a per-dataset RT ordering check — this skill
  aggregates medians across many datasets.
- Lipids lack subclass/chain-composition annotations, so ECN or ESCN scores cannot
  be computed and the ordering predicates are undefined.
- The input is raw MS spectra rather than already-annotated lipids with retention
  times — annotation must precede this aggregation.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: LipidIN
  role: Provides the lipid categories intelligence framework built on the three relative
    retention time rules and supplies the annotated lipid identities and RTs used
    for conformity computation
  repo: github.com/LinShuhaiLAB/LipidIN
- name: Python
  role: Implements the ECN/ESCN scoring and pair-conformity fraction computation,
    then aggregates per-dataset results to a median in conformity_summary.csv
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-59683-5
    title: 'LipidIN: a comprehensive repository for flash platform-independent annotation
      and reverse lipidomics'
  source_content_hash: sha256:dead4dd3dcad58a46e9fac2892834a87610e89c63afd904e5cf03435f20a5dd2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidIN
  - Python
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# multi-dataset-median-conformity-aggregation

## Summary

Aggregates per-dataset conformity statistics for lipid retention-time rules (ECN, intra-subclass unsaturation parallelism, ESCN) across >100 published LC-MS datasets to obtain a median conformity value and average absolute RT deviation rates.

## When to use

Use when you have multiple annotated lipidomics datasets with retention times and need to quantify how consistently a relative retention-time ordering rule holds across all of them, rather than within a single dataset.

## When NOT to use

- You have only a single dataset and want a per-dataset RT ordering check — this skill aggregates medians across many datasets.
- Lipids lack subclass/chain-composition annotations, so ECN or ESCN scores cannot be computed and the ordering predicates are undefined.
- The input is raw MS spectra rather than already-annotated lipids with retention times — annotation must precede this aggregation.

## Inputs

- Annotated lipid species with observed retention times from multiple published LC-MS datasets (e.g. Fig. 2 / Supplementary Data 2 references)
- Per-lipid structural annotations (total acyl carbons, double-bond counts, head group/subclass) sufficient to compute ECN and ESCN scores

## Outputs

- conformity_summary.csv with per-dataset conformity fractions, median conformity across datasets, and average absolute RT deviation rates per rule
- Aggregated average absolute RT deviation rates (reported ~1.44% for ECN, ~1.28% for ESCN)

## How to apply

Implement each rule as an ordering predicate: compute the ECN score (total acyl carbons minus twice the number of double bonds), the intra-subclass unsaturation parallelism criterion (equal double-bond counts elute in parallel RT order across differing chain lengths), and the ESCN score (accounting for separated versus total acyl carbons), then for each dataset compute the fraction of lipid pairs whose observed RT ordering conforms to the rule-predicted ordering. Also compute the average absolute RT deviation rate per rule per dataset. Aggregate conformity fractions and deviation rates across all datasets and take the median, since the paper reports ~90% median conformity across >100 published datasets, with average absolute RT deviation rates of 1.44% for ECN and 1.28% for ESCN. Write the result as a conformity_summary.csv containing both per-dataset and aggregated metrics so individual dataset outliers remain inspectable.

## Related tools

- **LipidIN** (Provides the lipid categories intelligence framework built on the three relative retention time rules and supplies the annotated lipid identities and RTs used for conformity computation) — github.com/LinShuhaiLAB/LipidIN
- **Python** (Implements the ECN/ESCN scoring and pair-conformity fraction computation, then aggregates per-dataset results to a median in conformity_summary.csv)

## Examples

```
python compute_conformity.py --datasets datasets/*.csv --rules ECN,ESCN --out conformity_summary.csv
```

## Evaluation signals

- Median conformity across datasets reproduces the reported ~90% value
- Average absolute RT deviation rate reproduces ~1.44% for the ECN rule
- Average absolute RT deviation rate reproduces ~1.28% for the ESCN rule
- conformity_summary.csv contains both per-dataset rows and aggregated median rows with no missing rule/dataset combinations
- Per-dataset conformity fractions fall within the plausible 0–1 range and show the expected spread across >100 datasets

## Limitations

- The conformity statistics depend on the quality and subclass annotation completeness of each source dataset; datasets with incomplete chain-composition annotations cannot be scored for ECN/ESCN.
- The paper does not investigate the deeper relationship between metabolites and relative RT, deferring that to future work, so rule-level conformity is a summary statistic rather than a mechanistic RT model.
- Pre-training on a larger and more diverse dataset is a stated future need for extending reverse lipidomics, implying current dataset coverage may not generalize to all lipid classes.

## Evidence

- [methods] median conformity across datasets reproduces the reported ~90% value and that the average absolute RT deviation rates reproduce the reported 1.44% (ECN) and 1.28% (ESCN): [evidence span withheld — anchor status: not_found]
- [abstract] three relative retention time rules: "developed using three relative retention time rules, reducing false positive annotations and predicting unannotated lipids with a 5.7% estimated false discovery rate"
- [discussion] investigating the deeper relationship between metabolites and relative RT will provide a robust foundation: "investigating the deeper relationship between metabolites and relative RT will provide a robust foundation for metabolite identification, prediction, and causal network analysis"
