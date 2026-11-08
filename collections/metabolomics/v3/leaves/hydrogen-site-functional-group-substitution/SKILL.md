---
name: hydrogen-site-functional-group-substitution
description: Use when building a synthetic compound database from a known illicit-drug
  template (e.g., cathinone) and no reference spectra exist for emerging NPS analogs.
when_to_use_negative:
- When the analyte's mass spectrum already matches an established reference spectrum
  (NIST, Wiley, SWGDRUG) — PS2MS enumeration is only triggered when no reference match
  is found
- When the chemical structure of the NPS is already known — in silico EI-MS prediction
  via NEIMS can be applied directly without derivative enumeration
- When the target drug class has a core structure other than the preselected template
  (here cathinone) — the substitution is defined on a specific drug template's hydrogen
  sites
edam_operation: http://edamontology.org/operation_2481
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: rdkit
  role: Performs the iterative replacement of hydrogen atoms in the drug core template
    with functional-group R-groups; installed from conda with C++ code built from
    source
  repo: https://www.rdkit.org/docs/Install.html
- name: SCScore
  role: Computes average synthetic complexity scores of n-fold modified compounds
    to screen functional groups whose ΔSCScore exceeds αn (α=1.2)
- name: PS2MS
  role: Hosts the enumeration step (built with g++-10/CMake 3.16.0) that generates
    the synthetic NPS database by hydrogen-site substitution
  repo: https://github.com/jhhung/PS2MS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c05019
    title: 'PS<sup>2</sup>MS: A Deep Learning-Based Prediction System for Identifying
      New Psychoactive Substances Using Mass Spectrometry'
  source_content_hash: sha256:b50e5c9f45b7d6a27f46284024ae93a82e6028cd4e036433c3e30f8cc47724cd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PS2MS
  - SCScore
  - python
  - rdkit
  edam_operation: http://edamontology.org/operation_2481
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# hydrogen-site-functional-group-substitution

## Summary

Enumerates candidate derivatives of a known drug core (e.g., cathinone) by recursively replacing hydrogen atoms with functional-group R-groups using RDKit, then prunes the combinatorial space with SCScore-based synthesis-difficulty thresholds and Virshup drug-like criteria to build PS2MS's synthetic NPS database.

## When to use

Use when building a synthetic compound database from a known illicit-drug template (e.g., cathinone) and no reference spectra exist for emerging NPS analogs.

## When NOT to use

- When the analyte's mass spectrum already matches an established reference spectrum (NIST, Wiley, SWGDRUG) — PS2MS enumeration is only triggered when no reference match is found
- When the chemical structure of the NPS is already known — in silico EI-MS prediction via NEIMS can be applied directly without derivative enumeration
- When the target drug class has a core structure other than the preselected template (here cathinone) — the substitution is defined on a specific drug template's hydrogen sites

## Inputs

- Template drug core structure in SMILES notation (e.g., cathinone)
- Functional-group definitions: 204 compiled groups (full configuration) and 30 UNODC groups (restricted configuration)
- Recursion depth parameter n (maximum 3) and SCScore threshold alpha (=1.2)
- Virshup et al. drug-like criteria (57 rules)

## Outputs

- Derivative SMILES sets per configuration (204-group and 30-group runs)
- Synthetic NPS database of enumerated cathinone derivatives (~422 million compounds for 204 groups, ~2.3 million for 30 groups)
- Summary log with compound count per configuration and enumeration runtime
- Per-depth functional-group retention statistics (e.g., 131 groups retained at level 2, 97 at level 3)

## How to apply

Load the cathinone template SMILES plus a functional-group definition set (204 compiled groups for the full run, the 30 UNODC groups that cover all known cathinone-type drugs for the restricted run), preserving the heavy-atom core. Iteratively replace hydrogen atoms with functional-group R-groups using RDKit, recursing up to depth n=3; at each depth n>1 compute the average SCScore of n-fold modified compounds per group and retain only groups whose average ΔSCScore falls below αn with α=1.2. Apply the 57 drug-like criteria of Virshup et al. to filter enumerated derivatives, prioritizing the most effective rule (Messy rings: more than 7 smallest sets of smallest rings) to save computational resources. Run the enumeration twice (204-group and 30-group configurations), write derivative SMILES per configuration, and log compound counts (~422 million for the 204-group run, ~2.3 million for the 30-group run) and runtime. Rationale: SCScore screening avoids including highly challenging-to-synthesize compounds, keeping the database chemically plausible and tractable for downstream NEIMS spectrum prediction.

## Related tools

- **rdkit** (Performs the iterative replacement of hydrogen atoms in the drug core template with functional-group R-groups; installed from conda with C++ code built from source) — https://www.rdkit.org/docs/Install.html
- **SCScore** (Computes average synthetic complexity scores of n-fold modified compounds to screen functional groups whose ΔSCScore exceeds αn (α=1.2))
- **PS2MS** (Hosts the enumeration step (built with g++-10/CMake 3.16.0) that generates the synthetic NPS database by hydrogen-site substitution) — https://github.com/jhhung/PS2MS

## Evaluation signals

- Full-parameter enumeration (α=1.2, n=3) must recover all known illicit cathinone-type drugs listed in the UNODC database (Figure S2)
- Functional-group retention counts match expected values: 131 groups at the second and 97 at the third recursion level
- Total compound counts fall near expected magnitudes: ~422 million for the 204-group run and ~2.3 million for the 30-group run
- All enumerated derivatives satisfy the 57 Virshup drug-like criteria, including the Messy rings rule (no more than 7 smallest sets of smallest rings)
- Summary log records a compound count and runtime for each of the two configurations (204-group and 30-group)

## Limitations

- Enumeration is explosive in scale (~422 million compounds for 204 groups), so rule-based filtering — prioritizing the Messy rings rule — is required to save computational resources
- Only the 30 UNODC functional groups are guaranteed to encompass all known cathinone-type drugs; the full 204-group set trades tractability for coverage
- The number of enumeration levels has a similar impact on results but the exact optimal level count is not fully specified; SCScore screening biases the database toward synthetically accessible compounds only

## Evidence

- [readme] PS2MS will generate a synthetic database by substituting hydrogen atoms in the core structure of a given drug with functional groups.: "PS<sup>2</sup>MS will generate a synthetic database by substituting hydrogen atoms in the core structure of a given drug with functional groups."
- [methods] Enumeration and filtering of cathinone derivatives resulted in a total of approximately 422 million compounds.: "the process of enumeration and filtering resulted in a total of approximately 422 million compounds."
- [methods] Functional groups retained only if average ΔSCScore fell below threshold αn.: "retained only the functional groups whose average synthesis difficulty differential (represented as ΔSCScore) fell below a threshold of αn"
- [methods] Messy rings was the most effective drug-like filtering rule.: "The most effective rule was Messy rings (more than 7 smallest sets of smallest rings)"
- [discussion] 30 distinct functional groups encompass all known cathinone-type drugs.: "there are 30 distinct functional groups that encompass all known cathinone-type
drugs"
