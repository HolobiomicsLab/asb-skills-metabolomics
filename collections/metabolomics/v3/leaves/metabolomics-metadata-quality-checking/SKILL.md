---
name: metabolomics-metadata-quality-checking
description: Use when you have downloaded JSON project documents from the PoDP Zenodo
  archive and need to verify, per project, that each document conforms to the community-defined
  project format before downstream data mining or linking analysis.
when_to_use_negative:
- The input is raw metabolomics or genomics data (mzML, FASTA, genome accessions)
  rather than a curated PoDP project JSON document — the PoDP 'does not store any
  metabolomics or genomics datasets' itself
- You need to validate submitted data against MassIVE or MetaboLights, not against
  the PoDP project schema — this skill only checks the paired-data-form project JSON
  format
- The goal is content/semantic quality of free-text 'Other' fields, which by design
  are not machine-readable and cannot be schema-checked
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3173
tools:
- name: Paired Omics Data Platform
  role: Source of the community-curated project documents being validated; its paired-data-form
    repository publishes the defining JSON schema
  repo: https://github.com/iomega/paired-data-form
- name: Zenodo
  role: Hosts the regularly archived JSON project dataset that constitutes the validation
    input corpus
- name: paired-data-form
  role: Repository supplying app/public/schema.json — the web application renders
    the submission form from this JSON schema, making it the format specification
    for project documents
  repo: https://github.com/iomega/paired-data-form
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41589-020-00724-z
    title: A community resource for paired genomic and metabolomic data mining
  source_content_hash: sha256:2c19d2bae41061137a659f0bf4fd7e17e632b29c4a7e8a7d74c896a8b6849869
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/paired-data-form
  tools:
  - Paired Omics Data Platform
  - Zenodo
  - paired-data-form
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# metabolomics-metadata-quality-checking

## Summary

Validates PoDP paired-omics project documents — JSON files archived monthly to Zenodo by the paired-data-form platform — against the published JSON schema at app/public/schema.json in the iomega/paired-data-form repository, flagging structural or field-constraint violations per project.

## When to use

Use when you have downloaded JSON project documents from the PoDP Zenodo archive and need to verify, per project, that each document conforms to the community-defined project format before downstream data mining or linking analysis.

## When NOT to use

- The input is raw metabolomics or genomics data (mzML, FASTA, genome accessions) rather than a curated PoDP project JSON document — the PoDP 'does not store any metabolomics or genomics datasets' itself
- You need to validate submitted data against MassIVE or MetaboLights, not against the PoDP project schema — this skill only checks the paired-data-form project JSON format
- The goal is content/semantic quality of free-text 'Other' fields, which by design are not machine-readable and cannot be schema-checked

## Inputs

- JSON documents of PoDP paired-omics projects downloaded from the Zenodo archive
- JSON schema file app/public/schema.json from github.com/iomega/paired-data-form

## Outputs

- validation_report.csv with one row per project and its pass/fail validation status plus error details for failures
- Per-project validation error listings for non-conforming documents

## How to apply

Download the project JSON documents from the PoDP Zenodo archive, which is populated by the paired-data-form web service (each project is stored as a file on disk and uploaded to Zenodo monthly). Fetch the authoritative JSON schema from app/public/schema.json in the paired-data-form repository — this same schema is what the React web application renders the submission form from, so it defines the canonical field set (project identifiers, MassIVE/MetaboLights experiment accessions, genome labels, metadata labels, MS/MS URLs). For each project JSON, run a schema validation library such as Python's jsonschema against the schema and capture pass/fail status plus detailed validation errors. Write one row per project into validation_report.csv; a project is marked pass only when it conforms to all schema constraints in app/public/schema.json. Inspect failures for known structural weak points, e.g. free text entered in 'Other' boxes, which the paper notes is inherently not machine-readable.

## Related tools

- **Paired Omics Data Platform** (Source of the community-curated project documents being validated; its paired-data-form repository publishes the defining JSON schema) — https://github.com/iomega/paired-data-form
- **Zenodo** (Hosts the regularly archived JSON project dataset that constitutes the validation input corpus)
- **paired-data-form** (Repository supplying app/public/schema.json — the web application renders the submission form from this JSON schema, making it the format specification for project documents) — https://github.com/iomega/paired-data-form

## Examples

```
import json, jsonschema; jsonschema.validate(json.load(open('project.json')), json.load(open('schema.json')))
```

## Evaluation signals

- validation_report.csv contains exactly one row per downloaded project JSON — row count matches the number of archived project documents
- Every project is marked pass only when jsonschema reports zero violations against app/public/schema.json; any 'pass' with reported errors is a correctness failure
- Failed projects carry error details (failing field/constraint) sufficient to locate the offending element in the JSON document
- The schema used for validation is byte-identical to app/public/schema.json in the paired-data-form version that generated the archived projects (schema and data versions should correspond)

## Limitations

- Schema validation only enforces structure and field constraints; free text entered in 'Other' boxes is inherently not machine-readable, so schema-passing documents may still contain non-machine-readable content
- The PoDP stores no raw metabolomics or genomics datasets — a schema-conformant project says nothing about the quality of the underlying MS/genome data it links to
- Zenodo archives are updated monthly by the paired-data-form web service, so validation results reflect a snapshot at a given deposition

## Evidence

- [readme] The [JSON schema (app/public/schema.json)](app/public/schema.json) describes the format of an project.: "The [JSON schema (app/public/schema.json)](app/public/schema.json) describes the format of an project."
- [abstract] These community-curated data are regularly archived to a Zenodo dataset and made available for download in JSON format: "These community-curated data are regularly archived to a Zenodo dataset and made available for download in JSON format"
- [readme] The web service stores each project as a file on disk.: [evidence span withheld — anchor status: external_doc]
