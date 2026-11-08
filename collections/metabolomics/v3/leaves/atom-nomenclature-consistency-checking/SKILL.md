---
name: atom-nomenclature-consistency-checking
description: Use when converting or curating BMRB NMR-STAR entries (chemical shifts,
  NOE/dihedral restraints) toward NEF or OneDep submission and residue/atom names
  must stay consistent across files.
when_to_use_negative:
- The input only needs table format conversion (e.g., plain tab-delimited shifts to
  STAR format) and atom nomenclature is already consistent — STARch alone suffices,
  since it converts table formats but not atom nomenclature
- The task is renaming atoms to ALATIS identifiers for metabolomics reproducibility
  — that is a distinct BMRB naming scheme, not IUPAC consistency checking across shift/coordinate
  files
- The input is not NMR-STAR/chemical shift data (e.g., time-domain FID files or processed
  spectra), which have no atom nomenclature table to compare
tools:
- name: PyNMR-STAR
  role: Parses and syntax-validates NMR-STAR entries against the BMRB NMR-STAR 3.2
    schema so chemical shift loops and restraint saveframes can be read and compared
    for nomenclature consistency
  repo: https://github.com/uwbmrb/PyNMRSTAR
- name: STARch
  role: Converts chemical shift tables in several formats to STAR format; used only
    for table format conversion since it does not change atom nomenclature
  repo: http://www.bmrb.wisc.edu/software/starch
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/978-1-0716-0270-6_14
    title: BioMagResBank (BMRB) as a Resource for Structural Biology
  source_content_hash: sha256:54077bff1832ef2809918f19536832309b7399df33541195f91c3c6031a2cc51
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/uwbmrb/PyNMRSTAR
  tools:
  - PyNMR-STAR
  - STARch
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# atom-nomenclature-consistency-checking

## Summary

Verifies that residue and atom names in BMRB NMR-STAR chemical shift tables, restraint saveframes, and derived NEF/coordinate files follow consistent IUPAC nomenclature, because mismatched nomenclature tables are rejected during NMR data deposition.

## When to use

Use when converting or curating BMRB NMR-STAR entries (chemical shifts, NOE/dihedral restraints) toward NEF or OneDep submission and residue/atom names must stay consistent across files.

## When NOT to use

- The input only needs table format conversion (e.g., plain tab-delimited shifts to STAR format) and atom nomenclature is already consistent — STARch alone suffices, since it converts table formats but not atom nomenclature
- The task is renaming atoms to ALATIS identifiers for metabolomics reproducibility — that is a distinct BMRB naming scheme, not IUPAC consistency checking across shift/coordinate files
- The input is not NMR-STAR/chemical shift data (e.g., time-domain FID files or processed spectra), which have no atom nomenclature table to compare

## Inputs

- Archival BMRB NMR-STAR entry file (e.g., from http://www.bmrb.wisc.edu/ftp/pub/bmrb/)
- Chemical shift loop (_Atom_chem_shift) from the NMR-STAR entry
- Restraint saveframes (NOE distance restraints, dihedral angle restraints)
- Coordinate file for the same entry

## Outputs

- Consistency report comparing atom nomenclature between chemical shift and coordinates files
- Extracted shift and restraint tables with IUPAC-consistent residue/atom names
- Validated converted.nef file whose residue/atom names match the source NMR-STAR entry's chemical shift table

## How to apply

Retrieve the archival NMR-STAR entry from the BMRB FTP server and parse it with PyNMR-STAR, which validates records against the BMRB NMR-STAR 3.2 schema. Extract the chemical shift loop and restraint saveframes (e.g., NOE distance and dihedral angle restraints) using IUPAC atom nomenclature so residue and atom names remain consistent between the shift table, restraints, and any coordinate file. When a format conversion of chemical shift tables is needed, route plain tab-delimited or other table formats through the STARch converter rather than hand-editing, but note that STARch converts table formats only — it does not fix atom nomenclature, so any renaming must be done separately. Finally, compare atom nomenclature across the chemical shift and coordinates files (and the emitted NEF) for consistency before deposition. The decision point is any discrepancy in residue/atom naming between files: these must be reconciled prior to submission because deposition rules reject mismatched nomenclature tables.

## Related tools

- **PyNMR-STAR** (Parses and syntax-validates NMR-STAR entries against the BMRB NMR-STAR 3.2 schema so chemical shift loops and restraint saveframes can be read and compared for nomenclature consistency) — https://github.com/uwbmrb/PyNMRSTAR
- **STARch** (Converts chemical shift tables in several formats to STAR format; used only for table format conversion since it does not change atom nomenclature) — http://www.bmrb.wisc.edu/software/starch

## Examples

```
import pynmrstar; entry = pynmrstar.Entry.from_file('bmr17074.str')
```

## Evaluation signals

- The atom nomenclature in the chemical shift file and the coordinates file match for every residue/atom pair
- The converted.nef file parses without errors and its residue/atom names are identical to the source NMR-STAR entry's chemical shift table
- The source NMR-STAR file parses cleanly under the BMRB NMR-STAR 3.2 schema via PyNMR-STAR
- No hand-edited nomenclature changes were introduced — any format change went through STARch or a scripted IUPAC renaming step
- The entry would not be flagged by the deposition rule that rejects mismatched nomenclature tables

## Limitations

- STARch converts table formats but not atom nomenclature, so it cannot fix naming inconsistencies on its own
- Mismatched nomenclature tables between chemical shift and coordinates files cause rejection during deposition, so failures surface late if the check is skipped
- A few older-format BMRB entries (~25) are excluded from current query results and may need extra handling before validation

## Evidence

- [methods] atom nomenclature consistency check: "the atom nomenclature in the chemical shift and coordinates files is compared for consistency"
- [methods] STARch converts formats not nomenclature: "The STARch file converter (http://www.bmrb.wisc.edu/software/starch) can take chemical shift tables in several formats"
- [readme] PyNMR-STAR purpose: [evidence span withheld — anchor status: external_doc]
- [intro] BMRB as NMR archive: "The Biological Magnetic Resonance Data Bank (BioMagResBank or BMRB) [1] has served for the past 30 years as the primary archive for spectral and derived data"
