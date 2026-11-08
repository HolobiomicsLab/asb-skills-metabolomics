---
name: compound-spectra-data-model-mapping
description: Use when a laboratory needs to ingest LC-MS/MS (or NMR) spectra with
  their metadata into a local PeakForest database via the prefilled spreadsheet import
  template, associating each spectrum with a compound and a biological matrix such
  as human urine.
when_to_use_negative:
- The input is raw spectral data (mzML/netCDF etc.) needing peak detection or clean-up
  — PeakForest does not include data clean-up or extraction tools and expects completely-cleaned
  data
- The goal is large-scale public-repository deposition with automated curation — the
  curation feature requires centralized, highly-reactive error detection and is 'less
  easy to deploy in larger-scale public repositories'
- Only a handful of compounds/spectra exist — a minimum number of compounds and spectra
  must be imported before a local PeakForest database is fully functional
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0605
tools:
- name: PeakForest
  role: Target Java Web application whose data-model entities (chemical compounds,
    spectra) define the schema the template columns are mapped onto; export/Web services
    consume the resulting records
  repo: https://github.com/peakforest/peakforest-webapp
- name: Java 8
  role: Runtime for the PeakForest webapp that the mapped records must be compatible
    with (JVM 1.8+)
  repo: https://github.com/peakforest/peakforest-webapp
- name: MySQL 5+
  role: Backend database into which the mapped compound and spectra entities are ultimately
    persisted
- name: Open Babel
  role: Chemistry toolbox used by the webapp for chemical compound library handling
    during mapping of compound metadata
  repo: http://openbabel.org/wiki/Main_Page
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-022-01899-3
    title: 'PeakForest: a multi-platform digital infrastructure for interoperable
      metabolite spectral data and metadata management'
  source_content_hash: sha256:b06db8003938056f153a5e7143109f637ae9e00886e98721aee77b5da6c02d17
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/peakforest/peakforest-webapp
  tools:
  - Java 8
  - MySQL 5+
  - Open Babel
  - PeakForest
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# compound-spectra-data-model-mapping

## Summary

Maps columns of a PeakForest LC-MS/MS Excel-like import template (e.g. the SI urine-matrix template from Online Resources 3–6) onto the PeakForest data-model entities — chemical compounds and spectra — including sample-type/biological-matrix qualifiers, producing clean JSON records compatible with the PeakForest database and Web services.

## When to use

Use when a laboratory needs to ingest LC-MS/MS (or NMR) spectra with their metadata into a local PeakForest database via the prefilled spreadsheet import template, associating each spectrum with a compound and a biological matrix such as human urine.

## When NOT to use

- The input is raw spectral data (mzML/netCDF etc.) needing peak detection or clean-up — PeakForest does not include data clean-up or extraction tools and expects completely-cleaned data
- The goal is large-scale public-repository deposition with automated curation — the curation feature requires centralized, highly-reactive error detection and is 'less easy to deploy in larger-scale public repositories'
- Only a handful of compounds/spectra exist — a minimum number of compounds and spectra must be imported before a local PeakForest database is fully functional

## Inputs

- PeakForest LC-MS/MS import template (XLSX, e.g. supplementary Online Resources 3–6) prefilled for spectra acquired on a specific analytical matrix (e.g. human urine)
- Compound metadata (names/structures curated via the PeakForest chemical compound library, supported by Open Babel)
- MS-MS spectral metadata manually extracted from raw data (PeakForest stores metadata, not raw spectra)
- Biological-matrix / sample-type annotations for each spectrum

## Outputs

- Validated structured records with compound metadata, MS-MS spectral metadata, and matrix linkage
- JSON file compatible with PeakForest database entities and Web services (e.g. lc_msms_urine_spectral_records.json)

## How to apply

Load the PeakForest LC-MS/MS supplementary import template (XLSX) that was generated with prefilled, predefined data and metadata derived from the user's analytical methods. Map each template column onto the PeakForest Java webapp data model (version 8, Hibernate-backed, requiring JVM 1.8+, Maven 4+, MySQL 5+), yielding structured records that hold compound metadata, MS-MS spectral metadata, and the biological-matrix annotation linkage. Store metadata about the raw data rather than raw spectra themselves, mirroring PeakForest's design of storing 'metadata related to raw data rather than raw data itself to minimize storage space usage'. Validate that every required metadata field is populated, because PeakForest includes no clean-up tools and requires users to supply completely-cleaned data following team-edited SOPs. Serialize the validated records (e.g. lc_msms_urine_spectral_records.json) in a schema compatible with the PeakForest database entities and its REST Web services, which support filters such as LC/GC column characteristics and 'AND'/'OR' logic-gate queries.

## Related tools

- **PeakForest** (Target Java Web application whose data-model entities (chemical compounds, spectra) define the schema the template columns are mapped onto; export/Web services consume the resulting records) — https://github.com/peakforest/peakforest-webapp
- **Java 8** (Runtime for the PeakForest webapp that the mapped records must be compatible with (JVM 1.8+)) — https://github.com/peakforest/peakforest-webapp
- **MySQL 5+** (Backend database into which the mapped compound and spectra entities are ultimately persisted)
- **Open Babel** (Chemistry toolbox used by the webapp for chemical compound library handling during mapping of compound metadata) — http://openbabel.org/wiki/Main_Page

## Examples

```
python parse_pf_template.py --template PeakForest_LCMSMS_urine_template.xlsx --matrix human-urine --out lc_msms_urine_spectral_records.json
```

## Evaluation signals

- Every required metadata field in the template is populated — no nulls, since PeakForest does no clean-up and requires completely-cleaned input
- Each record links to exactly one chemical compound and carries a sample-type/biological-matrix qualifier (e.g. presence in human urine)
- Output JSON serializes cleanly against the PeakForest database entity schema and is accepted by the REST Web services
- Records contain metadata about raw data rather than raw spectra themselves (storage-minimizing design)
- Round-trip check: records re-imported via the PeakForest webapp (mvn clean install WAR on Tomcat 7) appear correctly in the spectra library manager

## Limitations

- Metadata must be manually entered; PeakForest does not include data clean-up or extraction tools per se
- Installation and local deployment require personnel with computing or bioinformatics skills
- Additional developments are needed to enhance MS-MS matching capabilities, so mapped MS-MS records may not fully support automated matching yet
- Initial database construction and spectral collection takes time; deployment is iterative

## Evidence

- [supplementary] excel-like file template with prefilled and predefined data/metadata: [evidence span withheld — anchor status: not_found]
- [methods] metadata instead of raw data storage: "PeakForest stores metadata related to raw data rather than raw data itself to minimize storage space usage."
- [discussion] no clean-up tools, cleaned data required: "PeakForest does not include data clean-up or extraction tools per se and requires metadata to be manually entered in the database"
- [readme] build requirements for the webapp data model: "This project uses and requires:
- java JVM 1.8+
- maven 4+
- MySQL 5+"
- [methods] import–curate–exploit workflow: "Users can import compounds and spectral data manually extracted from raw data, curate information and exploit this knowledge via data export or PeakForest Web services connected tools"
