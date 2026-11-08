---
name: rest-web-service-data-export
description: Use when a laboratory needs to move locally curated PeakForest spectral
  records (peak lists, chromatography and ionization metadata) out of the database
  for reuse by external tools or submission to international repositories.
when_to_use_negative:
- Input is raw spectral data that still needs clean-up or extraction — PeakForest
  does not include data clean-up tools and requires metadata to be manually entered
  and completely cleaned following SOPs
- The goal is long-term storage of raw spectral data — PeakForest is designed for
  metabolite annotation report data and stores metadata related to raw data rather
  than raw data itself
- Only one or very few spectra are available — a minimum number of compounds and spectra
  must be imported before local PeakForest databases are fully functional
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: PeakForest
  role: Open-source Java Web application providing the API toolbox, REST Web services
    and data-export path for curated NMR/MS spectral records
  repo: https://services.pfem.clermont.inrae.fr/gitlab/peakforest/peakforest-webapp
- name: Tomcat 7
  role: Hosts the PeakForest Web application and its REST documentation
- name: Maven 4+
  role: Builds peakforest-webapp-X.war via mvn clean install for Tomcat deployment
- name: MySQL 5+
  role: Backend database storing the compounds and spectra served through the export
    path
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s11306-022-01899-3
    title: 'PeakForest: a multi-platform digital infrastructure for interoperable
      metabolite spectral data and metadata management'
  source_content_hash: sha256:64e0b3606a8b4fa9507a122d65709a79731431ce32cd9153c0086e44f4453283
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Maven 4+
  - MySQL 5+
  - PeakForest
  - Tomcat 7
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# rest-web-service-data-export

## Summary

Exposes locally curated PeakForest compound, LC-MS/GC-MS and NMR spectral records through the PeakForest REST Web services and data-export path so external repositories (MassBank, MoNA, HMDB, GNPS, MetaboLights, Metabolomics Workbench) and connected tools can consume them.

## When to use

Use when a laboratory needs to move locally curated PeakForest spectral records (peak lists, chromatography and ionization metadata) out of the database for reuse by external tools or submission to international repositories.

## When NOT to use

- Input is raw spectral data that still needs clean-up or extraction — PeakForest does not include data clean-up tools and requires metadata to be manually entered and completely cleaned following SOPs
- The goal is long-term storage of raw spectral data — PeakForest is designed for metabolite annotation report data and stores metadata related to raw data rather than raw data itself
- Only one or very few spectra are available — a minimum number of compounds and spectra must be imported before local PeakForest databases are fully functional

## Inputs

- Curated PeakForest compound and spectral metadata collection (JSON/CSV records with peak lists, chromatography and ionization metadata)
- PeakForest webapp source tree (git clone of peakforest-webapp) with configuration files conf.properties, info.properties, hibernate.cfg.xml, metExploreData.
- MySQL 5+ database populated with a minimum number of compounds and spectra

## Outputs

- lcms_spectra_export.mzML (standardized mzML spectral records)
- massbank_mona_records.txt (MassBank/MoNA submission-style tagged text records, one per spectrum)
- REST Web service query endpoints returning filtered spectral/compound records (nmrML for 1D-NMR, MSP for mass spectra)

## How to apply

Build the PeakForest Java webapp with Maven 4+ (mvn clean install) and deploy the resulting peakforest-webapp-X.war on a Tomcat 7 server backed by MySQL 5+, since the Web application and the REST documentation are hosted on a Tomcat server. Load the curated LC-MS compound and spectral metadata collection (JSON/CSV records with peak lists, chromatography and ionization metadata) that was manually extracted from raw data and curated in the database. Map each compound and spectrum record onto standardized spectral exchange schemas — carrying chemical identifiers, acquisition metadata and peak annotations — and serialize with the Java-based converter into an mzML file (e.g. lcms_spectra_export.mzML) plus MassBank/MoNA-style tagged text records (massbank_mona_records.txt); technique-adapted formats such as nmrML for 1D-NMR and MSP for mass spectra keep outputs compatible with tools like MS-DIAL and NIST MSsearch. Expose the converted records through the PeakForest data-export path and REST Web services, using the built-in filter system (column characteristics such as column brand, type, name, length, diameter as filters, with 'AND'/'OR' logic gates) so clients can query and retrieve exactly the spectra of interest.

## Related tools

- **PeakForest** (Open-source Java Web application providing the API toolbox, REST Web services and data-export path for curated NMR/MS spectral records) — https://services.pfem.clermont.inrae.fr/gitlab/peakforest/peakforest-webapp
- **Tomcat 7** (Hosts the PeakForest Web application and its REST documentation)
- **Maven 4+** (Builds peakforest-webapp-X.war via mvn clean install for Tomcat deployment)
- **MySQL 5+** (Backend database storing the compounds and spectra served through the export path)

## Examples

```
git clone git@services.pfem.clermont.inrae.fr:peakforest/peakforest-webapp.git && mvn clean install
```

## Evaluation signals

- The mzML output (lcms_spectra_export.mzML) parses as well-formed standardized spectral records
- The number of exported records equals the number of spectra in the input collection
- All compound metadata fields (chemical identifiers, acquisition metadata, peak annotations) are preserved in the exported records
- MassBank/MoNA records in massbank_mona_records.txt are one tagged record per spectrum and load into connected tools such as MS-DIAL or NIST MSsearch
- REST queries with column-characteristic filters and 'AND'/'OR' logic gates return the expected subset of spectra

## Limitations

- Installation requires personnel with computing or bioinformatics skills; deployment is iterative and initial database construction takes time
- Users must supply completely-cleaned data following SOPs, since PeakForest does not include data clean-up or extraction tools and metadata must be manually entered
- PeakForest stores metadata related to raw data rather than raw data itself, so exports carry annotation-report metadata, not raw acquisitions

## Evidence

- [methods] Export via REST Web services and connected tools: "Users can import compounds and spectral data manually extracted from raw data, curate information and exploit this knowledge via data export or PeakForest Web services connected tools"
- [methods] PeakForest designed as API toolbox: "PeakForest is a Java Web application (version 8) and has been designed as an API’s toolbox (Online Resource 1)."
- [results] Column characteristics usable as Web service filters: "The column characteristics can be used as filter in Web service"
- [methods] Tomcat hosting of Web application and REST docs: "The Web application and the REST documentation are hosted on a Tomcat server (version 7)"
- [readme] Building the webapp WAR artifact: "Build command `mvn clean install` produce: - `peakforest-webapp-X.war` (java bin file, ready for a tomcat server deploy; `X` matching the current version number)"
