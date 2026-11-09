---
name: genbank-coordinate-extraction
description: Use when a candidate BGC of known function is being prepared for deposition
  into MIBiG and the accession-request stage requires product name(s) plus BGC sequence
  coordinates, preferentially derived from an NCBI Genbank accession.
when_to_use_negative:
- The BGC is a computationally predicted cluster from tools like antiSMASH or ClusterFinder
  without a known product — MIBiG is restricted to BGCs of known function.
- The goal is bulk querying/filtering of existing MIBiG entries rather than submitting
  a new one; use the repository's annotation-metadata filters or Boolean queries instead.
- The source cluster has no Genbank/NCBI accession or traceable sequence coordinates,
  so the required accession-request fields cannot be compiled.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_3174
tools:
- name: MIBiG Repository
  role: Online submission portal whose two-stage workflow receives the extracted Genbank
    coordinates and product names as an accession request, then exposes the extended
    entry form after staff approval.
  repo: https://mibig.secondarymetabolites.org/
- name: JSON schema
  role: Schema description and validation technology used to enforce field validity
    and dependency rules on the submitted MIBiG annotation, including the extended
    entry-form data.
  repo: https://json-schema.org
- name: mibig-api
  role: REST-like web API server handling access to the underlying MIBiG PostgreSQL
    database, through which submitted records are served.
  repo: https://github.com/mibig-secmet/mibig-api/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkz882
    title: 'MIBiG 2.0: a repository for biosynthetic gene clusters of known function'
  source_content_hash: sha256:63426ef530faf4c2170120f12e2ec12a93081eb43cfdc8b253c1e0a58921545c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - JSON schema
  - MIBiG Repository
  - mibig-api
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# genbank-coordinate-extraction

## Summary

Compile the sequence coordinates for a biosynthetic gene cluster (BGC) from NCBI Genbank provenance so they can be packaged with product name(s) into a MIBiG accession request, the first stage of the two-stage MIBiG crowdsourcing submission workflow.

## When to use

Use when a candidate BGC of known function is being prepared for deposition into MIBiG and the accession-request stage requires product name(s) plus BGC sequence coordinates, preferentially derived from an NCBI Genbank accession.

## When NOT to use

- The BGC is a computationally predicted cluster from tools like antiSMASH or ClusterFinder without a known product — MIBiG is restricted to BGCs of known function.
- The goal is bulk querying/filtering of existing MIBiG entries rather than submitting a new one; use the repository's annotation-metadata filters or Boolean queries instead.
- The source cluster has no Genbank/NCBI accession or traceable sequence coordinates, so the required accession-request fields cannot be compiled.

## Inputs

- NCBI Genbank accession or Genbank-format record containing the BGC nucleotide sequence
- BGC boundary/coordinate information for the cluster within the Genbank record
- Product name(s) of the biosynthetic gene cluster's known compound(s)

## Outputs

- Accession-request record containing product name(s) and BGC sequence coordinates/Genbank accession
- Structured JSON two-stage submission record capturing the accession-request stage and (post-approval) the extended-annotation stage (e.g. submission_record.json)

## How to apply

First, locate the source Genbank record (or NCBI accession) that contains the BGC of known function and extract the cluster boundary coordinates and the contig/record identifiers, since MIBiG explicitly asks submitters to provide 'BGC sequence information (preferably Genbank coordinates)'. Second, pair the extracted coordinates with the product name(s) of the cluster's known metabolites — these two field sets are the minimum required content of the accession request. Third, treat the assembled record as pending until MIBiG staff approve it; approval gates access to the extended entry form used for detailed annotation. Rationale: MIBiG's two-stage design uses the lightweight accession request (product names + Genbank-derived coordinates) as the entry gate, with full JSON-schema-validated annotation deferred to the extended form after staff approval.

## Related tools

- **MIBiG Repository** (Online submission portal whose two-stage workflow receives the extracted Genbank coordinates and product names as an accession request, then exposes the extended entry form after staff approval.) — https://mibig.secondarymetabolites.org/
- **JSON schema** (Schema description and validation technology used to enforce field validity and dependency rules on the submitted MIBiG annotation, including the extended entry-form data.) — https://json-schema.org
- **mibig-api** (REST-like web API server handling access to the underlying MIBiG PostgreSQL database, through which submitted records are served.) — https://github.com/mibig-secmet/mibig-api/

## Examples

```
python scripts/build_submission_record.py --genbank NC_003888.3 --coords 1..128796 --products "actinorhodin" --out submission_record.json
```

## Evaluation signals

- The output JSON record is valid and parses without error.
- The accession-request stage contains both required field sets: product name(s) and BGC sequence coordinates or Genbank accession.
- Extracted coordinates fall within the bounds of the referenced Genbank record and consistently locate the same cluster.
- The request is marked pending/staff-approved only after all required fields are present, mirroring the MIBiG approval gate.
- The two-stage record distinguishes the accession-request stage from the extended-annotation stage rather than merging them.

## Limitations

- Coordinates are 'preferably' rather than mandatorily Genbank-derived, so quality depends on the provenance of the sequence information supplied.
- The workflow cannot proceed past the accession request without MIBiG staff approval; the extended entry form is unavailable until then.
- MIBiG curation may fix or reject entries — MIBiG 2.0 curation involved 'removing duplicate entries' and retiring low-quality entries, so submitted records are subject to staff review.

## Evidence

- [intro] Request a MIBiG accession number by submitting product names and BGC sequence information (Genbank coordinates): [evidence span withheld — anchor status: not_found]
- [intro] approval gate and extended entry form: "After the request is approved by MIBiG staff, the workflow subsequently provides an extended entry form"
- [intro] two-stage workflow description: "a user starts by requesting a MIBiG accession number"
- [intro] validation technology for submitted annotations: "embed validation and dependency rules into the schema"
