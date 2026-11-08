---
name: metabolomics-entry-retrieval-via-rest-api
description: Use when a task requires fetching a BMRB metabolomics (small molecule)
  entry — e.g. its assigned chemical shifts or spectral data — programmatically from
  api.bmrb.io rather than browsing the BMRB website or FTP server.
when_to_use_negative:
- The target entry is a macromolecule (protein, nucleic acid) — the API defaults to
  the 'macromolecules' database, so querying metabolite IDs without specifying database=metabolomics
  will return no results
- The goal is interactive visualization of chemical shift histograms rather than retrieval
  — PyBMRB fetches and plots shift data directly and may be more appropriate
- Bulk FTP download of the whole metabolomics directory is needed — the API is query-oriented;
  use the BMRB FTP server (http://www.bmrb.wisc.edu/ftp/pub/bmrb/) instead
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3272
tools:
- name: BMRB-API
  role: RESTful query interface used to list metabolomics entries and retrieve a small-molecule
    entry in NMR-STAR (text) or JSON format
  repo: https://github.com/uwbmrb/BMRB-API
- name: PyNMR-STAR
  role: Python library used to parse the retrieved NMR-STAR entry and extract the
    assigned chemical shift loop
  repo: https://github.com/uwbmrb/PyNMRSTAR
- name: BMRB metabolomics database
  role: The small-molecule archive holding 1H/13C 1D and 2D spectra and assigned shifts
    for over 1000 biological small molecules
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/978-1-0716-0270-6_14
    title: BioMagResBank (BMRB) as a Resource for Structural Biology
  source_content_hash: sha256:d460a6cba9b6a6d5e09ae110ff51c70d049d8aed351d9ccd36b19aa2f8178568
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/uwbmrb/BMRB-API
  tools:
  - BMRB metabolomics database
  - BMRB-API
  - PyNMR-STAR
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# metabolomics-entry-retrieval-via-rest-api

## Summary

Retrieves a small-molecule/metabolite entry (1D/2D NMR spectra, time-domain data, and assigned chemical shifts) from BMRB's metabolomics database via the BMRB RESTful API, returning the record in NMR-STAR format for downstream parsing with PyNMR-STAR.

## When to use

Use when a task requires fetching a BMRB metabolomics (small molecule) entry — e.g. its assigned chemical shifts or spectral data — programmatically from api.bmrb.io rather than browsing the BMRB website or FTP server.

## When NOT to use

- The target entry is a macromolecule (protein, nucleic acid) — the API defaults to the 'macromolecules' database, so querying metabolite IDs without specifying database=metabolomics will return no results
- The goal is interactive visualization of chemical shift histograms rather than retrieval — PyBMRB fetches and plots shift data directly and may be more appropriate
- Bulk FTP download of the whole metabolomics directory is needed — the API is query-oriented; use the BMRB FTP server (http://www.bmrb.wisc.edu/ftp/pub/bmrb/) instead

## Inputs

- BMRB metabolomics entry identifier (e.g. a bmse### ID)
- BMRB API base URL and version (e.g. http://api.bmrb.io/v2/)
- Target database selector: 'metabolomics'

## Outputs

- NMR-STAR format entry file (e.g. metabolite_entry.str) containing assigned chemical shifts and spectrum-associated time-domain/peak-list references
- Comma-delimited chemical shift table (e.g. metabolite_shifts.csv) with atom/residue identifier and shift value columns

## How to apply

Query the BMRB REST API at api.bmrb.io, selecting the 'metabolomics' database explicitly (queries default to 'macromolecules'), e.g. via /list_entries?database=metabolomics to enumerate small-molecule entries. Retrieve the entry with /entry/$entry_id?format=str to obtain NMR-STAR text rather than the default JSON, and include an 'Application: <name> <version>' HTTP header if the client will be distributed. Parse the returned NMR-STAR file with PyNMR-STAR to locate the assigned chemical shift loop, then extract atom/residue identifiers and shift values into a comma-delimited table. Respect the API's rate limit of 50 queries per second per IP; on an HTTP 403, wait at least 10 seconds before resending any queries. Use /current/ for development but pin a fixed major version (e.g. /v2/) in deployed applications since major-version increments can break queries.

## Related tools

- **BMRB-API** (RESTful query interface used to list metabolomics entries and retrieve a small-molecule entry in NMR-STAR (text) or JSON format) — https://github.com/uwbmrb/BMRB-API
- **PyNMR-STAR** (Python library used to parse the retrieved NMR-STAR entry and extract the assigned chemical shift loop) — https://github.com/uwbmrb/PyNMRSTAR
- **BMRB metabolomics database** (The small-molecule archive holding 1H/13C 1D and 2D spectra and assigned shifts for over 1000 biological small molecules)

## Examples

```
import requests; r = requests.get('http://api.bmrb.io/v2/entry/bmse000001?format=str', headers={'Application':'MyApp 1.0'}); open('metabolite_entry.str','w').write(r.text)
```

## Evaluation signals

- metabolite_shifts.csv exists and parses as a valid CSV
- The CSV contains populated chemical-shift value and atom-identifier columns matching the entry's assigned shifts in the NMR-STAR file
- The retrieved record is valid NMR-STAR (PyNMR-STAR parses it without error) and corresponds to a metabolomics-database entry, not macromolecules
- HTTP responses do not include 403 rate-limit errors; query rate stays under 50 requests/second per IP
- Spectrum-associated time-domain or peak-list data referenced in the entry resolve against the Metabolomics directory of the BMRB FTP server

## Limitations

- The API enforces a rate limit of up to 50 queries per second per IP; exceeding it yields HTTP 403 for all requests until at least 10 seconds elapse
- A few (~25) older-format BMRB entries are not included in current query grid results
- Queries use the 'macromolecules' database by default; searching for metabolites without specifying database=metabolomics silently returns no results rather than an error
- Major API version increments may break existing queries, so fixed-version URLs should be used in deployed software

## Evidence

- [readme] metabolomics database listing: [evidence span withheld — anchor status: not_found]
- [readme] rate limit: [evidence span withheld — anchor status: not_found]
- [methods] metabolomics archive scope: [evidence span withheld — anchor status: not_found]
- [intro] ALATIS library scope: "BMRB also offers support for studies of metabolomics and natural products through a library of a variety of 1D and two-dimensional (2D) NMR spectra of pure compounds"
- [readme] PyNMRSTAR purpose: [evidence span withheld — anchor status: external_doc]
