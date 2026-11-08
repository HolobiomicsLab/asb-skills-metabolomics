---
name: nmr-star-schema-validation
description: Use when input is an archival BMRB NMR-STAR entry file that must be parsed,
  validated against the NMR-STAR 3.2 schema, or converted (e.g., to NEF) before deposition
  or downstream analysis.
when_to_use_negative:
- The data is raw time-domain or processed spectra not represented in NMR-STAR format
  — BMRB accepts these as separate uploaded data files, not via schema-parsed STAR
  entries
- Only the atom nomenclature needs remapping (e.g., to ALATIS identifiers) — STARch
  converts table formats but not atom nomenclature, so a different tool/step is required
- The goal is querying or visualizing existing BMRB chemical shift data rather than
  validating/converting a specific entry — the BMRB REST API or PyBMRB/RBMRB is more
  appropriate
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3452
tools:
- name: PyNMR-STAR
  role: Python library for reading, writing, parsing, and schema-validating NMR-STAR
    files against the BMRB NMR-STAR 3.2 schema
  repo: https://github.com/uwbmrb/PyNMRSTAR
- name: STARch
  role: File converter that takes chemical shift tables in several formats and converts
    them for deposition; used for format conversion only, not nomenclature correction
- name: BMRB-API
  role: REST API alternative for retrieving BMRB macromolecule, metabolomics, and
    chemcomp entries (in NMR-STAR or JSON) without FTP download
  repo: https://github.com/uwbmrb/BMRB-API
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/978-1-0716-0270-6_14
    title: BioMagResBank (BMRB) as a Resource for Structural Biology
  source_content_hash: sha256:e750db5aef20636518deb5acebd8da00791e9314d4cf4c6da9d4eff5933ce58b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/uwbmrb/PyNMRSTAR
  tools:
  - BMRB-API
  - PyNMR-STAR
  - STARch
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# nmr-star-schema-validation

## Summary

Parse and syntax-validate NMR-STAR files (the data format BMRB uses to archive NMR experiments, spectral and derived data, and supporting metadata) against the BMRB NMR-STAR 3.2 schema using the PyNMR-STAR library, as a prerequisite for converting entries to NEF for deposition.

## When to use

Use when input is an archival BMRB NMR-STAR entry file that must be parsed, validated against the NMR-STAR 3.2 schema, or converted (e.g., to NEF) before deposition or downstream analysis.

## When NOT to use

- The data is raw time-domain or processed spectra not represented in NMR-STAR format — BMRB accepts these as separate uploaded data files, not via schema-parsed STAR entries
- Only the atom nomenclature needs remapping (e.g., to ALATIS identifiers) — STARch converts table formats but not atom nomenclature, so a different tool/step is required
- The goal is querying or visualizing existing BMRB chemical shift data rather than validating/converting a specific entry — the BMRB REST API or PyBMRB/RBMRB is more appropriate

## Inputs

- Archival BMRB NMR-STAR entry file (downloaded from http://www.bmrb.wisc.edu/ftp/pub/bmrb/)
- Chemical shift tables in various formats (for STARch conversion)
- Restraint saveframes (NOE distance and dihedral angle restraints) within NMR-STAR entries

## Outputs

- Parsed, schema-validated NMR-STAR entry object (entry/saveframe/loop structure)
- Extracted chemical shift loop and restraint tables with consistent IUPAC atom nomenclature
- Validated NEF-format file (e.g., converted.nef) conforming to the NEF specification

## How to apply

Retrieve the archival NMR-STAR entry from the public BMRB FTP server (http://www.bmrb.wisc.edu/ftp/pub/bmrb/), which hosts entry and derived-data directories. Parse the file with the PyNMR-STAR Python library, which validates records against the BMRB NMR-STAR 3.2 schema; use the library's entry, saveframe, and loop objects to access chemical shift loops and restraint saveframes (NOE distance and dihedral angle restraints). When extracting shifts and restraints, preserve IUPAC atom nomenclature so residue and atom names stay consistent, since depositions with mismatched nomenclature tables are rejected. If converting formats, route plain tab-delimited intermediates through the STARch converter rather than hand-editing — STARch converts table formats but not atom nomenclature. Judge success by whether the file parses without schema errors and any emitted NEF contains well-formed saveframes/loops with valid tags and atom names matching the source chemical shift table.

## Related tools

- **PyNMR-STAR** (Python library for reading, writing, parsing, and schema-validating NMR-STAR files against the BMRB NMR-STAR 3.2 schema) — https://github.com/uwbmrb/PyNMRSTAR
- **STARch** (File converter that takes chemical shift tables in several formats and converts them for deposition; used for format conversion only, not nomenclature correction)
- **BMRB-API** (REST API alternative for retrieving BMRB macromolecule, metabolomics, and chemcomp entries (in NMR-STAR or JSON) without FTP download) — https://github.com/uwbmrb/BMRB-API

## Examples

```
import pynmrstar; entry = pynmrstar.Entry.from_file('bmr_entry.str'); entry.validate(); shifts = entry.get_saveframe('assigned_chemical_shifts')
```

## Evaluation signals

- The NMR-STAR file parses with PyNMR-STAR without schema errors against the NMR-STAR 3.2 schema
- The emitted converted.nef exists and parses without errors as a valid NEF file with well-formed saveframes/loops and valid tags
- Residue and atom names in the NEF output exactly match the source NMR-STAR entry's chemical shift table (per the deposition rule that mismatched nomenclature tables are rejected)
- Chemical shift values pass outlier checking (standard deviation-based statistical filtering, as done by BMRB annotators during data curation)
- Atom nomenclature in chemical shift and coordinates files is consistent (compared as part of BMRB/PDB deposition validation)

## Limitations

- A few (~25) older-format BMRB entries are not included in current query results, so schema-parsing/query tooling may miss them
- STARch converts table formats but not atom nomenclature — nomenclature inconsistencies must be fixed separately before conversion
- Uploading entries to the BMRB API /entry/ endpoint makes them publicly accessible via the assigned entry_id and entries expire after a week of inactivity
- The BMRB API rate-limits to 50 queries per second per IP; exceeding it yields HTTP 403 until a 10+ second pause

## Evidence

- [methods] PyNMR-STAR validates against 3.2 schema: [evidence span withheld — anchor status: not_found]
- [readme] PyNMR-STAR purpose from README: [evidence span withheld — anchor status: external_doc]
