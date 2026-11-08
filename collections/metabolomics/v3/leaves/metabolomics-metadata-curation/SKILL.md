---
name: metabolomics-metadata-curation
description: Use when preparing spectral metadata records (e.g. LC-MS/MS spectra from
  a urine analytical matrix) for import into PeakForest, where users must supply completely-cleaned
  data from an Excel-like template.
when_to_use_negative:
- You need data clean-up, extraction, or processing of raw spectral files — PeakForest
  is designed for metabolite annotation report data rather than storage of raw spectral
  data
- Metadata is incomplete or noisy and cannot be fully cleaned before import, since
  PeakForest requires completely-cleaned data and offers no clean-up tools
- The task is browsing/querying existing spectra (use PeakForest's filter/search Web
  service with column characteristics or AND/OR logic instead of re-curating records)
edam_topics:
- http://edamontology.org/topic_3391
- http://edamontology.org/topic_3272
tools:
- name: PeakForest
  role: 'Target infrastructure: open-source Java Web application (version 8) whose
    data model integrates chemical compounds and spectra, with Web services for import
    and curation'
  repo: https://github.com/peakforest
- name: PeakForest webapp
  role: Java webapp whose entity model (compounds, spectra) and REST services define
    the schema the curated records must conform to
  repo: https://github.com/peakforest/peakforest-webapp
- name: Java 8 / Maven / MySQL
  role: Runtime stack required to run and test the local PeakForest instance into
    which curated metadata is imported
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-022-01899-3
    title: 'PeakForest: a multi-platform digital infrastructure for interoperable
      metabolite spectral data and metadata management'
  source_content_hash: sha256:9db99ac0ed0ce37d90af68a003163eb4cf6d2abd83cf991f78bd900180ebdd37
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java 8 / Maven / MySQL
  - PeakForest
  - PeakForest webapp
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3391
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# metabolomics-metadata-curation

## Summary

Curates and validates metabolite spectral metadata (compound metadata, MS-MS spectral metadata, biological-matrix annotations) for ingestion into the PeakForest database, which stores metadata about raw data rather than the raw spectra themselves.

## When to use

Use when preparing spectral metadata records (e.g. LC-MS/MS spectra from a urine analytical matrix) for import into PeakForest, where users must supply completely-cleaned data from an Excel-like template.

## When NOT to use

- You need data clean-up, extraction, or processing of raw spectral files — PeakForest is designed for metabolite annotation report data rather than storage of raw spectral data
- Metadata is incomplete or noisy and cannot be fully cleaned before import, since PeakForest requires completely-cleaned data and offers no clean-up tools
- The task is browsing/querying existing spectra (use PeakForest's filter/search Web service with column characteristics or AND/OR logic instead of re-curating records)

## Inputs

- PeakForest LC-MS/MS Excel-like import template (XLSX) with prefilled/predefined data and metadata columns (Online Resources 3–6)
- Chemical compound metadata associated with each imported spectrum
- Sample-type / biological-matrix annotations (e.g. presence in human urine)

## Outputs

- Validated structured metadata records (e.g. lc_msms_urine_spectral_records.) compatible with PeakForest database entities
- Records ingestable via PeakForest Web services / REST API

## How to apply

Load the PeakForest LC-MS/MS import template (XLSX, e.g. Online Resources 3–6) for spectra acquired on the analytical matrix of interest (e.g. human urine) with a spreadsheet-parsing library such as pandas. Map template columns onto the PeakForest Java webapp data-model entities — chemical compounds and spectra — parsing each entry into a structured record holding compound metadata, spectral metadata, and the biological-matrix linkage. PeakForest stores metadata related to raw data rather than the raw data itself, so record descriptors rather than spectra. Validate that required metadata fields are populated in every record, because PeakForest does not include data clean-up or extraction tools and requires users to supply completely-cleaned data following SOPs edited by the PeakForest team. Serialize validated records to a schema-compatible JSON file aligned with the PeakForest database entities and REST Web services.

## Related tools

- **PeakForest** (Target infrastructure: open-source Java Web application (version 8) whose data model integrates chemical compounds and spectra, with Web services for import and curation) — https://github.com/peakforest
- **PeakForest webapp** (Java webapp whose entity model (compounds, spectra) and REST services define the schema the curated records must conform to) — https://github.com/peakforest/peakforest-webapp
- **Java 8 / Maven / MySQL** (Runtime stack required to run and test the local PeakForest instance into which curated metadata is imported)

## Examples

```
python -c "import pandas as pd, ; df = pd.read_excel('PeakForest_LCMSMS_urine_template.xlsx'); recs = df.to_dict('records'); .dump(recs, open('lc_msms_urine_spectral_records.','w'))"
```

## Evaluation signals

- Every serialized record has all required metadata fields populated (no empty compound, spectral, or matrix-annotation fields), since PeakForest accepts only completely-cleaned data
- Records reference a chemical compound entity and a spectrum entity consistent with the PeakForest Java webapp data model (compounds and spectra integrated in one schema)
- Each spectrum record carries a qualified sample-type / biological-matrix annotation (e.g. presence in human urine)
- Output JSON round-trips through the PeakForest schema without missing-field errors on import via the Web services
- Output contains metadata about raw data, not raw spectral signals, mirroring PeakForest's storage policy

## Limitations

- PeakForest does not include data clean-up or extraction tools per se; metadata must be manually entered and cleaned before import
- A minimum number of compounds and spectra must be imported before a local PeakForest database is fully functional; initial construction takes time and deployment is iterative
- Installation and curation workflows require personnel with computing or bioinformatics skills
- Additional developments are needed to enhance MS-MS matching capabilities; UV-Vis spectra and ion-mobility CCS data are not yet integrated

## Evidence

- [methods] PeakForest stores metadata related to raw data rather than raw data itself to minimize storage space usage.: "PeakForest stores metadata related to raw data rather than raw data itself to minimize storage space usage."
- [methods] Users can import compounds and spectral data manually extracted from raw data, curate information, and exploit via data export or Web services: "Users can import compounds and spectral data manually extracted from raw data, curate information and exploit this knowledge via data export or PeakForest Web services connected tools"
- [discussion] PeakForest does not include data clean-up or extraction tools; metadata must be manually entered: "PeakForest does not include data clean-up or extraction tools per se and requires metadata to be manually entered in the database"
- [discussion] Users must supply completely-cleaned data following SOPs, assisted by tutorials: "users supply completely-cleaned data following SOPs edited by the PeakForest team, and PeakForest tutorials are available to assist in this step"
- [methods] PeakForest is a Java Web application (version 8) and has been designed as an API's toolbox: "PeakForest is a Java Web application (version 8) and has been designed as an API’s toolbox (Online Resource 1)."
