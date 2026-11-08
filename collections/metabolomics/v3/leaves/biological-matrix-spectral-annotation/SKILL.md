---
name: biological-matrix-spectral-annotation
description: Use when populating a PeakForest instance with LC-MS/MS spectra from
  a supplementary import template (Online Resources 3–6 style XLSX) and spectra must
  be associated with compounds and qualified by biological-matrix presence.
when_to_use_negative:
- You need to store or process raw spectral data files — PeakForest is designed for
  metabolite annotation report data rather than storage of raw spectral data
- Your spectra files are dirty or uncleaned — PeakForest does not include data clean-up
  or extraction tools and requires users to supply completely-cleaned data
- You are annotating spectra without linking them to a chemical compound or biological-matrix
  sample type — the import path is specifically structured around these entities
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: PeakForest
  role: Open-source Java web infrastructure hosting the compound/spectra libraries,
    the Excel-like LC-MS/MS import template mechanism, curation interface, and REST
    Web services used to store and query biological-matrix spectral annotations
  repo: https://github.com/peakforest/peakforest-webapp
- name: Java 8
  role: Runtime (JVM 1.8+) on which the PeakForest webapp data model for compounds
    and spectra is deployed
  repo: https://github.com/peakforest/peakforest-webapp
- name: MySQL 5+
  role: Database backend (MySQL 5+) persisting the PeakForest compound and spectra
    entities into which imported records are loaded
- name: Tomcat 7
  role: Server hosting the deployed peakforest-webapp WAR and its REST documentation/Web
    services
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-022-01899-3
    title: 'PeakForest: a multi-platform digital infrastructure for interoperable
      metabolite spectral data and metadata management'
  source_content_hash: sha256:b53a74b9af52f2a683f9919b59f3b8059da4e44245c7f5a450cce1b4da2924e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/peakforest/peakforest-webapp
  tools:
  - Java 8
  - MySQL 5+
  - PeakForest
  - Tomcat 7
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# biological-matrix-spectral-annotation

## Summary

Import and annotate LC-MS/MS spectra acquired on a biological matrix (e.g. human urine) into a PeakForest database using the Excel-like XLSX import template, so spectra are linked to chemical compounds and qualified by sample type. PeakForest stores metadata about the raw data rather than the raw spectra themselves.

## When to use

Use when populating a PeakForest instance with LC-MS/MS spectra from a supplementary import template (Online Resources 3–6 style XLSX) and spectra must be associated with compounds and qualified by biological-matrix presence.

## When NOT to use

- You need to store or process raw spectral data files — PeakForest is designed for metabolite annotation report data rather than storage of raw spectral data
- Your spectra files are dirty or uncleaned — PeakForest does not include data clean-up or extraction tools and requires users to supply completely-cleaned data
- You are annotating spectra without linking them to a chemical compound or biological-matrix sample type — the import path is specifically structured around these entities

## Inputs

- PeakForest LC-MSMS import template XLSX (e.g. Online Resources 3–6 supplementary files) for spectra acquired on the human urine analytical matrix
- PeakForest Java webapp data model (chemical compound and spectra entities)
- Analytical method definitions used to prefill template metadata

## Outputs

- lc_msms_urine_spectral_records. with validated compound metadata, MS-MS spectral metadata, and biological-matrix (human urine) annotations, compatible with PeakForest database entities and Web services
- Curated entries in a local PeakForest database (compound library and spectra library)

## How to apply

Deploy the PeakForest Java webapp (JVM 1.8+, Maven 4+, MySQL 5+, Tomcat 7, built via `mvn clean install` into a WAR) and obtain the LC-MS/MS import template prefilled from the user's analytical methods. Load the XLSX with a spreadsheet-parsing library and map template columns onto the PeakForest data-model entities — chemical compounds and spectra — following the Java webapp data model. Parse each row into a structured record carrying compound metadata, MS-MS spectral metadata, and the biological-matrix (human urine) sample-type annotation, storing metadata about the raw data rather than the raw spectra. Validate that every required metadata field is populated, because PeakForest has no clean-up tools and users must supply completely-cleaned data following SOPs. Finally, serialize validated records to JSON in a schema compatible with the PeakForest database entities and REST Web services, where column characteristics and logic-gate ('AND', 'OR') filters can later be used for querying.

## Related tools

- **PeakForest** (Open-source Java web infrastructure hosting the compound/spectra libraries, the Excel-like LC-MS/MS import template mechanism, curation interface, and REST Web services used to store and query biological-matrix spectral annotations) — https://github.com/peakforest/peakforest-webapp
- **Java 8** (Runtime (JVM 1.8+) on which the PeakForest webapp data model for compounds and spectra is deployed) — https://github.com/peakforest/peakforest-webapp
- **MySQL 5+** (Database backend (MySQL 5+) persisting the PeakForest compound and spectra entities into which imported records are loaded)
- **Tomcat 7** (Server hosting the deployed peakforest-webapp WAR and its REST documentation/Web services)

## Examples

```
git clone git@services.pfem.clermont.inrae.fr:peakforest/peakforest-webapp.git && mvn clean install && mvn test
```

## Evaluation signals

- Every parsed record has all required metadata fields populated (PeakForest requires completely-cleaned data; no clean-up tools exist to compensate for gaps)
- Each spectral record is associated with exactly one chemical compound and a sample-type qualification including presence in a biological matrix (human urine)
- Output JSON records conform to the PeakForest database entity schema and are retrievable/exploitable through PeakForest Web services
- Records contain spectral metadata (not raw spectra), consistent with PeakForest storing metadata about raw data to minimize storage space usage
- Template-to-entity column mapping is complete — no template columns silently dropped during parsing of the XLSX

## Limitations

- PeakForest does not include data clean-up or extraction tools and requires metadata to be manually entered in the database
- A minimum number of compounds and spectra must be imported before local PeakForest databases are fully functional
- Additional developments are needed to enhance MS-MS matching capabilities; installation requires personnel with computing or bioinformatics skills

## Evidence

- [methods] The spectral import path operates via an excel-like file template: "PeakForest stores metadata related to raw data rather than raw data itself to minimize storage space usage."
- [methods] Users import and curate compounds and spectral data: "Users can import compounds and spectral data manually extracted from raw data, curate information and exploit this knowledge via data export or PeakForest Web services connected tools"
- [results] Annotation of spectra in biological matrices: "the annotation of spectra in biological matrices via external peak matching tools, to the spectra linkage with external resources for biological interpretation"
- [discussion] No clean-up tools; metadata must be cleaned before import: "PeakForest does not include data clean-up or extraction tools per se and requires metadata to be manually entered in the database"
- [readme] Webapp build/runtime prerequisites: "This project uses and requires:
- java JVM 1.8+
- maven 4+
- MySQL 5+"
