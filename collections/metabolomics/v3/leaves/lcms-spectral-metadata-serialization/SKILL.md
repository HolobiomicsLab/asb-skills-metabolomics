---
name: lcms-spectral-metadata-serialization
description: Use when a curated PeakForest compound/spectrum collection (JSON/CSV
  peak lists with chromatography and ionization metadata) must be exported for MassBank/MoNA
  submission or consumed by tools like MS-DIAL and NIST MSsearch.
when_to_use_negative:
- 'Input is raw spectral data: PeakForest stores metadata related to raw data rather
  than raw data itself and is designed for metabolite annotation report data'
- 'Data has not been cleaned: PeakForest does not include data clean-up or extraction
  tools and requires completely-cleaned data following SOPs to be manually entered'
- The goal is MS-MS or 2D-NMR matching, which the authors note still requires additional
  development of matching capabilities
edam_operation: http://edamontology.org/operation_0335
tools:
- name: PeakForest
  role: Open-source Java webapp providing the export module that converts local spectral
    collections into mzML, MassBank, nmrML and MSP formats and exposes them via REST
    Web services
  repo: https://github.com/peakforest
- name: Maven 4+
  role: Build tool used to compile the PeakForest webapp converter (mvn clean install)
    into a deployable WAR artifact
- name: Java 8
  role: Runtime environment in which the PeakForest webapp and its serialization converter
    execute
- name: Tomcat 7
  role: Server hosting the PeakForest Web application and REST documentation through
    which exported records are served
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s11306-022-01899-3
    title: 'PeakForest: a multi-platform digital infrastructure for interoperable
      metabolite spectral data and metadata management'
  source_content_hash: sha256:12e695fe5d0be3cf097316fd73facf671746a85c5156913a9b1dbe8f462c5303
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java 8
  - Maven 4+
  - PeakForest
  - Tomcat 7
  edam_operation: http://edamontology.org/operation_0335
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# lcms-spectral-metadata-serialization

## Summary

Serializes locally curated LC-MS/NMR spectral and compound metadata collections from PeakForest into standardized exchange formats (mzML, MassBank text, nmrML, MSP) so that laboratory-internal records become interoperable with international repositories such as MassBank, MoNA, HMDB, GNPS, MetaboLights and Metabolomics Workbench.

## When to use

Use when a curated PeakForest compound/spectrum collection (JSON/CSV peak lists with chromatography and ionization metadata) must be exported for MassBank/MoNA submission or consumed by tools like MS-DIAL and NIST MSsearch.

## When NOT to use

- Input is raw spectral data: PeakForest stores metadata related to raw data rather than raw data itself and is designed for metabolite annotation report data
- Data has not been cleaned: PeakForest does not include data clean-up or extraction tools and requires completely-cleaned data following SOPs to be manually entered
- The goal is MS-MS or 2D-NMR matching, which the authors note still requires additional development of matching capabilities

## Inputs

- PeakForest database export of curated LC-MS compound and spectral metadata (JSON/CSV records with peak lists, chromatography and ionization metadata)
- Manually extracted compound and spectral metadata curated via the PeakForest curation interface

## Outputs

- lcms_spectra_export.mzml (standardized mzML spectral records)
- massbank_mona_records.txt (MassBank/MoNA submission-style tagged text records, one per spectrum)
- nmrML records for 1D-NMR spectra and MSP records for mass spectra
- REST Web service responses exposing the exported records to external repositories and tools

## How to apply

Load the curated JSON/CSV records from the PeakForest database containing peak lists, chromatography and ionization metadata, and map each compound and spectrum record onto the standardized spectral exchange schema, carrying over chemical identifiers, acquisition metadata and peak annotations. Serialize the collection with the Java-based converter built in the PeakForest webapp (a Java 8 / Maven 4+ / MySQL 5+ project) into the mzML-format file lcms_spectra_export.mzml. In parallel, generate MassBank/MoNA submission-style tagged text records (one record per spectrum) into massbank_mona_records.txt, and also emit technique-adapted open formats (nmrML for 1D-NMR spectra, MSP for mass spectra). Expose the converted records through the PeakForest data-export path and REST Web services hosted on Tomcat so external repositories and tools can consume them.

## Related tools

- **PeakForest** (Open-source Java webapp providing the export module that converts local spectral collections into mzML, MassBank, nmrML and MSP formats and exposes them via REST Web services) — https://github.com/peakforest
- **Maven 4+** (Build tool used to compile the PeakForest webapp converter (mvn clean install) into a deployable WAR artifact)
- **Java 8** (Runtime environment in which the PeakForest webapp and its serialization converter execute)
- **Tomcat 7** (Server hosting the PeakForest Web application and REST documentation through which exported records are served)

## Examples

```
git clone git@services.pfem.clermont.inrae.fr:peakforest/peakforest-webapp.git && mvn clean install && mvn test
```

## Evaluation signals

- The mzML output must parse as well-formed standardized spectral records
- The number of exported records must equal the number of spectra in the input collection
- All compound metadata fields (chemical identifiers, acquisition metadata, peak annotations) must be preserved after serialization
- The mvn clean install build must produce peakforest-webapp-X.war and mvn test unit tests must pass
- Exported records must be retrievable through the PeakForest data-export path / REST Web services

## Limitations

- Technical installation requires personnel with computing or bioinformatics skills
- PeakForest does not include data clean-up or extraction tools; metadata must be manually entered and users must supply completely-cleaned data following SOPs
- A minimum number of compounds and spectra must be imported before local databases are fully functional
- The curation feature is less easy to deploy in larger-scale public repositories as it requires centralised, highly-reactive error detection and correction processes

## Evidence

- [abstract] Standardized operating procedures and formats were included to ensure data quality and interoperability, in line with international recommendations and FAIR principles.: "Standardized operating procedures and formats were included to ensure data quality and interoperability, in line with international recommendations and FAIR principles."
- [methods] Import/curate/export via Web services: "Users can import compounds and spectral data manually extracted from raw data, curate information and exploit this knowledge via data export or PeakForest Web services connected tools"
- [readme] Maven build produces deployable WAR: "Build command `mvn clean install` produce: `peakforest-webapp-X.war` (java bin file, ready for a tomcat server deploy; `X` matching the current version number)"
- [methods] Webapp tech requirements: "This project uses and requires:
- java JVM 1.8+
- maven 4+
- MySQL 5+"
