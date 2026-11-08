---
name: bgc-accession-request-submission
description: Use when a task requires submitting or simulating submission of a new
  biosynthetic gene cluster entry to MIBiG, or structuring BGC metadata into the accession-request
  → staff-approval → extended-annotation pipeline.
when_to_use_negative:
- The BGC is a candidate cluster from genome mining (e.g. antiSMASH output) of unknown
  function — MIBiG only accepts BGCs associated with molecules of known chemical structure
- The goal is querying/filtering existing MIBiG entries (e.g. 'find all complete RiPP
  BGCs from the genus Streptomyces') rather than depositing a new one
- The sequence is already a published, fully annotated MIBiG entry — use the repository/API
  instead of re-submitting
tools:
- name: MIBiG Repository
  role: Online portal (https://mibig.secondarymetabolites.org/) through which the
    accession request and extended entry form are submitted
  repo: https://mibig.secondarymetabolites.org/
- name: mibig-api
  role: REST-like web API backend handling access to the underlying PostgreSQL database
    for the repository
  repo: https://github.com/mibig-secmet/mibig-api/
- name: JSON schema
  role: Description and validation technology used to embed validation and dependency
    rules for the annotation data
  repo: https://json-schema.org
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkz882
    title: 'MIBiG 2.0: a repository for biosynthetic gene clusters of known function'
  source_content_hash: sha256:6ca7e6d532facafa63f10c25b1f49fe5ee8c390b2ba0e20fa0f4d06e35c35a82
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - JSON schema
  - MIBiG Repository
  - mibig-api
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# bgc-accession-request-submission

## Summary

Reconstructs the two-stage MIBiG crowdsourcing submission workflow: first a minimal accession request (product name(s) plus BGC sequence/Genbank coordinates), then, after MIBiG staff approval, an extended entry form for detailed annotation. Useful for simulating curated BGC deposition into the MIBiG 2.0 repository.

## When to use

Use when a task requires submitting or simulating submission of a new biosynthetic gene cluster entry to MIBiG, or structuring BGC metadata into the accession-request → staff-approval → extended-annotation pipeline.

## When NOT to use

- The BGC is a candidate cluster from genome mining (e.g. antiSMASH output) of unknown function — MIBiG only accepts BGCs associated with molecules of known chemical structure
- The goal is querying/filtering existing MIBiG entries (e.g. 'find all complete RiPP BGCs from the genus Streptomyces') rather than depositing a new one
- The sequence is already a published, fully annotated MIBiG entry — use the repository/API instead of re-submitting

## Inputs

- BGC product name(s)
- BGC sequence information, preferentially NCBI Genbank accession coordinates
- MIBiG submission workflow documentation describing the extended entry form

## Outputs

- Accession-request record (product names, sequence coordinates/Genbank accession, pending-approval status)
- Two-stage structured JSON submission record (e.g. submission_record.json) combining accession-request and extended-annotation fields

## How to apply

Stage 1: compile the minimal accession-request metadata — product name(s) and BGC sequence information, preferentially as NCBI Genbank accession coordinates — since the paper states 'a user starts by requesting a MIBiG accession number' with exactly these fields. Stage 2: apply the approval gate by marking the request approved only once the required fields (product name(s) and Genbank-derived coordinates) are present, mirroring the MIBiG staff approval step. Stage 3: enumerate the extended entry form fields that the approved submitter must fill for detailed annotation, per the MIBiG submission documentation. Validate the assembled two-stage record against the JSON schema description and validation technology that MIBiG 2.0 adopted to 'embed validation and dependency rules into the schema', then emit a structured JSON submission record (e.g. submission_record.json) capturing both stages.

## Related tools

- **MIBiG Repository** (Online portal (https://mibig.secondarymetabolites.org/) through which the accession request and extended entry form are submitted) — https://mibig.secondarymetabolites.org/
- **mibig-api** (REST-like web API backend handling access to the underlying PostgreSQL database for the repository) — https://github.com/mibig-secmet/mibig-api/
- **JSON schema** (Description and validation technology used to embed validation and dependency rules for the annotation data) — https://json-schema.org

## Examples

```
python -c "import json; json.load(open('submission_record.json'))"
```

## Evaluation signals

- Output file submission_record.json exists and parses as valid JSON
- Record contains the accession-request fields: product name(s) and BGC sequence coordinates or Genbank accession
- Record contains the extended entry form annotation fields for the approved stage
- Approval-gate logic: request is only marked approved when all required accession-request fields are present
- Record matches the two-stage workflow documented in the paper's submission description

## Limitations

- Accession requests are gated by MIBiG staff approval; the simulated gate is only a proxy for the real review process
- The extended entry form fields must be enumerated from the MIBiG submission documentation; the paper does not list every field explicitly
- The repository and its API are community-maintained (Apache-2.0 licensed Go server); the README provides no changelog or usage examples for the submission endpoint

## Evidence

- [intro] Request a MIBiG accession number by submitting product names and BGC sequence information (Genbank coordinates): "a user starts by requesting a MIBiG accession number"
- [intro] After approval by MIBiG staff, submit detailed information via an extended entry form: "After the request is approved by MIBiG staff, the workflow subsequently provides an extended entry form"
- [intro] Validate and embed dependency rules into the schema using JSON schema technology: "embed validation and dependency rules into the schema"
- [readme] The MIBiG API server is the WebAPI backend for the repository: "The WebAPI server driving the [Minimum Information about a Biosynthetic Gene cluster (MIBiG)](http://mibig.secondarymetabolites.org/) repository."
