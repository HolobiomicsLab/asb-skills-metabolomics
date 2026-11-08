---
name: molecular-graph-component-verification
description: Use when validating that a molecular-graph similarity method correctly
  treats double-bond positions, e.g. checking MCS overlaps between structurally similar
  FAs like FA 18:2(9Z,12Z) and FA 18:1(9Z).
when_to_use_negative:
- Input lipid names or structures lack double-bond position annotations (e.g. only
  FA 18:2 without 9Z,12Z), so the DB-position-aware mode cannot be meaningfully exercised
- The goal is lipidome-level comparison of many lipids with abundances (Hausdorff
  distances on the lipid space model) rather than pairwise structural verification
  of single species
- Lipid sn-positions are unspecified in a way that requires the 'Ignore lipid sn-positions'
  all-combinations mode, since that changes which fatty acyl chains are paired and
  compared
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: LipidSpace
  role: Performs the LaWeCSE-based maximum-common-subgraph comparison of lipid molecular
    graphs with and without double-bond positions, combined with the Jaccard index
  repo: https://github.com/lifs-tools/lipidspace
- name: Goslin (cppgoslin)
  role: Parses and standardizes lipid shorthand names before graph construction
  repo: https://github.com/lifs-tools/cppgoslin
- name: LIPID MAPS
  role: Source database for the lipid structures used in the comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c02449
    title: 'LipidSpace: Simple Exploration, Reanalysis, and Quality Control of Large-Scale
      Lipidomics Studies'
  source_content_hash: sha256:8f1f7a1fa385b70aad292a24cd50a34da724bc018a5d969fb8bb21ca70f81331
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/lipidspace
  tools:
  - Goslin
  - Goslin (cppgoslin)
  - LIPID MAPS
  - LipidSpace
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# Molecular-graph component verification of double-bond-aware lipid structure comparison

## Summary

A verification procedure for LipidSpace's graph-based lipid structure comparison that checks whether counting double-bond (DB) positions in the maximum-common-subgraph (MCS) computation changes pairwise similarity between fatty acids such as FA 18:2(9Z,12Z) and FA 18:1(9Z). It validates the tool's 'Correctness of Double Bond Comparisons' claim by comparing DB-position-aware vs DB-agnostic intersection/union component counts.

## When to use

Use when validating that a molecular-graph similarity method correctly treats double-bond positions, e.g. checking MCS overlaps between structurally similar FAs like FA 18:2(9Z,12Z) and FA 18:1(9Z).

## When NOT to use

- Input lipid names or structures lack double-bond position annotations (e.g. only FA 18:2 without 9Z,12Z), so the DB-position-aware mode cannot be meaningfully exercised
- The goal is lipidome-level comparison of many lipids with abundances (Hausdorff distances on the lipid space model) rather than pairwise structural verification of single species
- Lipid sn-positions are unspecified in a way that requires the 'Ignore lipid sn-positions' all-combinations mode, since that changes which fatty acyl chains are paired and compared

## Inputs

- LIPID MAPS structure records for FA 18:2(9Z,12Z) (linoleic acid) and FA 18:1(9Z) (oleic acid)
- Lipid shorthand names parsed/standardized via Goslin (cppgoslin)
- Molecular graphs with atoms and bonds as components for each lipid

## Outputs

- Intersection and united component counts per comparison mode (DB-aware: 38/39; DB-agnostic: 36/39)
- Verification table double_bond_mcs_verification.csv with mode/intersection/union rows

## How to apply

Retrieve the LIPID MAPS structure records for the two fatty acids (e.g. linoleic acid FA 18:2(9Z,12Z) and oleic acid FA 18:1(9Z)) and parse their shorthand names with the Goslin C++ parser (cppgoslin), which LipidSpace uses to standardize lipid names. Build the two molecular graphs with atoms and bonds as components and compute the maximum common subgraph via the LaWeCSE-based MCS approach combined with the Jaccard index, counting double-bond positions in the forward (carbonyl-side) direction for the DB-position-aware mode. Recompute the same pairwise comparison in the DB-agnostic mode in which all double-bond matches count as mismatches. Record the shared (intersection) and united (union) component counts for each mode; the expected result is 38 of 39 shared components with DB positions considered versus 36 of 39 without, confirming that DB-aware comparison yields higher structural similarity for these lipids.

## Related tools

- **LipidSpace** (Performs the LaWeCSE-based maximum-common-subgraph comparison of lipid molecular graphs with and without double-bond positions, combined with the Jaccard index) — https://github.com/lifs-tools/lipidspace
- **Goslin (cppgoslin)** (Parses and standardizes lipid shorthand names before graph construction) — https://github.com/lifs-tools/cppgoslin
- **LIPID MAPS** (Source database for the lipid structures used in the comparison)

## Evaluation signals

- DB-position-aware mode yields 38 of 39 shared components for FA 18:2(9Z,12Z) vs FA 18:1(9Z)
- DB-agnostic mode yields 36 of 39 shared components for the same pair
- The union component count is 39 in both modes (only the intersection differs)
- The intersection difference between modes is attributable specifically to double-bond position matches being counted as mismatches in the DB-agnostic mode
- Verification rows written to double_bond_mcs_verification.csv contain exactly one row per mode with matching intersection/union integers

## Limitations

- In default mode LipidSpace compares the first FA of one lipid with the first FA of the other by sn-position; activating 'Ignore lipid sn-positions' to compare all FA combinations gives more accurate results but decreases performance
- LipidSpace analyses cannot currently be stored or loaded, so verification runs must be repeated rather than resumed
- The DB-agnostic mode deliberately counts all double-bond matches as mismatches, so the resulting distance is more conservative and only meaningful as a lower bound on structural similarity

## Evidence

- [intro] MCS plus Jaccard basis: "we applied the maximum common subgraph (MCS) approach in combination with the Jaccard index to determine a pairwise similarity of lipid species based on the molecular structure"
- [methods] Goslin name parsing: "It utilizes the C++ library of Goslin4,5 to parse and standardize lipid names"
- [methods] Structure source databases: "extracted all lipids from databases such as LIPID MAPS and SwissLipids on the highest structural resolution level"
- [readme] Default FA chain pairing: "In default mode, LipidSpace is comparing the first fatty acyl chain (FA) of the first lipid with the first FA of the second lipid, the second FA of the first lipid with the second FA of the second"
