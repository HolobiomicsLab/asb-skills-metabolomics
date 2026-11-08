---
name: nmr-star-file-parsing
description: Use when a task requires reading, validating, or extracting saveframes
  and loops (e.g., the assigned Atom_chem_shift chemical shift loop, NOE/dihedral
  restraint saveframes) from a BMRB entry in NMR-STAR format.
when_to_use_negative:
- The chemical shift table only needs format conversion between table formats — use
  the STARch converter, which converts table formats but not atom nomenclature.
- Input is already NEF or a plain tab-delimited/CSV shift table — there is no NMR-STAR
  to parse.
- Goal is visualization of chemical shift lists as spectra or shift-distribution histograms
  — use PyBMRB, which fetches and parses the data for you.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_1317
- http://edamontology.org/topic_3172
tools:
- name: PyNMR-STAR
  role: Python module used to read, write, syntax-validate, and manipulate NMR-STAR
    files; exposes saveframes/loops (e.g., Atom_chem_shift) for record extraction.
  repo: https://github.com/uwbmrb/PyNMRSTAR
- name: BMRB-API
  role: REST API used to query and retrieve BMRB macromolecule and metabolomics entries
    in NMR-STAR format before parsing.
  repo: https://github.com/uwbmrb/BMRB-API
- name: PyBMRB
  role: Visualization layer that uses PyNMR-STAR to parse NMR-STAR files and BMRB-API
    to fetch chemical shift data for spectral plotting.
  repo: https://github.com/uwbmrb/PyBMRB
- name: RBMRB
  role: R-language library from the BMRB GitHub repository for analyzing, visualizing,
    and manipulating NMR-STAR formatted chemical shift data as an alternative to the
    Python stack.
  repo: https://github.com/uwbmrb/RBMRB
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_001
  source_papers:
  - doi: 10.1007/978-1-0716-0270-6_14
    title: BioMagResBank (BMRB) as a Resource for Structural Biology
  source_content_hash: sha256:19c790dd529c3d7dc7f80d98a960604b7df88f09566ebaacf3c3347279c8fd8e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/uwbmrb/PyNMRSTAR
  tools:
  - BMRB-API
  - NMR-STAR
  - PyBMRB
  - PyNMR-STAR
  - RBMRB
  - STARch
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_1317
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# nmr-star-file-parsing

## Summary

Parses and syntax-validates BMRB entries in the NMR-STAR data format using PyNMR-STAR, exposing saveframes/loops such as the Atom_chem_shift chemical shift loop for downstream extraction, conversion, and analysis. This is the core handling step for any BMRB archival data, since BMRB archives spectral and derived data in NMR-STAR.

## When to use

Use when a task requires reading, validating, or extracting saveframes and loops (e.g., the assigned Atom_chem_shift chemical shift loop, NOE/dihedral restraint saveframes) from a BMRB entry in NMR-STAR format.

## When NOT to use

- The chemical shift table only needs format conversion between table formats — use the STARch converter, which converts table formats but not atom nomenclature.
- Input is already NEF or a plain tab-delimited/CSV shift table — there is no NMR-STAR to parse.
- Goal is visualization of chemical shift lists as spectra or shift-distribution histograms — use PyBMRB, which fetches and parses the data for you.

## Inputs

