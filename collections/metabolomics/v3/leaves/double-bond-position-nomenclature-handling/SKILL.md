---
name: double-bond-position-nomenclature-handling
description: Use when parsing lipid shorthand names that carry double-bond position/geometry
  annotations (e.g., FA 18:2(9Z,12Z)) and you must verify that a structural similarity
  computation either counts DB positions or treats all DB matches as mismatches.
when_to_use_negative:
- The comparison goal is sn-position handling (e.g., PC 18:0_16:1 with unspecified
  sn-positions) rather than DB-position correctness — LipidSpace handles that with
  the separate 'Ignore lipid sn-positions' mode
- Input lipid names lack DB position annotations entirely (e.g., species-level names
  like FA 18:2), so there is no positional information to compare
- The analysis is quantitative lipidome reanalysis (abundance tables, PCA, Hausdorff
  distances) rather than pairwise structural MCS verification
edam_operation: http://edamontology.org/operation_0502
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: LipidSpace
  role: Standalone tool providing the LaWeCSE-based maximum-common-subgraph structural
    comparison with DB-position-aware and DB-agnostic modes
  repo: https://github.com/lifs-tools/lipidspace
- name: Goslin (cppgoslin)
  role: C++ library bundled with LipidSpace to parse and standardize lipid names,
    including DB positions, into structured components
  repo: https://github.com/lifs-tools/cppgoslin
- name: LIPID MAPS
  role: Structure database from which the compared lipid records (e.g., linoleic and
    oleic acid) are retrieved
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c02449
    title: 'LipidSpace: Simple Exploration, Reanalysis, and Quality Control of Large-Scale
      Lipidomics Studies'
  source_content_hash: sha256:78e980cbfa81bd0767b9e48fb106e2722504cb4e06c3eda7beed743bb504a38c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/lipidspace
  tools:
  - Goslin
  - Goslin (cppgoslin)
  - LIPID MAPS
  - LipidSpace
  edam_operation: http://edamontology.org/operation_0502
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# double-bond-position-nomenclature-handling

## Summary

Handling of double-bond (DB) position and geometry nomenclature in lipid shorthand names (e.g., FA 18:2(9Z,12Z) vs FA 18:1(9Z)) so that graph-based structural similarity computations correctly count or deliberately ignore DB positions. LipidSpace validates this by comparing MCS overlaps of the same lipid pair in DB-aware and DB-agnostic modes.

## When to use

Use when parsing lipid shorthand names that carry double-bond position/geometry annotations (e.g., FA 18:2(9Z,12Z)) and you must verify that a structural similarity computation either counts DB positions or treats all DB matches as mismatches.

## When NOT to use

- The comparison goal is sn-position handling (e.g., PC 18:0_16:1 with unspecified sn-positions) rather than DB-position correctness — LipidSpace handles that with the separate 'Ignore lipid sn-positions' mode
- Input lipid names lack DB position annotations entirely (e.g., species-level names like FA 18:2), so there is no positional information to compare
- The analysis is quantitative lipidome reanalysis (abundance tables, PCA, Hausdorff distances) rather than pairwise structural MCS verification

## Inputs

- Lipid shorthand names with DB position/geometry, e.g., FA 18:2(9Z,12Z) (linoleic acid) and FA 18:1(9Z) (oleic acid)
- LIPID MAPS structure records for the compared lipids
- Molecular graphs of the two lipids (atoms and bonds as components)

## Outputs

- Per-mode intersection (shared) and union component counts for the pairwise MCS comparison
- Mode/intersection/union rows written to double_bond_mcs_verification.csv (expected: DB-aware 38/39, DB-agnostic 36/39)

## How to apply

Parse the lipid shorthand names with the Goslin parser (cppgoslin) to standardize them into structured components including DB positions and stereochemistry. Build the molecular graphs (atoms and bonds as components) and compute the maximum common subgraph with the LaWeCSE-based MCS approach combined with the Jaccard index, counting double-bond positions in the forward (carbonyl-side) direction for the DB-position-aware mode. Recompute the same pairwise comparison in the DB-agnostic mode, in which all double-bond matches count as mismatches. For the canonical FA 18:2(9Z,12Z) vs FA 18:1(9Z) pair, the DB-aware mode must yield 38 of 39 shared components while the DB-agnostic mode yields 36 of 39; record intersection and union counts per mode (e.g., to double_bond_mcs_verification.csv). Rationale: counting DB positions in the carbonyl-forward direction distinguishes positional isomers such as 9Z,12Z from 9Z, which a DB-agnostic comparison would score as identical.

## Related tools

- **LipidSpace** (Standalone tool providing the LaWeCSE-based maximum-common-subgraph structural comparison with DB-position-aware and DB-agnostic modes) — https://github.com/lifs-tools/lipidspace
- **Goslin (cppgoslin)** (C++ library bundled with LipidSpace to parse and standardize lipid names, including DB positions, into structured components) — https://github.com/lifs-tools/cppgoslin
- **LIPID MAPS** (Structure database from which the compared lipid records (e.g., linoleic and oleic acid) are retrieved)

## Examples

```
cd examples && make && ./lipid_name_parser
```

## Evaluation signals

- DB-position-aware MCS comparison of FA 18:2(9Z,12Z) vs FA 18:1(9Z) yields 38 shared components out of 39
- DB-agnostic recomputation of the same pair yields 36 shared components out of 39
- Intersection/union counts differ between the two modes by exactly the two DB-position components (38 vs 36 of 39)
- Goslin parses both shorthand names without error and reproduces the DB positions (9Z,12Z and 9Z) in the structured output
- Total component count (39) is identical across both modes, confirming only DB-position matching changes

## Limitations

- In default mode LipidSpace compares the first FA of the first lipid with the first FA of the second lipid etc.; all-combinations FA comparison ('Ignore lipid sn-positions') is more accurate but decreases performance
- The default distance metric is bound between 0 and 1; unbound metrics give more accurate but less visually comparable lipid spaces
- Without DB positions considered, positional isomers (e.g., 9Z,12Z vs 9Z) score as more similar than their structures warrant

## Evidence

- [methods] Correctness of Double Bond Comparisons: "Correctness of Double Bond Comparisons"
- [methods] Goslin parsing: "It utilizes the C++ library of Goslin4,5 to parse and standardize lipid names"
- [methods] MCS + Jaccard: "we decided to utilize the maximum common subgraph (MCS) approach14 in combination with the Jaccard index15"
- [readme] sn-position vs all-combinations FA comparison: "a mode can be activated to compare all combinations of FA comparisons for both lipids and picking the lowest distance. The results are more accurate, but the performance is decreased."
- [abstract] graph-based structural comparison rationale: "A graph-based comparison of lipid structures is the basis for calculating structural space models and subsequently computing lipidome similarities."
