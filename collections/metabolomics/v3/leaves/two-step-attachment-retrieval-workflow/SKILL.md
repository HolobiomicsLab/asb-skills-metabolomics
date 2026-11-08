---
name: two-step-attachment-retrieval-workflow
description: Reference for retrieving PubChem resources that are not directly addressable
  by CID alone — biologic depiction images (SVG) and annotation attachments (e.g.
  Mass Spectrometry) — via the two-step PUG-View /data/ → /image/ or /data/key/ pattern.
when_to_use_negative:
- When the target is a standard PUG-REST computed property (e.g. molecular formula,
  molecular weight) that is directly addressable by CID in a single call — PUG-REST
  is the better interface
- When multiple annotation headings must be retrieved in one request, since PUG-View
  does not support multi-heading or multi-identifier requests except via full-annotation
  retrieval
- When retrieving specialized tabular data served from other backend services that
  are not accessible through PUG-View at all
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: PUG-View
  role: Primary web-service interface used to issue both the intermediate /data/ JSON
    request and the final /image/ or /data/key/ call; handles the two-step resolution
    of internal identifiers to biologic depiction images and annotation attachments
- name: PUG-REST
  role: Contrasting single-request interface for computed compound properties; the
    two-step workflow is only needed when PUG-REST cannot address the resource directly,
    so PUG-REST is the fallback to prefer where applicable
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-019-0375-2
    title: 'PUG-View: programmatic access to chemical annotations integrated in PubChem'
  source_content_hash: sha256:d339ff0b99b8ca1d836a313c6e72e5636a57155d1a560d6a57b00aa0d17aff8c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PUG-REST
  - PUG-View
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# two-step-attachment-retrieval-workflow

## Summary

A PUG-View retrieval pattern for biologic depiction images and annotation attachments where the target resource is addressed by an internal identifier (e.g. biologic ID 138308 or attachment key 3413726_1) that must first be obtained from an intermediate /data/ JSON request before issuing the second call that returns the SVG image or attachment payload.

## When to use

Reference for retrieving PubChem resources that are not directly addressable by CID alone — biologic depiction images (SVG) and annotation attachments (e.g. Mass Spectrometry) — via the two-step PUG-View /data/ → /image/ or /data/key/ pattern.

## When NOT to use

- When the target is a standard PUG-REST computed property (e.g. molecular formula, molecular weight) that is directly addressable by CID in a single call — PUG-REST is the better interface
- When multiple annotation headings must be retrieved in one request, since PUG-View does not support multi-heading or multi-identifier requests except via full-annotation retrieval
- When retrieving specialized tabular data served from other backend services that are not accessible through PUG-View at all

## Inputs

- PubChem Compound Identifier (CID) of the biologic molecule or annotated compound record
- Heading string identifying the annotation section (e.g. 'Biologic Description', 'Mass Spectrometry')

## Outputs

- SVG depiction image for the biologic molecule (from /image/biologic/<id>/SVG)
- Annotation attachment payload with MIME type set in the Content-Type response header (from /data/key/<key>)
- Intermediate full-annotation JSON response containing the internal biologic identifier or attachment key

## How to apply

Issue an intermediate PUG-View request to /data/compound/<CID>/JSON with a heading query — ?heading=Biologic+Description for biologic depictions or ?heading=Mass+Spectrometry for annotation attachments — to obtain the internal identifier embedded in the section annotation data (e.g. biologic ID 138308 for CID 91848714, attachment key 3413726_1). For biologic depictions, the second call is /image/biologic/<id>/SVG, which returns an SVG image. For attachments, the second call is /data/key/<key> with no output-format segment, because the MIME type is delivered via the Content-Type response header rather than the URL path. The rationale is that biologic and attachment resources are sub-records within the full annotation JSON and cannot be addressed by CID in a single request, so a two-hop resolution is required.

## Related tools

- **PUG-View** (Primary web-service interface used to issue both the intermediate /data/ JSON request and the final /image/ or /data/key/ call; handles the two-step resolution of internal identifiers to biologic depiction images and annotation attachments)
- **PUG-REST** (Contrasting single-request interface for computed compound properties; the two-step workflow is only needed when PUG-REST cannot address the resource directly, so PUG-REST is the fallback to prefer where applicable)

## Examples

```
curl 'https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/data/compound/91848714/JSON?heading=Biologic+Description'
```

## Evaluation signals

- The intermediate /data/ JSON response must contain an internal biologic identifier (e.g. 138308 for CID 91848714) or attachment key (e.g. 3413726_1) embedded in the section annotation data — if absent, the heading query or CID is wrong
- The /image/biologic/<id>/SVG call must return an SVG image; any non-image content-type indicates a failed resolution
- The /data/key/<key> call must return no output-format segment in the URL and a valid MIME type in the Content-Type response header
- Requests must be constrained to one record identifier per call — PUG-View cannot accept multiple input record identifiers in a single request
- Only annotation data under a single heading can be retrieved per intermediate request unless the full annotation JSON is requested

## Limitations

- A PUG-View request cannot take multiple input record identifiers
- Annotation data under multiple headings for a given compound cannot be retrieved with a single request, unless the full annotation data retrieval is requested
- Some specialized (especially tabular) data come from other backend service(s) and are not currently accessible through PUG-View

## Evidence

- [other] Biologic depiction image retrieval via PUG-View is a two-step process: [evidence span withheld — anchor status: not_found]
- [other] Two-step variants: [evidence span withheld — anchor status: not_found]
- [other] Attachment key retrieval: [evidence span withheld — anchor status: not_found]
- [discussion] Single-identifier constraint: "a PUG-View request cannot take multiple input record identifiers"
- [discussion] Multi-heading constraint: "annotation data under multiple headings for a given compound cannot be retrieved with a single request, unless the full annotation data retrieval is requested"
- [intro] PUG-View purpose: "which serves information needed to render interactive web pages but that also allows one to programmatically access the chemical annotations and summary information in PubChem"
