---
name: mass-spectral-library-import-and-preprocessing
description: Reference for ingesting MS/MS library files (msp, mzML, mzXML, MGF, JSON,
  metabolomics-USI) into matchms and running default-filter preprocessing with automatic
  YAML/report logging before library searching or ML training.
when_to_use_negative:
- Input spectra already have complete, validated annotations and only need similarity
  scoring or fingerprinting rather than cleaning — the cleaning pipeline's repair/derivation
  filters would add unnecessary work
- 'The task requires fragment-matching plausibility checks against measured peaks:
  the article notes current matchms filters ''lack plausibility checks that consider
  both metadata and measured fragments'' — such fragment-matching filters are only
  a possible future expansion'
- 'You need to standardize instrument type or collision-energy metadata: the article
  states these ''could be important fields that might require additional standardization''
  that is not yet implemented'
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: matchms
  role: Python framework that imports the spectral library in standard formats and
    runs the default-filter cleaning pipeline, automatically emitting the YAML workflow
    file and processing report
  repo: https://github.com/matchms/matchms
- name: RDKit
  role: 'Validates structure annotations during cleaning: SMILES, InChI and InChIKey
    are loaded by RDKit and compared to each other, and mismatching spectra are removed'
- name: GNPS
  role: Hosts the public mass spectral library that is the demonstration input, and
    integrates the matchms pipeline so MS/MS spectral libraries are cleaned every
    24 h
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-024-00878-1
    title: Reproducible MS/MS library cleaning pipeline in matchms
  source_content_hash: sha256:ca3c65ac83f0045a46eda7190b67a8e999c66b43201d36180cdb3a67a257bea3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - GNPS
  - Python
  - RDKit
  - matchms
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 23
---

# mass-spectral-library-import-and-preprocessing

## Summary

Import public tandem mass spectrometry (MS/MS) spectral libraries (e.g. GNPS, MassBank, MoNA) into matchms, apply the default metadata filters, and automatically emit a reusable YAML workflow file plus a processing report so each cleaning run is reproducible and auditable.

## When to use

Reference for ingesting MS/MS library files (msp, mzML, mzXML, MGF, JSON, metabolomics-USI) into matchms and running default-filter preprocessing with automatic YAML/report logging before library searching or ML training.

## When NOT to use

- Input spectra already have complete, validated annotations and only need similarity scoring or fingerprinting rather than cleaning — the cleaning pipeline's repair/derivation filters would add unnecessary work
- The task requires fragment-matching plausibility checks against measured peaks: the article notes current matchms filters 'lack plausibility checks that consider both metadata and measured fragments' — such fragment-matching filters are only a possible future expansion
- You need to standardize instrument type or collision-energy metadata: the article states these 'could be important fields that might require additional standardization' that is not yet implemented

## Inputs

- MS/MS spectral library file (msp, mzML, mzXML, MGF, JSON, or metabolomics-USI format), e.g. the public GNPS mass spectral library or the NIST library
- matchms library-cleaning pipeline configuration (default filter set: metadata harmonization, derivation of missing metadata from other fields, adduct/precursor m/z handling, annotation validation)

## Outputs

- Cleaned spectral library with harmonized metadata (derived SMILES/InChI/InChIKey, corrected adducts and parent masses)
- workflow.yaml file recording all settings and filters used for the cleaning run
- processing report (processing_report.csv) summarizing the effect of each filter: spectra removed, metadata changed, peak information changed

## How to apply

Load the spectral library with matchms (version 0.26.4 was used on the GNPS library downloaded 21-08-2023) via the SpectrumProcessor, selecting the 'default filters' preset, which runs basic metadata harmonization and also derives missing metadata from other fields. Matchms then automatically creates a YAML file containing all settings and filters used, which can be reused directly to rerun the cleaning pipeline unchanged. During the same run it generates a processing report (CSV) giving an overview of the number of spectra changed by each filter, differentiating complete removal, metadata changes, and peak-information changes. On the 500,569-spectrum GNPS library the run took 6 h 45 min; the two artifacts should be stored as workflow.yaml and processing_report.csv. Verify reproducibility by re-running from the YAML and confirming the report matches.

