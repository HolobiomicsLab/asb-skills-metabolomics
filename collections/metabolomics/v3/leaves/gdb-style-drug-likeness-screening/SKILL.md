---
name: gdb-style-drug-likeness-screening
description: Use when a large enumerated SMILES set of substituted derivatives must
  be reduced to a drug-like, synthetically tractable candidate database before spectrum
  prediction, using explicit rule-based filters.
when_to_use_negative:
- When the analyte's mass spectrum already matches an established reference library
  (NIST, Wiley, SWGDRUG) — the conventional screening path, not enumeration/filtering,
  applies
- When screening raw experimental spectra rather than enumerated candidate structures
  — this skill operates on SMILES sets, not MS data
- When the goal is to keep synthetically challenging or non-drug-like chemical space
  (e.g., GDB-style chemical-universe exploration) — the drug-like and ΔSCScore criteria
  would wrongly discard it
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: rdkit
  role: Molecular processing toolkit used to evaluate the 57 drug-like rules, perceive
    smallest sets of smallest rings for the messy-rings rule, and compute properties
    over each derivative
  repo: https://www.rdkit.org/docs/Install.html
- name: PS2MS
  role: Pipeline whose enumeration and rule-based filtering stage this skill implements;
    filtered compounds feed its NEIMS mass-spectrum prediction step
  repo: https://github.com/jhhung/PS2MS
- name: SCScore
  role: Synthetic-complexity metric used to compute the ΔSCScore differential that
    screens out highly challenging-to-synthesize functional groups before enumeration
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c05019
    title: 'PS<sup>2</sup>MS: A Deep Learning-Based Prediction System for Identifying
      New Psychoactive Substances Using Mass Spectrometry'
  source_content_hash: sha256:f9c196be82887fcffb72c227bbb46b740598a2560885e777dba6ca9929f64d15
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PS2MS
  - SCScore
  - python
  - rdkit
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# gdb-style-drug-likeness-screening

## Summary

A multi-criteria molecular filtering step (GDB/Virshup-style drug-like rules plus ring-complexity and synthetic-accessibility screens) that reduces an enumerated derivative set to a manageable, synthetically plausible NPS database (~422 million compounds in PS²MS).

## When to use

Use when a large enumerated SMILES set of substituted derivatives must be reduced to a drug-like, synthetically tractable candidate database before spectrum prediction, using explicit rule-based filters.

## When NOT to use

- When the analyte's mass spectrum already matches an established reference library (NIST, Wiley, SWGDRUG) — the conventional screening path, not enumeration/filtering, applies
- When screening raw experimental spectra rather than enumerated candidate structures — this skill operates on SMILES sets, not MS data
- When the goal is to keep synthetically challenging or non-drug-like chemical space (e.g., GDB-style chemical-universe exploration) — the drug-like and ΔSCScore criteria would wrongly discard it

## Inputs

- Enumerated derivative SMILES set (e.g., enumerated_compounds.smi from the hydrogen-to-functional-group enumeration step on a drug core such as cathinone)
- Per-derivative molecular structures for ring perception and property calculation (RDKit mol objects)
- SCScore values for functional groups (for ΔSCScore threshold αn)

## Outputs

- Filtered compound set (filtered_compounds.smi) of drug-like, synthetically tractable derivatives (~422 million for the cathinone template)
- Per-rule elimination counts (filter_counts.csv) ranking filter rules by number of derivatives removed

## How to apply

Load the enumerated derivative SMILES produced by hydrogen-to-functional-group substitution on the drug template (e.g., cathinone). Evaluate all 57 drug-like criteria from Virshup et al. against every derivative with RDKit molecular processing (smallest sets of smallest rings perception, valence/stability checks, property calculations), discarding compounds failing any rule. Apply the messy rings rule, removing any compound with more than 7 smallest sets of smallest rings — this was the single most effective (largest-removing) filter in the PS²MS evaluation. Additionally screen functional groups by synthetic accessibility: retain only functional groups whose average ΔSCScore falls below a threshold αn so that highly challenging-to-synthesize derivatives are excluded from enumeration. Tally per-rule elimination counts and assemble the survivors (filtered_compounds.smi) for downstream NEIMS mass-spectrum prediction.

## Related tools

- **rdkit** (Molecular processing toolkit used to evaluate the 57 drug-like rules, perceive smallest sets of smallest rings for the messy-rings rule, and compute properties over each derivative) — https://www.rdkit.org/docs/Install.html
- **PS2MS** (Pipeline whose enumeration and rule-based filtering stage this skill implements; filtered compounds feed its NEIMS mass-spectrum prediction step) — https://github.com/jhhung/PS2MS
- **SCScore** (Synthetic-complexity metric used to compute the ΔSCScore differential that screens out highly challenging-to-synthesize functional groups before enumeration)

## Examples

```
from rdkit import Chem; mol = Chem.MolFromSmiles(smi); keep = mol is not None and len(Chem.GetSymmSSSS(mol)) <= 7  # messy rings rule: discard >7 SSSR
```

## Evaluation signals

- filter_counts.csv per-rule tally must reproduce the paper's pattern that the messy rings rule (>7 SSSR) removes the largest share of derivatives
- Filtered set scale should be consistent with the paper's ~422 million compounds after enumeration and filtering of the cathinone template
- Every surviving compound passes all 57 Virshup et al. drug-like criteria (no rule violations)
- No retained compound has more than 7 smallest sets of smallest rings
- All retained functional groups have average ΔSCScore below the threshold αn

## Limitations

- Filter aggressiveness depends on the number of enumeration levels and thresholds; the paper notes the number of enumeration levels has a similar (unspecified) impact on results
- The rule set biases the database toward drug-like, synthetically accessible space, which may exclude unusual NPS structures
- RDKit must be built from source and installed via conda per the README; requires python 3.6.9+ and g++-10/CMake 3.16.0 for the pipeline

## Evidence

- [methods] 57 drug-like criteria from Virshup et al.: "We further introduced 57 drug-like criteria according to Virshup et al.25"
- [methods] Messy rings rule is most effective: "The most effective rule was Messy rings (more than 7 smallest sets of smallest rings)"
- [methods] Resulting database scale: "the process of enumeration and filtering resulted in a total of approximately 422 million compounds."
- [methods] ΔSCScore functional-group screening: "retained only the functional groups whose average synthesis difficulty differential (represented as ΔSCScore) fell below a threshold of αn"
- [readme] Enumeration input to filtering: "PS<sup>2</sup>MS will generate a synthetic database by substituting hydrogen atoms in the core structure of a given drug with functional groups."
