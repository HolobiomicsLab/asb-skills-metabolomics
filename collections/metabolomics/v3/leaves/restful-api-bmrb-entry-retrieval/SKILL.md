---
name: restful-api-bmrb-entry-retrieval
description: Use when a workflow needs a specific archived BMRB entry fetched programmatically
  (by entry_id) in NMR-STAR or JSON format, without manual browser download, as input
  for parsing or chemical-shift extraction.
when_to_use_negative:
- The data of interest is not archived in BMRB's macromolecule, metabolomics, or chemcomps
  databases — queries against tables a database does not contain simply return no
  results rather than an error, which can be mistaken for a broken query.
- The task is deposition of new NMR data rather than retrieval — use OneDep, ADIT-NMR/BMRBdep,
  or SMSDep deposition workflows instead.
- You need interactive visualization of chemical shift distributions rather than raw
  entry retrieval — PyBMRB is the appropriate layer, as it fetches and plots shifts
  directly.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3272
tools:
- name: BMRB-API
  role: RESTful query interface used to fetch the BMRB entry by entry_id in JSON or
    NMR-STAR format from api.bmrb.io
  repo: https://github.com/uwbmrb/BMRB-API
- name: PyNMR-STAR
  role: Python module that reads and validates the returned NMR-STAR entry in memory
    so the 'Atom_chem_shift' loop can be accessed
  repo: https://github.com/uwbmrb/PyNMRSTAR
- name: PyBMRB
  role: Optional downstream layer that fetches BMRB data via BMRB-API and parses NMR-STAR
    files for chemical shift visualization, avoiding manual download
  repo: https://github.com/uwbmrb/PyBMRB
- name: RBMRB
  role: R-language alternative for accessing and manipulating BMRB/NMR-STAR data when
    working in R
  repo: https://github.com/uwbmrb/RBMRB
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/978-1-0716-0270-6_14
    title: BioMagResBank (BMRB) as a Resource for Structural Biology
  source_content_hash: sha256:a8adb16d2c437c3ab242170a0390f6b1e13a5166cbf13986e966235945ee838a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/uwbmrb/BMRB-API
  tools:
  - BMRB-API
  - PyBMRB
  - PyNMR-STAR
  - RBMRB
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
emails_redacted: true
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# restful-api-bmrb-entry-retrieval

## Summary

Programmatically retrieve a BMRB entry (e.g., BMRB entry 17074) in NMR-STAR or JSON format via the BMRB RESTful API at api.bmrb.io, then parse it in memory with PyNMR-STAR for downstream extraction of assigned chemical shifts.

## When to use

Use when a workflow needs a specific archived BMRB entry fetched programmatically (by entry_id) in NMR-STAR or JSON format, without manual browser download, as input for parsing or chemical-shift extraction.

## When NOT to use

- The data of interest is not archived in BMRB's macromolecule, metabolomics, or chemcomps databases — queries against tables a database does not contain simply return no results rather than an error, which can be mistaken for a broken query.
- The task is deposition of new NMR data rather than retrieval — use OneDep, ADIT-NMR/BMRBdep, or SMSDep deposition workflows instead.
- You need interactive visualization of chemical shift distributions rather than raw entry retrieval — PyBMRB is the appropriate layer, as it fetches and plots shifts directly.

## Inputs

- BMRB entry accession number / entry_id (e.g., 17074)
- Desired output format parameter for the /entry/ query (JSON default or NMR-STAR text)
- BMRB API version (e.g., v2 or current)
- 'Application' HTTP header value (application name plus version number)

## Outputs

- BMRB entry in NMR-STAR format (text) suitable for parsing with PyNMR-STAR
- BMRB entry in JSON format (default) keyed by entry, saveframe, or loop
- In-memory parsed NMR-STAR entry object from which the 'Atom_chem_shift' loop records (residue, atom name, shift value) can be extracted

## How to apply

Send an HTTP GET to api.bmrb.io with an explicit major version path (e.g., /v2/ for stable releases, /current/ for development) followed by the query type, e.g. /entry/$entry_id, which returns JSON by default and another format (such as NMR-STAR text) when the format parameter is supplied. Include an 'Application' HTTP header naming your application and version so BMRB can track usage, and respect the rate limit of up to 50 queries per second per IP — a HTTP 403 response signals rate limiting, and you must wait at least 10 seconds before any further queries. For deployed software, pin a fixed major version URL rather than /current/ so new API releases do not break your queries. Retrieve the entry in NMR-STAR format and parse it in memory with PyNMR-STAR, which validates syntactic correctness against the NMR-STAR schema, then locate the 'Atom_chem_shift' saveframe/loop to extract per-atom records (atom type, residue, atom name, chemical shift value). Note that only ~25 older-format BMRB entries are not included in current query results, so very old accessions may fail.

## Related tools

- **BMRB-API** (RESTful query interface used to fetch the BMRB entry by entry_id in JSON or NMR-STAR format from api.bmrb.io) — https://github.com/uwbmrb/BMRB-API
- **PyNMR-STAR** (Python module that reads and validates the returned NMR-STAR entry in memory so the 'Atom_chem_shift' loop can be accessed) — https://github.com/uwbmrb/PyNMRSTAR
- **PyBMRB** (Optional downstream layer that fetches BMRB data via BMRB-API and parses NMR-STAR files for chemical shift visualization, avoiding manual download) — https://github.com/uwbmrb/PyBMRB
- **RBMRB** (R-language alternative for accessing and manipulating BMRB/NMR-STAR data when working in R) — https://github.com/uwbmrb/RBMRB

## Examples

```
import requests; r = requests.get("http://api.bmrb.io/v2/entry/17074", headers={"Application":"My Application 1.0"})
```

## Evaluation signals

- The GET request to api.bmrb.io/v2/entry/$entry_id returns HTTP 200 and the body parses as a valid NMR-STAR entry (via PyNMR-STAR) rather than an error payload.
- The parsed entry contains the 'Atom_chem_shift' saveframe/loop with per-atom records carrying residue, atom name, and chemical shift value columns.
- The Application header is set and the client never issues more than 50 queries per second per IP; any HTTP 403 triggers a wait of at least 10 seconds before retry.
- A pinned major version URL (e.g., /v2/) is used for released software, and the same query still succeeds after minor version updates (backwards compatibility expectation).
- The script completes without error and the extracted per-shift record count matches the number of assigned chemical shifts in the entry.

## Limitations

- Rate limiting: up to 50 queries per second per IP; exceeding it returns HTTP 403 for all requests and requires waiting at least 10 seconds; heavy users must contact <maintainer e-mail, see the repository README> for an exception.
- Data uploaded via the POST /entry/ endpoint is publicly accessible to anyone holding the assigned entry_id and expires after one week.
- A few (~25) BMRB entries in an older format are not included in the current query grid results.
- Major API version increments can break existing queries, so applications must pin a specific major version.

## Evidence

- [full_text] BMRB offers an API and Python/R libraries enabling programmatic analysis and manipulation of NMR-STAR formatted files.: [evidence span withheld — anchor status: not_found]
