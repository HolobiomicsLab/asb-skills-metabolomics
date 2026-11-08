---
name: charge-neutralization-rule-handling
description: Use when a spectral library (MGF/CSV with a SMILES field) contains charged
  or zwitterionic molecules that must be neutralized with rule-based RDKit transforms,
  and net-charge changes require duplicating spectra as charged and uncharged forms.
when_to_use_negative:
- Input spectra already carry standardized, neutralized structures and no SMILES re-standardization
  is needed
- Spectra lack structure annotations entirely (SMILES field missing), so only spectral
  filters (e.g., matchms library_cleaning_1.yaml) apply
- The goal is peak-level spectrum cleaning (e.g., removing spectra with identical
  fragment intensities or low-resolution spectra) rather than chemical-structure standardization
edam_operation: http://edamontology.org/operation_0266
edam_topics:
- http://edamontology.org/topic_0154
tools:
- name: RDKit
  role: SanitizeMol, Cleanup, FragmentParent, stereochemistry removal, TautomerEnumerator
    canonicalization, and rule-based charge neutralization transforms
- name: Python
  role: Implementation language for the standardization module (Python 3.10 or higher),
    loading MGF/CSV spectra with pandas and emitting standardized SMILES
- name: matchms
  role: Second-round library cleaning (library_cleaning_2.yaml filters) in which charged
    and uncharged copies of the spectrum are carried forward so zwitterions are not
    removed
  repo: https://github.com/matchms/matchms
- name: spectraverse-analysis
  role: Public repository containing the preprocessing code implementing the structure
    standardization and neutralization pipeline
  repo: https://github.com/skinniderlab/spectraverse-analysis
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5c06256
    title: Comprehensive Curation and Harmonization of Small-Molecule MS/MS Libraries
      in Spectraverse
  source_content_hash: sha256:c8734f879456ea8622d6d56da94cce5988d3f6ffbc298960d88068b280bde81b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - Python
  - RDKit
  - matchms
  - spectraverse-analysis
  edam_operation: http://edamontology.org/operation_0266
  edam_topics:
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# charge-neutralization-rule-handling

## Summary

RDKit-based charge neutralization step of the Spectraverse chemical-structure standardization module: rule-based transforms neutralize formal charges after sanitization, cleanup, fragment-parent, and tautomer canonicalization, emitting both charged and uncharged copies of a spectrum when neutralization alters net charge so zwitterionic compounds are not removed.

## When to use

Use when a spectral library (MGF/CSV with a SMILES field) contains charged or zwitterionic molecules that must be neutralized with rule-based RDKit transforms, and net-charge changes require duplicating spectra as charged and uncharged forms.

## When NOT to use

- Input spectra already carry standardized, neutralized structures and no SMILES re-standardization is needed
- Spectra lack structure annotations entirely (SMILES field missing), so only spectral filters (e.g., matchms library_cleaning_1.yaml) apply
- The goal is peak-level spectrum cleaning (e.g., removing spectra with identical fragment intensities or low-resolution spectra) rather than chemical-structure standardization

## Inputs

- Spectra file in MGF or CSV format containing a SMILES field per record (loaded with pandas/Python)
- RDKit-parsable SMILES strings, including salts/counterions and charged or zwitterionic structures

## Outputs

- standardized_smiles.csv with valid, RDKit-parsable standardized SMILES per record
- Charged and uncharged duplicate spectrum records for all net-charge-altering neutralizations
- Separate log file recording every neutralization conflict (e.g., zwitterions)

## How to apply

Parse each SMILES field from the spectra file (MGF/CSV loaded with pandas) in RDKit, applying SanitizeMol followed by Cleanup and rejecting records that fail validation. Apply FragmentParent to strip salts/counterions, remove stereochemistry, and canonicalize tautomers with RDKit's TautomerEnumerator. Neutralize charges with rule-based transforms, adding the paper's special-case checks so sulfoxide and phosphoryl groups are not over-corrected. When neutralization changes the molecule's net charge, emit two copies of the spectrum — one with the charged form and one with the uncharged form — and carry both forward to the second matchms round (library_cleaning_2.yaml filters) to prevent removal of zwitterionic compounds. Write standardized structures to standardized_smiles.csv and record every neutralization conflict (e.g., zwitterions) in a separate log file.

## Related tools

- **RDKit** (SanitizeMol, Cleanup, FragmentParent, stereochemistry removal, TautomerEnumerator canonicalization, and rule-based charge neutralization transforms)
- **Python** (Implementation language for the standardization module (Python 3.10 or higher), loading MGF/CSV spectra with pandas and emitting standardized SMILES)
- **matchms** (Second-round library cleaning (library_cleaning_2.yaml filters) in which charged and uncharged copies of the spectrum are carried forward so zwitterions are not removed) — https://github.com/matchms/matchms
- **spectraverse-analysis** (Public repository containing the preprocessing code implementing the structure standardization and neutralization pipeline) — https://github.com/skinniderlab/spectraverse-analysis

## Examples

```
from rdkit import Chem; mol = Chem.MolFromSmiles(smi); Chem.SanitizeMol(mol); mol = Chem.Cleanup(mol)
```

## Evaluation signals

- The module runs end-to-end on the input file without unhandled exceptions
- Every emitted record contains a valid, RDKit-parseable standardized SMILES
- Charged/uncharged duplicates are present for all neutralizations that alter net charge
- Every neutralization conflict (e.g., zwitterions) is recorded in the separate log file
- Sulfoxide and phosphoryl groups survive neutralization without over-correction (special-case checks applied)

## Limitations

- Records whose SMILES fail RDKit SanitizeMol/Cleanup validation are rejected rather than repaired
- Neutralization conflicts such as zwitterions must be logged and handled by duplicating spectra rather than resolving to a single form
- RDKit has no distribution compatible with Python 3.13, constraining the runtime environment for this step

## Evidence

- [abstract] developed a preprocessing pipeline to harmonize metadata, standardize chemical structures, and remove low-quality or redundant spectra: "developed a preprocessing pipeline to harmonize metadata, standardize chemical structures, and remove low-quality or redundant spectra"
- [methods] Run matchms again using filters specified in matchms/library_cleaning_2.yaml.: "Run matchms again using filters specified in matchms/library_cleaning_2.yaml."
- [discussion] our preprocessing code is publicly available via GitHub (https://github.com/skinniderlab/spectraverse-analysis): "our preprocessing code is publicly available via GitHub (https://github.com/skinniderlab/spectraverse-analysis)"
- [readme] Matchms offers an array of tools for metadata cleaning and validation, alongside basic peak filtering, to ensure data accuracy and integrity.: [evidence span withheld — anchor status: external_doc]
