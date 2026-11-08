---
name: filter-efficiency-ranking-across-criteria
description: Use when you have applied multiple rule-based filters to an enumerated
  compound set and need per-rule elimination counts to identify the most effective
  criterion and validate filter behavior against a reference (e.g., Figure 2d).
when_to_use_negative:
- Input is a single known compound list rather than a large enumerated derivative
  set — per-rule efficiency ranking adds no value and the messy-rings dominance claim
  does not transfer.
- The goal is mass-weight filtering of compounds against an observed M+ peak with
  a tolerance range (a discussion-stage feature of PS²MS), not rule-efficiency analysis
  during database construction.
edam_operation: http://edamontology.org/operation_3675
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_2258
tools:
- name: rdkit
  role: 'Molecular processing for rule evaluation: SSSR ring perception, valence/stability
    checks, and property calculations on each enumerated derivative'
  repo: https://www.rdkit.org/docs/Install.html
- name: PS2MS
  role: Provides the enumeration-and-filtering pipeline whose rules are tallied and
    ranked; C++/CMake enumeration step produces the derivative set
  repo: https://github.com/jhhung/PS2MS
- name: SCScore
  role: Supplies average synthetic complexity scores used for the ΔSCScore filtering
    criterion applied alongside the drug-like rules
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c05019
    title: 'PS<sup>2</sup>MS: A Deep Learning-Based Prediction System for Identifying
      New Psychoactive Substances Using Mass Spectrometry'
  source_content_hash: sha256:c40818794c68010eeee238975d2edde68013eea16259a4a1b71cf90312d6a587
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PS2MS
  - SCScore
  - python
  - rdkit
  edam_operation: http://edamontology.org/operation_3675
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# filter-efficiency-ranking-across-criteria

## Summary

Tallies how many enumerated derivatives each filtering criterion eliminates (e.g., the 57 Virshup drug-like rules, messy rings, ΔSCScore) and ranks rules by elimination count, revealing which criterion contributes most to reducing the enumerated set to the ~422-million-compound synthetic NPS database.

## When to use

Use when you have applied multiple rule-based filters to an enumerated compound set and need per-rule elimination counts to identify the most effective criterion and validate filter behavior against a reference (e.g., Figure 2d).

## When NOT to use

- Input is a single known compound list rather than a large enumerated derivative set — per-rule efficiency ranking adds no value and the messy-rings dominance claim does not transfer.
- The goal is mass-weight filtering of compounds against an observed M+ peak with a tolerance range (a discussion-stage feature of PS²MS), not rule-efficiency analysis during database construction.

## Inputs

- Enumerated derivative SMILES set from hydrogen-to-functional-group substitution of the drug core structure (e.g., cathinone)
- Set of 57 drug-like filtering criteria from Virshup et al.
- Messy rings rule definition (more than 7 smallest sets of smallest rings)
- ΔSCScore threshold αn for synthesis-difficulty filtering

## Outputs

- filter_counts.csv — per-rule elimination counts ranked by number of derivatives removed
- filtered_compounds.smi — surviving derivatives passing all 57 criteria (~422 million compounds)
- Ranked rule-efficiency table identifying the most effective rule (Messy rings, Figure 2d)

## How to apply

Load the enumerated derivative SMILES set produced by hydrogen-to-functional-group substitution on the drug core structure (e.g., cathinone). Evaluate each of the 57 drug-like criteria from Virshup et al. against every derivative using RDKit molecular processing: ring perception via smallest sets of smallest rings (SSSR), valence/stability checks, and property calculations; also compute ΔSCScore thresholds. For each rule, tally the number of derivatives it eliminates and write per-rule counts to filter_counts.csv. Rank the rules by elimination count — in PS²MS the 'Messy rings' rule (more than 7 smallest sets of smallest rings) was the most effective, removing the largest share of molecules as reported in Figure 2d — then assemble the survivors of all 57 criteria into filtered_compounds.smi for downstream NEIMS spectrum prediction.

## Related tools

- **rdkit** (Molecular processing for rule evaluation: SSSR ring perception, valence/stability checks, and property calculations on each enumerated derivative) — https://www.rdkit.org/docs/Install.html
- **PS2MS** (Provides the enumeration-and-filtering pipeline whose rules are tallied and ranked; C++/CMake enumeration step produces the derivative set) — https://github.com/jhhung/PS2MS
- **SCScore** (Supplies average synthetic complexity scores used for the ΔSCScore filtering criterion applied alongside the drug-like rules)

## Examples

```
python enumerate.py --core cathinone.smi && python filter.py --counts filter_counts.csv --output filtered_compounds.smi
```

## Evaluation signals

- filter_counts.csv exists and contains one elimination count for each of the 57 Virshup drug-like criteria plus the messy rings and ΔSCScore rules.
- Ranking the per-rule counts confirms the Messy rings rule (more than 7 smallest sets of smallest rings) removes the largest share of molecules, reproducing the paper's Figure 2d pattern.
- The filtered set (filtered_compounds.smi) is consistent with the paper's reported scale of approximately 422 million compounds after enumeration and filtering.
- Every retained derivative passes all 57 drug-like criteria; spot-check random survivors with RDKit to confirm no rule violations (SSSR count ≤ 7, valid valences).

## Limitations

- The most effective rule is data-dependent: 'Messy rings' was dominant for the cathinone template; other drug cores may shift which rule removes the most derivatives.
- The paper does not specify the exact number of enumeration levels, which affects how many derivatives reach the filtering stage and thus the per-rule counts.
- Filtering removes highly challenging-to-synthesize compounds via SCScore thresholds, but this may exclude some valid NPS candidates.

## Evidence

- [methods] Messy rings rule is most effective: "The most effective rule was Messy rings (more than 7 smallest sets of smallest rings)"
- [methods] 57 drug-like criteria introduced: "We further introduced 57 drug-like criteria according to Virshup et al.25"
- [methods] ~422 million compounds after enumeration and filtering: "the process of enumeration and filtering resulted in a total of approximately 422 million compounds."
- [readme] PS2MS builds synthetic database by substitution: "PS<sup>2</sup>MS will generate a synthetic database by substituting hydrogen atoms in the core structure of a given drug with functional groups."
- [discussion] SCScore screening rationale: "We utilize these
scores to screen functional groups, thus avoiding the inclusion of
highly challenging-to-synthesize compounds in the enumeration process"