- BMRB entry in NMR-STAR format retrieved from the BMRB REST API (api.bmrb.io, macromolecule or metabolomics database)
- NMR-STAR .str entry file downloaded from the public BMRB FTP server (http://www.bmrb.wisc.edu/ftp/pub/bmrb/)
- NMR-STAR text or JSON entry uploaded to the API's /entry/ store endpoint (expires after one week)

## Outputs

- Parsed, schema-validated NMR-STAR entry object (saveframes and loops) in memory
- Extracted assigned chemical shift records (atom type, residue, atom name, shift value) as a comma-delimited table, e.g., chemical_shifts.csv
- Extracted chemical shift and restraint (NOE distance, dihedral angle) tables ready for NEF conversion, e.g., converted.nef

## How to apply

Retrieve the entry as NMR-STAR via the versioned BMRB REST API (e.g., api.bmrb.io/v2/entry/$entry_id with an optional format parameter) or download a .str file from the public BMRB FTP, then parse it with PyNMR-STAR, which validates the file against the BMRB NMR-STAR 3.2 schema as it loads. Navigate the parsed entry's saveframes and loops: assigned chemical shifts live in the 'Atom_chem_shift' saveframe/loop with per-atom records carrying atom type, residue, atom name, and shift value, while restraint saveframes hold NOE distance and dihedral angle restraints. Extract per-atom records programmatically — optionally filtered by residue and atom type — keeping IUPAC atom nomenclature so residue and atom names stay consistent with the source entry, since mismatched nomenclature tables are rejected at deposition. Write extracted shift records to a comma-delimited table (e.g., chemical_shifts.csv); if only a table-format conversion is needed, route through the STARch converter rather than hand-editing, because STARch converts table formats but not atom nomenclature. Validate the output: the emitted table or NEF file must parse without errors, and residue/atom names must match the source NMR-STAR chemical shift table. Respect API etiquette: at most 50 queries per second per IP, wait at least 10 seconds after any HTTP 403, and send an 'Application' HTTP header naming your tool and version.

## Related tools

- **PyNMR-STAR** (Python module used to read, write, syntax-validate, and manipulate NMR-STAR files; exposes saveframes/loops (e.g., Atom_chem_shift) for record extraction.) — https://github.com/uwbmrb/PyNMRSTAR
- **BMRB-API** (REST API used to query and retrieve BMRB macromolecule and metabolomics entries in NMR-STAR format before parsing.) — https://github.com/uwbmrb/BMRB-API
- **PyBMRB** (Visualization layer that uses PyNMR-STAR to parse NMR-STAR files and BMRB-API to fetch chemical shift data for spectral plotting.) — https://github.com/uwbmrb/PyBMRB
- **RBMRB** (R-language library from the BMRB GitHub repository for analyzing, visualizing, and manipulating NMR-STAR formatted chemical shift data as an alternative to the Python stack.) — https://github.com/uwbmrb/RBMRB

## Examples

```
import pynmrstar; entry = pynmrstar.Entry.from_file('metabolite_entry.str'); shifts = entry.get_loop_by_category('_Atom_chem_shift')
```

## Evaluation signals

- PyNMR-STAR parses the file without errors, i.e., the entry is syntactically valid against the BMRB NMR-STAR 3.2 schema.
- chemical_shifts.csv exists, is comma-delimited, has one row per assigned chemical shift in the entry, and contains populated residue, atom name, and shift value columns.
- Residue and atom names in the extracted table exactly match the source NMR-STAR entry's chemical shift table (IUPAC nomenclature preserved, no mismatches).
- Downstream NEF output (converted.nef) exists and parses as a valid NEF file with well-formed saveframes/loops, valid tags, and consistent atom nomenclature.
- API-based retrieval returns the requested entry content (not an HTTP 403 rate-limit response) and includes the 'Application' header identifying the calling tool.

## Limitations

- BMRB API rate limiting: up to 50 queries per second per IP; exceeding the limit returns HTTP 403 for all requests and requires waiting at least 10 seconds before any further query.
- Currently a few BMRB entries (~25) in an older format are not included in the current queries.
- Entries uploaded via the API's /entry/ store endpoint are publicly accessible to anyone with the assigned entry_id and expire one week after the last request using that entry_id.
- Parsing and STARch conversion do not reconcile atom nomenclature — STARch converts table formats but not atom nomenclature, so nomenclature mismatches must be fixed upstream.

## Evidence

- [readme] PyNMRSTAR overview: [evidence span withheld — anchor status: external_doc]
- [readme] BMRB-API purpose: [evidence span withheld — anchor status: external_doc]
- [readme] API rate limit: [evidence span withheld — anchor status: external_doc]
- [methods] PyNMRSTAR documentation: "There is extensive documentation and an introduction to working with the NMR-STAR format at the GitHub page for the project: https://github.com/uwbmrb/PyNMRSTAR."
