---
name: mass-spectral-library-cleaning
description: Use when an annotated MS/MS spectral library (GNPS, MassBank, MoNA) contains
  inconsistent metadata — missing SMILES, salt-formatted SMILES, parent masses computed
  from molar mass, mismatched adducts or annotations — and a curated subset with validated
  structure annotations is needed.
when_to_use_negative:
- Input spectra lack structure annotations entirely — the requirement filters (Require
  Valid Annotation via RDKit) will remove spectra rather than repair them, since Derive
  Annotation From Compound Name only succeeds when a compound name matches the given
  parent mass
- Validation of fragment peaks against the annotation is needed — the pipeline explicitly
  lacks fragment-matching plausibility checks, so wrong chemical annotations consistent
  with the measured mass go unnoticed
- Standardization of instrument type or collision-energy metadata is the goal — these
  fields are not yet cleaned by current matchms filters
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: matchms
  role: Python framework providing the Pipeline class, create_workflow, the default/repair/requirement
    filter functions, YAML export, and processing-report generation
  repo: https://github.com/matchms/matchms
- name: RDKit
  role: Loads SMILES, InChI and InChIKey and compares them to validate structure annotations
    (Require Valid Annotation)
- name: PubChem
  role: Source of canonical SMILES, InChI and InChIKey derived from compound names
    (Derive Annotation From Compound Name), accepted only if the SMILES matches the
    given parent mass
- name: GNPS
  role: Hosts the input public MS/MS library (cleaned every 24 h with matchms) and
    integrates the matchms pipeline into its ecosystem
- name: bioconda-recipes
  role: Conda packaging channel through which matchms versions are distributed
  repo: https://github.com/bioconda/bioconda-recipes
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-024-00878-1
    title: Reproducible MS/MS library cleaning pipeline in matchms
  source_content_hash: sha256:1968018286b27cdbb4382b8803519e21b568bbe3911f67a0265a78a36b0664bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - GNPS
  - PubChem
  - Python
  - RDKit
  - bioconda-recipes
  - matchms
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.967741935483871
n_claims: 31
---

# mass-spectral-library-cleaning

## Summary

A reproducible matchms pipeline (v0.26.4) that cleans annotated MS/MS spectral libraries such as the GNPS public library by running default metadata-harmonization filters plus repair filters (Derive Annotation From Compound Name via PubChem, Repair Smiles Of Salts, Repair Parent Mass Is Molar Mass, Repair Adduct And Parent Mass Based On Smiles, Repair Not Matching Annotation) followed by requirement filters, yielding a curated library ready for library searching and machine-learning training.

## When to use

Use when an annotated MS/MS spectral library (GNPS, MassBank, MoNA) contains inconsistent metadata — missing SMILES, salt-formatted SMILES, parent masses computed from molar mass, mismatched adducts or annotations — and a curated subset with validated structure annotations is needed.

## When NOT to use

- Input spectra lack structure annotations entirely — the requirement filters (Require Valid Annotation via RDKit) will remove spectra rather than repair them, since Derive Annotation From Compound Name only succeeds when a compound name matches the given parent mass
- Validation of fragment peaks against the annotation is needed — the pipeline explicitly lacks fragment-matching plausibility checks, so wrong chemical annotations consistent with the measured mass go unnoticed
- Standardization of instrument type or collision-energy metadata is the goal — these fields are not yet cleaned by current matchms filters

## Inputs

- Public GNPS mass spectral library file (no-propagated version, 21-08-2023 snapshot; 500,569 spectra; also tested on MassBank and MoNA)
- matchms-supported spectral file formats (mzML, mzXML, msp, MGF, JSON, metabolomics-USI)

## Outputs

- Cleaned spectral library (448,485 curated spectra for GNPS), deposited on Zenodo
- Auto-generated YAML file containing all pipeline settings and filters used (rerunnable for reproducibility)
- Processing report counting per filter spectra removed, metadata changed, and peaks changed (examples in Table S1/S2)

## How to apply

