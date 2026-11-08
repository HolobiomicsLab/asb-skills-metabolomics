---
name: rest-api-parameter-documentation
description: Use when a paper documents a REST API (e.g. PUG-View) with concrete example
  request URLs in its Construction And Content section, and you must reconstruct machine-readable
  URL templates, identifier-type constraints, and output-format rules.
when_to_use_negative:
- The goal is retrieving annotation data itself rather than documenting request syntax
  — PUG-View cannot take multiple input record identifiers or multiple headings in
  one request, so bulk retrieval needs separate full-retrieval requests
- The data of interest are specialized (especially tabular) data served by other backend
  services and not accessible through PUG-View
- The target is individual computed compound properties, for which PUG-REST is the
  better-suited interface
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: PUG-View
  role: REST-style web service whose special-report request variants and URL syntax
    are being documented
- name: PUG-REST
  role: Complementary interface noted in the documentation for retrieving all compound
    neighbors and individual computed properties not covered by the special reports
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-019-0375-2
    title: 'PUG-View: programmatic access to chemical annotations integrated in PubChem'
  source_content_hash: sha256:b027aee9bebc50a0691cc789982c297ffa7c3cd53f355da719f829a917d702e7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PUG-REST
  - PUG-View
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# rest-api-parameter-documentation

## Summary

Documents a REST-style web service's request grammar by extracting the paper's concrete example request URLs (e.g. PUG-View special-report forms) and generalizing them into parameterized URL templates with required parameters, output formats, and multi-step flags.

## When to use

Use when a paper documents a REST API (e.g. PUG-View) with concrete example request URLs in its Construction And Content section, and you must reconstruct machine-readable URL templates, identifier-type constraints, and output-format rules.

## When NOT to use

- The goal is retrieving annotation data itself rather than documenting request syntax — PUG-View cannot take multiple input record identifiers or multiple headings in one request, so bulk retrieval needs separate full-retrieval requests
- The data of interest are specialized (especially tabular) data served by other backend services and not accessible through PUG-View
- The target is individual computed compound properties, for which PUG-REST is the better-suited interface

## Inputs

- Paper text (Construction And Content section) containing example request URLs
- Concrete example request URLs per special-report variant (categories, neighbors, literature, structure, image, key, qr, linkout)
- Record identifier examples (e.g. CID 91848714, internal biologic identifier 138308, attachment key 3413726_1)

## Outputs

- pug_view_special_reports.json — a single JSON object keyed by the eight ⟨ANNOTATION⟩ values, each with url_template, required_parameters, output_format, and multi_step fields

## How to apply

Parse the paper's Construction And Content section and extract every concrete example request URL (e.g. /rest/pug_view/categories/compound/24/JSON, /rest/pug_view/neighbors/compound/60823/JSON, /rest/pug_view/literature/compound/2244/JSON, /rest/pug_view/structure/compound/123631/JSON, /rest/pug_view/linkout/compound/2244/JSON). Generalize each example into a URL template with placeholders for the record identifier, recording which identifier type each variant takes (CID for compound-based variants; NCBI Gene ID / protein accession handled only via the data/ record form). For two-step variants, document the intermediate call explicitly: biologic depiction retrieval first requests /data/compound/<CID>/JSON?heading=Biologic+Description to obtain the internal biologic identifier (e.g. 138308 for CID 91848714) before /image/biologic/<id>/SVG, and annotation attachments first request /data/compound/<CID>/JSON?heading=Mass+Spectrometry to obtain the attachment key (e.g. 3413726_1) before /data/key/<key>, which takes NO output-format segment because the mime type comes from the Content-Type header. Record per-variant format constraints: SVG output for biologic images and QR codes (with long/short QR variants plus the LCSS-targeted form /qr/long/compound/702/LCSS/SVG), and note that the neighbors special report returns only neighbors carrying pre-defined annotation types while PUG-REST is required for all neighbors. Assemble all eight variants into a single JSON object keyed by the ⟨ANNOTATION⟩ value (categories, neighbors, literature, structure, image, key, qr, linkout) with url_template, required_parameters, output_format, and multi_step fields.

## Related tools

- **PUG-View** (REST-style web service whose special-report request variants and URL syntax are being documented)
- **PUG-REST** (Complementary interface noted in the documentation for retrieving all compound neighbors and individual computed properties not covered by the special reports)

## Examples

```
curl https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/categories/compound/24/JSON
```

## Evaluation signals

- Output file is valid JSON containing all eight variant keys: categories, neighbors, literature, structure, image, key, qr, linkout
- Every variant has a populated url_template and required_parameters field matching the paper's Construction And Content example requests (e.g. /rest/pug_view/categories/compound/24/JSON)
- Two-step variants carry the intermediate call: heading-filtered /data/compound/<CID>/JSON?heading=Biologic+Description returning internal biologic identifier 138308 for CID 91848714, and ?heading=Mass+Spectrometry returning attachment key 3413726_1
- The /data/key/<key> variant is recorded WITHOUT an output-format segment, with a note that the mime type comes from the Content-Type header
- Identifier-type constraints are correct: compound-based variants take CID; NCBI Gene ID / protein accession are only handled via the data/ record form

## Limitations

- A PUG-View request cannot take multiple input record identifiers
- Annotation data under multiple headings for a given compound cannot be retrieved with a single request, unless the full annotation data retrieval is requested
- Some specialized (especially tabular) data come from other backend service(s) and are not currently accessible through PUG-View
- The neighbors special report returns only neighbors carrying pre-defined annotation types; PUG-REST is required for all neighbors

## Evidence

- [abstract] PUG-View is a REST-style annotation service whose URL syntax the paper documents: "One of these is PUG-View, which is a Representational State Transfer (REST)-style web service interface specialized for accessing annotation data contained in PubChem."
- [intro] The paper explicitly documents the request-URL syntax and scope of PUG-View: "The present paper will describe important aspects of PUG-View, including the scope of data accessible through PUG-View, the syntax for formulation a PUG-View request URL"
- [discussion] PUG-View is used as a backend service to programmatically access annotation content: "PUG-View allows one to programmatically access this annotation content in PubChem and is also used as a backend service"
- [discussion] Single-request constraint: no multiple record identifiers per request: "a PUG-View request cannot take multiple input record identifiers"
- [discussion] Heading retrieval constraint: one heading per request unless full retrieval: "annotation data under multiple headings for a given compound cannot be retrieved with a single request, unless the full annotation data retrieval is requested"
- [discussion] Some tabular backend data are inaccessible via PUG-View: "some specialized (especially tabular) data come from other backend service(s) and are not currently accessible through PUG-View"
