---
name: peakforest-data-model-entity-mapping
description: Use when a batch compound list must be loaded into a local PeakForest
  database and each template row's metadata (common name, InChI, InChIKey, etc.) must
  be mapped to the chemical compound entity before validation and database ingestion.
when_to_use_negative:
- PeakForest does not include data clean-up or extraction tools, so do not use this
  mapping to clean or normalize dirty spectra/metadata — users must supply completely-cleaned
  data following the PeakForest team's SOPs
- The goal is storing raw spectral data rather than metabolite annotation report data
  and compound metadata; PeakForest is designed for the latter, not raw spectral data
  storage
- Only a handful of ad hoc compounds need manual entry — the manual 'Import compounds
  and spectral data' workflow via the Web interface is simpler than building a template-based
  batch import
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: PeakForest
  role: 'Target infrastructure: its data model''s chemical compound entity defines
    the fields that template columns are mapped onto, and its batch compound-import
    module consumes the resulting records'
  repo: https://github.com/peakforest/peakforest-webapp
- name: Java 8
  role: Implementation language for the parsing/validation module inside the PeakForest
    webapp (Java Web application, version 8)
- name: Maven 4+
  role: 'Build/test harness: `mvn test` runs the unit tests that verify each template
    row maps to a valid compound record or a logged validation error'
- name: MySQL 5+
  role: Backend database into which the mapped compound records are ingested
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s11306-022-01899-3
    title: 'PeakForest: a multi-platform digital infrastructure for interoperable
      metabolite spectral data and metadata management'
  source_content_hash: sha256:6e3ef27d5e315d8670cc642e67941a089f56a314f6f51a6e84c80da93907b73c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/peakforest/peakforest-webapp
  tools:
  - Java 8
  - Maven 4+
  - MySQL 5+
  - PeakForest
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# peakforest-data-model-entity-mapping

## Summary

Maps columns of a supplementary compound-import template (e.g. Online Resource 2 XLSX) onto the chemical compound entity fields of the PeakForest data model, so batch-imported compound lists can be validated and serialized for ingestion into the PeakForest MySQL-backed webapp.

## When to use

Use when a batch compound list must be loaded into a local PeakForest database and each template row's metadata (common name, InChI, InChIKey, etc.) must be mapped to the chemical compound entity before validation and database ingestion.

## When NOT to use

- PeakForest does not include data clean-up or extraction tools, so do not use this mapping to clean or normalize dirty spectra/metadata — users must supply completely-cleaned data following the PeakForest team's SOPs
- The goal is storing raw spectral data rather than metabolite annotation report data and compound metadata; PeakForest is designed for the latter, not raw spectral data storage
- Only a handful of ad hoc compounds need manual entry — the manual 'Import compounds and spectral data' workflow via the Web interface is simpler than building a template-based batch import

## Inputs

- Compound-import template XLSX file (supplementary template, e.g. Online Resource 2) with one row per compound
- PeakForest chemical compound entity definition from the PeakForest data model
- peakforest-webapp Maven project source tree (Java 8)

## Outputs

- compound_records.: structured compound records shaped for PeakForest database ingestion
- Row-level validation error log for template rows missing mandatory fields (common name, InChI, InChIKey)

## How to apply

Obtain the supplementary compound-import template (XLSX) and reference the chemical compound entity defined in the PeakForest data model; the batch import module creates compound cards from minimal compound information, with common name, InChI and InChIKey as mandatory fields. Load the XLSX and map each column header to the corresponding chemical compound metadata field of the PeakForest compound entity. Implement the parsing/validation logic in Java 8 within the Maven-built peakforest-webapp codebase, checking that every mandatory field is present in each row and collecting row-level validation errors for records that fail. Serialize each valid row into a structured compound record shaped for database ingestion (e.g. compound_records.), then run `mvn test` to confirm each template row yields either a well-formed compound record or an explicit, logged validation error.

## Related tools

- **PeakForest** (Target infrastructure: its data model's chemical compound entity defines the fields that template columns are mapped onto, and its batch compound-import module consumes the resulting records) — https://github.com/peakforest/peakforest-webapp
- **Java 8** (Implementation language for the parsing/validation module inside the PeakForest webapp (Java Web application, version 8))
- **Maven 4+** (Build/test harness: `mvn test` runs the unit tests that verify each template row maps to a valid compound record or a logged validation error)
- **MySQL 5+** (Backend database into which the mapped compound records are ingested)

## Examples

```
git clone git@services.pfem.clermont.inrae.fr:peakforest/peakforest-webapp.git && mvn test
```

## Evaluation signals

- Every template row with common name, InChI and InChIKey present yields a well-formed structured compound record in compound_records.
- Rows missing any mandatory field (common name, InChI, InChIKey) produce an explicit, logged row-level validation error rather than a partial record
- `mvn test` on the peakforest-webapp project passes, confirming the module's unit tests cover all template rows
- Mapped record fields correspond one-to-one to the chemical compound entity fields of the PeakForest data model (no unmapped or dropped column headers)
- Records are shaped for ingestion into the MySQL-backed PeakForest database (hibernate.cfg.xml-configured persistence layer)

## Limitations

- Common name, InChI and InChIKey are mandatory — rows lacking any of these cannot be imported by the batch module
- PeakForest requires metadata to be manually entered/cleaned beforehand; it has no data clean-up or extraction tools
- A minimum number of compounds and spectra must be imported before local PeakForest databases are fully functional
- Installation of the peakforest-webapp requires personnel with computing or bioinformatics skills (Java 8+, Maven 4+, MySQL 5+)

## Evidence

- [supplementary] The batch compound-import module creates associated compound cards based on minimal compound information found in the imported file, where common name, InChI and InChIKey are mandatory fields for the function to operate.: [evidence span withheld — anchor status: not_found]
- [methods] PeakForest is a Java Web application (version 8) and has been designed as an API’s toolbox: "PeakForest is a Java Web application (version 8) and has been designed as an API’s toolbox (Online Resource 1)."
- [readme] Run `mvn test` command to launch all unit test.: "Run `mvn test` command to launch all unit test."
- [methods] Users can import compounds and spectral data manually extracted from raw data, curate information and exploit this knowledge via data export or PeakForest Web services: "Users can import compounds and spectral data manually extracted from raw data, curate information and exploit this knowledge via data export or PeakForest Web services connected tools"
- [discussion] PeakForest does not include data clean-up or extraction tools per se and requires metadata to be manually entered in the database: "PeakForest does not include data clean-up or extraction tools per se and requires metadata to be manually entered in the database"
