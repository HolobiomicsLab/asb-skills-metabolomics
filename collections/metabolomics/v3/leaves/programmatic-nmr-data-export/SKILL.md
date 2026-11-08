---
name: programmatic-nmr-data-export
description: Use when a task requires programmatically retrieving a specific BMRB
  accession (e.g., BMRB entry 17074) and exporting its assigned per-atom chemical
  shift records into a structured tabular file for downstream analysis.
when_to_use_negative:
- The goal is deposition of new NMR data rather than retrieval/export — use the OneDep,
  ADIT-NMR, SMSDep, or BMRBdep deposition workflows instead
- The task is visualization of chemical shift distributions rather than tabular export
  — PyBMRB's plotting/histogram functionality is the more direct choice
- The data of interest are metabolomics small-molecule spectra or chemcomp entries
  not stored as assigned macromolecular chemical shift loops
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3366
tools:
- name: BMRB-API
  role: RESTful API used to query and retrieve the chosen BMRB accession in NMR-STAR
    format and access its chemical-shift query capability
  repo: https://github.com/uwbmrb/BMRB-API
- name: PyNMR-STAR
  role: Python library used to parse the returned NMR-STAR file in memory, ensuring
    syntactic validity against the NMR-STAR schema, and to access the Atom_chem_shift
    loop
  repo: https://github.com/uwbmrb/PyNMRSTAR
- name: PyBMRB
  role: Companion Python module that can fetch BMRB data directly via the API and
    parse NMR-STAR files, avoiding manual download steps; primarily for visualization
  repo: https://github.com/uwbmrb/PyBMRB
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/978-1-0716-0270-6_14
    title: BioMagResBank (BMRB) as a Resource for Structural Biology
  source_content_hash: sha256:3ca3c3fee8b1f7bcf7b8d982ae6b4de833091d2f28a58ff2598850716b7f8df6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/uwbmrb/BMRB-API
  tools:
  - BMRB-API
  - PyBMRB
  - PyNMR-STAR
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
emails_redacted: true
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# programmatic-nmr-data-export

## Summary

Retrieves BMRB entries in NMR-STAR format via the BMRB REST API, parses them in memory with PyNMR-STAR, and exports assigned chemical shift records (residue, atom name, shift value) to a comma-delimited table such as chemical_shifts.csv.

## When to use

Use when a task requires programmatically retrieving a specific BMRB accession (e.g., BMRB entry 17074) and exporting its assigned per-atom chemical shift records into a structured tabular file for downstream analysis.

## When NOT to use

- The goal is deposition of new NMR data rather than retrieval/export — use the OneDep, ADIT-NMR, SMSDep, or BMRBdep deposition workflows instead
- The task is visualization of chemical shift distributions rather than tabular export — PyBMRB's plotting/histogram functionality is the more direct choice
- The data of interest are metabolomics small-molecule spectra or chemcomp entries not stored as assigned macromolecular chemical shift loops

## Inputs

- BMRB accession number (e.g., entry 17074)
- NMR-STAR formatted entry retrieved from the BMRB REST API
- Optional residue and/or atom type filters for the chemical shift records

## Outputs

- chemical_shifts.csv — comma-delimited table with one row per assigned chemical shift and columns for residue, atom name, and shift value
- In-memory parsed NMR-STAR entry object (PyNMR-STAR Entry)

## How to apply

Query the BMRB RESTful API (e.g., api.bmrb.io/v2/entry/$entry_id?format=nmrstar) for the chosen accession, requesting the entry in NMR-STAR format; no API key is required but requests should include an 'Application' HTTP header and respect the rate limit of up to 50 queries per second per IP. Parse the returned NMR-STAR file in memory with PyNMR-STAR, which validates syntactic correctness against the NMR-STAR schema. Locate the assigned chemical shift loop (the 'Atom_chem_shift' saveframe/loop) and extract the per-atom records — atom type, residue, atom name, chemical shift value — optionally filtered by residue and atom type, matching the API's chemical-shift query capability. Write the extracted shift records to a comma-delimited chemical_shifts.csv with the required columns (residue, atom name, shift value), one row per assigned chemical shift.

## Related tools

- **BMRB-API** (RESTful API used to query and retrieve the chosen BMRB accession in NMR-STAR format and access its chemical-shift query capability) — https://github.com/uwbmrb/BMRB-API
- **PyNMR-STAR** (Python library used to parse the returned NMR-STAR file in memory, ensuring syntactic validity against the NMR-STAR schema, and to access the Atom_chem_shift loop) — https://github.com/uwbmrb/PyNMRSTAR
- **PyBMRB** (Companion Python module that can fetch BMRB data directly via the API and parse NMR-STAR files, avoiding manual download steps; primarily for visualization) — https://github.com/uwbmrb/PyBMRB

## Examples

```
import pynmrstar; entry = pynmrstar.Entry.from_database(17074); shifts = entry.get_loops_by_category('_Atom_chem_shift')[0].data; import csv; csv.writer(open('chemical_shifts.csv','w')).writerows(shifts)
```

## Evaluation signals

- The script runs without error end to end (API retrieval, parsing, extraction, export)
- chemical_shifts.csv exists, is comma-delimited, and contains the required columns (residue, atom name, shift value)
- chemical_shifts.csv has one row per assigned chemical shift in the entry — row count matches the Atom_chem_shift loop record count in the source NMR-STAR entry
- PyNMR-STAR parsing raises no schema/syntax errors, confirming the NMR-STAR file is syntactically valid
- Spot-check a few exported shift values against the raw NMR-STAR Atom_chem_shift loop (or the API's chemical shift query) for residue/atom/value agreement

## Limitations

- A few (~25) older-format BMRB entries are not included in current API query grid results
- The API enforces a rate limit of up to 50 queries per second per IP; exceeding it returns HTTP 403 for all requests and requires waiting at least 10 seconds to be removed from the blacklist
- Large-scale querying may require contacting <maintainer e-mail, see the repository README> for a rate-limit exception; applications should send an 'Application' header with name and version
- Data uploaded to the API's temporary entry store is publicly accessible via the assigned entry_id and expires after one week of inactivity

## Evidence

- [readme] The Biological Magnetic Resonance Bank is developing a REST API to use for querying against the substantial BMRB databases.: [evidence span withheld — anchor status: external_doc]
- [readme] Limits: Up to 50 queries per second per IP.: [evidence span withheld — anchor status: external_doc]