Download the target public library (e.g. the no-propagated GNPS library snapshot, 21-08-2023; 500,569 spectra) and build a workflow with matchms's create_workflow function so filters run in a predefined global order, letting dependent filters run after their prerequisites. Run the full library-cleaning pipeline: basic/default metadata-harmonization filters, then repair filters — Derive Annotation From Compound Name (canonical SMILES/InChI/InChIKey from PubChem), Repair Smiles Of Salts, Repair Parent Mass Is Molar Mass, Repair Adduct And Parent Mass Based On Smiles, Repair Not Matching Annotation — then requirement filters (Require Valid Annotation via RDKit, Require Parent Mass Match Smiles, Require Matching Adduct/Precursor m/z/Parent Mass, Require Matching Adduct And Ionmode, Require Minimum Number Of High Peaks). The run automatically emits a YAML file with all settings/filters for reproducibility and a processing report counting per filter how many spectra were removed, had metadata changed, or had peaks changed. On the GNPS library the pipeline took 6 h 45 min; with repairs enabled only 31,758 spectra were removed (83,843 without repair), repairing 52,084 spectra and leaving 448,485 curated spectra.

## Related tools

- **matchms** (Python framework providing the Pipeline class, create_workflow, the default/repair/requirement filter functions, YAML export, and processing-report generation) — https://github.com/matchms/matchms
- **RDKit** (Loads SMILES, InChI and InChIKey and compares them to validate structure annotations (Require Valid Annotation))
- **PubChem** (Source of canonical SMILES, InChI and InChIKey derived from compound names (Derive Annotation From Compound Name), accepted only if the SMILES matches the given parent mass)
- **GNPS** (Hosts the input public MS/MS library (cleaned every 24 h with matchms) and integrates the matchms pipeline into its ecosystem)
- **bioconda-recipes** (Conda packaging channel through which matchms versions are distributed) — https://github.com/bioconda/bioconda-recipes

## Examples

```
from matchms.importing.load_spectra import load_spectra; from matchms.pipeline import Pipeline; pipeline = Pipeline.from_yaml("library_cleaning.yaml"); cleaned = pipeline.run(load_spectra("GNPS_21-08-2023.msp"))
```

## Evaluation signals

- Flow invariant: input spectra count (500,569 for GNPS) minus removed spectra (31,758 with repair functions enabled; 83,843 without) equals the curated library size (448,485); repaired spectra = 52,084
- Processing report exists and shows per-filter counts of spectra removed, metadata changed, and peaks changed
- Auto-generated YAML file present and matches the filter list, so the run can be rerun for reproducibility
- Structure-annotation consistency: SMILES/InChI/InChIKey fields complete and mutually consistent per RDKit; adduct, precursor m/z, and parent mass agree with the SMILES-derived mass
- Error-rate estimate via re-annotation check: remove SMILES from selected spectra, re-derive from compound name, and compare

## Limitations

- No plausibility checks that consider both metadata and measured fragments; wrong chemical annotations consistent with the measured mass go unnoticed — fragment-matching filters are only a possible future expansion
- Instrument type and collision-energy metadata fields are not yet standardized by current matchms filters
- Repair depends on PubChem lookups matching the parent mass; annotations not found in PubChem cannot be derived and spectra are then removed by requirement filters
- No validator tool yet for automatic curation during upload to public libraries (proposed as future work)

## Evidence

- [methods] The public GNPS library was downloaded on 21-08-2023: "The public GNPS library was downloaded on 21-08-2023 (GNPS [13])."
- [methods] Matchms version 0.26.4 was used to run these pipelines: "Matchms version 0.26.4 was used to run these pipelines."
- [methods] SMILES, InChI and InChIKey are loaded by RDKit and compared to each other: "SMILES, InChI and InChIKey are loaded by RDKit [18] and compared to each other."
- [results] Running the pipeline on the GNPS library took 6 h and 45 min: "Running the machms pipeline with the filters given in Supplementary Table S1 on the GNPS library of 500,569 spectra took 6 h and 45 min."
- [results] The final cleaned GNPS public mass spectral library contains 448.485 curated mass spectra: "The final cleaned GNPS public mass spectral library contains 448.485 curated mass spectra and is available on Zenodo [5]."
- [methods] it automatically creates a YAML file containing all settings and filters used: "it automatically creates a YAML file containing all settings and filters used."
- [readme] README: matchms imports, processes, cleans, and compares mass spectrometry data: "Matchms is a versatile open-source Python package developed for importing, processing, cleaning, and comparing mass spectrometry data (MS/MS)."
- [readme] Pipeline class handles complex workflows via yaml files: [evidence span withheld — anchor status: external_doc]
