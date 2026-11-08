---
name: nef-format-conversion
description: Use when an archival BMRB NMR-STAR entry must be syntax-validated and
  its chemical shift loop and restraint saveframes (NOE distances, dihedral angles)
  converted to the NEF interchange format for OneDep/wwPDB deposition with IUPAC atom
  nomenclature preserved.
when_to_use_negative:
- The shift and restraint data are already in NEF or NMR-STAR format accepted by OneDep
  — no format conversion is needed.
- 'The actual problem is atom-nomenclature remapping itself: STARch converts table
  formats but not atom nomenclature, so a format converter will not fix inconsistent
  names.'
- The goal is visualizing or exploring chemical shift distributions rather than producing
  a depositable NEF file (analysis/visualization libraries such as PyBMRB are the
  better fit).
edam_topics:
- http://edamontology.org/topic_1317
tools:
- name: PyNMR-STAR
  role: Python library used to parse and syntax-validate the BMRB NMR-STAR entry against
    the NMR-STAR 3.2 schema and to extract the chemical shift loop and restraint saveframes
  repo: https://github.com/uwbmrb/PyNMRSTAR
- name: STARch
  role: BMRB file converter applied when a plain tab-delimited chemical shift table
    intermediate is needed; converts table formats but not atom nomenclature
  repo: http://www.bmrb.wisc.edu/software/starch
- name: OneDep
  role: wwPDB deposition system whose requirement that restraints be provided in NEF
    or NMR-STAR format defines the target output of the conversion
  repo: https://deposit.wwpdb.org/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/978-1-0716-0270-6_14
    title: BioMagResBank (BMRB) as a Resource for Structural Biology
  source_content_hash: sha256:a6591bfade20db4866cf6ab6d44b2ccfcf8b8073d8cfa6e58297ebf82613631c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/uwbmrb/PyNMRSTAR
  tools:
  - OneDep
  - PyNMR-STAR
  - STARch
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_1317
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# nef-format-conversion

## Summary

Validates archival BMRB NMR-STAR entries with PyNMR-STAR against the BMRB NMR-STAR 3.2 schema, then converts their chemical shift loop and restraint saveframes (NOE distance, dihedral angle) into the NEF interchange format that OneDep accepts. Rationale: deposition requires restraints in NEF or NMR-STAR format, and mismatched atom nomenclature tables are rejected.

## When to use

Use when an archival BMRB NMR-STAR entry must be syntax-validated and its chemical shift loop and restraint saveframes (NOE distances, dihedral angles) converted to the NEF interchange format for OneDep/wwPDB deposition with IUPAC atom nomenclature preserved.

## When NOT to use

- The shift and restraint data are already in NEF or NMR-STAR format accepted by OneDep — no format conversion is needed.
- The actual problem is atom-nomenclature remapping itself: STARch converts table formats but not atom nomenclature, so a format converter will not fix inconsistent names.
- The goal is visualizing or exploring chemical shift distributions rather than producing a depositable NEF file (analysis/visualization libraries such as PyBMRB are the better fit).

## Inputs

