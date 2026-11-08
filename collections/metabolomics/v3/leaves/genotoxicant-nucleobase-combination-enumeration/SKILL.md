---
name: genotoxicant-nucleobase-combination-enumeration
description: Use when expanding a DNA adductomics database beyond literature-reported
  adducts with suspected adducts from all chemically plausible genotoxicant–nucleobase
  combinations, needing charged monoisotopic masses [M+H]+/[M]+ and [M+H-dR]+/[M-dR]+,
  structures, and SMILES/InChI identifiers.
when_to_use_negative:
- When the task is to curate only literature-reported adducts (the harmonization step
  of the paper), not to expand the database with suspected combinations
- When computing in silico MS/MS fragments or building a spectral library — use the
  separate spectral-library step with CFM-ID instead
edam_operation: http://edamontology.org/operation_2406
edam_topics:
- http://edamontology.org/topic_0199
tools:
- name: ChemDraw
  role: Draw each suspected adduct structure and molecular formula; compute charged
    monoisotopic masses [M+H]+/[M]+ and [M+H-dR]+/[M-dR]+
- name: ChemScript
  role: Convert .cdxml files to .mol files and IUPAC nomenclature
- name: Open Babel 3.1.1
  role: Convert .mol files to SMILES, InChI, and InChIKey
- name: liquid chromatography (LC)
  role: Contextual analytical workflow (LC-HRMS) the resulting database supports;
    not directly used during enumeration
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3389/fchem.2022.908572
    title: A Comprehensive Database for DNA Adductomics
  source_content_hash: sha256:b2de9007990ed44dfdf9513780464a58d693065735168a2eab653c00c244e287
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemDraw
  - ChemScript
  - Open Babel 3.1.1
  - liquid chromatography (LC)
  edam_operation: http://edamontology.org/operation_2406
  edam_topics:
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# genotoxicant-nucleobase-combination-enumeration

## Summary

Enumerate chemically plausible theoretical genotoxicant–nucleobase combinations and convert each to a charged monoisotopic-mass adduct record, expanding a curated DNA adduct database (279 literature entries → 582 entries with 303 suspected adducts).

## When to use

Use when expanding a DNA adductomics database beyond literature-reported adducts with suspected adducts from all chemically plausible genotoxicant–nucleobase combinations, needing charged monoisotopic masses [M+H]+/[M]+ and [M+H-dR]+/[M-dR]+, structures, and SMILES/InChI identifiers.

## When NOT to use

- When the task is to curate only literature-reported adducts (the harmonization step of the paper), not to expand the database with suspected combinations
- When computing in silico MS/MS fragments or building a spectral library — use the separate spectral-library step with CFM-ID instead

## Inputs

- Curated table of literature-reported DNA adducts (279 entries)
- List of genotoxicants and the four nucleobases
- ChemDraw .cdxml structure files
- Open Babel 3.1.1 for SMILES/InChI/InChIKey conversion
- ChemScript for .mol/IUPAC conversion

## Outputs

- 303 suspected DNA adduct entries with structures, molecular formulas, and monoisotopic masses
- 582-entry database (.xlsx) merging literature-reported and suspected adducts
- Per-adduct identifier set: .cdxml, .mol, SMILES, InChI, InChIKey, IUPAC name

## How to apply

Start from the curated 279 literature-reported DNA adducts, then enumerate all chemically plausible theoretical combinations of potential genotoxicants with the four nucleobases (dT adducts fewer, since e.g. etheno-dT cannot form without an amino group). Draw each suspected adduct structure and molecular formula in ChemDraw v.19.1.0.8, leaving modification positions unspecified in the name and omitting literature references. For every DNA adduct calculate the charged monoisotopic masses [M+H]+ or [M]+ and [M+H-dR]+ or [M-dR]+, accounting for the positively charged form used for modifications on N3/N7 of dG, N1/N3/N7 of dA, and N3 of dC. Convert .cdxml files to .mol files and IUPAC names with ChemScript, then to SMILES, InChI, and InChIKey with Open Babel 3.1.1, and merge the 303 suspected entries with the 279 curated ones in an .xlsx file including hyperlinks to the .cdxml, .mol files and references.

## Related tools

- **ChemDraw** (Draw each suspected adduct structure and molecular formula; compute charged monoisotopic masses [M+H]+/[M]+ and [M+H-dR]+/[M-dR]+)
- **ChemScript** (Convert .cdxml files to .mol files and IUPAC nomenclature)
- **Open Babel 3.1.1** (Convert .mol files to SMILES, InChI, and InChIKey)
- **liquid chromatography (LC)** (Contextual analytical workflow (LC-HRMS) the resulting database supports; not directly used during enumeration)

## Examples

```
Draw suspected dG/dA/dC/dT genotoxicant adducts in ChemDraw v.19.1.0.8; export .cdxml; ChemScript → .mol + IUPAC; Open Babel 3.1.1 → SMILES/InChI/InChIKey; compute [M+H]+ and [M-dR]+ monoisotopic masses; merge 303 suspected + 279 literature entries into .xlsx with hyperlinks.
```

## Evaluation signals

- Final database contains exactly 582 entries: 279 literature-reported plus 303 suspected
- Suspected entries are distinguishable by having no linked reference in the .xlsx
- Every entry carries both [M+H]+/[M]+ and [M+H-dR]+/[M-dR]+ monoisotopic masses
- Positively charged forms correctly assigned for modifications on N3/N7 of dG, N1/N3/N7 of dA, and N3 of dC
- Each suspected adduct yields .cdxml, .mol, SMILES, InChI, and InChIKey via ChemScript and Open Babel 3.1.1

## Limitations

- dT adducts are fewer because etheno-dT and similar structures cannot form without an amino group
- Modification positions are left unspecified in suspected-adduct names, limiting positional specificity
- Suspected adducts lack literature references and require downstream MS/MS confirmation

## Evidence

- [full_text] Implementation added suspected DNA adducts from all theoretical combinations of genotoxicants with the four nucleobases, with [M+H]+/[M]+ and [M+H-dR]+/[M-dR]+ monoisotopic masses calculated, producing a database of 582 entries: [evidence span withheld — anchor status: not_found]
- [abstract] Implementing the database with 303 new potential DNA adducts: "implementing the database with 303 new potential DNA adducts coming from different combinations of genotoxicants with nucleobases"
- [abstract] Four-step approach including literature search, harmonization, implementation, spectral library: "we present a four-step approach used to compile and curate a database for the annotation of DNA adducts in biological samples"
- [abstract] Harmonization of literature structures before expansion: "The second step consisted in harmonizing structures, molecular formulas, and names"
