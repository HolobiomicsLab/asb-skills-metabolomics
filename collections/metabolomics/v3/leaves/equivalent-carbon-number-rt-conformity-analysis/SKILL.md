---
name: equivalent-carbon-number-rt-conformity-analysis
description: 'Use when validating a LipidIN LCI retention-order model or auditing
  any annotated LC-MS/MS lipidomics dataset: you have lipid identities with retention
  times across glycerophospholipid subclasses and need to quantify conformity to ECN/IUP/ESCN
  RT rules and reproduce the paper''s deviation.'
when_to_use_negative:
- Input data lack observed retention times or chain-composition/double-bond annotations
  — conformity cannot be computed without both.
- The dataset is used as training/positive evidence for the LCI retention-order model
  itself rather than as an independent benchmark; the article notes the deeper relationship
  between metabolites and relative RT remains uninvestigated.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: LipidIN
  role: Source of the three relative retention time rules (ECN, IUP, ESCN) built into
    the LCI module; used to obtain annotated lipid identities with retention times
    from its datasets and theoretical library
  repo: github.com/LinShuhaiLAB/LipidIN
- name: Python
  role: Implementation language for computing ECN/ESCN scores, checking RT monotonicity
    per subclass, and aggregating conformity metrics across datasets
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-59683-5
    title: 'LipidIN: a comprehensive repository for flash platform-independent annotation
      and reverse lipidomics'
  source_content_hash: sha256:8aa57bf56ad1283ecce4a3797c46a9395888895c5bc4600e4041a9c10679b5e2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidIN
  - Python
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# equivalent-carbon-number-rt-conformity-analysis

## Summary

A validation approach that quantifies how well published lipidomics RT datasets conform to three relative retention-time rules — ECN, intra-subclass unsaturation parallelism, and ESCN — and computes average absolute RT deviation rates for ECN and ESCN.

## When to use

Use when validating a LipidIN LCI retention-order model or auditing any annotated LC-MS/MS lipidomics dataset: you have lipid identities with retention times across glycerophospholipid subclasses and need to quantify conformity to ECN/IUP/ESCN RT rules and reproduce the paper's deviation benchmarks (1.44% ECN, 1.28% ESCN).

## When NOT to use

- Input data lack observed retention times or chain-composition/double-bond annotations — conformity cannot be computed without both.
- The dataset is used as training/positive evidence for the LCI retention-order model itself rather than as an independent benchmark; the article notes the deeper relationship between metabolites and relative RT remains uninvestigated.

## Inputs

- Annotated lipidomics datasets with lipid identities and observed LC retention times (Fig. 2 / Supplementary Data 2 collections, >100 published datasets)
- Lipid subclass annotations including acyl-chain carbon counts and double-bond positions, e.g. from the LipidIN theoretical library of 168.6 million fragmentation entries

## Outputs

- conformity_summary.csv with per-dataset and aggregated ECN, IUP, and ESCN conformity fractions
- Average absolute RT deviation rates per rule (target: 1.44% ECN, 1.28% ESCN) and median cross-dataset conformity (~90%)

## How to apply

For each published dataset referenced in Fig. 2 and Supplementary Data 2, compute the equivalent carbon number (ECN) score as total acyl carbons minus twice the number of double bonds and verify that observed RT ordering of lipids within a subclass matches ECN-predicted monotonicity. Second, group lipids of the same subclass by double-bond count and check that equal-double-bond species elute in parallel RT order as chain length varies (IUP rule). Third, compute the equivalent separated carbon number (ESCN) score, which accounts for separated rather than total acyl carbons, and verify RT conformity per ESCN ordering. For each rule and dataset, compute the fraction of lipid pairs whose observed RT ordering conforms to the rule-predicted ordering, and the average absolute RT deviation rate; aggregate across >100 datasets to obtain a median conformity value. Validation: the paper reports ~90% median conformity and average absolute RT deviation rates of 1.44% (ECN) and 1.28% (ESCN); outputs are collected in conformity_summary.csv with per-dataset and aggregated metrics.

## Related tools

- **LipidIN** (Source of the three relative retention time rules (ECN, IUP, ESCN) built into the LCI module; used to obtain annotated lipid identities with retention times from its datasets and theoretical library) — github.com/LinShuhaiLAB/LipidIN
- **Python** (Implementation language for computing ECN/ESCN scores, checking RT monotonicity per subclass, and aggregating conformity metrics across datasets)

## Examples

```
import pandas as pd; df=pd.read_csv('dataset.tsv',sep='\t'); df['ecn']=df['total_carbons']-2*df['double_bonds']; df['rt_conform']=(df['rt']==df.sort_values('ecn')['rt'].values); conformity=df['rt_conform'].mean()
```

## Evaluation signals

- Reproduce median conformity across >100 datasets of ~90% for the three RT rules
- Reproduce average absolute RT deviation rates of 1.44% for ECN and 1.28% for ESCN
- Per-dataset conformity fractions written for all three rules in conformity_summary.csv
- ECN score computed as total acyl carbons minus twice the double-bond count matches rule-predicted RT ordering within each subclass
- ESCN score accounts for separated acyl carbons and its ordering is verified per dataset

## Limitations

- The article defers deeper investigation of metabolite–relative RT relationships to future work; conformity is an empirical rule check, not a causal RT model.
- Coverage is limited to lipid subclasses whose annotated chain compositions and double-bond locations are present in the input; unannotated or ambiguous RT orderings are excluded.
- Conformity is a median across heterogeneous published datasets; individual datasets may deviate substantially from the ~90% aggregate.

## Evidence

- [abstract] ECN/IUP/ESCN rules defined and conformity metrics reported: "developed using three relative retention time rules, reducing false positive annotations and predicting unannotated lipids with a 5.7% estimated false discovery rate"
- [intro] Deep-learning mapping hypothesis between structure and retention: "deep learning models could extract the mapping relationship between lipid structure features (e.g., head group, chain length, and degree of unsaturation of each fatty acyl or"
- [intro] Combining MS/MS and RT scores improves annotation: "combining MS/MS- and RT-based scores could significantly improve annotation performance in LC-MS/MS-based lipidomics analyses"
- [discussion] Future need to investigate deeper RT relationships: "investigating the deeper relationship between metabolites and relative RT will provide a robust foundation for metabolite identification, prediction, and causal network analysis"
- [readme] LipidIN theoretical library scale: "168.6 million fragmentation entries (all chain compositions + C=C double-bond positions)"
