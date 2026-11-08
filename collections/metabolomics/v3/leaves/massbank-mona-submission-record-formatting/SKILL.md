---
name: massbank-mona-submission-record-formatting
description: Use when a laboratory's curated local spectral library (PeakForest JSON/CSV
  records with peak lists, acquisition and compound metadata) must be exported for
  submission to MassBank, MoNA or other public metabolomics repositories.
when_to_use_negative:
- Input is raw spectral data rather than curated metabolite annotation report data
  — PeakForest is designed for metabolite annotation report data rather than storage
  of raw spectral data
- 'Data has not been cleaned and curated: PeakForest does not include data clean-up
  or extraction tools and requires users to supply completely-cleaned data following
  SOPs'
- The goal is annotating unknown spectra against external banks rather than exporting
  existing curated records — use the external banks API / peak matching tools instead
edam_operation: http://edamontology.org/operation_3434
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: PeakForest
  role: Hosts the curated NMR/MS spectral database and provides the export module
    and REST Web services that convert local records into MassBank/MoNA-compatible
    and open formats
  repo: https://github.com/peakforest
- name: Java 8
  role: Runtime JVM for the PeakForest webapp in which the spectral format converter
    is implemented
- name: Maven 4+
  role: Build tool used to compile the PeakForest webapp converter (mvn clean install
    producing peakforest-webapp-X.war)
- name: Tomcat 7
  role: Server hosting the PeakForest web application and REST Web services that expose
    the exported records
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s11306-022-01899-3
    title: 'PeakForest: a multi-platform digital infrastructure for interoperable
      metabolite spectral data and metadata management'
  source_content_hash: sha256:7ff1c2bbb5e29c8bee694d7d9b3d465184973cd4a92f50c91a8b4bc918293e15
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java 8
  - Maven 4+
  - PeakForest
  - Tomcat 7
  edam_operation: http://edamontology.org/operation_3434
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# massbank-mona-submission-record-formatting

## Summary

Formats locally curated PeakForest spectral records (LC-MS/GC-MS peak lists with chromatography and ionization metadata) into MassBank/MoNA-compatible submission text records and standardized open formats (mzML, nmrML, MSP) so they can be deposited in international repositories such as MassBank, MoNA, HMDB, GNPS, MetaboLights and Metabolomics Workbench.

## When to use

Use when a laboratory's curated local spectral library (PeakForest JSON/CSV records with peak lists, acquisition and compound metadata) must be exported for submission to MassBank, MoNA or other public metabolomics repositories.

## When NOT to use

- Input is raw spectral data rather than curated metabolite annotation report data — PeakForest is designed for metabolite annotation report data rather than storage of raw spectral data
- Data has not been cleaned and curated: PeakForest does not include data clean-up or extraction tools and requires users to supply completely-cleaned data following SOPs
- The goal is annotating unknown spectra against external banks rather than exporting existing curated records — use the external banks API / peak matching tools instead

## Inputs

- Curated LC-MS compound and spectral metadata collection exported from PeakForest (JSON/CSV records with peak lists, chromatography and ionization metadata)
- PeakForest webapp source tree (git clone of peakforest-webapp) with Maven build configuration
- MassBank/MoNA submission format requirements for tagged spectral records

## Outputs

- lcms_spectra_export.mzml — standardized mzML spectral records
- massbank_mona_records.txt — MassBank/MoNA submission-style text records, one tagged record per spectrum
- nmrML files for 1D-NMR spectra and MSP files for mass spectra (technique-adapted open formats)
- Records exposed via the PeakForest data-export path / REST Web services

## How to apply

Load the curated compound and spectral metadata collection exported from the PeakForest database (JSON/CSV records containing peak lists, chromatography and ionization metadata). Map each compound and spectrum record onto the standardized spectral exchange schema, carrying over chemical identifiers, acquisition metadata and peak annotations. Serialize the collection with the Java-based converter built in the PeakForest webapp (compiled with Maven 4+ on a Java 8 JVM) into an mzML file (e.g. lcms_spectra_export.mzml), and generate MassBank/MoNA submission-style tagged text records, one per spectrum, in massbank_mona_records.txt. For technique-specific outputs, emit nmrML for 1D-NMR spectra and MSP for mass spectra, keeping outputs linkable to downstream tools such as MS-DIAL and GCMS NIST MSsearch. Expose the converted records through the PeakForest data-export path / REST Web services so external repositories and tools can consume them; validate that mzML parses as well-formed standardized spectral records and that exported record counts equal the number of input spectra with all compound metadata fields preserved.

## Related tools

- **PeakForest** (Hosts the curated NMR/MS spectral database and provides the export module and REST Web services that convert local records into MassBank/MoNA-compatible and open formats) — https://github.com/peakforest
- **Java 8** (Runtime JVM for the PeakForest webapp in which the spectral format converter is implemented)
- **Maven 4+** (Build tool used to compile the PeakForest webapp converter (mvn clean install producing peakforest-webapp-X.war))
- **Tomcat 7** (Server hosting the PeakForest web application and REST Web services that expose the exported records)

## Examples

```
git clone git@services.pfem.clermont.inrae.fr:peakforest/peakforest-webapp.git; mvn clean install
```

## Evaluation signals

- The mzML output must parse as well-formed standardized spectral records
- The number of exported records must equal the number of spectra in the input collection
- All compound metadata fields (chemical identifiers, acquisition metadata, peak annotations) must be preserved in the exported records
- MassBank/MoNA text output contains one tagged record per input spectrum
- mvn test passes on the PeakForest webapp build producing the converter WAR

## Limitations

- Installation requires personnel with computing or bioinformatics skills; a minimum number of compounds and spectra must be imported before local PeakForest databases are fully functional
- PeakForest does not include data clean-up or extraction tools — metadata must be manually entered and completely-cleaned data supplied following SOPs
- PeakForest stores metadata related to raw data rather than raw data itself, so raw spectra are not carried through the export
- The converter requires the full webapp stack (Java JVM 1.8+, Maven 4+, MySQL 5+, Open Babel 2.3.2 with java bundle)

## Evidence

- [results] Export module supporting MassBank/MoNA submissions: [evidence span withheld — anchor status: not_found]
- [results] Repository interoperability: [evidence span withheld — anchor status: not_found]
- [methods] Build prerequisites: "This project uses and requires:
- java JVM 1.8+
- maven 4+
- MySQL 5+"
- [readme] WAR artifact for Tomcat deployment: "Build command `mvn clean install` produce:
   - `peakforest-webapp-X.war` (java bin file, ready for a tomcat server deploy; `X` matching the current version number)"
- [abstract] FAIR/interoperability rationale: "Standardized operating procedures and formats were included to ensure data quality and interoperability, in line with international recommendations and FAIR principles."
