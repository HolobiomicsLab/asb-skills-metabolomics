---
name: lipid-retention-time-rule-implementation
description: Use when you have annotated lipid identities paired with LC-MS retention
  times in one or more datasets and want to quantify conformity of the ECN/IUP/ESCN
  relative RT ordering rules.
when_to_use_negative:
- Do not use for direct MS/MS spectral annotation or false-positive removal — that
  is the province of LipidIN's annotation pipeline, not the RT rule benchmark
- Do not use when only raw MS data without lipid annotation (subclass, chain length,
  double-bond counts) is available; the rules require already-annotated lipid identities
- Do not use for absolute RT prediction from chemical structure alone (e.g., for de
  novo retention modelling) — this skill evaluates relative ordering consistency,
  not absolute RT prediction
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
tools:
- name: LipidIN
  role: Provides the lipid annotation context (subclass, chain composition, unsaturation)
    and references the >100 published datasets used for the RT-rule conformity benchmark
  repo: github.com/LinShuhaiLAB/LipidIN
- name: Python
  role: Implementation language for the ECN/IUP/ESCN rule calculations and the per-dataset
    / aggregated conformity statistics
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-59683-5
    title: 'LipidIN: a comprehensive repository for flash platform-independent annotation
      and reverse lipidomics'
  source_content_hash: sha256:f1e82a23b77e068a12a5181e0a9c0d0123ec9efb6d1c381923cd13f99fae8d02
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidIN
  - Python
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# lipid-retention-time-rule-implementation

## Summary

A benchmark skill for evaluating how consistently the three relative retention-time rules (ECN, IUP, ESCN) hold across lipid subclasses in LC-MS/MS datasets, computing per-dataset conformity fractions and average absolute RT deviation rates (1.44% for ECN, 1.28% for ESCN).

## When to use

Use when you have annotated lipid identities paired with LC-MS retention times in one or more datasets and want to quantify conformity of the ECN/IUP/ESCN relative RT ordering rules.

## When NOT to use

- Do not use for direct MS/MS spectral annotation or false-positive removal — that is the province of LipidIN's annotation pipeline, not the RT rule benchmark
- Do not use when only raw MS data without lipid annotation (subclass, chain length, double-bond counts) is available; the rules require already-annotated lipid identities
- Do not use for absolute RT prediction from chemical structure alone (e.g., for de novo retention modelling) — this skill evaluates relative ordering consistency, not absolute RT prediction

## Inputs

- Published LC-MS/MS lipidomics datasets providing annotated lipid identities with retention times (e.g., those referenced in Fig. 2 and Supplementary Data 2 of the LipidIN article)
- Annotated lipid subclass labels, acyl chain lengths, and double-bond counts per lipid
- Python execution environment for implementing the three RT rules and computing conformity statistics

## Outputs

- conformity_summary.csv containing per-dataset and aggregated metrics: conformity fraction per rule (ECN, IUP, ESCN), median conformity across datasets (~90%), and average absolute RT deviation rates (1.44% for ECN, 1.28% for ESCN)
- Per-rule per-pair conformity classifications within each dataset

## How to apply

Implement three relative retention-time checks per lipid subclass: (1) the equivalent carbon number (ECN) rule — compute ECN = total acyl carbons minus twice the number of double bonds and verify RT monotonicity against ECN ordering within each subclass; (2) the intra-subclass unsaturation parallelism (IUP) rule — group lipids of the same subclass with equal double-bond counts and confirm they elute in parallel RT order across differing chain lengths; (3) the equivalent separated carbon number (ESCN) rule — compute ESCN from separated (not total) acyl carbons and verify RT conformity per ESCN ordering. For each rule and dataset, compute the fraction of lipid pairs whose observed RT ordering conforms to the rule-predicted ordering and the average absolute RT deviation rate. Aggregate conformity fractions across >100 published datasets (e.g., those referenced in Fig. 2 and Supplementary Data 2) to obtain a median conformity value (~90%). Write per-dataset and aggregated metrics to conformity_summary.csv.

## Related tools

- **LipidIN** (Provides the lipid annotation context (subclass, chain composition, unsaturation) and references the >100 published datasets used for the RT-rule conformity benchmark) — github.com/LinShuhaiLAB/LipidIN
- **Python** (Implementation language for the ECN/IUP/ESCN rule calculations and the per-dataset / aggregated conformity statistics)

## Examples

```
python scripts/rt_rule_conformity.py --datasets fig2_suppdata2/*.csv --rules ECN,IUP,ESCN --out conformity_summary.csv
```

## Evaluation signals

- Per-dataset ECN conformity fraction matches the reported ~90% median conformity across datasets
- Average absolute RT deviation rate for the ECN rule reproduces 1.44%
- Average absolute RT deviation rate for the ESCN rule reproduces 1.28%
- For each subclass, lipids with equal double-bond counts (IUP grouping) show parallel RT ordering across differing chain lengths
- conformity_summary.csv is produced with both per-dataset and aggregated metric rows

## Limitations

- The conformity estimate (~90%) and RT deviation rates are derived from published datasets referenced in Fig. 2 and Supplementary Data 2; datasets not represented there may yield different values
- The article notes the deeper relationship between metabolites and relative RT remains uninvestigated and deferred to future work, so the rules are empirical rather than mechanistically derived
- Datasets lacking complete subclass, chain-length, and double-bond annotations will produce incomplete conformity statistics

## Evidence

- [abstract] Conformity across datasets ~90% median: "developed using three relative retention time rules, reducing false positive annotations and predicting unannotated lipids with a 5.7% estimated false discovery rate"
- [methods] RT-rule benchmark workflow (task_003): [evidence span withheld — anchor status: not_found]
- [results] per-rule conformity metric definition: [evidence span withheld — anchor status: not_found]
- [methods] ESCN rule definition: [evidence span withheld — anchor status: not_found]
- [methods] intra-subclass unsaturation parallelism rule: [evidence span withheld — anchor status: not_found]
- [supplementary] benchmark dataset scope: [evidence span withheld — anchor status: not_found]
- [intro] RT-rule motivation in LipidIN: "We hypothesized that deep learning models could extract the mapping relationship between lipid structure features (e.g., head group, chain length, and degree of unsaturation of each fatty acyl"
