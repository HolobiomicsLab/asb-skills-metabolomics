---
name: pubchem-annotation-retrieval
description: Use when you need programmatic access to the full annotation content
  of a PubChem record — e.g. a Compound Summary for CID 1983, a Substance record SID
  46506142, or an Assay AID 1259416 — as JSON.
when_to_use_negative:
- You need tabular or computed per-compound properties for many CIDs at once — PUG-REST
  is better suited for those individual computed compound properties, and some specialized
  (especially tabular) data are not accessible through PUG-View at all.
- You need annotation data under multiple headings for a given compound in one request
  — that requires requesting the full annotation data retrieval, not a targeted heading
  query.
- You have multiple record identifiers to fetch — PUG-View cannot take multiple input
  record identifiers in a single request.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: PUG-View
  role: Primary REST web-service interface used to issue the pug_view/data or pug_view/index
    GET requests that return annotation JSON for a PubChem record
  repo: https://pubchem.ncbi.nlm.nih.gov
- name: PUG-REST
  role: Alternative PubChem programmatic route, better for retrieving specific individual
    computed compound properties; contrasted with PUG-View when choosing an interface
  repo: https://pubchem.ncbi.nlm.nih.gov
- name: PUG-SOAP
  role: One of the other PubChem programmatic access routes mentioned alongside PUG-View;
    not the interface for this skill
- name: E-Utilities
  role: NCBI programmatic access route mentioned as an alternative; not used for PUG-View
    annotation retrieval
- name: PUG
  role: Power User Gateway, an older PubChem programmatic access route; not the interface
    for this skill
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1186/s13321-019-0375-2
    title: 'PUG-View: programmatic access to chemical annotations integrated in PubChem'
  source_content_hash: sha256:04fa6a37277c3868f6d5d07bc7cbc71690a15b888755cc561aa345bc095ab1ec
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - E-Utilities
  - PUG
  - PUG-REST
  - PUG-SOAP
  - PUG-View
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# pubchem-annotation-retrieval

## Summary

Retrieve chemical annotations and summary information for a PubChem record (compound, substance, or assay) by issuing PUG-View REST requests against the pubchem.ncbi.nlm.nih.gov/pug_view endpoint, saving structured JSON annotation or Table-of-Contents responses.

## When to use

Use when you need programmatic access to the full annotation content of a PubChem record — e.g. a Compound Summary for CID 1983, a Substance record SID 46506142, or an Assay AID 1259416 — as JSON.

## When NOT to use

- You need tabular or computed per-compound properties for many CIDs at once — PUG-REST is better suited for those individual computed compound properties, and some specialized (especially tabular) data are not accessible through PUG-View at all.
- You need annotation data under multiple headings for a given compound in one request — that requires requesting the full annotation data retrieval, not a targeted heading query.
- You have multiple record identifiers to fetch — PUG-View cannot take multiple input record identifiers in a single request.

## Inputs

- Single PubChem record identifier (CID, SID, or AID) plus database type (compound, substance, or assay)
- Desired retrieval mode: full annotations ('data') or Table-of-Contents index ('index')
- Output format choice, e.g. JSON

## Outputs

- Valid JSON file containing the 'Record' node with full annotation content for the requested identifier (e.g. annotations_cid1983.json)
- Valid JSON file listing available (sub)headings for the record's Summary page (e.g. compound_toc.json)

## How to apply

Construct a PUG-View URL embedding three pieces of information: the annotation type ('data' for full record annotations or 'index' for the Table of Contents), the record specification as database type plus a SINGLE identifier (e.g. compound/1983, substance/46506142, assay/1259416), and the output format ('JSON'). Issue one HTTP GET per record against https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/{data|index}/{compound|substance|assay}/{identifier}/JSON and save each response as a JSON file (annotations_cid1983.json, compound_toc.json, etc.). The 'index' variant returns the available (sub)headings for the record's Summary page without the full annotation content, letting you inspect scope before committing to a heavy full-data request. Because a PUG-View request cannot take multiple input record identifiers, loop over identifiers one at a time rather than batching. Validation: each saved file must parse as valid JSON, contain the 'Record' node for the requested identifier, and match the content served on the corresponding Summary/Record page; error responses (e.g. unknown record) count as failures.

## Related tools

- **PUG-View** (Primary REST web-service interface used to issue the pug_view/data or pug_view/index GET requests that return annotation JSON for a PubChem record) — https://pubchem.ncbi.nlm.nih.gov
- **PUG-REST** (Alternative PubChem programmatic route, better for retrieving specific individual computed compound properties; contrasted with PUG-View when choosing an interface) — https://pubchem.ncbi.nlm.nih.gov
- **PUG-SOAP** (One of the other PubChem programmatic access routes mentioned alongside PUG-View; not the interface for this skill)
- **E-Utilities** (NCBI programmatic access route mentioned as an alternative; not used for PUG-View annotation retrieval)
- **PUG** (Power User Gateway, an older PubChem programmatic access route; not the interface for this skill)

## Examples

```
requests.get('https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/index/compound/1983/JSON')
```

## Evaluation signals

- Each returned JSON parses without error and contains the 'Record' node for the requested CID/SID/AID.
- The content of the retrieved annotation matches what is served on the corresponding Summary/Record web page for the same identifier.
- An 'index' request returns the available (sub)headings without the full annotation data content — confirming you used the correct endpoint variant.
- Requests that target a nonexistent identifier return an error message rather than silently succeeding; such responses are reported as failures.
- Exactly one identifier is embedded per request URL; no attempt to batch multiple CIDs/SIDs/AIDs in a single PUG-View call.

## Limitations

- A PUG-View request cannot take multiple input record identifiers — each record must be requested individually.
- Some specialized (especially tabular) data come from other backend services and are not currently accessible through PUG-View.
- Annotation data under multiple headings for a given compound cannot be retrieved with a single request, unless the full annotation data retrieval is requested.

## Evidence

- [results] full annotation content: [evidence span withheld — anchor status: not_found]
- [results] index request returns Table of Contents: [evidence span withheld — anchor status: not_found]
- [abstract] PUG-View scope: "One of these is PUG-View, which is a Representational State Transfer (REST)-style web service interface specialized for accessing annotation data contained in PubChem."
- [discussion] single identifier limitation: "a PUG-View request cannot take multiple input record identifiers"
- [discussion] comparison with PUG-REST: "while PUG-REST is best for accessing specific individual computed compound properties"
