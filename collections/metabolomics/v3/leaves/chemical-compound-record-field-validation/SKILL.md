---
name: chemical-compound-record-field-validation
description: Use when batch-importing large compound lists into PeakForest from the
  compound-import template file (XLSX, Online Resource 2) and each row must be checked
  for the mandatory common name, InChI and InChIKey fields before creating compound
  cards.
when_to_use_negative:
- Input is not the compound-import template but raw spectral data files — PeakForest
  is designed for metabolite annotation report data rather than storage of raw spectral
  data, and this skill only validates compound metadata fields.
- Only a few compounds need entering — the batch template import and its validation
  module target large compound lists; single compounds can be imported via the web
  interface.
- Rows lack structural identifiers (InChI/InChIKey) and cannot be completed beforehand
  — the import function will not operate, so records should be curated upstream instead.
edam_topics:
- http://edamontology.org/topic_3310
tools:
- name: PeakForest webapp
  role: Hosts the batch compound-import module and the chemical compound entity data
    model that defines the mandatory fields (common name, InChI, InChIKey)
  repo: github.com/peakforest/peakforest-webapp
- name: Java 8
  role: Implementation language for the parsing/validation module inside the PeakForest
    Java Web application (version 8)
- name: Maven
  role: 'Build and test tool: `mvn clean install` produces the deployable WAR; `mvn
    test` runs the import module''s unit tests'
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s11306-022-01899-3
    title: 'PeakForest: a multi-platform digital infrastructure for interoperable
      metabolite spectral data and metadata management'
  source_content_hash: sha256:cff95aca1a35c2b67e99b6a739ff95078eb7d8b0c6c7778c7f6cd12adb2c067f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java 8
  - Maven
  - PeakForest webapp
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3310
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# chemical-compound-record-field-validation

## Summary

A validation and serialization procedure for PeakForest's batch compound-import module: it parses an XLSX compound-import template, checks each row against the mandatory fields of the PeakForest chemical compound entity (common name, InChI, InChIKey), and emits structured compound records for database ingestion. It exists to guarantee that only complete, well-formed compound metadata enters a PeakForest instance.

## When to use

Use when batch-importing large compound lists into PeakForest from the compound-import template file (XLSX, Online Resource 2) and each row must be checked for the mandatory common name, InChI and InChIKey fields before creating compound cards.

## When NOT to use

- Input is not the compound-import template but raw spectral data files — PeakForest is designed for metabolite annotation report data rather than storage of raw spectral data, and this skill only validates compound metadata fields.
- Only a few compounds need entering — the batch template import and its validation module target large compound lists; single compounds can be imported via the web interface.
- Rows lack structural identifiers (InChI/InChIKey) and cannot be completed beforehand — the import function will not operate, so records should be curated upstream instead.

## Inputs

- Batch compound-import template file (XLSX, e.g. Online Resource 2) with one row per compound
- PeakForest chemical compound entity definition from the PeakForest data model
- PeakForest webapp source code (Java 8, Maven project)

## Outputs

- Structured compound_records. with one validated compound record per valid template row
- Row-level validation errors (logged) for rows missing mandatory fields (common name, InChI, InChIKey)
- Passing unit test suite under `mvn test`

## How to apply

Obtain the supplementary compound-import template (XLSX) and map each column header to the corresponding field of the chemical compound entity in the PeakForest data model. Implement the parsing/validation logic in Java 8 within the Maven-built PeakForest webapp codebase, treating common name, InChI and InChIKey as mandatory: any template row missing one of these fields must raise an explicit, row-level validation error rather than a partial record. Serialize each valid row into a structured compound record shaped for PeakForest database ingestion and collect all records (e.g. into compound_records.). Run `mvn test` to launch the module's unit tests and confirm that every template row yields either a well-formed compound record or a logged validation error. The rationale is that PeakForest does not include data clean-up or extraction tools, so template-level field validation is the gate that keeps incomplete compound cards out of the chemical compound library.

## Related tools

- **PeakForest webapp** (Hosts the batch compound-import module and the chemical compound entity data model that defines the mandatory fields (common name, InChI, InChIKey)) — github.com/peakforest/peakforest-webapp
- **Java 8** (Implementation language for the parsing/validation module inside the PeakForest Java Web application (version 8))
- **Maven** (Build and test tool: `mvn clean install` produces the deployable WAR; `mvn test` runs the import module's unit tests)

## Examples

```
git clone git@services.pfem.clermont.inrae.fr:peakforest/peakforest-webapp.git && mvn clean install && mvn test
```

## Evaluation signals

- Every row of the XLSX template yields exactly one of two outcomes: a well-formed compound record in the output JSON or an explicit, logged row-level validation error.
- Any row missing common name, InChI, or InChIKey produces a validation error and no partial compound card is created for it.
- `mvn test` completes successfully, covering the import module's unit tests.
- Output compound records conform to the field structure of the PeakForest chemical compound entity and are directly consumable by PeakForest database ingestion.
- `mvn clean install` produces the peakforest-webapp-X.war artifact containing the validated import module.

## Limitations

- PeakForest does not include data clean-up or extraction tools per se and requires metadata to be manually entered in the database — validation only checks completeness of mandatory fields, not content correctness.
- Users must supply completely-cleaned data following SOPs edited by the PeakForest team; the template rows must already be curated before import.
- A minimum number of compounds (and spectra) must be imported before local PeakForest databases are fully functional.
- Installation and development of the module require personnel with computing or bioinformatics skills (JVM 1.8+, Maven 4+, MySQL 5+).

## Evidence

- [supplementary] The batch compound-import module creates associated compound cards based on minimal compound information found in the imported file, where common name, InChI and InChIKey are mandatory fields: [evidence span withheld — anchor status: not_found]
- [methods] PeakForest is a Java Web application (version 8) and has been designed as an API's toolbox (Online Resource 1).: "PeakForest is a Java Web application (version 8) and has been designed as an API’s toolbox (Online Resource 1)."
- [methods] Users can import compounds and spectral data manually extracted from raw data, curate information and exploit this knowledge via data export or PeakForest Web services: "Users can import compounds and spectral data manually extracted from raw data, curate information and exploit this knowledge via data export or PeakForest Web services"
- [readme] This project uses and requires: - java JVM 1.8+ - maven 4+ - MySQL 5+: "This project uses and requires:
- java JVM 1.8+
- maven 4+
- MySQL 5+"
- [readme] Run `mvn test` command to launch all unit test.: "Run `mvn test` command to launch all unit test."
- [discussion] PeakForest does not include data clean-up or extraction tools per se and requires metadata to be manually entered in the database: "PeakForest does not include data clean-up or extraction tools per se and requires metadata to be manually entered in the database"
