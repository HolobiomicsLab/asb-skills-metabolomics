---
name: mass-spectrometry-detection-statistics-aggregation
description: Use when raw Thermo .raw / .mzML flow injection acquisitions have been
  processed into per-file feature lists and you need to summarize how many unique
  compounds were detected exclusively in positive mode, exclusively in negative mode,
  or in both ionization modes.
when_to_use_negative:
- Input is already an aggregated count table or published statistic — no re-aggregation
  from raw feature lists is needed.
- Data were acquired by LC-MS/MS rather than flow injection–Orbitrap MSn and per-well
  metadata cannot be joined to features.
- Only a single ionization mode was acquired — the mode-breakdown logic (positive-only
  vs negative-only vs both) cannot be computed.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mzmine
  role: Open-source software used to run the high-throughput data acquisition and
    processing pipeline that produces per-file compound feature lists from the MSn
    acquisitions.
  repo: https://github.com/mzmine/mzmine
- name: ThermoRawFileParser
  role: Converts Thermo RAW mass spectrometer files to common open formats (mzML/MGF/Parquet)
    on all platforms supporting .NET Core, an optional step before feature processing
    in mzmine.
  repo: https://github.com/CompOmics/ThermoRawFileParser
- name: Orbitrap ID-X
  role: Instrument for which the acquisition sequence creation is set up via Xcalibur;
    the resulting .raw/.mzML files are the upstream input to the detection pipeline.
- name: Prefect 2
  role: Runs the metadata_cleanup_prefect.py flow that serves the metadata-cleanup
    service and processes submitted jobs, yielding the cleaned metadata table used
    to join detections to compound identifiers.
  repo: https://github.com/corinnabrungs/msn_tree_library
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41592-025-02813-0
    title: 'MSnLib: efficient generation of open multi-stage fragmentation mass spectral
      libraries'
  source_content_hash: sha256:6c0971712b0b89c0cd99b2b07a7c5a0a75a499619b29e6ae5562e4ee48eedf92
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine
  tools:
  - Orbitrap ID-X
  - Prefect 2
  - ThermoRawFileParser
  - mzmine
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# mass-spectrometry-detection-statistics-aggregation

## Summary

Aggregate per-file compound detections from flow injection–Orbitrap MSn data into unique-compound counts broken down by ionization mode (positive-only, negative-only, and both), producing a validated detection-statistics table such as the 30,008-compound breakdown reported for MSnLib.

## When to use

Use when raw Thermo .raw / .mzML flow injection acquisitions have been processed into per-file feature lists and you need to summarize how many unique compounds were detected exclusively in positive mode, exclusively in negative mode, or in both ionization modes.

## When NOT to use

- Input is already an aggregated count table or published statistic — no re-aggregation from raw feature lists is needed.
- Data were acquired by LC-MS/MS rather than flow injection–Orbitrap MSn and per-well metadata cannot be joined to features.
- Only a single ionization mode was acquired — the mode-breakdown logic (positive-only vs negative-only vs both) cannot be computed.

## Inputs

- Deposited .mzML or Thermo .raw acquisition files (Zenodo 10966280/10966404/10967081; MassIVE MSV000094528)
- Cleaned metadata table following the shared template (unique sample id, plate_id, well/vial location) produced by the metadata cleanup flow
- Per-file feature lists from mzmine processing of each ionization mode

## Outputs

- detection_statistics.csv containing unique compound counts per ionization mode
- Count breakdown: total unique compounds, positive-only, negative-only, and both-mode detections
- Set intersection of positive- and negative-mode compound detections

## How to apply

Start from the deposited .mzML/.raw files (e.g. Zenodo 10966280, 10966404, 10967081; MassIVE MSV000094528) and the cleaned metadata table produced by the metadata_cleanup_prefect.py flow. Convert Thermo .raw files to open formats with ThermoRawFileParser where needed, then process the feature lists in mzmine to obtain per-file compound detections for each ionization mode. Join the feature lists against the cleaned metadata so each detected feature carries a unique compound identifier (structure information is queried from PubChem by Name search when absent). Compute unique-compound counts per ionization mode and the set intersection of positive- and negative-mode detections, then validate against the expected breakdown — for MSnLib: 30,008 unique small molecules total, >12,200 positive-only, >3,400 negative-only, and ~14,300 detected in both modes. Write the count breakdown table to detection_statistics.csv.

## Related tools

- **mzmine** (Open-source software used to run the high-throughput data acquisition and processing pipeline that produces per-file compound feature lists from the MSn acquisitions.) — https://github.com/mzmine/mzmine
- **ThermoRawFileParser** (Converts Thermo RAW mass spectrometer files to common open formats (mzML/MGF/Parquet) on all platforms supporting .NET Core, an optional step before feature processing in mzmine.) — https://github.com/CompOmics/ThermoRawFileParser
- **Orbitrap ID-X** (Instrument for which the acquisition sequence creation is set up via Xcalibur; the resulting .raw/.mzML files are the upstream input to the detection pipeline.)
- **Prefect 2** (Runs the metadata_cleanup_prefect.py flow that serves the metadata-cleanup service and processes submitted jobs, yielding the cleaned metadata table used to join detections to compound identifiers.) — https://github.com/corinnabrungs/msn_tree_library

## Examples

```
prefect deploy metadata_cleanup_prefect:cleanup_file --name local-deploy --pool local-work
```

## Evaluation signals

- The aggregated total of unique compounds matches the reported value of 30,008 for the MSnLib dataset.
- Positive-only detections exceed 12,200 and negative-only detections exceed 3,400.
- The count of compounds detected in both ionization modes is approximately 14,300.
- The three mode categories (positive-only, negative-only, both) sum to the total unique compound count without overlap.
- The breakdown is written to detection_statistics.csv with rows keyed by ionization-mode category.

## Limitations

- Counts depend on the fidelity of the cleaned metadata; the shared template requires the same column names and minimum needed information (unique sample id, plate_id, well/vial location) for the join to succeed.
- Some external databases (Dictionary of Natural Products, DrugBank, Broad Drug Repurposing Hub) need a local file and/or special access, otherwise they must be set to False in jobs.py, which can limit compound annotation coverage.
- Counts are specific to flow injection–Orbitrap MSn data processed with mzmine; re-processing feature lists with different parameters will change the resulting statistics.

## Evidence

- [abstract] MSnLib, a machine learning-ready open resource of >2 million spectra in MSn trees of 30,008 unique small molecules: [evidence span withheld — anchor status: not_found]
- [abstract] built with a high-throughput data acquisition and processing pipeline in the open-source software mzmine: "built with a high-throughput data acquisition and processing pipeline in the open-source software mzmine"
- [methods] Start serving the flow locally by running metadata_cleanup_prefect.py, the service will then run and wait for jobs to be submitted (jobs.py): "Start serving the flow locally by running `metadata_cleanup_prefect.py`, the service will then run
  and wait for jobs to be submitted (jobs.py)"
- [readme] A tool allowing reading Thermo RAW mass spectrometer files and converting to common open formats on all platforms supporting .NET Core.: [evidence span withheld — anchor status: external_doc]
