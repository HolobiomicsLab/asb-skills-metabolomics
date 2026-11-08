---
name: zwitterion-spectral-library-curation
description: Use when standardizing SMILES annotations in an MS/MS spectral library
  (MGF/CSV) and rule-based charge neutralization would change the net charge — e.g.,
  zwitterionic metabolites — causing one ionic form to be lost in later filtering.
when_to_use_negative:
- Input records fail RDKit SMILES parsing/sanitization — they are rejected before
  neutralization is ever attempted
- Neutralization does not change the molecule's net charge — no duplication is needed;
  a single standardized copy suffices
- The task is generic peak-quality filtering (e.g., removing spectra with identical
  fragment intensities or low-resolution spectra via library_cleaning yaml filters)
  rather than charge-state handling of chemical structures
edam_operation: http://edamontology.org/operation_4003
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: RDKit
  role: Parse SMILES, apply SanitizeMol, Cleanup, FragmentParent, TautomerEnumerator,
    and rule-based charge neutralization transforms
- name: Python
  role: Implements the standardization module (Python 3.10+), loading MGF/CSV with
    pandas and emitting charged/uncharged spectrum copies
- name: matchms
  role: Receives both charged and uncharged spectrum copies in the second library
    cleaning round (library_cleaning_2.yaml) so zwitterionic compounds are not removed
  repo: https://github.com/matchms/matchms
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5c06256
    title: Comprehensive Curation and Harmonization of Small-Molecule MS/MS Libraries
      in Spectraverse
  source_content_hash: sha256:3f8c8afd94a0c472878a2998c852df4ce1096b47b9511bd201fedef39d3398d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - Python
  - RDKit
  - matchms
  edam_operation: http://edamontology.org/operation_4003
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# zwitterion-spectral-library-curation

## Summary

Handles charge-altering neutralization during chemical-structure standardization of MS/MS spectral libraries: when neutralization changes a molecule's net charge (e.g., zwitterions), both the charged and uncharged forms are emitted as separate spectrum copies so zwitterionic compounds are not removed in downstream matchms cleaning rounds.

## When to use

Use when standardizing SMILES annotations in an MS/MS spectral library (MGF/CSV) and rule-based charge neutralization would change the net charge — e.g., zwitterionic metabolites — causing one ionic form to be lost in later filtering.

## When NOT to use

- Input records fail RDKit SMILES parsing/sanitization — they are rejected before neutralization is ever attempted
- Neutralization does not change the molecule's net charge — no duplication is needed; a single standardized copy suffices
- The task is generic peak-quality filtering (e.g., removing spectra with identical fragment intensities or low-resolution spectra via library_cleaning yaml filters) rather than charge-state handling of chemical structures

## Inputs

- MGF or CSV spectra file with a SMILES field per record
- Per-record SMILES strings parseable by RDKit (after SanitizeMol/Cleanup and FragmentParent salt-stripping)

## Outputs

- standardized_smiles.csv containing the standardized structures
- Separate log file recording every neutralization conflict (e.g., zwitterions)
- Duplicated spectrum records (charged + uncharged copies) for all net-charge-altering neutralizations, carried into the second matchms cleaning round

## How to apply

During the structure-standardization stage, parse each SMILES from the spectra file with RDKit, apply SanitizeMol and Cleanup, then apply FragmentParent to strip salts/counterions and remove stereochemistry. Canonicalize tautomers with RDKit's TautomerEnumerator, then neutralize charges using rule-based transforms that include special-case checks so sulfoxide and phosphoryl groups are not over-corrected. At the decision point: if neutralization changes the molecule's net charge, emit TWO copies of the spectrum — one associated with the charged form and one with the uncharged form — and carry both forward to the second matchms round (library_cleaning_2.yaml filters) to prevent removal of zwitterionic compounds. Record every neutralization conflict (e.g., zwitterions) in a separate log file, and write the standardized structures to standardized_smiles.csv.

## Related tools

- **RDKit** (Parse SMILES, apply SanitizeMol, Cleanup, FragmentParent, TautomerEnumerator, and rule-based charge neutralization transforms)
- **Python** (Implements the standardization module (Python 3.10+), loading MGF/CSV with pandas and emitting charged/uncharged spectrum copies)
- **matchms** (Receives both charged and uncharged spectrum copies in the second library cleaning round (library_cleaning_2.yaml) so zwitterionic compounds are not removed) — https://github.com/matchms/matchms

## Examples

```
from rdkit import Chem; mol = Chem.MolFromSmiles(smi); Chem.SanitizeMol(mol); mol = rdMolStandardize.FragmentParent(mol)
```

## Evaluation signals

- The module runs end-to-end on the input spectra file without unhandled exceptions
- Every emitted record contains a valid, RDKit-parseable standardized SMILES
- Charged/uncharged duplicates are present for all net-charge-altering neutralizations (zwitterions yield two spectrum copies)
- Every neutralization conflict is recorded in the separate log file
- Both copies survive the second matchms round (library_cleaning_2.yaml) rather than being removed as redundant

## Limitations

- Rule-based neutralization needs special-case checks so sulfoxide and phosphoryl groups are not over-corrected
- RDKit lacks a distribution compatible with Python 3.13; Python 3.10–3.12 (project requires 3.10+) is the safe range
- Duplicating spectra doubles record counts for zwitterions, which must be accounted for in downstream de-duplication

## Evidence

- [methods] When neutralization changes the net charge, emit both the charged and uncharged copies of the structure.: [evidence span withheld — anchor status: not_found]
- [methods] Neutralization conflict handling in the pipeline: [evidence span withheld — anchor status: not_found]
- [methods] As part of the structure standardization pipeline, when neutralization altered a molecule's overall charge, two copies of the spectrum were created: [evidence span withheld — anchor status: not_found]
- [readme] Preprocessing code availability: "our preprocessing code is publicly available via GitHub (https://github.com/skinniderlab/spectraverse-analysis), accompanied by extensive documentation"
