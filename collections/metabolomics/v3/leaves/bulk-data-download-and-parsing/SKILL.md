---
name: bulk-data-download-and-parsing
description: Use when you need the set of deposited PoDP project JSON documents (e.g.
  70 paired datasets with 4,853 MS samples) and must check each document's structure
  against the platform's official schema before downstream analysis.
when_to_use_negative:
- You need the raw metabolomics (mzML/MS2) or genomics datasets themselves — the PoDP
  'does not store any metabolomics or genomics datasets, but captures metadata' linking
  to MassIVE, MetaboLights, GenBank, and JGI identifiers
- You need to create or edit a paired-omics project via the web form rather than programmatically
  consume deposited documents
- Your target is free-text 'Other' metadata fields, which are 'inherently not machine-readable'
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0219
tools:
- name: Zenodo
  role: Source of the regularly archived bulk download of community-curated PoDP project
    JSON documents
- name: paired-data-form
  role: Repository hosting app/public/schema.json, the artifact specifying the required
    format of a deposited project document
  repo: https://github.com/iomega/paired-data-form
- name: Paired Omics Data Platform
  role: The community resource whose project documents are downloaded, parsed, and
    schema-validated
  repo: https://github.com/iomega/paired-data-form
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41589-020-00724-z
    title: A community resource for paired genomic and metabolomic data mining
  source_content_hash: sha256:7fdc269ec49f8f4299338b9d49454b16a1a8ef939fadaa4f717674b77b87ba67
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/paired-data-form
  tools:
  - Paired Omics Data Platform
  - Zenodo
  - paired-data-form
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# bulk-data-download-and-parsing

## Summary

Downloads the community-curated Paired Omics Data Platform (PoDP) project documents, which are regularly archived to Zenodo in JSON format, and parses/validates them against the published JSON schema (app/public/schema.json) from the paired-data-form repository.

## When to use

Use when you need the set of deposited PoDP project JSON documents (e.g. 70 paired datasets with 4,853 MS samples) and must check each document's structure against the platform's official schema before downstream analysis.

## When NOT to use

- You need the raw metabolomics (mzML/MS2) or genomics datasets themselves — the PoDP 'does not store any metabolomics or genomics datasets, but captures metadata' linking to MassIVE, MetaboLights, GenBank, and JGI identifiers
- You need to create or edit a paired-omics project via the web form rather than programmatically consume deposited documents
- Your target is free-text 'Other' metadata fields, which are 'inherently not machine-readable'

## Inputs

- PoDP project JSON documents downloaded from the Zenodo archive (Zenodo DOI 10.5281/zenodo.6091663)
- Published JSON schema file app/public/schema.json from the iomega/paired-data-form GitHub repository

## Outputs

- Per-project validation results table validation_report.csv with one row per project and pass/fail status
- Parsed project JSON objects (project identifiers, MassIVE/MetaboLights accessions, genome labels, MS/MS URLs, MIBiG links) for downstream analysis
- Error details (validation messages) for schema-conformance failures

## How to apply

Download the JSON project documents from the regularly updated Zenodo archive of the PoDP. Load the published schema file app/public/schema.json from the iomega/paired-data-form repository — the README states this schema 'describes the format of an project'. For each project JSON, run a schema validation library such as Python's jsonschema, checking document structure and field constraints (e.g. MassIVE accession or MetaboLights study identifier fields, public genome identifiers, genome–metabolome and BGC–MS/MS links). Record per-project pass/fail status with error details for failures, and write one row per project to a validation_report.csv. A project is marked pass only when its JSON conforms to all schema constraints in app/public/schema.json.

## Related tools

- **Zenodo** (Source of the regularly archived bulk download of community-curated PoDP project JSON documents)
- **paired-data-form** (Repository hosting app/public/schema.json, the artifact specifying the required format of a deposited project document) — https://github.com/iomega/paired-data-form
- **Paired Omics Data Platform** (The community resource whose project documents are downloaded, parsed, and schema-validated) — https://github.com/iomega/paired-data-form

## Examples

```
import json, jsonschema; schema=json.load(open('schema.json')); doc=json.load(open('project.json')); jsonschema.validate(doc, schema)
```

## Evaluation signals

- validation_report.csv contains exactly one row per downloaded project JSON with a pass/fail status
- A project is marked pass only if jsonschema validation against app/public/schema.json reports zero constraint errors
- Count of successfully parsed projects is consistent with the number of JSON files present in the Zenodo download
- Parsed fields resolve to public identifiers (MassIVE accessions, MetaboLights study identifiers, NCBI GenBank/JGI genome identifiers) as required by the schema
- Failed validations carry non-empty error details pinpointing the violated schema constraint

## Limitations

- Free text entered in 'Other' boxes is 'inherently not machine-readable but gives an option for customization by the user', so schema validation cannot capture that content
- The schema covers project metadata only — the platform 'does not store any metabolomics or genomics datasets', so validation says nothing about the linked raw data in MassIVE/MetaboLights/GenBank
- The Zenodo archive is refreshed on a monthly upload schedule (the web service uploads 'the projects to Zenodo each month'), so downloads may lag the live platform

## Evidence

- [abstract] Zenodo archive format: "These community-curated data are regularly archived to a Zenodo dataset and made available for download in JSON format"
- [readme] Schema location and purpose: "The [JSON schema (app/public/schema.json)](app/public/schema.json) describes the format of an project."
- [readme] Projects stored as files: [evidence span withheld — anchor status: external_doc]
- [abstract] Project structure and identifiers: "The metadata is organized in projects that can consist of multiple related experiments, identified by their MassIVE accession or MetaboLights study identifier"
