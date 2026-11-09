---
name: chemical-shift-table-extraction
description: Use when a task requires per-atom assigned chemical shifts (residue,
  atom name, shift value) from a BMRB macromolecule or metabolomics entry, or when
  building CSV/NEF tables from NMR-STAR files.
when_to_use_negative:
- Input is a processed/derived table rather than an archival NMR-STAR entry — route
  chemical shift tables through the STARch converter instead of re-parsing NMR-STAR.
- The goal is structure deposition or coordinate validation (e.g. OneDep/ADIT-NMR
  workflow) rather than shift data extraction.
- You need time-domain or processed spectra files themselves rather than the assigned
  shift loop — fetch those from the BMRB FTP metabolomics directory instead.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3366
tools:
- name: BMRB-API
  role: RESTful API used to query and retrieve the chosen BMRB accession in NMR-STAR
    format (macromolecule or metabolomics database)
  repo: https://github.com/uwbmrb/BMRB-API
- name: PyNMR-STAR
  role: Python library that parses and syntax-validates the NMR-STAR file and provides
    access to the chemical shift saveframe/loop
  repo: https://github.com/uwbmrb/PyNMRSTAR
- name: PyBMRB
  role: Python module for visualizing the extracted chemical shift data as 1D/multi-dimensional
    NMR spectra and shift histograms
  repo: https://github.com/uwbmrb/PyBMRB
- name: RBMRB
  role: R-language library alternative for accessing and manipulating BMRB chemical
    shift data
  repo: https://github.com/uwbmrb/RBMRB
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_002
  source_papers:
  - doi: 10.1007/978-1-0716-0270-6_14
    title: BioMagResBank (BMRB) as a Resource for Structural Biology
  source_content_hash: sha256:972113ab8011f229255aec3254613fae7df4c432e23d266ec5e217e8c4438392
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/uwbmrb/BMRB-API
  tools:
  - BMRB-API
  - PyBMRB
  - PyNMR-STAR
  - RBMRB
  - STARch
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# chemical-shift-table-extraction

## Summary

Programmatically retrieve a BMRB entry in NMR-STAR format via the BMRB REST API, parse and validate it with PyNMR-STAR, and extract the assigned chemical shift loop (Atom_chem_shift records) into a comma-delimited shift table.

## When to use

Use when a task requires per-atom assigned chemical shifts (residue, atom name, shift value) from a BMRB macromolecule or metabolomics entry, or when building CSV/NEF tables from NMR-STAR files.

## When NOT to use

- Input is a processed/derived table rather than an archival NMR-STAR entry — route chemical shift tables through the STARch converter instead of re-parsing NMR-STAR.
- The goal is structure deposition or coordinate validation (e.g. OneDep/ADIT-NMR workflow) rather than shift data extraction.
- You need time-domain or processed spectra files themselves rather than the assigned shift loop — fetch those from the BMRB FTP metabolomics directory instead.

## Inputs

- BMRB entry accession (e.g. BMRB entry 17074) or metabolomics entry ID
- NMR-STAR format entry file retrieved from the BMRB API or BMRB FTP server (http://www.bmrb.wisc.edu/ftp/pub/bmrb/)

## Outputs

- chemical_shifts.csv — comma-delimited table with one row per assigned chemical shift and required columns (residue, atom name, shift value)
- metabolite_shifts.csv — analogous shift table for a small-molecule metabolomics entry
- Optionally an intermediate NMR-STAR file (e.g. metabolite_entry.str) saved for provenance

## How to apply

Query the BMRB RESTful API (e.g. api.bmrb.io/v2/entry/$entry_id?format=str) for the chosen accession, requesting NMR-STAR format; the API serves the macromolecules, metabolomics, and chemcomps databases. Parse the returned file in memory with PyNMR-STAR, which validates syntax against the NMR-STAR 3.2 schema. Locate the assigned chemical shift loop (Atom_chem_shift saveframe/loop) and extract per-atom records — atom type, residue, atom name, shift value — optionally filtered by residue or atom type; use IUPAC atom nomenclature so names stay consistent for downstream NEF conversion. Write the records to a comma-delimited table (e.g. chemical_shifts.csv). If shifts must be combined with restraints for NEF output, convert via STARch rather than hand-editing, since STARch converts table formats but not atom nomenclature.

## Related tools

- **BMRB-API** (RESTful API used to query and retrieve the chosen BMRB accession in NMR-STAR format (macromolecule or metabolomics database)) — https://github.com/uwbmrb/BMRB-API
- **PyNMR-STAR** (Python library that parses and syntax-validates the NMR-STAR file and provides access to the chemical shift saveframe/loop) — https://github.com/uwbmrb/PyNMRSTAR
- **PyBMRB** (Python module for visualizing the extracted chemical shift data as 1D/multi-dimensional NMR spectra and shift histograms) — https://github.com/uwbmrb/PyBMRB
- **RBMRB** (R-language library alternative for accessing and manipulating BMRB chemical shift data) — https://github.com/uwbmrb/RBMRB

## Examples

```
import pynmrstar; entry = pynmrstar.Entry.from_database(17074); shifts = entry.get_tags(['Atom_chem_shift.Val_ID','Atom_chem_shift.Comp_ID','Atom_chem_shift.Atom_ID','Atom_chem_shift.Val'])
```

## Evaluation signals

- The PyNMR-STAR parse completes without schema errors against the NMR-STAR 3.2 schema.
- chemical_shifts.csv exists, is comma-delimited, and has the required columns (residue, atom name, shift value).
- Row count of the CSV equals the number of assigned chemical shifts in the entry's Atom_chem_shift loop.
- Residue and atom names in the output match the source NMR-STAR chemical shift table verbatim (IUPAC nomenclature preserved), a requirement for downstream NEF/OneDep acceptance.
- If a plain tab-delimited intermediate was needed, it was produced via STARch conversion, not hand-editing.

## Limitations

- A few (~25) older-format BMRB entries are not included in current API query results.
- The API rate-limits at up to 50 queries per second per IP; exceeding it yields HTTP 403 and requires waiting ~10 seconds.
- Applications distributed to others should send an 'Application' HTTP header naming the app and version.
- STARch converts table formats but not atom nomenclature; mismatched nomenclature tables are rejected at deposition.

## Evidence

- [other] Query the BMRB RESTful API for the chosen accession (e.g., BMRB entry 17074), requesting the entry in NMR-STAR format as supported by the API's query types.: [evidence span withheld — anchor status: not_found]
- [other] Parse the returned NMR-STAR file in memory with PyNMR-STAR, which ensures syntactic validity against the NMR-STAR schema.: [evidence span withheld — anchor status: not_found]
- [other] Locate the assigned chemical shift loop and extract per-atom records.: [evidence span withheld — anchor status: not_found]
- [readme] A Python module for reading, writing, and manipulating NMR-STAR files.: [evidence span withheld — anchor status: external_doc]
- [methods] The STARch file converter can take chemical shift tables in several formats.: "The STARch file converter (http://www.bmrb.wisc.edu/software/starch) can take chemical shift tables in several formats"
- [other] BMRB offers an API and Python/R libraries for NMR-STAR data access.: [evidence span withheld — anchor status: not_found]