## Related tools

- **matchms** (Python framework that imports the spectral library in standard formats and runs the default-filter cleaning pipeline, automatically emitting the YAML workflow file and processing report) — https://github.com/matchms/matchms
- **RDKit** (Validates structure annotations during cleaning: SMILES, InChI and InChIKey are loaded by RDKit and compared to each other, and mismatching spectra are removed)
- **GNPS** (Hosts the public mass spectral library that is the demonstration input, and integrates the matchms pipeline so MS/MS spectral libraries are cleaned every 24 h)

## Examples

```
from matchms.filtering import SpectrumProcessor; pipeline = SpectrumProcessor('default'); cleaned = pipeline.from_file('gnps_library.msp')  # emits workflow.yaml + processing_report.csv
```

## Evaluation signals

- Running the pipeline on the GNPS library (500,569 input spectra) yields 448,485 curated mass spectra after cleaning; a large discrepancy from this count indicates filters were misconfigured
- A workflow.yaml file is present alongside the cleaned library, and re-running the pipeline from that YAML reproduces the same output — confirming reproducibility
- The processing_report.csv exists and shows, per filter, the number of spectra removed, metadata-changed, and peak-information-changed, e.g. spectra removed by 'Require Valid Annotation' where SMILES/InChI/InChIKey did not match
- The cleaned library's annotations pass the consistency check that canonical SMILES, InChI and InChIKey loaded by RDKit all match the spectrum
- Runtime is on the order of hours for a ~500k-spectrum library (6 h 45 min on GNPS); sub-minute times suggest the full default filter set did not run

## Limitations

- The pipeline 'currently still lack[s] plausibility checks that consider both metadata and measured fragments', so wrong chemical annotations that are consistent with the measured mass go unnoticed
- Fragment-matching filters (checking if fragments match the given annotation) are only a possible future expansion, not implemented
- Metadata fields such as instrument type and collision energies are not yet standardized by current matchms filters
- There is no validator tool yet for automatic curation during upload to public libraries — this is proposed future work
- Examples of processing reports are only in supplementary tables (Table S1 and Table S2), and supplementary file 1 appears without content in the article

## Evidence

- [readme] matchms now offers a `Pipeline` class to handle complex workflows. This also allows to create, import, export, or modify workflows using yaml files.: [evidence span withheld — anchor status: external_doc]
- [readme] Matchms supports a range of popular spectral data formats, including mzML, mzXML, msp, metabolomics-USI, MGF, and JSON.: [evidence span withheld — anchor status: external_doc]
- [methods] it automatically creates a YAML file containing all settings and filters used.: "it automatically creates a YAML file containing all settings and filters used."
- [methods] a processing report is generated. This report gives an overview of the number of spectra that were changed in the library by each filter.: "a processing report is generated. This report gives an overview of the number of spectra that were changed in the library by each filter."
- [methods] Default filters. Runs basic metadata harmonization, but also derives missing metadata from other fields: "Default filters. Runs basic metadata harmonization, but also derives missing metadata from other fields"
- [methods] Matchms version 0.26.4 was used to run these pipelines.: "Matchms version 0.26.4 was used to run these pipelines."
- [methods] The matchms pipeline was also run on the Massbank library: "The matchms pipeline was also run on the Massbank library"
- [results] Running the machms pipeline with the filters given in Supplementary Table S1 on the GNPS library of 500,569 spectra took 6 h and 45 min.: "Running the machms pipeline with the filters given in Supplementary Table S1 on the GNPS library of 500,569 spectra took 6 h and 45 min."
- [discussion] they currently still lack plausibility checks that consider both metadata and measured fragments. Wrong chemical annotations that are consistent with the measured mass, for instance, will go unnoticed: "they currently still lack plausibility checks that consider both metadata and measured fragments. Wrong chemical annotations that are consistent with the measured mass, for instance, will go unnoticed"
- [readme] Matchms is a versatile open-source Python package developed for importing, processing, cleaning, and comparing mass spectrometry data (MS/MS).: "Matchms is a versatile open-source Python package developed for importing, processing, cleaning, and comparing mass spectrometry data (MS/MS)."
