---
name: nmr-star-entry-sequence-extraction
description: Use when starting from a BMRB accession (or a local NMR-STAR entry) and
  you need the protein entity sequence in FASTA format to run BLAST-based cross-linking
  of BMRB archive data to external public databases.
when_to_use_negative:
- The sequence already exists in FASTA or the entry is from a non-BMRB archive (fetch
  directly from UniProt/PDB instead of parsing NMR-STAR).
- You need chemical shifts, restraints, or spectra rather than the entity sequence
  — use PyBMRB/RBMRB visualization or the BMRB query grid instead.
- The BMRB entry is one of the ~25 older-format entries excluded from current query
  grids; verify entry availability via /list_entries or /status first.
edam_operation: http://edamontology.org/operation_0292
tools:
- name: BMRB-API
  role: RESTful API used to fetch the BMRB entry (JSON or text format) containing
    the entity amino-acid sequence
  repo: https://github.com/uwbmrb/BMRB-API
- name: PyNMR-STAR
  role: Python module for reading and manipulating NMR-STAR files, used to parse the
    entry locally and extract the entity sequence
  repo: https://github.com/uwbmrb/PyNMRSTAR
- name: BLAST
  role: 'Downstream consumer: the extracted FASTA is submitted as blastp query against
    a curated protein database (BLOSUM62 defaults, E-value <= 1e-5 cutoff)'
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/978-1-0716-0270-6_14
    title: BioMagResBank (BMRB) as a Resource for Structural Biology
  source_content_hash: sha256:db44bd9ed49881725938bde5cbed5e09dc01c85a3ce1c706e8a16216f5f36b40
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/uwbmrb/BMRB-API
  tools:
  - BLAST
  - BMRB-API
  - PyNMR-STAR
  edam_operation: http://edamontology.org/operation_0292
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# nmr-star-entry-sequence-extraction

## Summary

Retrieves a BMRB entry's entity amino-acid sequence via the BMRB RESTful API and/or PyNMR-STAR parsing of the NMR-STAR file, emitting it in FASTA format as the query for downstream sequence-homology searches (e.g., NCBI blastp against UniProtKB/Swiss-Prot).

## When to use

Use when starting from a BMRB accession (or a local NMR-STAR entry) and you need the protein entity sequence in FASTA format to run BLAST-based cross-linking of BMRB archive data to external public databases.

## When NOT to use

- The sequence already exists in FASTA or the entry is from a non-BMRB archive (fetch directly from UniProt/PDB instead of parsing NMR-STAR).
- You need chemical shifts, restraints, or spectra rather than the entity sequence — use PyBMRB/RBMRB visualization or the BMRB query grid instead.
- The BMRB entry is one of the ~25 older-format entries excluded from current query grids; verify entry availability via /list_entries or /status first.

## Inputs

- BMRB accession number (macromolecules, metabolomics, or chemcomps database)
- NMR-STAR entry file (text or JSON format)
- BMRB REST API endpoint URL (api.bmrb.io/v2/ or /current/)

## Outputs

- FASTA-formatted entity amino-acid sequence with BMRB accession as query_id
- Query sequence ready for NCBI blastp submission against UniProtKB/Swiss-Prot

## How to apply

Fetch the chosen BMRB accession from the BMRB REST API (e.g., http://api.bmrb.io/current/entry/$entry_id, optionally with an entry_format parameter), requesting the 'Application: <name> <version>' HTTP header so BMRB can track usage. If working from a local NMR-STAR file instead, parse it with PyNMR-STAR (pip package pynmrstar) to walk the entity saveframe and extract the entity amino-acid sequence. Convert the extracted sequence to FASTA with the BMRB accession as query_id, since the BLAST tabular output columns (query_id, subject_id, ...) key on that identifier. Respect API rate limits (up to 50 queries/second per IP; HTTP 403 on breach, wait at least 10 seconds before retrying). The resulting FASTA is then submitted to blastp with BLOSUM62 defaults, filtering hits at E-value <= 1e-5 into homology_hits.csv.

## Related tools

- **BMRB-API** (RESTful API used to fetch the BMRB entry (JSON or text format) containing the entity amino-acid sequence) — https://github.com/uwbmrb/BMRB-API
- **PyNMR-STAR** (Python module for reading and manipulating NMR-STAR files, used to parse the entry locally and extract the entity sequence) — https://github.com/uwbmrb/PyNMRSTAR
- **BLAST** (Downstream consumer: the extracted FASTA is submitted as blastp query against a curated protein database (BLOSUM62 defaults, E-value <= 1e-5 cutoff))

## Examples

```
import requests; r = requests.get('http://api.bmrb.io/current/entry/15060?format=text', headers={'Application':'seq-extract 1.0'}); star = r.text; seq = pynmrstar.Entry.from_string(star).get_saveframe_by_name('entity').get_tag('Entity.Polymer_seq_one_letter_code')
```

## Evaluation signals

- FASTA output parses as valid FASTA: a single '>' header line with the BMRB accession as query_id followed by a standard amino-acid alphabet sequence.
- Extracted sequence length matches the entity SEQ_LENGTH in the NMR-STAR entry's entity saveframe.
- BLAST run on the extracted sequence yields homology_hits.csv that parses as CSV with columns query_id, subject_id, subject_description, percent_identity, alignment_length, e_value.
- homology_hits.csv contains at least one significant hit (E-value <= 1e-5), and hits are ranked by E-value.
- API calls succeed without HTTP 403 (rate-limit) responses; any 403 is followed by a >= 10 second wait before retrying.

## Limitations

- BMRB API rate limit is 50 queries per second per IP; exceeding it returns HTTP 403 and blacklisting requires waiting at least 10 seconds before any further queries.
- HTTPS is available but slightly slower due to TLS overhead; recommended only for uploading private data.
- A few (~25) older-format BMRB entries are not included in current query results, so some accessions may be unavailable through the API/query grid.
- Data uploaded to the API /entry/ store endpoint is publicly accessible via the assigned entry_id, so it must not be used for confidential sequences.

## Evidence

- [readme] REST API fetch: [evidence span withheld — anchor status: external_doc]
- [readme] entry retrieval: [evidence span withheld — anchor status: external_doc]
- [readme] rate limiting: [evidence span withheld — anchor status: external_doc]
- [methods] PyNMR-STAR purpose: "There is extensive documentation and an introduction to working with the NMR-STAR format at the GitHub page for the project: https://github.com/uwbmrb/PyNMRSTAR."
- [methods] older-format entries: "Currently a few BMRB entries (~25) in an older format are not included in the current queries."
