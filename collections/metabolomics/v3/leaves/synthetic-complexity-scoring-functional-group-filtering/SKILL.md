---
name: synthetic-complexity-scoring-functional-group-filtering
description: Use when enumerating molecular derivatives from a core scaffold and you
  must decide which functional groups (from a library, e.g. 204 groups or 30 UNODC
  groups) are synthetically plausible at each recursion depth.
when_to_use_negative:
- When the analyte's chemical structure is already known and matches a reference spectrum
  — direct spectral matching against NIST/Wiley/SWGDRUG libraries suffices and enumeration
  is unnecessary
- 'When the input is a finished list of enumerated derivatives — at that stage the
  57 Virshup et al. drug-like criteria (e.g. ''Messy rings'': more than 7 smallest
  sets of smallest rings) apply, not functional-group SCScore screening'
- When the target compounds are expected to be synthetically inaccessible — this filter
  deliberately removes groups with high average synthesis difficulty differential
  (ΔSCScore ≥ α*n)
tools:
- name: rdkit
  role: Used to load the cathinone template SMILES, perform iterative hydrogen-atom
    replacement with functional-group R-groups, and support SCScore computation of
    the resulting compounds
  repo: https://www.rdkit.org/docs/Install.html
- name: SCScore
  role: Provides the synthetic complexity score used to compute the average ΔSCScore
    per functional group at each recursion depth for filtering
- name: PS2MS
  role: Pipeline that hosts the enumeration and rule-based filtering step; built from
    C++ source with CMake, invoked prior to NEIMS/DeepEI spectrum and fingerprint
    prediction
  repo: https://github.com/jhhung/PS2MS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c05019
    title: 'PS<sup>2</sup>MS: A Deep Learning-Based Prediction System for Identifying
      New Psychoactive Substances Using Mass Spectrometry'
  source_content_hash: sha256:62f4c5885e95a943ec7d4b6d932c517811a84879ff170eb12fff641140f445a7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PS2MS
  - SCScore
  - python
  - rdkit
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# synthetic-complexity-scoring-functional-group-filtering

## Summary

Filters candidate functional groups during derivative enumeration by computing the average SCScore (synthetic complexity score) of n-fold modified compounds and retaining only groups whose average ΔSCScore falls below the threshold α*n (α=1.2), so that hard-to-synthesize substituents are excluded before large-scale enumeration.

## When to use

Use when enumerating molecular derivatives from a core scaffold and you must decide which functional groups (from a library, e.g. 204 groups or 30 UNODC groups) are synthetically plausible at each recursion depth.

## When NOT to use

- When the analyte's chemical structure is already known and matches a reference spectrum — direct spectral matching against NIST/Wiley/SWGDRUG libraries suffices and enumeration is unnecessary
- When the input is a finished list of enumerated derivatives — at that stage the 57 Virshup et al. drug-like criteria (e.g. 'Messy rings': more than 7 smallest sets of smallest rings) apply, not functional-group SCScore screening
- When the target compounds are expected to be synthetically inaccessible — this filter deliberately removes groups with high average synthesis difficulty differential (ΔSCScore ≥ α*n)

## Inputs

- Core drug-template SMILES (e.g. cathinone heavy-atom core)
- Functional-group R-group definition library (204 compiled groups; 30 UNODC groups for the restricted configuration)
- SCScore model/parameters for computing synthetic complexity of enumerated derivatives
- Recursion depth n (3) and threshold multiplier alpha (1.2)

## Outputs

- Filtered set of functional groups per recursion depth (retention counts, e.g. 131 groups at the second and 97 at the third level)
- Enumerated derivative SMILES for each configuration (~422 million compounds for the 204-group run; ~2.3 million for the 30-group run)
- Summary log with compound counts per configuration and enumeration runtime

## How to apply

Load the core-structure SMILES (e.g. cathinone) and the functional-group R-group definitions, then iteratively replace hydrogen atoms with each functional group using RDKit, recursively up to depth n=3. At each depth n>1, compute the average SCScore of the n-fold modified compounds for every functional group and retain only groups whose average synthesis difficulty differential (ΔSCScore) falls below the threshold α*n with α=1.2. This screening, applied before the 57 Virshup et al. drug-like criteria, avoids including highly challenging-to-synthesize compounds in the enumeration process and reduces the candidate space. Run the filtered enumeration per configuration (204-group full set vs 30-group UNODC subset) and record per-group retention counts and total compound counts in a summary log.

## Related tools

- **rdkit** (Used to load the cathinone template SMILES, perform iterative hydrogen-atom replacement with functional-group R-groups, and support SCScore computation of the resulting compounds) — https://www.rdkit.org/docs/Install.html
- **SCScore** (Provides the synthetic complexity score used to compute the average ΔSCScore per functional group at each recursion depth for filtering)
- **PS2MS** (Pipeline that hosts the enumeration and rule-based filtering step; built from C++ source with CMake, invoked prior to NEIMS/DeepEI spectrum and fingerprint prediction) — https://github.com/jhhung/PS2MS

## Evaluation signals

- Validation run (alpha=1.2, n=3) must recover all known illicit cathinone-type drugs listed in the UNODC database (Figure S2)
- Functional-group retention counts match expectations: 131 groups retained at the second and 97 at the third recursion level
- Total compound count for the 204-group configuration is approximately 422 million (and ~2.3 million for the 30-group UNODC configuration)
- All retained groups satisfy the invariant: average ΔSCScore < α*n with α=1.2 at every depth n>1
- Enumeration runtime and per-configuration compound counts are recorded in the summary log

## Limitations

- The threshold (α=1.2) and depth (n=3) jointly determine coverage; different enumeration levels have a similar impact on the result space, so tuning changes the database scale substantially
- SCScore screening may exclude genuinely present but synthetically challenging NPS variants from the synthetic database
- Requires building RDKit C++ code from source and installing the python package from conda; the enumeration step needs CMake 3.16.0+ and g++-10 or higher

## Evidence

- [methods] retained only the functional groups whose average synthesis difficulty differential fell below a threshold of αn: "retained only the functional groups whose average synthesis difficulty differential (represented as ΔSCScore) fell below a threshold of αn"
- [methods] average synthetic complexity score of the resulting compounds using SCScore: "We then calculated the average synthetic complexity score of the resulting compounds using SCScore."
- [discussion] We utilize these scores to screen functional groups, thus avoiding the inclusion of highly challenging-to-synthesize compounds in the enumeration process: "We utilize these
scores to screen functional groups, thus avoiding the inclusion of
highly challenging-to-synthesize compounds in the enumeration process"
- [readme] PS2MS will generate a synthetic database by substituting hydrogen atoms in the core structure of a given drug with functional groups.: "PS<sup>2</sup>MS will generate a synthetic database by substituting hydrogen atoms in the core structure of a given drug with functional groups."
