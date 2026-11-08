---
name: lcmsms-annotation-record-structuring
description: Use when LC-MS/MS spectra acquired on a biological matrix (e.g. human
  urine) must be imported into PeakForest via its prefilled XLSX template, and template
  columns must be mapped onto chemical-compound and spectral entities.
when_to_use_negative:
- You need to store or process raw spectral data itself — PeakForest is designed for
  metabolite annotation report data rather than storage of raw spectral data, storing
  only metadata about the raw data
- The input is uncleaned data requiring clean-up or extraction — PeakForest does not
  include data clean-up or extraction tools and metadata must be manually entered
- The goal is NMR rather than LC-MS/MS import — use the PeakForest NMR import path
  and tooling (e.g. the NMRPro viewer) instead
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: PeakForest
  role: Open-source Java webapp (version 8) whose data model of chemical compounds
    and spectra defines the record schema and whose excel-like import template supplies
    the prefilled data/metadata columns
  repo: https://github.com/peakforest/peakforest-webapp
- name: Java 8
  role: Runtime for deploying the PeakForest webapp whose entities the structured
    records must match (JVM 1.8+, built with mvn clean install into a WAR for Tomcat)
  repo: https://github.com/peakforest/peakforest-webapp
- name: MySQL 5+
  role: Backend database into which the structured LC-MS/MS compound and spectral
    records are ultimately imported
  repo: https://github.com/peakforest/peakforest-webapp
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-022-01899-3
    title: 'PeakForest: a multi-platform digital infrastructure for interoperable
      metabolite spectral data and metadata management'
  source_content_hash: sha256:498f939796571277381a0017fcc0d31a95d3b6050c20fe19ca73f083c96bf682
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# lcmsms-annotation-record-structuring

## Summary

Structures LC-MS/MS spectral annotation records by parsing a PeakForest excel-like import template (e.g. for a human urine analytical matrix) into validated, compound-linked, biological-matrix-qualified records serialized for the PeakForest database and Web services.

## When to use

Use when LC-MS/MS spectra acquired on a biological matrix (e.g. human urine) must be imported into PeakForest via its prefilled XLSX template, and template columns must be mapped onto chemical-compound and spectral entities.

## When NOT to use

- You need to store or process raw spectral data itself — PeakForest is designed for metabolite annotation report data rather than storage of raw spectral data, storing only metadata about the raw data
- The input is uncleaned data requiring clean-up or extraction — PeakForest does not include data clean-up or extraction tools and metadata must be manually entered
- The goal is NMR rather than LC-MS/MS import — use the PeakForest NMR import path and tooling (e.g. the NMRPro viewer) instead

## Inputs

- PeakForest LC-MS/MS XLSX import template prefilled from the user's analytical methods (e.g. supplementary file for a human urine analytical matrix, analogous to Online Resources 3–6)
- Compound metadata (chemical compound identifiers/names) associated with each acquired spectrum
- MS-MS spectral metadata (instrument/analytical-method attributes) and biological-matrix annotations (e.g. presence in human urine)

## Outputs

- lc_msms_urine_spectral_records. — validated per-spectrum records with compound metadata, MS-MS spectral metadata, and biological-matrix linkage, schema-compatible with PeakForest database entities and Web services

## How to apply

Load the supplementary PeakForest LC-MS/MS import template (one of the XLSX files analogous to Online Resources 3–6) with a spreadsheet-parsing library such as pandas, relying on the prefilled and predefined data/metadata columns derived from the user's analytical methods. Map each template column onto the PeakForest Java webapp data-model entities — chemical compounds and spectra — so every record carries compound metadata, MS-MS spectral metadata, and the biological-matrix (human urine) annotation linkage. Follow PeakForest's design decision of storing metadata related to raw data rather than the raw spectra themselves. Validate that all required metadata fields are populated, because PeakForest requires users to supply completely-cleaned data and provides no clean-up tools. Serialize the validated records as JSON (e.g. lc_msms_urine_spectral_records.) in a schema compatible with PeakForest database entities and Web services.

## Related tools

- **PeakForest** (Open-source Java webapp (version 8) whose data model of chemical compounds and spectra defines the record schema and whose excel-like import template supplies the prefilled data/metadata columns) — https://github.com/peakforest/peakforest-webapp
- **Java 8** (Runtime for deploying the PeakForest webapp whose entities the structured records must match (JVM 1.8+, built with mvn clean install into a WAR for Tomcat)) — https://github.com/peakforest/peakforest-webapp
- **MySQL 5+** (Backend database into which the structured LC-MS/MS compound and spectral records are ultimately imported) — https://github.com/peakforest/peakforest-webapp

## Examples

```
import pandas as pd; records = pd.read_excel('PeakForest_LC-MSMS_urine_template.xlsx').to_dict('records'); import ; .dump([r for r in records if all(v not in (None,'') for v in r.values())], open('lc_msms_urine_spectral_records.','w'))
```

## Evaluation signals

- Every parsed record has all required metadata fields populated (no nulls/empties), since users must supply completely-cleaned data following PeakForest SOPs
- Each record links to a chemical compound entity and carries a sample-type/biological-matrix qualifier (e.g. presence in human urine)
- Records contain metadata about the raw data only, not raw spectra, consistent with PeakForest's storage design
- The serialized JSON loads against a schema mirroring the PeakForest database entities used by the Web services
- Column mapping is complete: every prefilled/predefined template column from the XLSX is accounted for in the output schema

## Limitations

- PeakForest does not include data clean-up or extraction tools per se and requires metadata to be manually entered in the database; the template must already be completely cleaned
- PeakForest stores metadata related to raw data rather than raw data itself, so raw spectra cannot be archived via this import path
- A minimum number of compounds and spectra need to be imported before local PeakForest databases become fully-functional; initial database construction takes time
- Installation/deployment requires personnel with computing or bioinformatics skills (JVM 1.8+, maven 4+, MySQL 5+, Tomcat 7)

## Evidence

- [abstract] abstract: "PeakForest is able to capture and store experimental spectral MS and NMR metadata as well as collect and display signal annotations."
- [methods] methods: "PeakForest stores metadata related to raw data rather than raw data itself to minimize storage space usage."
- [discussion] discussion: "PeakForest does not include data clean-up or extraction tools per se and requires metadata to be manually entered in the database"
- [readme] readme: "This project uses and requires:
- java JVM 1.8+
- maven 4+
- MySQL 5+"
- [results] results: "the annotation of spectra in biological matrices via external peak matching tools, to the spectra linkage with external resources for biological interpretation or publication"
