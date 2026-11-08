---
name: chemical-structure-format-conversion
description: Use when editable ChemDraw .cdxml structure records (e.g., DNA adducts
  drawn with unspecified modification positions) must be converted to .mol files and
  standard identifiers (SMILES, InChI, InChIKey) for a chemical database.
when_to_use_negative:
- Input identifiers are already SMILES/InChI and only need merging or harmonization
  of names and formulas — the conversion step is redundant.
- The objects are spectra (e.g., mzML HRMS/MS files from the LC-HRMS workflow) rather
  than chemical structure records — use spectral processing, not structure-format
  conversion.
tools:
- name: ChemDraw
  role: Draws each adduct structure and molecular formula as .cdxml files (v.19.1.0.8)
- name: ChemScript
  role: Batch-converts .cdxml structure files to .mol files and generates IUPAC nomenclature
- name: Open Babel
  role: Converts the .mol files to SMILES, InChI, and InChIKey (v.3.1.1)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3389/fchem.2022.908572
    title: A Comprehensive Database for DNA Adductomics
  source_content_hash: sha256:56afbfe5db19c85108a935db9c50d8df4e36d13d72beba710337ee8514663efe
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemDraw
  - ChemScript
  - Open Babel
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# chemical-structure-format-conversion

## Summary

Convert ChemDraw-drawn chemical structures (.cdxml) into interoperable computational identifiers (.mol files, IUPAC nomenclature, SMILES, InChI, InChIKey) so a small-molecule database — here, 582 DNA adduct entries — can be searched, linked, and merged into .xlsx tables.

## When to use

Use when editable ChemDraw .cdxml structure records (e.g., DNA adducts drawn with unspecified modification positions) must be converted to .mol files and standard identifiers (SMILES, InChI, InChIKey) for a chemical database.

## When NOT to use

- Input identifiers are already SMILES/InChI and only need merging or harmonization of names and formulas — the conversion step is redundant.
- The objects are spectra (e.g., mzML HRMS/MS files from the LC-HRMS workflow) rather than chemical structure records — use spectral processing, not structure-format conversion.

## Inputs

- ChemDraw v.19.1.0.8 structure files (.cdxml) with molecular formulas
- Structures drawn for suspected adducts with unspecified modification positions in the name
- Charged monoisotopic mass annotations ([M + H]+ / [M]+ and [M + H-dR]+ / [M-dR]+)

## Outputs

- .mol structure files per adduct
- IUPAC nomenclature strings
- SMILES, InChI, and InChIKey identifiers per entry
- .xlsx database rows (582 entries) with hyperlinks to .cdxml and .mol files

## How to apply

Draw each structure and molecular formula in ChemDraw v.19.1.0.8 (here, 303 suspected adducts from genotoxicant × nucleobase combinations, with modification positions left unspecified in the names and charged monoisotopic masses calculated as [M + H]+ or [M]+ and [M + H-dR]+ or [M-dR]+ for modifications on N3/N7 of dG, N1/N3/N7 of dA, and N3 of dC). Convert the .cdxml structure files to .mol files and IUPAC nomenclature using ChemScript. Then convert the .mol files to SMILES, InChI, and InChIKey using Open Babel 3.1.1. Finally, attach the identifiers to the 279 literature-reported plus 303 suspected entries in an .xlsx file with hyperlinks to the .cdxml and .mol files, yielding the 582-entry database released at the gitlab.com/nexs-metabolomics/projects/dna_adductomics_database repository.

## Related tools

- **ChemDraw** (Draws each adduct structure and molecular formula as .cdxml files (v.19.1.0.8))
- **ChemScript** (Batch-converts .cdxml structure files to .mol files and generates IUPAC nomenclature)
- **Open Babel** (Converts the .mol files to SMILES, InChI, and InChIKey (v.3.1.1))

## Examples

```
obabel -imol adduct.mol -osmi -O adduct.smi; obabel -ismi adduct.smi -oinchi -O adduct.inchi --append InChIKey
```

## Evaluation signals

- Every one of the 582 entries carries a non-empty SMILES, InChI, and InChIKey plus a linked .cdxml and .mol file
- Formula/monoisotopic mass in the entry matches the structure in the converted .mol file, including the charged form ([M + H]+ or [M]+ and [M + H-dR]+ or [M-dR]+) used for the specified protonation/nucleobase positions
- Suspected (non-literature) entries remain distinguishable by having no linked reference in the merged .xlsx
- Round-trip conversion check: canonicalizing the SMILES regenerated from the .mol file yields the same InChIKey
- Final merged table contains exactly 279 + 303 = 582 entries

## Limitations

- Modification positions are left unspecified in the suspected-adduct names, so the identifiers represent positional isomers ambiguously — regioisomers cannot be distinguished by formula/mass alone.
- Conversion inherits any drawing errors made in ChemDraw; the [M+H]+/[M]+ charged forms depend on correct per-position choice (N3/N7 of dG, N1/N3/N7 of dA, N3 of dC).
- Specified tool versions (ChemDraw v.19.1.0.8, ChemScript, Open Babel 3.1.1) may produce slightly different canonical SMILES/InChIKey layer representations across versions.

## Evidence

- [methods] Convert .cdxml to .mol and identifiers via ChemScript and Open Babel 3.1.1: [evidence span withheld — anchor status: not_found]
- [methods] Structures drawn in ChemDraw with unspecified modification positions: [evidence span withheld — anchor status: not_found]
- [abstract] Abstract lists the identifier set generated for each adduct: "monoisotopic masses, chemical formulas, .cdxml files, .mol files, SMILES, InChI, InChIKey and IUPAC nomenclature"
