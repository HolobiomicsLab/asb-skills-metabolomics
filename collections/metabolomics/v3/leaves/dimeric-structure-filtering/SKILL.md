---
name: dimeric-structure-filtering
description: Use when a cleaned structure table contains entries whose SMILES still
  has a '.' after RDKit parsing and MolVS sanitization, so dimeric entries must be
  dropped before standardizing to InChI.
when_to_use_negative:
- Input is already a cleaned, single-component structure table with no '.' in the
  SMILES column
- You need to keep covalent multi-component records intact rather than reject them
  — this filter discards, it does not split or merge components
- You are working with spectra or mass spectrometry peak lists rather than SMILES/structure
  tables
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3336
tools:
- name: RDKit
  role: Parses SMILES strings into molecular objects and flags entries containing
    '.' (dimeric structures)
- name: MolVS
  role: Sanitizes each molecule via standardizing, fragment removal, and uncharging
    so multi-component inputs are resolved before the residual dimer check
- name: sanitizing.py
  role: Script from the lotus-processor pipeline implementing the three-stage sanitization
    feeding this filter
  repo: https://github.com/lotusnprod/lotus-processor
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.7554/elife.70780
    title: The LOTUS initiative for open knowledge management in natural products
      research
  source_content_hash: sha256:7347de49486265125bd71b534d8afa2a2ae2a49f05753bd929dc91fb4de50387
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/lotusnprod/lotus-processor
  tools:
  - MolVS
  - RDKit
  - sanitizing.py
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# dimeric-structure-filtering

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Discard molecular structures that remain dimeric after sanitization by checking for a '.' separator in the SMILES string, as applied in the LOTUS natural products processing pipeline.

## When to use

Use when a cleaned structure table contains entries whose SMILES still has a '.' after RDKit parsing and MolVS sanitization, so dimeric entries must be dropped before standardizing to InChI.

## When NOT to use

- Input is already a cleaned, single-component structure table with no '.' in the SMILES column
- You need to keep covalent multi-component records intact rather than reject them — this filter discards, it does not split or merge components
- You are working with spectra or mass spectrometry peak lists rather than SMILES/structure tables

## Inputs

- Raw structure table with SMILES strings (e.g. smiles.tsv.gz, gzipped TSV)
- Chemical name table (e.g. nominal.tsv.gz) for name-to-structure fallback via OPSIN

## Outputs

- Cleaned structure table interim/tables/2_cleaned/structure/cleaned.tsv.gz with an InChI column and no remaining '.' in SMILES
- Sanitized single-component molecular records ready for InChI standardization

## How to apply

Load the SMILES column of the raw structure table (e.g. smiles.tsv.gz in the LOTUS pipeline) with pandas and parse each string with RDKit, discarding any entry whose SMILES contains a '.' as a first-pass dimer removal. Then run MolVS sanitization in three stages — standardizing, fragment removal, and uncharging — so multi-component records are broken up or neutralized. Re-check the sanitized records: any structure still containing a '.' in its SMILES is a residual dimer and is discarded before InChI generation, since a dimer cannot yield a single valid standard InChI for the natural product. The surviving single-component records are written to the cleaned table (interim/tables/2_cleaned/structure/cleaned.tsv.gz) with their InChI field populated.

## Related tools

- **RDKit** (Parses SMILES strings into molecular objects and flags entries containing '.' (dimeric structures))
- **MolVS** (Sanitizes each molecule via standardizing, fragment removal, and uncharging so multi-component inputs are resolved before the residual dimer check)
- **sanitizing.py** (Script from the lotus-processor pipeline implementing the three-stage sanitization feeding this filter) — https://github.com/lotusnprod/lotus-processor

## Examples

```
python scripts/3_cleaningAndEnriching/sanitizing.py --input smiles.tsv.gz --output interim/tables/2_cleaned/structure/cleaned.tsv.gz
```

## Evaluation signals

- Cleaned.tsv.gz exists in gzipped TSV format with a populated InChI field
- No retained record contains a '.' in its SMILES column (dimer check)
- Record count drops relative to raw input, reflecting dimer and failed-sanitization removals
- Every retained molecule is a valid, sanitized RDKit Mol object (no parse failures or recharged residues)

## Limitations

- The dot-in-SMILES rule is a hard discard: any structurally legitimate multi-component record is lost rather than repaired
- The LOTUS pipeline notes that correct salts, charged or dimeric molecules in the input 'will suffer an unwanted sanitization step', so sanitization can degrade correct inputs
- This filtering operates only at the structure level; it does not validate structure-organism-reference occurrence triples

## Evidence

- [results] Structures with 'dot' in SMILES (remaining dimers) discarded: [evidence span withheld — anchor status: not_found]
