---
name: smiles-structure-standardization
description: Use when MS/MS spectra (e.g., MGF records with a SMILES metadata field)
  carry chemically incoherent or inconsistent structure annotations — salts/counterions,
  unspecified stereochemistry, mixed tautomers, or charged vs. neutral forms — and
  need harmonization before library matching.
when_to_use_negative:
- The structures are already canonical standardized SMILES and only spectral-level
  cleaning (e.g., matchms library_cleaning_1/2.yaml filters) is needed
- Stereochemistry must be preserved for the downstream task — this pipeline removes
  stereochemistry from the parent structure by design
- Input records lack a SMILES field or the structure cannot be parsed by RDKit; these
  are rejected rather than standardized
edam_operation: http://edamontology.org/operation_3436
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: Python
  role: Implementation language for the standardization module (Python 3.10 or higher);
    RDKit transforms, pandas loading, and CSV/log output are all scripted in Python
- name: matchms
  role: 'Downstream consumer: standardized (and duplicated charged/uncharged) structures
    feed the second matchms round so zwitterionic compounds are not removed'
  repo: https://github.com/matchms/matchms
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5c06256
    title: Comprehensive Curation and Harmonization of Small-Molecule MS/MS Libraries
      in Spectraverse
  source_content_hash: sha256:a9d1d709a0e0f97599f5ac052299307106844b1e7df5a85b7757f39d58933126
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - Python
  - matchms
  edam_operation: http://edamontology.org/operation_3436
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# smiles-structure-standardization

## Summary

Standardizes chemical structure annotations (SMILES) attached to MS/MS spectra using an RDKit pipeline: sanitization, cleanup, salt stripping via FragmentParent, stereochemistry removal, tautomer canonicalization, and rule-based charge neutralization with special-case guards. It ensures each spectrum in the Spectraverse library carries a coherent, RDKit-parseable structure representation before spectral matching.

## When to use

Use when MS/MS spectra (e.g., MGF records with a SMILES metadata field) carry chemically incoherent or inconsistent structure annotations — salts/counterions, unspecified stereochemistry, mixed tautomers, or charged vs. neutral forms — and need harmonization before library matching.

## When NOT to use

- The structures are already canonical standardized SMILES and only spectral-level cleaning (e.g., matchms library_cleaning_1/2.yaml filters) is needed
- Stereochemistry must be preserved for the downstream task — this pipeline removes stereochemistry from the parent structure by design
- Input records lack a SMILES field or the structure cannot be parsed by RDKit; these are rejected rather than standardized

## Inputs

- MGF-format MS/MS spectra files with a SMILES metadata field per record (placed in the data/ directory)
- CSV/SMILES records parsed into RDKit Mol objects

## Outputs

- standardized_smiles.csv containing a valid, RDKit-parseable standardized SMILES per emitted record
- Charged/uncharged duplicate records for all net-charge-altering neutralizations
- A separate log file recording every neutralization conflict (e.g., zwitterions)

## How to apply

Load the spectra file (MGF/CSV) with pandas and extract each record's SMILES field. Parse each SMILES in RDKit, apply SanitizeMol followed by Cleanup, and reject records that fail validation. Apply FragmentParent to strip salts/counterions, then RemoveStereochemistry from the parent. Canonicalize tautomers with RDKit's TautomerEnumerator, then neutralize charges with rule-based transforms, adding the paper's special-case checks so sulfoxide and phosphoryl groups are not over-corrected. When neutralization changes the net charge, emit both the charged and uncharged copies of the structure (carrying the spectrum forward to the second matchms round) so zwitterionic compounds are not removed, and log every neutralization conflict to a separate file.

## Related tools

- **Python** (Implementation language for the standardization module (Python 3.10 or higher); RDKit transforms, pandas loading, and CSV/log output are all scripted in Python)
- **matchms** (Downstream consumer: standardized (and duplicated charged/uncharged) structures feed the second matchms round so zwitterionic compounds are not removed) — https://github.com/matchms/matchms

## Examples

```
mol = Chem.MolFromSmiles(smiles); Chem.SanitizeMol(mol); mol = Chem.FragmentParent(mol); Chem.RemoveStereochemistry(mol); standardized = Chem.MolToSmiles(enumerated_neutralized_mol)
```

## Evaluation signals

- The module runs end-to-end on the input MGF/SMILES file without unhandled exceptions
- Every emitted record in standardized_smiles.csv contains a valid, RDKit-parseable standardized SMILES
- Charged/uncharged duplicates are present for all net-charge-altering neutralizations (one spectrum copy per form)
- Sulfoxide and phosphoryl groups are not over-corrected by the neutralization transforms (paper's special-case checks pass)
- Every neutralization conflict (e.g., zwitterions) appears in the separate log file

## Limitations

- Records whose SMILES fail RDKit SanitizeMol/Cleanup validation are rejected entirely, not repaired
- Neutralization must be guarded: without the special-case checks, rule-based transforms over-correct sulfoxide and phosphoryl groups
- Emitting both charged and uncharged copies deliberately inflates record counts for charge-altering neutralizations; downstream deduplication must account for this

## Evidence

- [methods] task_006 workflow step 6: [evidence span withheld — anchor status: not_found]
- [results] task_006 finding: [evidence span withheld — anchor status: not_found]
- [methods] task_006 workflow steps 2-5: [evidence span withheld — anchor status: not_found]
- [abstract] article workflow: "developed a preprocessing pipeline to harmonize metadata, standardize chemical structures, and remove low-quality or redundant spectra"
- [readme] spectraverse-analysis README prerequisites: "- Python 3.10 or higher
- pip or conda package manager"
