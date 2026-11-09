---
name: gnps-library-spectrum-processing
description: 'Use when a public GNPS/MassBank/MoNA .msp library needs reproducible
  cleaning before library matching or machine-learning training: metadata harmonization,
  adduct/parent-mass repair, annotation validation.'
when_to_use_negative:
- The input is already a cleaned library (e.g. the Zenodo-published cleaned GNPS version
  with 448,485 spectra); re-running the repair filters adds no information.
- 'You need plausibility checks that consider measured fragments, not just metadata:
  wrong chemical annotations consistent with the measured mass will go unnoticed because
  fragment-matching filters are not implemented.'
- You need cleaned instrument-type or collision-energy metadata; current matchms filters
  do not yet standardize these fields.
- The task is interactive curation during upload to a public library rather than offline
  cleaning; a dedicated validator tool for that is only proposed as future work.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: matchms
  role: Python framework supplying the Pipeline class, the library-cleaning filters
    (repair/derive/require), .msp import/export, and processing-report generation;
    version 0.26.4 used in the article
  repo: https://github.com/matchms/matchms
- name: RDKit
  role: Loads and compares SMILES, InChI and InChIKey annotations during 'Require
    Valid Annotation' and adduct-repair steps
- name: matchms
  role: MS/MS spectral libraries at GNPS are cleaned every 24 h with matchms and made
    available for download
  repo: https://github.com/matchms/matchms
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-024-00878-1
    title: Reproducible MS/MS library cleaning pipeline in matchms
  source_content_hash: sha256:971dc77a7afb026ba6a08e4a010bfc6824466f4cdf81c647d0d259719d00227c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - Python
  - RDKit
  - matchms
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# gnps-library-spectrum-processing

## Summary

Run the matchms library-cleaning pipeline over annotated MS/MS library spectra (e.g. the GNPS public library) to harmonize metadata, derive missing annotations from PubChem, and repair or discard inconsistent structure annotations.

## When to use

Use when a public GNPS/MassBank/MoNA .msp library needs reproducible cleaning before library matching or machine-learning training: metadata harmonization, adduct/parent-mass repair, annotation validation.

## When NOT to use

- The input is already a cleaned library (e.g. the Zenodo-published cleaned GNPS version with 448,485 spectra); re-running the repair filters adds no information.
- You need plausibility checks that consider measured fragments, not just metadata: wrong chemical annotations consistent with the measured mass will go unnoticed because fragment-matching filters are not implemented.
- You need cleaned instrument-type or collision-energy metadata; current matchms filters do not yet standardize these fields.
- The task is interactive curation during upload to a public library rather than offline cleaning; a dedicated validator tool for that is only proposed as future work.

## Inputs

- Annotated MS/MS spectral library in .msp (or mzML, mzXML, MGF, JSON) format, e.g. the GNPS public library downloaded 21-08-2023
- YAML workflow file specifying the library-cleaning filter set (or the built-in pipeline defaults)

## Outputs

- Cleaned MS/MS spectral library file with harmonized metadata and validated annotations
- Automatically generated YAML file containing all settings and filters used
- Processing report giving the number of spectra changed by each filter

## How to apply

Import the .msp library with matchms 0.26.4 and apply the 'library cleaning' pipeline, which runs all default filters (basic metadata harmonization plus derivation of missing metadata) and additionally repairs annotation errors and requires complete annotations. Key repair filters include 'Derive Annotation From Compound Name' (canonical SMILES/InChI/InChIKey from PubChem, accepting only name matches consistent with the given parent mass), 'Repair Smiles Of Salts' (test whether the parent mass matches separate or combined salt ions in the SMILES), 'Repair Parent Mass Is Molar Mass' (replace molar mass with monoisotopic mass), 'Repair Adduct And Parent Mass Based On Smiles' (select the adduct that explains the precursor m/z given the SMILES), and 'Repair Not Matching Annotation' (reconcile SMILES/InChI/InChIKey/parent mass conflicts), followed by 'Require Valid Annotation' where RDKit loads SMILES, InChI and InChIKey and spectra with incomplete or mismatched annotations are removed. The pipeline automatically writes a YAML file recording all settings and filters used and a processing report counting how many spectra each filter changed. On the GNPS library of 500,569 spectra the run took about 6 h 45 min and yielded 448,485 curated spectra, so budget runtime accordingly and compare the report's filter-level change counts against expectations for your library.

