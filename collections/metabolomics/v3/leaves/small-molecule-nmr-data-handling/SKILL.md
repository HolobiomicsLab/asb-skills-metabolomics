---
name: small-molecule-nmr-data-handling
description: Use when the task requires retrieving a metabolite or biological small-molecule
  NMR entry (spectra, time-domain data, or assigned chemical shifts) from BMRB and
  converting its NMR-STAR record into a tabular shift table.
when_to_use_negative:
- The target entry is a macromolecule (protein, nucleic acid, carbohydrate) — query
  the default 'macromolecules' database instead of 'metabolomics'.
- The goal is depositing new NMR data rather than retrieving existing data — use BMRBdep/ADIT-NMR
  deposition workflow instead.
- The input is already a parsed chemical shift table (e.g. a CSV) — parsing with PyNMR-STAR
  would be redundant.
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3314
tools:
- name: BMRB-API
  role: REST API used to query the metabolomics database and retrieve small-molecule
    entries in NMR-STAR or JSON format
  repo: https://github.com/uwbmrb/BMRB-API
- name: PyNMR-STAR
  role: Python library used to parse the downloaded NMR-STAR file and extract the
    assigned chemical shift loop
  repo: https://github.com/uwbmrb/PyNMRSTAR
- name: PyBMRB
  role: Optional Python module to visualize the extracted 1D chemical shift lists
    as multi-dimensional NMR spectra
  repo: https://github.com/uwbmrb/PyBMRB
- name: BMRB metabolomics database
  role: Source archive of 1H and 13C 1D/2D NMR spectra and assigned chemical shifts
    for over 1000 biological small molecules
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/978-1-0716-0270-6_14
    title: BioMagResBank (BMRB) as a Resource for Structural Biology
  source_content_hash: sha256:82c0d132a2bebd5a271fd86b111fd2467a4fd8d72cf890b5b1a34526d098a844
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/uwbmrb/BMRB-API
  tools:
  - BMRB metabolomics database
  - BMRB-API
  - PyBMRB
  - PyNMR-STAR
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# small-molecule-nmr-data-handling

## Summary

Retrieves and parses small-molecule (metabolomics) NMR entries from BMRB — 1H/13C 1D and 2D spectra and assigned chemical shifts for over 1000 biological small molecules — using the BMRB REST API and the PyNMR-STAR parser to produce machine-readable shift tables.

## When to use

Use when the task requires retrieving a metabolite or biological small-molecule NMR entry (spectra, time-domain data, or assigned chemical shifts) from BMRB and converting its NMR-STAR record into a tabular shift table.

## When NOT to use

- The target entry is a macromolecule (protein, nucleic acid, carbohydrate) — query the default 'macromolecules' database instead of 'metabolomics'.
- The goal is depositing new NMR data rather than retrieving existing data — use BMRBdep/ADIT-NMR deposition workflow instead.
- The input is already a parsed chemical shift table (e.g. a CSV) — parsing with PyNMR-STAR would be redundant.

## Inputs

- BMRB metabolomics (small molecule) entry identifier or compound name
- BMRB REST API endpoint (api.bmrb.io, e.g. /v2/ or /current/)
- NMR-STAR format entry file (metabolite_entry.str), optionally including time-domain or peak-list data referenced from the BMRB FTP directory

## Outputs

- NMR-STAR entry record for the metabolite (metabolite_entry.str)
- Comma-delimited chemical shift table (metabolite_shifts.csv) with atom-identifier and shift-value columns
- Optionally, 1D/2D spectra or peak lists retrieved from the entry

## How to apply

Query the BMRB RESTful API (api.bmrb.io) against the 'metabolomics' database — not the default 'macromolecules' database — e.g. via /list_entries?database=metabolomics to enumerate small-molecule entries. Request the full entry record in NMR-STAR format (GET /entry/$entry_id with entry_format set, or fetch from the BMRB FTP server at http://www.bmrb.wisc.edu/ftp/pub/bmrb/) and save it as a .str file. Parse the NMR-STAR file with the PyNMR-STAR Python library to locate the assigned chemical shift loop for the compound's atoms, extracting atom/residue identifiers (with ALATIS atom naming where applicable) and their shift values. Write the results to a comma-delimited CSV (e.g. metabolite_shifts.csv). Rationale: BMRB's metabolomics archive is maintained as a separate database from macromolecule entries, so queries must target the correct database; the API is free, keyless, and rate-limited to 50 queries/second/IP with HTTP 403 issued on violation.

## Related tools

- **BMRB-API** (REST API used to query the metabolomics database and retrieve small-molecule entries in NMR-STAR or JSON format) — https://github.com/uwbmrb/BMRB-API
- **PyNMR-STAR** (Python library used to parse the downloaded NMR-STAR file and extract the assigned chemical shift loop) — https://github.com/uwbmrb/PyNMRSTAR
- **PyBMRB** (Optional Python module to visualize the extracted 1D chemical shift lists as multi-dimensional NMR spectra) — https://github.com/uwbmrb/PyBMRB
- **BMRB metabolomics database** (Source archive of 1H and 13C 1D/2D NMR spectra and assigned chemical shifts for over 1000 biological small molecules)

## Examples

```
import requests, pynmrstar; entry = pynmrstar.Entry.from_database('bmse000123', 'metabolomics'); shifts = entry.get_loops_by_category('_Atom_chem_shift')[0]
```

## Evaluation signals

- metabolite_shifts.csv exists and is a valid comma-delimited CSV.
- The CSV contains populated chemical-shift value and atom-identifier columns matching the entry's assigned shifts.
- The entry ID returned by the API corresponds to an entry in the 'metabolomics' database (verifiable via /list_entries?database=metabolomics).
- The parsed NMR-STAR file opens cleanly in PyNMR-STAR (parses without error) and the chemical shift loop is non-empty.
- API responses do not return HTTP 403, confirming the 50 queries/second rate limit was respected.

## Limitations

- The API enforces a rate limit of up to 50 queries per second per IP; exceeding it returns HTTP 403 and requires waiting at least 10 seconds.
- Uploaded/stored entries via POST /entry/ expire after one week unless refreshed, and uploaded data is publicly accessible via the assigned entry_id.
- Queries against a table a given database does not contain silently return no results rather than an error, so a wrong database choice can look like an empty result set.
- About 25 older-format BMRB entries are not included in current query grid results.
- Applications distributing API calls should send an 'Application' HTTP header with name and version so BMRB can track usage.

## Evidence

- [other] BMRB's metabolomics resource contains a growing database of 1H and 13C 1D and 2D NMR spectra (including time-domain data) and assigned chemical shifts for over 1000 biological small molecules: [evidence span withheld — anchor status: not_found]
- [readme] Note that not all databases contain the same tables. In general, a search in a table that a given database doesn't contain will not produce an error; instead it will return that no results were found.: [evidence span withheld — anchor status: external_doc]
- [readme] The Biological Magnetic Resonance Bank is developing a REST API to use for querying against the substantial BMRB databases. Most data in the macromolecule and metabolomics databases is accessible through the API.: [evidence span withheld — anchor status: external_doc]
- [readme] This library was developed by the BMRB to give the Python-using NMR community tools to work with the NMR-STAR data format.: [evidence span withheld — anchor status: external_doc]
