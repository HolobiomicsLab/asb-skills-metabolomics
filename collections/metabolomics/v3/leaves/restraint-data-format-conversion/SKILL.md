---
name: restraint-data-format-conversion
description: Use when an archival BMRB NMR-STAR entry must be validated, its chemical
  shift loop and restraint saveframes extracted, and its tables emitted as a NEF file
  (converted.nef) conformant with the NEF specification.
when_to_use_negative:
- The data are already in NEF or a deposition-acceptable NMR-STAR form — no conversion
  step is needed
- Atom nomenclature itself must be corrected or remapped (e.g., to ALATIS identifiers)
  — STARch converts table formats but not atom nomenclature
- The goal is visual/analytical exploration of chemical shift distributions rather
  than format conversion — use PyBMRB/RBMRB visualization tooling instead
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_1317
tools:
- name: PyNMR-STAR
  role: Parses and syntax-validates the NMR-STAR entry against the BMRB NMR-STAR 3.2
    schema; reads, writes, and manipulates the entry before extraction
  repo: https://github.com/uwbmrb/PyNMRSTAR
- name: STARch
  role: Converts chemical shift and restraint tables between tabular formats (but
    not atom nomenclature) as an intermediate step before NEF emission
  repo: http://www.bmrb.wisc.edu/software/starch
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/978-1-0716-0270-6_14
    title: BioMagResBank (BMRB) as a Resource for Structural Biology
  source_content_hash: sha256:552152b2b317d1cee2b7d3d1739a029c1138512c4f198fe93e6d26b41a69f4df
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/uwbmrb/PyNMRSTAR
  tools:
  - PyNMR-STAR
  - STARch
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_1317
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# restraint-data-format-conversion

## Summary

Converts chemical shift and restraint tables (e.g., NOE distance and dihedral angle restraints) out of a BMRB NMR-STAR entry into the NEF interchange format required for wwPDB OneDep deposition, using PyNMR-STAR for schema-valid parsing and STARch for table format conversion while preserving IUPAC atom nomenclature.

## When to use

Use when an archival BMRB NMR-STAR entry must be validated, its chemical shift loop and restraint saveframes extracted, and its tables emitted as a NEF file (converted.nef) conformant with the NEF specification.

## When NOT to use

- The data are already in NEF or a deposition-acceptable NMR-STAR form — no conversion step is needed
- Atom nomenclature itself must be corrected or remapped (e.g., to ALATIS identifiers) — STARch converts table formats but not atom nomenclature
- The goal is visual/analytical exploration of chemical shift distributions rather than format conversion — use PyBMRB/RBMRB visualization tooling instead

## Inputs

- NMR-STAR entry file retrieved from the BMRB FTP server (http://www.bmrb.wisc.edu/ftp/pub/bmrb/)
- Chemical shift loop from the NMR-STAR entry
- Restraint saveframes (e.g., NOE distance restraints, dihedral angle restraints)

## Outputs

- converted.nef — a NEF-format interchange file with well-formed saveframes/loops, valid tags, and atom nomenclature matching the source entry
- Validated NMR-STAR parse results (schema-3.2 conformance report from PyNMR-STAR)
- Optionally, tab-delimited intermediate tables produced via STARch

## How to apply

Retrieve the NMR-STAR entry from the public BMRB FTP server (http://www.bmrb.wisc.edu/ftp/pub/bmrb/) and parse it with the PyNMR-STAR library, which validates records against the BMRB NMR-STAR 3.2 schema. Extract the chemical shift loop and the restraint saveframes (NOE distance and dihedral angle restraints) using IUPAC atom nomenclature so residue and atom names stay consistent with the source entry's chemical shift table. Convert the extracted shift and restraint tables to NEF; if an intermediate tab-delimited table is needed, route it through the STARch converter rather than hand-editing, since STARch converts table formats but not atom nomenclature. Finally, check the emitted NEF for well-formed saveframes/loops, valid tags, and consistent atom nomenclature before writing converted.nef, because depositions with mismatched nomenclature tables are rejected.

## Related tools

- **PyNMR-STAR** (Parses and syntax-validates the NMR-STAR entry against the BMRB NMR-STAR 3.2 schema; reads, writes, and manipulates the entry before extraction) — https://github.com/uwbmrb/PyNMRSTAR
- **STARch** (Converts chemical shift and restraint tables between tabular formats (but not atom nomenclature) as an intermediate step before NEF emission) — http://www.bmrb.wisc.edu/software/starch

## Examples

```
import pynmrstar; entry = pynmrstar.Entry.from_file('bmrb_entry.str'); entry.validate()
```

## Evaluation signals

- converted.nef exists and parses without errors as a valid NEF file
- NEF saveframes and loops are well-formed and tags are valid per the NEF specification
- Residue and atom names in the NEF output match the source NMR-STAR entry's chemical shift table exactly
- Source NMR-STAR file passes PyNMR-STAR validation against the NMR-STAR 3.2 schema
- Atom nomenclature is consistent between the chemical shift data and the coordinate/restraint data, per the deposition rule that mismatched nomenclature tables are rejected

## Limitations

- STARch converts table formats but not atom nomenclature, so nomenclature consistency must be ensured upstream using IUPAC atom names
- A few (~25) older-format BMRB entries are not included in current query grid results and may need special handling
- Mismatched nomenclature tables between shift and coordinate files are grounds for deposition rejection

## Evidence

- [readme] PyNMR-STAR is the BMRB-maintained tool for working with NMR-STAR files: [evidence span withheld — anchor status: external_doc]
- [readme] PyNMR-STAR scope covers reading, writing, and manipulating NMR-STAR files: [evidence span withheld — anchor status: external_doc]
