---
name: chemical-structure-identifier-extraction
description: Use when building or extending a small-molecule/chemical database where
  each entry needs machine-readable structure identifiers (.mol, .cdxml, SMILES, InChI,
  InChIKey) before spectral prediction or search.
when_to_use_negative:
- The input is already a normalized identifier table (e.g. an SDF or SMILES list)
  and you only need spectral prediction downstream.
- You are annotating spectra without any new chemical structure — this skill assumes
  you have or can generate structure files for each adduct.
- Your target molecules lack defined chemical structures (e.g. you only have m/z peaks
  with no candidate formula).
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3335
- http://edamontology.org/topic_0154
tools:
- name: ChemDraw
  role: Source of monoisotopic masses, chemical formulas, .cdxml files, .mol files,
    SMILES, InChI, InChIKey, and IUPAC names for each adduct entry
- name: CFM-ID
  role: Downstream consumer of the extracted structure identifiers, used to generate
    in silico MS/MS fragments for all 582 adducts
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3389/fchem.2022.908572
    title: A Comprehensive Database for DNA Adductomics
  source_content_hash: sha256:b8bfc4603ca4ceacdaa35068f35df36febbb6e5965e6f70893e409d230acc052
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CFM-ID
  - ChemDraw
  - DT
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3335
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# Chemical structure identifier extraction

## Summary

Extracts computer-readable chemical structure identifiers (SMILES, InChI, InChIKey, IUPAC names) plus structure files (.cdxml, .mol) and monoisotopic masses from a curated DNA adduct database, so each entry can be fed to downstream in silico fragmentation tools like CFM-ID.

## When to use

Use when building or extending a small-molecule/chemical database where each entry needs machine-readable structure identifiers (.mol, .cdxml, SMILES, InChI, InChIKey) before spectral prediction or search.

## When NOT to use

- The input is already a normalized identifier table (e.g. an SDF or SMILES list) and you only need spectral prediction downstream.
- You are annotating spectra without any new chemical structure — this skill assumes you have or can generate structure files for each adduct.
- Your target molecules lack defined chemical structures (e.g. you only have m/z peaks with no candidate formula).

## Inputs

- Curated list of DNA adduct entries (279 literature-reported + 303 newly proposed) with harmonized names, formulas, and structures
- ChemDraw structure files (.cdxml) per adduct
- Molfile (.mol) per adduct

## Outputs

- Per-entry structure identifier set: SMILES, InChI, InChIKey, IUPAC nomenclature
- Per-entry monoisotopic mass and chemical formula
- Keyed table linking each adduct's identifiers to its experimental and predicted MS/MS fragments

## How to apply

First collect the canonical chemical structures for every database entry — for this DNA adduct database the 279 literature-reported and 303 newly proposed adducts — using ChemDraw to obtain monoisotopic masses, chemical formulas, .cdxml files, and .mol files. Then derive or extract the standardized identifiers for each structure: SMILES, InChI, InChIKey, and IUPAC nomenclature, ensuring every entry carries all of these fields so that downstream consumers (spectral search engines, CFM-ID) can unambiguously address it. The extraction follows the paper's harmonization step: structures, molecular formulas, and names are made consistent across all 582 entries before the identifiers are written into the database table. Decide, during extraction, whether an entry came from an unequivocally identified literature source or a tentative/newly proposed combination of genotoxicant with nucleobase, since this affects whether experimental MS/MS exists for it. Finally, assemble the identifier columns alongside the fragment/energy-level data (three energy levels, fragments above 5% intensity) so that structure-level and spectrum-level fields are keyed to the same adduct entry.

## Related tools

- **ChemDraw** (Source of monoisotopic masses, chemical formulas, .cdxml files, .mol files, SMILES, InChI, InChIKey, and IUPAC names for each adduct entry)
- **CFM-ID** (Downstream consumer of the extracted structure identifiers, used to generate in silico MS/MS fragments for all 582 adducts)

## Examples

```
R: library(DT); datatable(adduct_table[, c("SMILES","InChI","InChIKey","monoisotopic_mass")]) — renders the extracted identifier columns in an interactive web table, per the paper's interactive web datatables.
```

## Evaluation signals

- Every one of the 582 entries has a non-empty SMILES, InChI, and InChIKey field (no nulls or placeholder strings).
- Monoisotopic masses in the table agree with the chemical formula to within typical HRMS tolerance (the paper reports high-resolution MS mass accuracy as the operating regime).
- The number of entries with identifiers equals the count reported by the paper: 582 total (279 literature + 303 newly proposed).
- Cross-checking a handful of entries against the source .mol/.cdxml files yields consistent InChIKeys, confirming no structure-file mismapping.
- InChIKeys are all 27-character hashes (standard InChI format), not truncated or hand-written variants.

## Limitations

- Extraction is only as good as the underlying literature curation: adducts that were only 'tentatively identified' in the source papers may carry less trustworthy structures.
- The database is DNA-adduct specific — nucleobase + genotoxicant combinations — so applying the same identifier extraction to unrelated chemical classes requires a new literature curation pass.
- Only 15 reference standards had experimental MS/MS; for the remaining entries the identifiers support predicted fragments, not confirmed spectra.
- No specific retention-time matching or chromatographic separation details are given in the abstract, so identifier extraction alone does not enable RT-based disambiguation of isomers.

## Evidence

- [abstract] Structure files and identifiers produced via ChemDraw for every entry: "monoisotopic masses, chemical formulas, .cdxml files, .mol files, SMILES, InChI, InChIKey and IUPAC nomenclature"
- [abstract] Database size broken into literature and newly proposed adducts: "implementing the database with 303 new potential DNA adducts coming from different combinations of genotoxicants with nucleobases"
- [abstract] Harmonization step ensures consistent names/formulas/structures before identifiers are written: "The second step consisted in harmonizing structures, molecular formulas, and names"
- [readme] DT README confirms the R datatable() interface used to serve the resulting identifier/fragment tables: [evidence span withheld — anchor status: external_doc]
- [abstract] Downstream use of the identifiers for predicted fragmentation: "generating in silico MS/MS fragments for all the adducts"
