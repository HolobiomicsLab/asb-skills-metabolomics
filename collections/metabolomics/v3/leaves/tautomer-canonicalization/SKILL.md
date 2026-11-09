---
name: tautomer-canonicalization
description: Use when building standardized SMILES for MS/MS library entries whose
  source SMILES may depict the same molecule as different tautomers, and a single
  canonical tautomer is needed before neutralization and downstream matching.
when_to_use_negative:
- The structures are already standardized or come from a library whose tautomers were
  canonicalized upstream — re-applying the enumerator may switch representations.
- The goal is spectral processing (peak filtering, adduct correction, similarity scoring)
  rather than chemical structure standardization; use the matchms library_cleaning
  yaml filters instead.
- The molecule is not a small molecule with ionizable tautomers, or the SMILES cannot
  be parsed/sanitized by RDKit — these records are rejected before this stage.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: RDKit
  role: Provides the TautomerEnumerator used to canonicalize tautomers of the parent
    structure, within the SanitizeMol -> Cleanup -> FragmentParent -> destereo ->
    tautomer -> neutralization pipeline
- name: Python
  role: Implementation language for the standardization module (Python 3.10 or higher
    per the README)
- name: matchms
  role: Surrounding spectral preprocessing context; spectra carrying the standardized
    structures are carried forward to the second matchms round (library_cleaning_2.yaml
    filters)
  repo: https://github.com/matchms/matchms
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5c06256
    title: Comprehensive Curation and Harmonization of Small-Molecule MS/MS Libraries
      in Spectraverse
  source_content_hash: sha256:71c7cb21f6a1bc92466ce8ac0a08e2922c1b4305bb74bef71f99a309743604bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - Python
  - RDKit
  - matchms
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# tautomer-canonicalization

## Summary

Canonicalize tautomers with RDKit's TautomerEnumerator as one stage of the Spectraverse chemical-structure standardization pipeline, so that the same compound appearing in different tautomeric forms across MGF-derived records collapses to a single standardized SMILES representation.

## When to use

Use when building standardized SMILES for MS/MS library entries whose source SMILES may depict the same molecule as different tautomers, and a single canonical tautomer is needed before neutralization and downstream matching.

## When NOT to use

- The structures are already standardized or come from a library whose tautomers were canonicalized upstream — re-applying the enumerator may switch representations.
- The goal is spectral processing (peak filtering, adduct correction, similarity scoring) rather than chemical structure standardization; use the matchms library_cleaning yaml filters instead.
- The molecule is not a small molecule with ionizable tautomers, or the SMILES cannot be parsed/sanitized by RDKit — these records are rejected before this stage.

## Inputs

- RDKit Mol objects representing salt-stripped, stereochemistry-stripped parent structures (post SanitizeMol/Cleanup/FragmentParent)
- SMILES field extracted per record from the input MGF/CSV spectra file

## Outputs

- Canonical tautomer Mol objects for downstream rule-based charge neutralization
- Standardized SMILES written to standardized_smiles.csv (including charged/uncharged duplicate rows for net-charge-altering neutralizations)
- Neutralization conflict log (e.g., zwitterions)

## How to apply

Run this stage after SanitizeMol, Cleanup, FragmentParent (salt stripping), and stereochemistry removal, operating on the parent structure of each record parsed from the SMILES field of the input MGF/CSV file. Instantiate RDKit's TautomerEnumerator (rdkit.Chem.MolStandardize) and call Canonicalize on each parent molecule to select its canonical tautomer, eliminating spurious differences between spectra that arise solely from tautomeric representation. Records that fail RDKit parsing or earlier sanitization are rejected before reaching this stage, so every molecule entering the enumerator is a valid sanitized parent. Emit the canonicalized structure (with charged/uncharged duplicates when subsequent neutralization alters net charge) into standardized_smiles.csv alongside a separate log of neutralization conflicts such as zwitterions.

## Related tools

- **RDKit** (Provides the TautomerEnumerator used to canonicalize tautomers of the parent structure, within the SanitizeMol -> Cleanup -> FragmentParent -> destereo -> tautomer -> neutralization pipeline)
- **Python** (Implementation language for the standardization module (Python 3.10 or higher per the README))
- **matchms** (Surrounding spectral preprocessing context; spectra carrying the standardized structures are carried forward to the second matchms round (library_cleaning_2.yaml filters)) — https://github.com/matchms/matchms

## Examples

```
from rdkit.Chem.MolStandardize import rdMolStandardize; canon = rdMolStandardize.TautomerEnumerator().Canonicalize(parent_mol)
```

## Evaluation signals

- Every emitted record in standardized_smiles.csv contains a valid, RDKit-parseable standardized SMILES.
- The module runs end-to-end on the full input MGF/CSV file without unhandled exceptions.
- Records differing only in tautomeric representation of the same parent compound produce identical canonical SMILES.
- Charged/uncharged duplicate rows are present for all neutralizations that alter net charge (e.g., zwitterions carried forward to the second matchms round).
- All neutralization conflicts, including zwitterions, are recorded in the separate log file.

## Limitations

- Tautomer canonicalization occurs before neutralization, so tautomers are standardized on the pre-neutralization parent structure; neutralization conflicts (e.g., zwitterions) are logged rather than resolved.
- Records that fail RDKit parsing or sanitization (SanitizeMol/Cleanup) are rejected upstream and never reach tautomer canonicalization.
- The environment requires Python 3.10 or higher; spectral_denoising's README notes RDKit currently has no distribution compatible with Python 3.13.

## Evidence

- [methods] Tautomer canonicalization step: [evidence span withheld — anchor status: not_found]
- [methods] Output and conflict logging: [evidence span withheld — anchor status: not_found]
- [results] Charged/uncharged duplication rationale: [evidence span withheld — anchor status: not_found]
- [methods] Pipeline ordering and validation: [evidence span withheld — anchor status: not_found]
- [readme] Input data placement: "Prepare your data: Place raw data files (MGF format) in the `data/` directory."
