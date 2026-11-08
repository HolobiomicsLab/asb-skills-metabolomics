---
name: spreadsheet-to-database-record-mapping
description: Use when a batch of compounds (or other entities) must be loaded from
  a supplied spreadsheet template into a structured database, and each row must be
  validated against mandatory metadata fields before ingestion.
when_to_use_negative:
- The input is already structured database records or API payloads rather than a spreadsheet
  template
- Only a single compound needs to be added — PeakForest supports manual import and
  curation of compounds via the web interface
edam_operation: http://edamontology.org/operation_1812
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: PeakForest
  role: Target infrastructure whose chemical compound data model defines the record
    schema and whose batch compound-import module consumes the mapped records
  repo: https://github.com/peakforest/peakforest-webapp
- name: Java 8
  role: Implementation language for the parsing/validation module inside the PeakForest
    webapp codebase
- name: Maven
  role: 'Build and test tool: `mvn test` runs the unit tests validating every template
    row, `mvn clean install` builds the webapp'
  repo: https://github.com/peakforest/peakforest-webapp
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s11306-022-01899-3
    title: 'PeakForest: a multi-platform digital infrastructure for interoperable
      metabolite spectral data and metadata management'
  source_content_hash: sha256:ba87e9684f3c6c267bec8502b7a3c23e77dd613c7b790689843af0b26533da33
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/peakforest/peakforest-webapp
  tools:
  - Java 8
  - Maven
  - PeakForest
  edam_operation: http://edamontology.org/operation_1812
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# spreadsheet-to-database-record-mapping

## Summary

Maps rows of a tabular template file (XLSX) onto a target database entity — here, the chemical compound entity of the PeakForest data model — validating mandatory fields per row and serializing each valid row into a structured record ready for database ingestion.

## When to use

Use when a batch of compounds (or other entities) must be loaded from a supplied spreadsheet template into a structured database, and each row must be validated against mandatory metadata fields before ingestion.

## When NOT to use

- The input is already structured database records or API payloads rather than a spreadsheet template
- Only a single compound needs to be added — PeakForest supports manual import and curation of compounds via the web interface

## Inputs

- XLSX compound-import template (Online Resource 2) with one compound per row
- Chemical compound entity definition from the PeakForest data model

## Outputs

- compound_records. containing structured compound records for PeakForest database ingestion
- Row-level validation error log for rows missing mandatory fields (common name, InChI, InChIKey)

## How to apply

Obtain the supplementary compound-import template (XLSX, Online Resource 2) and the chemical compound entity definition from the PeakForest data model. Load the workbook and map each column header to the corresponding chemical compound metadata field of the PeakForest compound entity. Implement a parsing/validation module in Java 8 within the Maven-built PeakForest webapp codebase; enforce the mandatory fields — common name, InChI, and InChIKey — on every template row, collecting row-level validation errors for records that fail rather than aborting the whole batch. Serialize each valid row into a structured compound record shaped for PeakForest database ingestion and write all records to compound_records.. Run `mvn test` to execute the module's unit tests and confirm every template row yields either a well-formed compound record or an explicit, logged validation error.

## Related tools

- **PeakForest** (Target infrastructure whose chemical compound data model defines the record schema and whose batch compound-import module consumes the mapped records) — https://github.com/peakforest/peakforest-webapp
- **Java 8** (Implementation language for the parsing/validation module inside the PeakForest webapp codebase)
- **Maven** (Build and test tool: `mvn test` runs the unit tests validating every template row, `mvn clean install` builds the webapp) — https://github.com/peakforest/peakforest-webapp

## Examples

```
mvn test -Dtest=CompoundImportModuleTest && mvn clean install  # produces peakforest-webapp-X.war after validating template rows into compound_records.
```

## Evaluation signals

- Every template row yields either a well-formed compound record in compound_records. or an explicit, logged validation error — no rows silently dropped
- Every serialized compound record contains non-empty common name, InChI, and InChIKey (the three mandatory fields)
- `mvn test` passes, covering the module's unit tests over the template rows
- Column-to-field mapping covers all headers present in the Online Resource 2 XLSX template with no unmapped columns

## Limitations

- Rows missing any mandatory field (common name, InChI, InChIKey) cannot be imported and are reported as validation errors rather than partially loaded
- PeakForest does not include data clean-up or extraction tools — template metadata must be complete and cleaned by the user before import
- A minimum number of compounds (and spectra) must be imported before a local PeakForest database is fully functional

## Evidence

- [supplementary] The batch compound-import module creates associated compound cards based on minimal compound information found in the imported file, where common name, InChI and InChIKey are mandatory fields: [evidence span withheld — anchor status: not_found]
- [methods] Users can import compounds and spectral data manually extracted from raw data, curate information and exploit this knowledge via data export or PeakForest Web services: "Users can import compounds and spectral data manually extracted from raw data, curate information and exploit this knowledge via data export or PeakForest Web services connected tools"
- [readme] Run `mvn test` command to launch all unit test.: "Run `mvn test` command to launch all unit test."
- [discussion] PeakForest does not include data clean-up or extraction tools per se and requires metadata to be manually entered in the database: "PeakForest does not include data clean-up or extraction tools per se and requires metadata to be manually entered in the database"
