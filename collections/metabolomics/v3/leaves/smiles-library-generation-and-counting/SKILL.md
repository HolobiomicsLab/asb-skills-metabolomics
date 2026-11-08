---
name: smiles-library-generation-and-counting
description: 'Use when building a large synthetic derivative library from a known
  drug core structure: you have a template SMILES, a functional-group (R-group) set,
  and need combinatorial hydrogen replacement with rule-based filtering and per-configuration
  compound counts.'
when_to_use_negative:
- The analyte's chemical structure and reference spectrum are already known — PS2MS
  enumeration is only triggered when the GC/EI-MS spectrum matches no established
  reference (NIST, Wiley, SWGDRUG).
- A small, targeted analog set suffices — the full 204-group enumeration produces
  ~422 million compounds and needs heavy filtering; the 30 UNODC-group configuration
  exists precisely to restrict scale.
- The target scaffold's derivatives are expected to be highly synthesis-challenging
  — the ΔSCScore screen (alpha*n, alpha=1.2) intentionally excludes groups yielding
  difficult-to-synthesize compounds.
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_3332
tools:
- name: rdkit
  role: Performs the heavy-atom-preserving hydrogen-to-R-group substitution on the
    cathinone core SMILES and computes smallest sets of smallest rings (SSSR) for
    the Messy rings filter
  repo: https://github.com/jhhung/PS2MS
- name: SCScore
  role: Computes synthetic complexity scores used to screen functional groups, retaining
    only those with average ΔSCScore below alpha*n (alpha=1.2)
- name: PS2MS
  role: Hosts the enumeration module (built with g++-10/CMake 3.16) that generates
    the synthetic database from core-structure substitutions and records counts/runtime
  repo: https://github.com/jhhung/PS2MS
- name: python 3.6.9
  role: Runtime for the deep learning tools and RDKit Python bindings used in the
    enumeration/detection pipeline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c05019
    title: 'PS<sup>2</sup>MS: A Deep Learning-Based Prediction System for Identifying
      New Psychoactive Substances Using Mass Spectrometry'
  source_content_hash: sha256:8e568f816a80ba5b871c5f540f8e2f699211de87dbbdb279143ec121ad0e5d68
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jhhung/PS2MS
  tools:
  - PS2MS
  - SCScore
  - python
  - python 3.6.9
  - rdkit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# SMILES library generation and counting

## Summary

Enumerates a synthetic NPS database by iteratively substituting hydrogen atoms in a drug core SMILES (e.g., cathinone) with R-groups via RDKit, then filters with SCScore and Virshup drug-like criteria, yielding candidate SMILES sets (~422M for 204 groups; ~2.3M for the 30 UNODC groups).

## When to use

Use when building a large synthetic derivative library from a known drug core structure: you have a template SMILES, a functional-group (R-group) set, and need combinatorial hydrogen replacement with rule-based filtering and per-configuration compound counts.

## When NOT to use

- The analyte's chemical structure and reference spectrum are already known — PS2MS enumeration is only triggered when the GC/EI-MS spectrum matches no established reference (NIST, Wiley, SWGDRUG).
- A small, targeted analog set suffices — the full 204-group enumeration produces ~422 million compounds and needs heavy filtering; the 30 UNODC-group configuration exists precisely to restrict scale.
- The target scaffold's derivatives are expected to be highly synthesis-challenging — the ΔSCScore screen (alpha*n, alpha=1.2) intentionally excludes groups yielding difficult-to-synthesize compounds.

## Inputs

- Cathinone template SMILES (heavy-atom core preserved)
- Functional-group R-group definitions: 204 compiled groups (full configuration) and/or 30 UNODC groups (restricted configuration)
- SCScore threshold parameter alpha=1.2 and recursion depth n=3
- Virshup et al. 57 drug-like filter criteria

## Outputs