## Related tools

- **matchms** (Python framework supplying the Pipeline class, the library-cleaning filters (repair/derive/require), .msp import/export, and processing-report generation; version 0.26.4 used in the article) — https://github.com/matchms/matchms
- **RDKit** (Loads and compares SMILES, InChI and InChIKey annotations during 'Require Valid Annotation' and adduct-repair steps)
- **matchms** (MS/MS spectral libraries at GNPS are cleaned every 24 h with matchms and made available for download) — https://github.com/matchms/matchms

## Examples

```
from matchms.pipeline import Pipeline; p = Pipeline.from_yaml('library_cleaning.yaml'); p.run()
```

## Evaluation signals

- The processing report counts spectra changed by each filter; e.g. for 'Repair adduct and parent mass based on SMILES' expect the adduct to be derivable for 99.98% of spectra with only 0.024% left incorrect after repair.
- The automatically generated YAML file must list every filter and setting actually used, allowing byte-level comparison of the workflow against the reported Supplementary Table S1 pipeline.
- Compare spectra counts: the pipeline should drop spectra with incomplete annotations; on the GNPS library 500,569 input spectra reduced to 448,485 output spectra.
- 'Require Valid Annotation' should leave no spectrum whose RDKit-parsed SMILES, InChI and InChIKey disagree with each other or the precursor metadata.
- Re-running the same YAML on the same input should produce a reproducible cleaned library, since the pipeline is designed for reproducibility and matchms 0.26.4 pinning.

## Limitations

- Filters only check metadata consistency; wrong chemical annotations that are consistent with the measured mass go unnoticed (no fragment-matching plausibility checks yet).
- Instrument type and collision-energy metadata are not yet cleaned or standardized by current matchms filters.
- Processing reports are documented in supplementary tables (Table S1/S2) and the Zenodo record for the cleaned library is cited only as reference [5] in the extracted text, so exact artifact IDs may need lookup.
- Running the full pipeline on ~500k spectra takes hours (6 h 45 min for GNPS); interactive use is impractical.

## Evidence

- [methods] library cleaning runs all default filters and repairs annotations: "Library cleaning. Runs all default filters, but in addition repairs errors in the annotations and requires complete annotations"
- [methods] metadata derivation from PubChem by compound name: "This filter derives the canonical SMILES, InChI and InChIKey from PubChem."
- [methods] adduct repair based on SMILES and precursor m/z: "Corrects the adduct of a spectrum based on the SMILES and precursor m/z."
- [methods] RDKit validation of annotations: "SMILES, InChI and InChIKey are loaded by RDKit [18] and compared to each other."
- [methods] YAML reproducibility artifact: "it automatically creates a YAML file containing all settings and filters used."
- [results] GNPS run scale and duration: "Running the machms pipeline with the filters given in Supplementary Table S1 on the GNPS library of 500,569 spectra took 6 h and 45 min."
- [results] cleaned library size: "The final cleaned GNPS public mass spectral library contains 448.485 curated mass spectra and is available on Zenodo"
- [results] error-rate estimate for adduct repair filter: [evidence span withheld — anchor status: not_found]
- [discussion] no fragment-based plausibility checks: "Wrong chemical annotations that are consistent with the measured mass, for instance, will go unnoticed"
- [discussion] instrument/collision-energy fields uncleaned: "the instrument type or collision energies could be important fields that might require additional standardization"
- [readme] matchms pipeline class and YAML workflows: [evidence span withheld — anchor status: external_doc]
- [readme] supported spectral input formats: [evidence span withheld — anchor status: external_doc]
- [methods] pinning version 0.26.4: "Matchms version 0.26.4 was used to run these pipelines."
- [methods] annual cleaning at GNPS: "MS/MS spectral libraries at GNPS are cleaned every 24 h with matchms and made available for download"
- [discussion] PubChem parent-mass guard on name-to-structure derivation: "the SMILES is only derived from PubChem based on a compound name if it matches the given parent mass"
