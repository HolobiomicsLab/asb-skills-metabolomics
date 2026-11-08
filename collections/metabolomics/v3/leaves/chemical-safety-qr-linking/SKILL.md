---
name: chemical-safety-qr-linking
description: Use when you need programmatic SVG QR codes linking a compound (CID)
  to its PubChem safety summary, or to retrieve biologic depiction images and annotation
  attachments via PUG-View special reports.
when_to_use_negative:
- When you need specific individual computed compound properties — PUG-REST, not PUG-View,
  is the interface best suited for that.
- When you need all structural neighbors of a compound — the PUG-View neighbors form
  returns only neighbors with pre-defined annotation types; use PUG-REST instead.
- When a request must accept multiple record identifiers or retrieve annotations under
  multiple headings at once — PUG-View cannot take multiple input record identifiers
  and multiple headings require full-annotation retrieval.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0605
tools:
- name: PUG-View
  role: Primary REST-style interface used to issue the special-report requests (qr,
    image, key, linkout, categories, neighbors, literature, structure) that surface
    annotations and safety-summary QR codes
- name: PUG-REST
  role: Complementary interface required to enumerate all compound neighbors (PUG-View's
    neighbors form covers only pre-defined annotation types) and for individual computed
    compound properties
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-019-0375-2
    title: 'PUG-View: programmatic access to chemical annotations integrated in PubChem'
  source_content_hash: sha256:0a171adefe31c01db9028bb05ff53a7e8d57c8f43fadf50b3343ec98a921c56c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PUG-REST
  - PUG-View
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# chemical-safety-qr-linking

## Summary

Reconstructs and applies the PUG-View special-report request variants — QR codes that encode a compound's link to its PubChem chemical safety summary (LCSS), biologic depiction images, and annotation attachment retrieval — as executable URL templates.

## When to use

Use when you need programmatic SVG QR codes linking a compound (CID) to its PubChem safety summary, or to retrieve biologic depiction images and annotation attachments via PUG-View special reports.

## When NOT to use

- When you need specific individual computed compound properties — PUG-REST, not PUG-View, is the interface best suited for that.
- When you need all structural neighbors of a compound — the PUG-View neighbors form returns only neighbors with pre-defined annotation types; use PUG-REST instead.
- When a request must accept multiple record identifiers or retrieve annotations under multiple headings at once — PUG-View cannot take multiple input record identifiers and multiple headings require full-annotation retrieval.

## Inputs

- PubChem Compound Identifier (CID) for compound-based special-report variants (e.g., 702 for LCSS QR, 91848714 for biologic depiction, 2244 for linkout/literature)
- PUG-View /data/ JSON response for a heading (e.g., heading=Biologic+Description or heading=Mass+Spectrometry) from which the internal biologic identifier or attachment key is parsed
- Internal biologic identifier (e.g., 138308) or attachment key (e.g., 3413726_1) produced by the first step of a two-step variant

## Outputs

- SVG QR code linking a compound to its PubChem chemical safety summary (short, long, and LCSS-targeted /qr/long/compound/702/LCSS/SVG forms)
- SVG biologic depiction image via /image/biologic/<id>/SVG
- Annotation attachment content served via /data/key/<key> with mime type taken from the Content-Type header
- pug_view_special_reports.json: one object keyed by ANNOTATION (categories, neighbors, literature, structure, image, key, qr, linkout) with url_template, required_parameters, output_format, and multi_step fields

## How to apply

Parse the paper's Construction And Content section to extract the concrete example request URLs for each special-report variant (e.g., /rest/pug_view/categories/compound/24/JSON, /rest/pug_view/neighbors/compound/60823/JSON, /rest/pug_view/structure/compound/123631/JSON, /rest/pug_view/linkout/compound/2244/JSON) and generalize each into a URL template with placeholders, recording that compound-based variants take a CID. For safety QR linking, request SVG output in short or long form (e.g., /qr/long/compound/702/LCSS/SVG for the LCSS-targeted variant); SVG is the required output format for QR codes and biologic images alike. For two-step variants, first request /data/compound/<CID>/JSON?heading=Biologic+Description to obtain the internal biologic identifier (e.g., 138308 for CID 91848714), then call /image/biologic/<id>/SVG; analogously, request ?heading=Mass+Spectrometry to obtain an attachment key (e.g., 3413726_1) and then /data/key/<key> with NO output-format segment, because the mime type is conveyed by the Content-Type header. Note that the neighbors form returns only neighbors carrying pre-defined annotation types, so PUG-REST is required to retrieve all neighbors. Assemble all eight variants (categories, neighbors, literature, structure, image, key, qr, linkout) into a single JSON object keyed by the ANNOTATION value, each with url_template, required_parameters, output_format, and multi_step flags, written to pug_view_special_reports.json.

## Related tools

- **PUG-View** (Primary REST-style interface used to issue the special-report requests (qr, image, key, linkout, categories, neighbors, literature, structure) that surface annotations and safety-summary QR codes)
- **PUG-REST** (Complementary interface required to enumerate all compound neighbors (PUG-View's neighbors form covers only pre-defined annotation types) and for individual computed compound properties)

## Examples

```
curl "https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/qr/long/compound/702/LCSS/SVG" -o safety_qr.svg
```

## Evaluation signals

- pug_view_special_reports.json parses as valid JSON and contains all eight variant keys: categories, neighbors, literature, structure, image, key, qr, linkout, each with url_template and required_parameters populated from the paper's Construction And Content examples.
- QR requests return SVG output (checked via Content-Type), matching the recorded output_format for the qr variant, including the LCSS-targeted form /qr/long/compound/702/LCSS/SVG.
- The biologic-depiction two-step chain reproduces the documented intermediate value: heading=Biologic+Description for CID 91848714 yields internal identifier 138308, and /image/biologic/138308/SVG returns an SVG image.
- The attachment two-step chain yields key 3413726_1 from heading=Mass+Spectrometry, and /data/key/3413726_1 is issued without an output-format segment, with the mime type read from the Content-Type header.
- Identifier-type constraints hold: compound-based variants accept a CID, while NCBI Gene IDs / protein accessions are handled only via the data/ record form, never through these special reports.

## Limitations

- A PUG-View request cannot take multiple input record identifiers — one record per request.
- Annotation data under multiple headings for a given compound cannot be retrieved with a single request, unless the full annotation data retrieval is requested.
- Some specialized (especially tabular) data come from other backend service(s) and are not currently accessible through PUG-View.
- The neighbors special-report form returns only neighbors carrying pre-defined annotation types; complete neighbor listings require PUG-REST.

## Evidence

- [abstract] PUG-View, which is a Representational State Transfer (REST)-style web service interface specialized for accessing annotation data contained in PubChem: "One of these is PUG-View, which is a Representational State Transfer (REST)-style web service interface specialized for accessing annotation data contained in PubChem."
- [full_text] Biologic depiction two-step retrieval with internal identifier: [evidence span withheld — anchor status: not_found]
- [full_text] Neighbors form limitation vs PUG-REST: [evidence span withheld — anchor status: not_found]
- [discussion] No multiple input record identifiers: "a PUG-View request cannot take multiple input record identifiers"
- [discussion] Specialized tabular data not accessible: "some specialized (especially tabular) data come from other backend service(s) and are not currently accessible through PUG-View"