- Derivative SMILES files per configuration (204-group and 30-group runs)
- Summary log with per-configuration compound counts (~422 million vs ~2.3 million) and enumeration runtime
- Per-depth list of surviving functional groups (131 at level 2, 97 at level 3) with average ΔSCScore

## How to apply

Load the core template SMILES (cathinone) and functional-group definitions (204 compiled groups for the full run, 30 UNODC groups for the restricted run). Using RDKit, iteratively replace hydrogen atoms with R-groups recursively up to depth n=3; at each depth n>1 compute the average SCScore of n-fold modified compounds per group and retain only groups whose average ΔSCScore falls below the threshold alpha*n with alpha=1.2. Filter enumerated derivatives with the 57 drug-like criteria of Virshup et al., applying the most effective rule first — Messy rings (more than 7 smallest sets of smallest rings, via SSSR) — to save computational resources. Run the enumeration once per functional-group configuration, writing derivative SMILES per configuration, and log compound counts (~422 million for 204 groups; ~2.3 million for 30 groups) plus runtime. Validation gate: the full-parameter run (alpha=1.2, n=3) must recover all known illicit cathinone-type drugs in the UNODC database (Figure S2), with 131 groups retained at recursion level 2 and 97 at level 3.

## Related tools

- **rdkit** (Performs the heavy-atom-preserving hydrogen-to-R-group substitution on the cathinone core SMILES and computes smallest sets of smallest rings (SSSR) for the Messy rings filter) — https://github.com/jhhung/PS2MS
- **SCScore** (Computes synthetic complexity scores used to screen functional groups, retaining only those with average ΔSCScore below alpha*n (alpha=1.2))
- **PS2MS** (Hosts the enumeration module (built with g++-10/CMake 3.16) that generates the synthetic database from core-structure substitutions and records counts/runtime) — https://github.com/jhhung/PS2MS
- **python 3.6.9** (Runtime for the deep learning tools and RDKit Python bindings used in the enumeration/detection pipeline)

## Examples

```
git clone https://github.com/jhhung/PS2MS && cd PS2MS/enumeration && cmake . && make  # CMake 3.16+/g++-10; see enumeration/README.md for run options
```

## Evaluation signals

- Full-parameter run (alpha=1.2, n=3) must recover all known illicit cathinone-type drugs listed in the UNODC database (Figure S2).
- Functional-group retention must match reported values: 131 groups at the second and 97 at the third recursion level.
- Compound counts must land at approximately 422 million (204-group run) and approximately 2.3 million (30-group run).
- Filter ordering check: Messy rings (>7 SSSR rings) applied first as the most effective of the 57 Virshup drug-like criteria.
- Summary log must record both per-configuration compound counts and enumeration runtime.

## Limitations

- Recursion depth directly scales enumeration cost — 'The number of enumeration levels also has a similar impact' on downstream load.
- The ΔSCScore screen excludes highly challenging-to-synthesize compounds, so the library is deliberately not exhaustive of chemical space.
- The restricted 30-group configuration covers only cathinone-type drugs ('30 distinct functional groups that encompass all known cathinone-type drugs'); other drug scaffolds need their own template and group sets.

## Evidence

- [readme] PS2MS will generate a synthetic database by substituting hydrogen atoms in the core structure of a given drug with functional groups.: "PS2MS will generate a synthetic database by substituting hydrogen atoms in the core structure of a given drug with functional groups."
- [methods] DeltaSCScore threshold: "retained only the functional groups whose average synthesis difficulty differential (represented as ΔSCScore) fell below a threshold of αn"
- [methods] Virshup criteria: "We further introduced 57 drug-like criteria according to Virshup et al.25"
- [methods] Messy rings rule: "The most effective rule was Messy rings (more than 7 smallest sets of smallest rings)"
- [methods] Compound count: "the process of enumeration and filtering resulted in a total of approximately 422 million compounds."
- [discussion] UNODC group coverage: "there are 30 distinct functional groups that encompass all known cathinone-type
 drugs"
