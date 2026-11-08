---
name: equivalent-separated-carbon-number-deviation-estimation
description: Use when you have published LC-MS/MS lipidomics datasets with annotated
  lipid identities and retention times and need to quantify how consistent the ECN,
  IUP, and ESCN RT rules are across datasets.
when_to_use_negative:
- Input is raw mass spectra rather than already annotated lipid identities with retention
  times — spectral library search and annotation must be done first (e.g., via LipidIN
  annotation) before RT rule conformity can be evaluated
- Dataset lacks sufficient coverage of a lipid subclass to compute pair-wise RT ordering
  (too few lipids within the same subclass to establish monotonic or parallel relationships)
- Only MS1 (no retention-time or structural) data is available and no lipid subclass
  or chain-composition annotation can be inferred, making ECN/ESCN scoring impossible
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0153
tools:
- name: LipidIN
  role: Source of the annotated lipid identities, subclass definitions, and the ECN/IUP/ESCN
    relative RT rule framework used to score each dataset
  repo: github.com/LinShuhaiLAB/LipidIN
- name: Python
  role: Implements the ECN, IUP, and ESCN rule computations, RT-ordering conformity
    checks, and per-dataset aggregation of conformity and deviation metrics
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-59683-5
    title: 'LipidIN: a comprehensive repository for flash platform-independent annotation
      and reverse lipidomics'
  source_content_hash: sha256:ecb46aeaea249561b9e8bd8188af381f4d8a801a2bfc66a7306469b8f6a224b7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidIN
  - Python
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# equivalent-separated-carbon-number-deviation-estimation

## Summary

Computes per-dataset conformity to three relative retention-time rules (ECN, IUP, ESCN) for lipid subclasses in LC-MS/MS datasets and aggregates median conformity and mean absolute RT deviation rates (1.44% ECN, 1.28% ESCN) for retention-based lipid annotation confidence.

## When to use

Use when you have published LC-MS/MS lipidomics datasets with annotated lipid identities and retention times and need to quantify how consistent the ECN, IUP, and ESCN RT rules are across datasets.

## When NOT to use

- Input is raw mass spectra rather than already annotated lipid identities with retention times — spectral library search and annotation must be done first (e.g., via LipidIN annotation) before RT rule conformity can be evaluated
- Dataset lacks sufficient coverage of a lipid subclass to compute pair-wise RT ordering (too few lipids within the same subclass to establish monotonic or parallel relationships)
- Only MS1 (no retention-time or structural) data is available and no lipid subclass or chain-composition annotation can be inferred, making ECN/ESCN scoring impossible

## Inputs

- published LC-MS/MS lipidomics datasets referenced in Fig. 2 and Supplementary Data 2 that provide annotated lipid identities with retention times
- lipid subclass annotations (head group, total acyl carbons, number of double bonds, and separated acyl carbons per chain)
- retention times (RT) for each annotated lipid within each dataset

## Outputs

- conformity_summary.csv with per-dataset and aggregated metrics: fraction of lipid pairs conforming to each rule (ECN, IUP, ESCN), median conformity across datasets (~90%), and mean absolute RT deviation rates (1.44% ECN, 1.28% ESCN)

## How to apply

Obtain the published datasets referenced in Fig. 2 and Supplementary Data 2 that supply annotated lipid identities with retention times. Implement the equivalent carbon number (ECN) rule by computing the ECN score (total acyl carbons minus twice the number of double bonds) for each lipid within a subclass and checking RT monotonicity against ECN ordering. Implement the intra-subclass unsaturation parallelism rule (IUP) by grouping lipids of the same subclass and verifying that lipids with equal double-bond counts elute in parallel RT order across differing chain lengths. Implement the equivalent separated carbon number (ESCN) rule by computing the ESCN score that accounts for separated (vs. total) acyl carbons and verifying RT conformity per ESCN ordering. For each rule and each dataset, compute the fraction of lipid pairs whose observed RT ordering conforms to the rule-predicted ordering and the average absolute RT deviation rate, then aggregate across datasets to obtain median conformity; validate against the paper's reported ~90% median conformity and the 1.44% (ECN) / 1.28% (ESCN) deviation rates, writing conformity_summary.csv with per-dataset and aggregated metrics.

## Related tools

- **LipidIN** (Source of the annotated lipid identities, subclass definitions, and the ECN/IUP/ESCN relative RT rule framework used to score each dataset) — github.com/LinShuhaiLAB/LipidIN
- **Python** (Implements the ECN, IUP, and ESCN rule computations, RT-ordering conformity checks, and per-dataset aggregation of conformity and deviation metrics)

## Examples

```
python -c "import pandas as pd; df = pd.read_csv('annotated_rt.csv'); df['ecn'] = df['acyl_carbons'] - 2*df['double_bonds']; df.sort_values(['subclass','ecn']).to_csv('conformity_summary.csv', index=False)"
```

## Evaluation signals

- Per-dataset conformity fractions for each of the three rules should be computed on a consistent pair-count basis across all datasets, enabling direct aggregation into a single median conformity value
- Aggregated median conformity across the >100 published datasets should reproduce the reported ~90% value
- Mean absolute RT deviation rate for the ECN rule should reproduce the reported 1.44%
- Mean absolute RT deviation rate for the ESCN rule should reproduce the reported 1.28%
- ECN score is computed as total acyl carbons minus twice the number of double bonds, and ESCN must explicitly account for separated (not total) acyl carbons, so a spot-check of scores against a known lipid species should match this definition

## Limitations

- The rule set relies on the article's curated set of >100 published datasets; conformity on novel datasets outside this corpus is not validated by this skill's context
- The paper notes the deeper relationship between metabolites and relative RT remains uninvestigated, so the RT-rule framework is heuristic rather than mechanistic
- LipidIN's ECN/IUP/ESCN rule framework carries an estimated 5.7% false discovery rate, so some fraction of annotated lipids may violate RT rules for reasons other than poor dataset quality

## Evidence

- [abstract] three relative retention time rules: [evidence span withheld — anchor status: not_found]
- [intro] RT-based scores combined with MS/MS-based scores improve annotation: "and that combining MS/MS- and RT-based scores could significantly improve annotation performance in LC-MS/MS-based lipidomics analyses"
- [supplementary] ESCN accounts for separated vs total acyl carbons: [evidence span withheld — anchor status: not_found]
- [readme] reported ECN/ESCN deviation rates: [evidence span withheld — anchor status: not_found]
