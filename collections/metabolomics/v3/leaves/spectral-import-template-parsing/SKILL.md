---
name: spectral-import-template-parsing
description: Use when a task asks to import LC-MS/MS (or MS/NMR) spectra into a PeakForest-style
  database from a prefilled spreadsheet template, or to map template columns onto
  compound/spectrum/biological-matrix entities before database or Web-service submission.
when_to_use_negative:
- The input is raw vendor spectral data (e.g. mzML/raw files) needing clean-up or
  extraction — PeakForest does not include data clean-up or extraction tools and expects
  completely-cleaned, manually curated metadata
- The goal is storage of raw spectral data itself rather than metabolite annotation
  report data and metadata
- Only a handful of spectra are available and a fully functional local PeakForest
  database is the goal — a minimum number of compounds and spectra must be imported
  first
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0153
tools:
- name: PeakForest
  role: 'Target infrastructure: its Java webapp data model, REST Web services, and
    excel-like import templates define the schema the parsed records must match'
  repo: https://github.com/peakforest/peakforest-webapp
- name: Java 8
  role: Runtime of the PeakForest web application whose entity model (compounds, spectra)
    governs the mapping of template columns
- name: MySQL 5+
  role: Backing database of the PeakForest webapp into which the serialized records
    must be loadable via the entity schema
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-022-01899-3
    title: 'PeakForest: a multi-platform digital infrastructure for interoperable
      metabolite spectral data and metadata management'
  source_content_hash: sha256:89f180f341e2eaa3f5d297e2e0021d7e48894108a77272b2f7e1027034aed223
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/peakforest/peakforest-webapp
  tools:
  - Java 8
  - MySQL 5+
  - PeakForest
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# spectral-import-template-parsing

## Summary

Reconstructs the PeakForest LC-MS/MS spectral import path by parsing the Excel-like XLSX import templates (e.g. the human urine analytical-matrix template from Online Resources 3–6) into structured records matching the PeakForest Java webapp data model of chemical compounds, spectra, and biological-matrix annotations.

## When to use

Use when a task asks to import LC-MS/MS (or MS/NMR) spectra into a PeakForest-style database from a prefilled spreadsheet template, or to map template columns onto compound/spectrum/biological-matrix entities before database or Web-service submission.

## When NOT to use

- The input is raw vendor spectral data (e.g. mzML/raw files) needing clean-up or extraction — PeakForest does not include data clean-up or extraction tools and expects completely-cleaned, manually curated metadata
- The goal is storage of raw spectral data itself rather than metabolite annotation report data and metadata
- Only a handful of spectra are available and a fully functional local PeakForest database is the goal — a minimum number of compounds and spectra must be imported first

## Inputs

- PeakForest LC-MSMS spectral import template (XLSX, e.g. Online Resources 3–6) for the human urine analytical matrix
- User analytical-method metadata used to prefill the template columns
- Chemical compound identifiers/metadata and MS-MS spectral metadata columns from the template
- Biological-matrix annotations (sample type, presence in human urine)

## Outputs

- lc_msms_urine_spectral_records. — validated per-spectrum records compatible with PeakForest database entities and Web services
- Validation report of missing required metadata fields per record

## How to apply

Load the supplementary PeakForest LC-MSMS XLSX import template (Online Resources 3–6) with a spreadsheet-parsing library such as pandas/openpyxl. Map each row onto the PeakForest data-model entities — chemical compounds and spectra — as defined by the Java (JVM 1.8+, Maven, MySQL 5+) webapp and its Hibernate-backed schema. Build one structured record per entry carrying compound metadata, MS-MS spectral metadata, and the biological-matrix linkage (e.g. presence in human urine); keep metadata about the raw data rather than raw spectra themselves, since PeakForest stores metadata to minimize storage space usage. Validate that every required metadata field is populated, because PeakForest supplies no clean-up tools and expects users to submit completely-cleaned data following team-edited SOPs. Finally serialize the validated records to a JSON schema compatible with the PeakForest database entities and REST Web services.

## Related tools

- **PeakForest** (Target infrastructure: its Java webapp data model, REST Web services, and excel-like import templates define the schema the parsed records must match) — https://github.com/peakforest/peakforest-webapp
- **Java 8** (Runtime of the PeakForest web application whose entity model (compounds, spectra) governs the mapping of template columns)
- **MySQL 5+** (Backing database of the PeakForest webapp into which the serialized records must be loadable via the entity schema)

## Examples

```
import pandas as pd; df = pd.read_excel('LC-MSMS_urine_import_template.xlsx'); df.dropna(how='all').to_('lc_msms_urine_spectral_records.', orient='records')
```

## Evaluation signals

- Every parsed record contains a non-null chemical-compound linkage and a non-null biological-matrix (human urine) annotation
- Serialized JSON keys match the PeakForest database entity fields (compound metadata, MS-MS spectral metadata, matrix annotation) and pass schema validation
- No raw spectral peak data appear in the output — only metadata about the raw data, per PeakForest's metadata-only storage design
- Row counts in the output JSON match the non-empty template rows after dropping prefilled header/example rows
- If records are loaded into a local PeakForest instance, imported spectra are browsable/curatable in the curation interface and queryable via Web services

## Limitations

- PeakForest does not include data clean-up or extraction tools; metadata must be manually entered and supplied completely cleaned following team-edited SOPs
- The Excel-like templates are prefilled/predefined based on the user's analytical methods, so parsing logic breaks if template column layout changes between Online Resource versions or analytical platforms
- Raw spectra are not stored — the import captures metadata and signal annotations only, and the local database needs a minimum number of imported compounds/spectra before it is fully functional

## Evidence

- [methods] Users can import compounds and spectral data manually extracted from raw data, curate information and exploit this knowledge via data export or PeakForest Web services connected tools: "Users can import compounds and spectral data manually extracted from raw data, curate information and exploit this knowledge via data export or PeakForest Web services connected tools"
- [methods] PeakForest stores metadata related to raw data rather than raw data itself to minimize storage space usage.: "PeakForest stores metadata related to raw data rather than raw data itself to minimize storage space usage."
- [discussion] PeakForest does not include data clean-up or extraction tools per se and requires metadata to be manually entered in the database: "PeakForest does not include data clean-up or extraction tools per se and requires metadata to be manually entered in the database"
- [readme] This project uses and requires: - java JVM 1.8+ - maven 4+ - MySQL 5+: "This project uses and requires:
- java JVM 1.8+
- maven 4+
- MySQL 5+"
- [readme] services provided: chemical compound library manager, spectra library manager, spectrum viewers, curation interface, admin interface: "- chemical compound library manager
- spectra library manager
- spectrum viewers
- curation interface
- admin interface"
