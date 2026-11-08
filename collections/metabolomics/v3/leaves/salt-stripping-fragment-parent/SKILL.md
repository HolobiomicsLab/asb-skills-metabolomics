---
name: salt-stripping-fragment-parent
description: Use when standardizing SMILES annotations attached to MS/MS spectra and
  records may contain salts, solvates, or counterion components that must be stripped
  to a single fragment parent before further structure harmonization.
when_to_use_negative:
- The input is a spectrum-level problem (peak filtering, low-resolution or zero/uniform-intensity
  fragment removal) rather than a structure-representation problem — use the matchms
  cleaning filters instead
- The salt/counterion identity itself must be preserved (e.g., you need the adduct
  or counterion for neutral-mass or adduct correction in _get_neutral_mass handling)
- Structures are not parseable by RDKit at all — FragmentParent requires a successfully
  sanitized Mol
edam_operation: http://edamontology.org/operation_3436
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_3520
tools:
- name: Python
  role: Implementation language for the standardization module that parses SMILES
    and calls RDKit SanitizeMol, Cleanup, and FragmentParent
- name: skinniderlab/spectraverse-analysis
  role: Public repository containing the preprocessing code, including the structure
    standardization module implementing fragment-parent salt stripping
  repo: https://github.com/skinniderlab/spectraverse-analysis
- name: matchms
  role: Downstream library-cleaning tool whose library_cleaning rounds consume the
    standardized single-fragment parent structures
  repo: https://github.com/matchms/matchms
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5c06256
    title: Comprehensive Curation and Harmonization of Small-Molecule MS/MS Libraries
      in Spectraverse
  source_content_hash: sha256:7ac6673da81030d973b330e89dae98c1488e573463f9d7f5141f7b2504af47bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/skinniderlab/spectraverse-analysis
  tools:
  - Python
  - matchms
  - skinniderlab/spectraverse-analysis
  edam_operation: http://edamontology.org/operation_3436
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# salt-stripping-fragment-parent

## Summary

Reduces each RDKit-parsed SMILES to its molecular fragment parent (stripping salts, solvents, and counterions) as part of the Spectraverse chemical-structure standardization stage, so that all spectra of a compound map to a single canonical parent structure before tautomer and charge standardization.

## When to use

Use when standardizing SMILES annotations attached to MS/MS spectra and records may contain salts, solvates, or counterion components that must be stripped to a single fragment parent before further structure harmonization.

## When NOT to use

- The input is a spectrum-level problem (peak filtering, low-resolution or zero/uniform-intensity fragment removal) rather than a structure-representation problem — use the matchms cleaning filters instead
- The salt/counterion identity itself must be preserved (e.g., you need the adduct or counterion for neutral-mass or adduct correction in _get_neutral_mass handling)
- Structures are not parseable by RDKit at all — FragmentParent requires a successfully sanitized Mol

## Inputs

- MGF or CSV spectra file with a SMILES metadata field per record
- RDKit Mol objects parsed from the SMILES strings (post SanitizeMol + Cleanup)

## Outputs

- Single-fragment parent SMILES per record (salt/counterion-free), feeding stereochemistry removal, TautomerEnumerator canonicalization, and charge neutralization
- Log of records rejected at sanitization/cleanup or FragmentParent stages

## How to apply

Load the spectra file (MGF/CSV) with pandas and extract the SMILES field per record, then parse each SMILES in RDKit and apply SanitizeMol followed by Cleanup, rejecting records that fail validation. On the sanitized molecule apply RDKit's FragmentParent to strip salts and counterions, selecting the largest remaining fragment as the parent structure. Perform this step after cleanup but before stereochemistry removal, tautomer canonicalization with the TautomerEnumerator, and rule-based charge neutralization. Fragments that fail FragmentParent (or the earlier sanitization) should be logged and excluded rather than passed downstream. Downstream steps in the Spectraverse pipeline (matchms rounds driven by library_cleaning_1.yaml and library_cleaning_2.yaml) assume every structure is a single-fragment parent, so salt stripping must be complete before those rounds run.

## Related tools

- **Python** (Implementation language for the standardization module that parses SMILES and calls RDKit SanitizeMol, Cleanup, and FragmentParent)
- **skinniderlab/spectraverse-analysis** (Public repository containing the preprocessing code, including the structure standardization module implementing fragment-parent salt stripping) — https://github.com/skinniderlab/spectraverse-analysis
- **matchms** (Downstream library-cleaning tool whose library_cleaning rounds consume the standardized single-fragment parent structures) — https://github.com/matchms/matchms

## Examples

```
mol = Chem.MolFromSmiles(smiles); Chem.SanitizeMol(mol); Chem.Cleanup(mol); parent = Chem.RemoveHs(Chem.MolFromSmiles(Chem.MolToSmiles(Chem.GetMolFrags(mol, asMols=True, largestOnly=True)[0])))
```

## Evaluation signals

- Every emitted standardized record is RDKit-parseable and contains exactly one fragment (no residual salt or counterion components)
- The module runs end-to-end on the input MGF/CSV without unhandled exceptions
- Records failing SanitizeMol/Cleanup or FragmentParent are logged and excluded rather than silently passed through
- Parent structures are consistent with subsequent steps: charged/uncharged duplicate SMILES appear for every net-charge-altering neutralization downstream
- Output is written to standardized_smiles.csv with every row carrying a valid standardized SMILES

## Limitations

- FragmentParent selection assumes the desired parent is the retained fragment; compounds where the salt/counterion is chemically meaningful may be over-stripped
- Salt stripping occurs before charge neutralization, so zwitterionic compounds whose neutralization changes net charge must still be duplicated downstream to avoid removal
- Records whose SMILES fail RDKit sanitization cannot be repaired and are rejected outright

## Evidence

- [methods] Apply FragmentParent to strip salts/counterions, then remove stereochemistry from the resulting parent structure.: [evidence span withheld — anchor status: not_found]
- [methods] Parse each SMILES in RDKit and apply SanitizeMol followed by Cleanup, rejecting records that fail validation.: [evidence span withheld — anchor status: not_found]
- [abstract] developed a preprocessing pipeline to harmonize metadata, standardize chemical structures, and remove low-quality or redundant spectra: "developed a preprocessing pipeline to harmonize metadata, standardize chemical structures, and remove low-quality or redundant spectra"
- [discussion] our preprocessing code is publicly available via GitHub (https://github.com/skinniderlab/spectraverse-analysis): "our preprocessing code is publicly available via GitHub (https://github.com/skinniderlab/spectraverse-analysis)"
