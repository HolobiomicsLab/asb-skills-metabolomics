---
name: drug-like-criteria-molecular-filtering
description: Use when a combinatorially enumerated derivative database (hydrogen-to-R-group
  substitutions on a drug core template) must be pruned to drug-like, synthesizable
  candidates before downstream spectrum prediction or ranking.
when_to_use_negative:
- The analyte's GC/EI-MS spectrum already matches a reference spectrum in NIST, Wiley,
  or SWGDRUG libraries — conventional screening and reference-standard confirmation
  suffice, so no derivative enumeration/filtering is needed.
- The input is a single known-structure compound to identify rather than an unenumerated
  drug core — filtering criteria operate on enumerated derivative sets, not individual
  structures.
- You are at the detection stage where candidates are already enumerated and ranked;
  there, PS2MS instead uses M+ peak-based mass weight filtering within a tolerance
  range, not the Virshup drug-like criteria.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: PS2MS
  role: Hosts the enumeration-and-rule-based-filtering step that generates the synthetic
    NPS database from substituted core structures
  repo: https://github.com/jhhung/PS2MS
- name: SCScore
  role: Computes average synthetic complexity of n-fold modified compounds to screen
    functional groups via ΔSCScore below α·n
  repo: https://github.com/jhhung/PS2MS
- name: rdkit
  role: Performs hydrogen-to-R-group substitution on the core SMILES and perceives
    smallest sets of smallest rings for the Messy rings criterion
  repo: https://github.com/jhhung/PS2MS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c05019
    title: 'PS<sup>2</sup>MS: A Deep Learning-Based Prediction System for Identifying
      New Psychoactive Substances Using Mass Spectrometry'
  source_content_hash: sha256:784a396129756ce457c27c6cb5989a06abc72cdf985dc1b7a0752738c5bc5c70
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jhhung/PS2MS
  tools:
  - PS2MS
  - SCScore
  - python
  - rdkit
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# drug-like-criteria-molecular-filtering

## Summary

Prunes an enumerated derivative set (e.g., cathinone-core compounds built with RDKit) using the 57 Virshup et al. drug-like criteria — prioritizing the 'Messy rings' rule (>7 smallest sets of smallest rings) — and an SCScore-based ΔSCScore threshold (α=1.2 per recursion depth, n=3) to keep the synthetic NPS database tractable and synthetically plausible.

## When to use

Use when a combinatorially enumerated derivative database (hydrogen-to-R-group substitutions on a drug core template) must be pruned to drug-like, synthesizable candidates before downstream spectrum prediction or ranking.

## When NOT to use

- The analyte's GC/EI-MS spectrum already matches a reference spectrum in NIST, Wiley, or SWGDRUG libraries — conventional screening and reference-standard confirmation suffice, so no derivative enumeration/filtering is needed.
- The input is a single known-structure compound to identify rather than an unenumerated drug core — filtering criteria operate on enumerated derivative sets, not individual structures.
- You are at the detection stage where candidates are already enumerated and ranked; there, PS2MS instead uses M+ peak-based mass weight filtering within a tolerance range, not the Virshup drug-like criteria.

## Inputs

- Cathinone template SMILES (heavy-atom core preserved)
- Functional-group R-group definitions: 204 compiled groups (full run) or 30 UNODC groups (restricted run)
- SCScore model for computing average synthesis difficulty of n-fold modified compounds
- Recursion depth n and threshold α (paper values: n=3, α=1.2)

## Outputs

- Filtered derivative SMILES per configuration (204-group and 30-group runs)
- Per-depth retained functional-group sets (e.g., 131 groups at depth 2, 97 at depth 3)
- Summary log with compound count (~422 million full; ~2.3 million UNODC) and enumeration runtime

## How to apply

Enumerate derivatives by iteratively replacing hydrogens on the preserved heavy-atom core (cathinone) with functional-group R-groups in RDKit, recursively up to depth n=3. At each depth n>1, compute the average SCScore of n-fold modified compounds per functional group and retain only groups whose average ΔSCScore falls below the threshold α·n with α=1.2, avoiding highly challenging-to-synthesize compounds. Then apply the 57 drug-like criteria of Virshup et al., prioritizing the most effective rule, 'Messy rings' (more than 7 smallest sets of smallest rings), to save computational resources. Run the filtering configuration twice — once with all 204 compiled functional groups and once with the 30 UNODC groups — and log compound counts (~422 million vs ~2.3 million) and runtime per configuration. Validation: the full-parameter setting (α=1.2, n=3) must recover all known illicit cathinone-type drugs in the UNODC database, with 131 functional groups retained at the second and 97 at the third recursion level.

## Related tools

- **PS2MS** (Hosts the enumeration-and-rule-based-filtering step that generates the synthetic NPS database from substituted core structures) — https://github.com/jhhung/PS2MS
- **SCScore** (Computes average synthetic complexity of n-fold modified compounds to screen functional groups via ΔSCScore below α·n) — https://github.com/jhhung/PS2MS
- **rdkit** (Performs hydrogen-to-R-group substitution on the core SMILES and perceives smallest sets of smallest rings for the Messy rings criterion) — https://github.com/jhhung/PS2MS

## Evaluation signals

- Full-parameter run (α=1.2, n=3) recovers all known illicit cathinone-type drugs listed in the UNODC database (Figure S2).
- Functional-group retention counts match expectations: 131 groups at the second and 97 at the third recursion level.
- Compound counts fall at the reported scale: ~422 million for the 204-group configuration and ~2.3 million for the 30-UNODC-group configuration.
- The Messy rings rule (>7 smallest sets of smallest rings) is confirmed as the most effective of the 57 Virshup et al. drug-like criteria.
- Summary log records enumeration runtime and per-configuration compound counts without contradiction between the two runs.

## Limitations

- α and n trade off database size vs coverage: deeper enumeration levels and stricter/looser thresholds change which functional groups survive and affect recovery of known drugs.
- SCScore screening excludes highly challenging-to-synthesize compounds, so genuinely novel but synthetically difficult NPS may be pruned.
- The Virshup drug-like criteria are rule-based; derivatives passing the filters still require downstream spectrum prediction (NEIMS) and mass-weight filtering via the M+ peak before identification.

## Evidence

- [methods] 57 drug-like criteria applied with Messy rings prioritized: "We further introduced 57 drug-like criteria according to Virshup et al.25"
- [methods] Messy rings is the most effective rule: "The most effective rule was Messy rings (more than 7 smallest sets of smallest rings)"
- [methods] ΔSCScore threshold for functional-group retention: "retained only the functional groups whose average synthesis difficulty differential (represented as ΔSCScore) fell below a threshold of αn"
- [readme] Enumeration by hydrogen substitution on the drug core: "PS<sup>2</sup>MS will generate a synthetic database by substituting hydrogen atoms in the core structure of a given drug with functional groups."
- [discussion] SCScore screening rationale: "We utilize these
scores to screen functional groups, thus avoiding the inclusion of
highly challenging-to-synthesize compounds in the enumeration process"
