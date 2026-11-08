---
name: compound-metadata-template-parsing
description: Use when a batch list of compounds must be imported into a PeakForest
  database from the supplementary compound-import template (Online Resource 2), where
  common name, InChI and InChIKey are mandatory per row.
when_to_use_negative:
- Input is raw spectral data (mzML/NMR FID) rather than a compound metadata template
  — PeakForest stores metadata related to raw data rather than raw data itself
- Compounds come from automatic extraction/clean-up of uncleaned data — PeakForest
  does not include data clean-up or extraction tools and requires users to supply
  completely-cleaned data following SOPs
- Only one or two compounds need adding — the manual import/curation interface in
  the PeakForest webapp is simpler than building a batch parsing module
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: PeakForest
  role: Target infrastructure whose chemical compound entity and batch compound-import
    module define the record schema and mandatory fields
  repo: https://github.com/peakforest/peakforest-webapp
- name: Java 8
  role: Language in which the parsing/validation module is implemented within the
    peakforest-webapp Maven project
- name: Maven
  role: Build/test tool used to run `mvn test` unit tests over the parsing module
- name: MySQL 5+
  role: Database backend into which the serialized compound records are ingested
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s11306-022-01899-3
    title: 'PeakForest: a multi-platform digital infrastructure for interoperable
      metabolite spectral data and metadata management'
  source_content_hash: sha256:ef0c59ff37e394e8c858a82f7d92baf96bdf1915c5cf1ac47c31048a3b503c32
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/peakforest/peakforest-webapp
  tools:
  - Java 8
  - Maven
  - MySQL 5+
  - PeakForest
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# compound-metadata-template-parsing

## Summary

Parses a structured compound-import template (XLSX) and maps each row to the chemical compound entity of the PeakForest data model, validating mandatory fields (common name, InChI, InChIKey) and emitting structured compound records for database ingestion.

## When to use

Use when a batch list of compounds must be imported into a PeakForest database from the supplementary compound-import template (Online Resource 2), where common name, InChI and InChIKey are mandatory per row.

## When NOT to use

- Input is raw spectral data (mzML/NMR FID) rather than a compound metadata template — PeakForest stores metadata related to raw data rather than raw data itself
- Compounds come from automatic extraction/clean-up of uncleaned data — PeakForest does not include data clean-up or extraction tools and requires users to supply completely-cleaned data following SOPs
- Only one or two compounds need adding — the manual import/curation interface in the PeakForest webapp is simpler than building a batch parsing module

## Inputs

- Compound-import template file (XLSX, Online Resource 2) with one compound per row
- PeakForest chemical compound entity definition from the PeakForest data model
- peakforest-webapp Maven project (Java 8, MySQL 5+)

## Outputs

- compound_records.json containing structured compound records for valid template rows
- Row-level validation error log for template rows missing mandatory fields (common name, InChI, InChIKey)
- Unit test results from `mvn test` covering the parsing module

## How to apply

Obtain the supplementary compound-import template (XLSX) and reference the chemical compound entity defined in the PeakForest data model, implemented as a Java 8 webapp built with Maven and backed by MySQL 5+. Load the XLSX and map each column header to the corresponding chemical compound metadata field of the PeakForest compound entity. Implement the parsing/validation module inside the peakforest-webapp codebase, checking that the mandatory fields (common name, InChI, InChIKey) are present in every template row and collecting row-level validation errors for records that fail rather than aborting the batch. Serialize each valid row into a structured compound record shaped for PeakForest database ingestion and write all records to compound_records.json. Run `mvn test` in the peakforest-webapp project to execute the module's unit tests, confirming every template row yields either a well-formed compound record or an explicit, logged validation error.

## Related tools

- **PeakForest** (Target infrastructure whose chemical compound entity and batch compound-import module define the record schema and mandatory fields) — https://github.com/peakforest/peakforest-webapp
- **Java 8** (Language in which the parsing/validation module is implemented within the peakforest-webapp Maven project)
- **Maven** (Build/test tool used to run `mvn test` unit tests over the parsing module)
- **MySQL 5+** (Database backend into which the serialized compound records are ingested)

## Examples

```
git clone git@services.pfem.clermont.inrae.fr:peakforest/peakforest-webapp.git && mvn test
```

## Evaluation signals

- Every template row produces either a well-formed compound record or an explicit, logged validation error — no silent row drops
- All records in compound_records.json carry non-empty mandatory fields: common name, InChI, and InChIKey
- Column-header-to-metadata-field mapping covers every column of the imported XLSX template with no unmapped headers
- `mvn test` in peakforest-webapp passes for the parsing module's unit tests
- Serialized records conform to the chemical compound entity shape of the PeakForest data model and load into the MySQL-backed database

## Limitations

- A minimum number of compounds and spectra need to be imported before local PeakForest databases are fully functional
- PeakForest does not include data clean-up or extraction tools per se — metadata must be manually cleaned before parsing, following SOPs edited by the PeakForest team
- Installation and module development require personnel with computing or bioinformatics skills

## Evidence

- [other] The batch compound-import module creates associated compound cards based on minimal compound information found in the imported file: [evidence span withheld — anchor status: not_found]
- [methods] PeakForest is a Java Web application (version 8): "PeakForest is a Java Web application (version 8) and has been designed as an API’s toolbox (Online Resource 1)."
- [readme] Run `mvn test` command to launch all unit test.: "Run `mvn test` command to launch all unit test.\"
- [methods] Users can import compounds and spectral data manually: "Users can import compounds and spectral data manually extracted from raw data, curate information and exploit this knowledge via data export or PeakForest Web services connected tools"
- [methods] PeakForest stores metadata related to raw data rather than raw data itself: "PeakForest stores metadata related to raw data rather than raw data itself to minimize storage space usage."
