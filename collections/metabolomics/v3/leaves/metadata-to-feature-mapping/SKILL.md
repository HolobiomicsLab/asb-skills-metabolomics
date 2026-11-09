---
name: metadata-to-feature-mapping
description: Use when you have mzmine feature lists from .raw/.mzML flow injection
  acquisitions plus a cleaned metadata sheet and need to map each feature to its compound
  ID (e.g. to reproduce coverage counts like 30,008 unique molecules across ionization
  modes).
when_to_use_negative:
- Input is already a compound-level feature table with assigned compound identifiers
  — re-running the metadata join would duplicate mappings
- Metadata has not passed through the metadata cleanup flow and lacks the template
  columns (unique sample id, plate_id, well location) — the join keys will not match
- The goal is spectral library generation (MSn tree building) rather than compound
  coverage/detection statistics
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: mzmine
  role: Open-source software used to process flow injection Orbitrap MSn data and
    produce per-file compound detections for each ionization mode
  repo: https://github.com/mzmine/mzmine
- name: ThermoRawFileParser
  role: Converts Thermo .raw acquisition files to open formats (mzML, indexed mzML,
    MGF, Parquet) before feature processing
  repo: https://github.com/CompOmics/ThermoRawFileParser
- name: Prefect 2
  role: Orchestrates the metadata cleanup flow (metadata_cleanup_prefect.py, jobs.py)
    that produces the cleaned metadata table used as the join target
  repo: https://github.com/corinnabrungs/msn_tree_library
- name: Orbitrap ID-X
  role: Instrument whose acquisition sequences (via Xcalibur) generate the data; metadata
    sheets must carry the unique sample id, plate name (plate_id) and vial/well location
    it expects
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41592-025-02813-0
    title: 'MSnLib: efficient generation of open multi-stage fragmentation mass spectral
      libraries'
  source_content_hash: sha256:e9a6d5277c47e720d8034a7da79c868ef2ed243abe7199ab6b99eac95d3eeca3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine
  tools:
  - Orbitrap ID-X
  - Prefect 2
  - ThermoRawFileParser
  - mzmine
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
  techniques:
  - direct-infusion-MS
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# metadata-to-feature-mapping

## Summary

Joins mzmine-extracted per-file feature lists from flow injection Orbitrap MSn acquisitions against the cleaned metadata table so every detected feature is assigned to a unique compound identifier, enabling per-ionization-mode compound counts and positive/negative mode set intersections.

## When to use

Use when you have mzmine feature lists from .raw/.mzML flow injection acquisitions plus a cleaned metadata sheet and need to map each feature to its compound ID (e.g. to reproduce coverage counts like 30,008 unique molecules across ionization modes).

## When NOT to use

- Input is already a compound-level feature table with assigned compound identifiers — re-running the metadata join would duplicate mappings
- Metadata has not passed through the metadata cleanup flow and lacks the template columns (unique sample id, plate_id, well location) — the join keys will not match
- The goal is spectral library generation (MSn tree building) rather than compound coverage/detection statistics

## Inputs

- mzmine feature lists (per-file compound detections per ionization mode)
- Cleaned metadata table from the metadata_cleanup_prefect.py Prefect flow, following the shared template (unique sample id, plate_id, vial/well location)
- Thermo .raw acquisition files (convertible to .mzML via ThermoRawFileParser) from Zenodo (10966280, 10966404, 10967081) and MassIVE (MSV000094528)

## Outputs

- Feature-to-compound mapping assigning each detected feature to its unique compound identifier
- Unique compound counts per ionization mode (positive-only, negative-only, both modes)
- detection_statistics.csv count breakdown table

## How to apply

First produce the cleaned metadata via the metadata cleanup Prefect flow (metadata_cleanup_prefect.py / jobs.py), ensuring the sheet follows the shared template with consistent column names and a unique sample id, plate_id (batch identifier), and vial/well location fields. Convert Thermo .raw files with ThermoRawFileParser to mzML where needed and process them in mzmine to obtain per-file compound detections for each ionization mode (positive and negative). Join the mzmine feature lists to the cleaned metadata on the unique sample id so each detected feature inherits its unique compound identifier; if no structure information is provided in the metadata, it is queried from PubChem by Name search. Then compute unique compound counts per ionization mode and the set intersection of positive- and negative-mode detections. Validate the breakdown against the reference values: 30,008 unique small molecules total, >12,200 positive-only, >3,400 negative-only, and ~14,300 detected in both modes, writing the result to detection_statistics.csv.

## Related tools

- **mzmine** (Open-source software used to process flow injection Orbitrap MSn data and produce per-file compound detections for each ionization mode) — https://github.com/mzmine/mzmine
- **ThermoRawFileParser** (Converts Thermo .raw acquisition files to open formats (mzML, indexed mzML, MGF, Parquet) before feature processing) — https://github.com/CompOmics/ThermoRawFileParser
- **Prefect 2** (Orchestrates the metadata cleanup flow (metadata_cleanup_prefect.py, jobs.py) that produces the cleaned metadata table used as the join target) — https://github.com/corinnabrungs/msn_tree_library
- **Orbitrap ID-X** (Instrument whose acquisition sequences (via Xcalibur) generate the data; metadata sheets must carry the unique sample id, plate name (plate_id) and vial/well location it expects)

## Examples

```
prefect deploy metadata_cleanup_prefect:cleanup_file --name local-deploy --pool local-work; prefect worker start --pool local-work
```

## Evaluation signals

- Total mapped unique compounds equals ~30,008 small molecules (matches MSnLib abstract figure)
- Ionization-mode breakdown matches reference values: >12,200 positive-only, >3,400 negative-only, ~14,300 in both modes
- Set intersection of positive- and negative-mode compound IDs equals the 'both modes' count (union of the three categories sums to the total)
- detection_statistics.csv is produced with one row per ionization-mode category and internally consistent counts
- Every feature in the mzmine feature lists either maps to a compound ID or is explicitly accounted for as unmapped (no silent drops)

## Limitations

- Join quality depends on the metadata following the shared template with the same column names and minimum needed information; non-conforming sheets will fail to map
- Structure information missing from metadata is only resolved via PubChem Name search, which can mis-resolve ambiguous names
- Some external databases (Dictionary of Natural Products, DrugBank) require local files or special access and must be enabled/disabled via jobs.py, otherwise mapping enrichment is incomplete

## Evidence

- [abstract] a machine learning-ready open resource of >2 million spectra in MS n trees of 30,008 unique small molecules: "a machine learning-ready open resource of >2 million spectra in MS n trees of 30,008 unique small molecules"
- [abstract] built with a high-throughput data acquisition and processing pipeline in the open-source software mzmine: "built with a high-throughput data acquisition and processing pipeline in the open-source software mzmine"
- [methods] The metadatasheet needs to hava a unique sample id, a plate name (batch identifier) as plate_id and the vial or well location: "The sequence creation is setup for the Orbitrap ID-X (Xcalibur). The metadatasheet needs to hava a
  unique sample id, a plate name (batch identifier) as plate_id and the vial or well location as"
- [readme] If no structure information is provided, it is queried from PubChem by Name search.: "If no
structure information is provided, it is queried from PubChem by Name search."
- [readme] Please use the [template]...for your metadata for having same column names and minimum needed information for the query.: "Please use the
[template](https://docs.google.com/spreadsheets/d/1v6_IlGS3VgycGc-mSSdNeocY-CFXpONVZbuh3XNLX2E/edit?usp=sharing)
for your metadata for having same column names and minimum needed"