- Archival BMRB NMR-STAR entry file (NMR-STAR 3.2 format) retrieved from the public BMRB FTP server (http://www.bmrb.wisc.edu/ftp/pub/bmrb/)
- Chemical shift loop from the NMR-STAR entry
- Restraint saveframes (e.g., NOE distance restraints and dihedral angle restraints)

## Outputs

- converted.nef — a NEF interchange file with well-formed saveframes/loops, valid tags, and residue/atom names matching the source NMR-STAR entry's chemical shift table

## How to apply

Retrieve the archival NMR-STAR entry from the public BMRB FTP server (http://www.bmrb.wisc.edu/ftp/pub/bmrb/), which hosts derived-data and entry directories in NMR-STAR format. Parse and syntax-validate the file with the PyNMR-STAR Python library, which checks records against the BMRB NMR-STAR 3.2 schema, before attempting any conversion. Extract the chemical shift loop and the restraint saveframes (e.g., NOE distance and dihedral angle restraints) using IUPAC atom nomenclature so residue and atom names remain consistent with the source entry. Convert the extracted shift and restraint tables into the NEF interchange format, mirroring the OneDep requirement that restraints be provided in NEF or NMR-STAR format; if a plain tab-delimited intermediate is needed, route it through the STARch converter rather than hand-editing, since STARch converts table formats but not atom nomenclature. Finally, check the emitted file for NEF conformance (well-formed saveframes/loops, valid tags, consistent atom nomenclature) and write it as converted.nef, which must parse without errors and preserve residue and atom names matching the source entry's chemical shift table.

## Related tools

- **PyNMR-STAR** (Python library used to parse and syntax-validate the BMRB NMR-STAR entry against the NMR-STAR 3.2 schema and to extract the chemical shift loop and restraint saveframes) — https://github.com/uwbmrb/PyNMRSTAR
- **STARch** (BMRB file converter applied when a plain tab-delimited chemical shift table intermediate is needed; converts table formats but not atom nomenclature) — http://www.bmrb.wisc.edu/software/starch
- **OneDep** (wwPDB deposition system whose requirement that restraints be provided in NEF or NMR-STAR format defines the target output of the conversion) — https://deposit.wwpdb.org/

## Examples

```
python -c "import pynmrstar; entry = pynmrstar.Entry.from_file('bmr10061.str'); entry.validate()"
```

## Evaluation signals

- converted.nef exists and parses without errors as a valid NEF file.
- Residue and atom names in converted.nef match the source NMR-STAR entry's chemical shift table (mismatched nomenclature tables are rejected at deposition).
- The source NMR-STAR file passed PyNMR-STAR validation against the BMRB NMR-STAR 3.2 schema before conversion was attempted.
- The emitted NEF contains well-formed saveframes/loops and valid tags per the NEF specification.
- All extracted restraint saveframes (NOE distance, dihedral angle) and the chemical shift loop are represented in the NEF output with no silent drops.

## Limitations

- STARch converts table formats but not atom nomenclature, so IUPAC residue/atom naming must be resolved outside the converter.
- Deposition rules reject entries whose chemical shift and coordinates files use inconsistent nomenclature tables, so any renaming must be applied consistently across files.
- A few (~25) BMRB entries in an older format are not included in current queries, so legacy entries may only be reachable through the FTP archive.

## Evidence

- [intro] The Biological Magnetic Resonance Data Bank (BioMagResBank or BMRB) [1] has served for the past 30 years as the primary archive for spectral and derived data: "The Biological Magnetic Resonance Data Bank (BioMagResBank or BMRB) [1] has served for the past 30 years as the primary archive for spectral and derived data"
- [readme] A Python module for reading, writing, and manipulating NMR-STAR files.: [evidence span withheld — anchor status: external_doc]
- [methods] Parse and syntax-validate the NMR-STAR file with the PyNMR-STAR Python library, which validates records against the BMRB NMR-STAR 3.2 schema.: [evidence span withheld — anchor status: not_found]
- [methods] The STARch file converter (http://www.bmrb.wisc.edu/software/starch) can take chemical shift tables in several formats: "The STARch file converter (http://www.bmrb.wisc.edu/software/starch) can take chemical shift tables in several formats"
- [methods] Convert the extracted shift and restraint tables into the NEF interchange format, mirroring the OneDep requirement that restraints be provided in NEF or NMR-STAR format: [evidence span withheld — anchor status: not_found]
- [methods] since STARch converts table formats but not atom nomenclature: [evidence span withheld — anchor status: not_found]
- [methods] the converted.nef file must exist, parse without errors as a valid NEF file, and preserve residue and atom names matching the source NMR-STAR entry's chemical shift table: [evidence span withheld — anchor status: not_found]
- [methods] the atom nomenclature in the chemical shift and coordinates files is compared for consistency: "the atom nomenclature in the chemical shift and coordinates files is compared for consistency"
