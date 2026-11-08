---
name: mibig-annotation-form-completion
description: 'Use when depositing a BGC of known function into the MIBiG repository,
  or when reconstructing/simulating its submission pipeline: an accession-request
  stage followed by an extended annotation form after staff approval.'
when_to_use_negative:
- The BGC is of unknown function — MIBiG is a repository for BGCs of known function,
  so such entries do not qualify for submission.
- The goal is to query or extract existing MIBiG entries programmatically rather than
  deposit a new one — use the MIBiG REST-like web API instead of the submission workflow.
- Only predicted/candidate BGCs from genome mining (e.g. antiSMASH or ClusterFinder
  output) are available without linked products of known chemical structure — these
  belong in databases like IMG-ABC or antiSMASH-DB, not the MIBiG annotation pipeline.
edam_topics:
- http://edamontology.org/topic_0219
tools:
- name: JSON schema
  role: Validates the extended-entry annotation and embeds dependency rules so submitted
    fields comply with the MIBiG 2.0 data standard
  repo: https://json-schema.org
- name: MIBiG Repository
  role: The online target of the submission workflow — accession requests and extended
    entry forms are submitted through it
  repo: https://mibig.secondarymetabolites.org/
- name: mibig-api
  role: REST-like web API server driving the MIBiG repository, handling access to
    the underlying PostgreSQL database into which submitted entries are stored
  repo: https://github.com/mibig-secmet/mibig-api/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkz882
    title: 'MIBiG 2.0: a repository for biosynthetic gene clusters of known function'
  source_content_hash: sha256:2136bac1488f306227e80224a617bf48a8ab296b5241a9117ef1a6e04c393c44
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - JSON schema
  - MIBiG Repository
  - mibig-api
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# mibig-annotation-form-completion

## Summary

Captures the documented two-stage MIBiG crowdsourcing submission workflow: first requesting a MIBiG accession number with product name(s) and BGC sequence information, then, after MIBiG staff approval, completing the extended entry form with detailed annotation data. It exists so future agents can model or replicate deposition of a biosynthetic gene cluster (BGC) of known function into MIBiG.

## When to use

Use when depositing a BGC of known function into the MIBiG repository, or when reconstructing/simulating its submission pipeline: an accession-request stage followed by an extended annotation form after staff approval.

## When NOT to use

- The BGC is of unknown function — MIBiG is a repository for BGCs of known function, so such entries do not qualify for submission.
- The goal is to query or extract existing MIBiG entries programmatically rather than deposit a new one — use the MIBiG REST-like web API instead of the submission workflow.
- Only predicted/candidate BGCs from genome mining (e.g. antiSMASH or ClusterFinder output) are available without linked products of known chemical structure — these belong in databases like IMG-ABC or antiSMASH-DB, not the MIBiG annotation pipeline.

## Inputs

- BGC product name(s) for the accession request
- BGC sequence information (nucleotide coordinates, preferably from an NCBI Genbank accession)
- MIBiG staff approval status for the requested accession number
- Detailed BGC annotation data for the extended entry form (e.g. biosynthetic class, literature references, associated chemical structure)

## Outputs

- MIBiG accession-number request record (pending/approved status)
- Completed extended-entry annotation record for the approved BGC
- Two-stage submission record saved as structured JSON (submission_record.json)

## How to apply

First compile the metadata required for the accession request: the BGC product name(s) and BGC sequence information, preferentially as coordinates from an NCBI Genbank accession, and submit it to request a MIBiG accession number. The request is then gated by MIBiG staff approval, so model this as a pending-to-approved state transition that fires only once the required fields (product names plus Genbank-derived coordinates) are present. After approval, populate the extended entry form with detailed annotation information for the BGC. Validate the resulting entry against MIBiG's JSON schema, which was adopted specifically to 'embed validation and dependency rules into the schema', so that submitted annotations comply with the MIBiG 2.0 data standard. Finally, assemble both stages — accession request and extended annotation — as a structured JSON submission record (e.g. submission_record.json), reflecting that MIBiG stores entries as JSON-validated records in its PostgreSQL-backed repository.

## Related tools

- **JSON schema** (Validates the extended-entry annotation and embeds dependency rules so submitted fields comply with the MIBiG 2.0 data standard) — https://json-schema.org
- **MIBiG Repository** (The online target of the submission workflow — accession requests and extended entry forms are submitted through it) — https://mibig.secondarymetabolites.org/
- **mibig-api** (REST-like web API server driving the MIBiG repository, handling access to the underlying PostgreSQL database into which submitted entries are stored) — https://github.com/mibig-secmet/mibig-api/

## Examples

```
import json; json.dump({'accession_request': {'product_names': prod, 'genbank_coordinates': coords, 'status': 'approved'}, 'extended_entry': entry}, open('submission_record.json','w'))
```

## Evaluation signals

- submission_record.json exists and parses as valid JSON
- The accession-request stage contains product name(s) and BGC sequence coordinates or a Genbank accession
- The extended-annotation stage is present only after the simulated staff-approval gate has marked the request approved
- The annotation record validates against the MIBiG JSON schema with its embedded dependency rules
- Both the accession-request stage and the extended-annotation stage are captured in the single submission record, matching the two-stage workflow

## Limitations

- The extended entry form is only provided after MIBiG staff approve the accession request, so the pipeline cannot be completed end-to-end in one pass and is gated on human curation.
- The sequence/coordinate requirement preferentially depends on an NCBI Genbank accession; BGCs lacking such coordinates complicate the request stage.
- MIBiG staff approval and curation quality are external to the workflow; the API repository README offers no documented procedural details beyond Go-version development prerequisites.

## Evidence

- [intro] Request a MIBiG accession number by submitting product names and BGC sequence information (Genbank coordinates): "a user starts by requesting a MIBiG accession number"
- [intro] After approval by MIBiG staff, submit detailed information via an extended entry form: "After the request is approved by MIBiG staff, the workflow subsequently provides an extended entry form"
- [intro] Validate and embed dependency rules into the schema using JSON schema technology: "embed validation and dependency rules into the schema"
- [supplementary] The repository is accessible online: "The MIBiG Repository is available at https://mibig.secondarymetabolites.org/."
- [readme] README describes the API server that backs the repository receiving submissions: "The WebAPI server driving the [Minimum Information about a Biosynthetic Gene cluster (MIBiG)](http://mibig.secondarymetabolites.org/) repository."
