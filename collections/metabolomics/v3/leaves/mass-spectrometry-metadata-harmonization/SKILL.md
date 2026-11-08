---
name: mass-spectrometry-metadata-harmonization
description: Use when a public MS/MS spectral library (e.g. GNPS, MassBank, MoNA)
  has inconsistent or missing metadata — compound-name-only annotations, mismatching
  SMILES/InChI/InChIKey, wrong adducts or parent masses — and must be cleaned reproducibly.
when_to_use_negative:
- When plausibility checks must consider both metadata and measured fragments — fragment-matching
  filters (checking if fragments match the given annotation) are not implemented,
  so wrong chemical annotations consistent with the measured mass go unnoticed
- When instrument type or collision energies need standardization — these metadata
  fields are not yet cleaned by current matchms filters
- When spectra already have complete, mutually consistent SMILES/InChI/InChIKey annotations
  — the repair filters would add runtime without value
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3318
tools:
- name: matchms
  role: Provides the Pipeline/SpectrumProcessor, the fixed global filter order, YAML
    workflow serialization and processing-report generation for library cleaning
  repo: github.com/matchms/matchms
- name: RDKit
  role: Loads and cross-compares SMILES, InChI and InChIKey annotations to validate
    structure annotations and remove incomplete/mismatching spectra
- name: PubChem
  role: Source from which canonical SMILES, InChI and InChIKey are derived when annotations
    are only compound names (matched against the given parent mass)
- name: Python
  role: Runtime language in which the matchms package imports, processes, cleans and
    compares the mass spectrometry data
  repo: github.com/matchms/matchms
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-024-00878-1
    title: Reproducible MS/MS library cleaning pipeline in matchms
  source_content_hash: sha256:89b22cf82f49171e8151788a277c0bfe5446d2d2a44a2429c206aab584417cd5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PubChem
  - Python
  - RDKit
  - matchms
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# mass-spectrometry-metadata-harmonization

## Summary

A matchms-based library-cleaning pipeline (version 0.26.4) that harmonizes, derives and repairs mass spectral metadata — SMILES/InChI/InChIKey annotations, adducts and parent masses — while automatically generating a YAML settings file and a per-filter processing report for reproducibility.

## When to use

Use when a public MS/MS spectral library (e.g. GNPS, MassBank, MoNA) has inconsistent or missing metadata — compound-name-only annotations, mismatching SMILES/InChI/InChIKey, wrong adducts or parent masses — and must be cleaned reproducibly.

## When NOT to use

- When plausibility checks must consider both metadata and measured fragments — fragment-matching filters (checking if fragments match the given annotation) are not implemented, so wrong chemical annotations consistent with the measured mass go unnoticed
- When instrument type or collision energies need standardization — these metadata fields are not yet cleaned by current matchms filters
- When spectra already have complete, mutually consistent SMILES/InChI/InChIKey annotations — the repair filters would add runtime without value

## Inputs

- Mass spectral library file in a supported format (mzML, mzXML, msp, MGF, JSON, metabolomics-USI)
- Library metadata records with compound names, SMILES, InChI, InChIKey, adduct and parent-mass fields
- matchms library-cleaning configuration (YAML) specifying the filter set and settings

## Outputs

- Cleaned spectral library file with harmonized/repaired metadata
- Automatically generated YAML workflow file containing all settings and filters used
- Processing report giving per-filter counts of removed spectra, changed metadata and changed peaks

## How to apply

Run matchms 0.26.4's SpectrumProcessor/Pipeline with the library-cleaning preset, which applies the fixed global filter order unchanged across libraries: basic metadata harmonization, default filters that derive missing metadata, then repair filters. Derive canonical SMILES, InChI and InChIKey from PubChem when annotations are only compound names ('Derive Annotation From Compound Name'); repair salts ('Repair Smiles Of Salts' by checking whether the parent mass matches separate or combined salt ions in the SMILES), correct parent masses computed from molar instead of monoisotopic mass ('Repair Parent Mass Is Molar Mass'), correct adducts from SMILES and precursor m/z ('Repair Adduct And Parent Mass Based On Smiles'), reconcile mismatching annotation fields ('Repair Not Matching Annotation'), and finally remove spectra whose SMILES/InChI/InChIKey (loaded and compared via RDKit) are incomplete or non-matching ('Require Valid Annotation'). Let the pipeline automatically write a YAML file capturing all settings and a processing report enumerating per-filter counts of removed spectra and changed metadata/peaks. Decision point: choose basic, default, or library-cleaning filter sets depending on whether annotation repair and complete annotations are required; the identical YAML configuration can be rerun on other libraries such as MassBank and MoNA.

## Related tools

- **matchms** (Provides the Pipeline/SpectrumProcessor, the fixed global filter order, YAML workflow serialization and processing-report generation for library cleaning) — github.com/matchms/matchms
- **RDKit** (Loads and cross-compares SMILES, InChI and InChIKey annotations to validate structure annotations and remove incomplete/mismatching spectra)
- **PubChem** (Source from which canonical SMILES, InChI and InChIKey are derived when annotations are only compound names (matched against the given parent mass))
- **Python** (Runtime language in which the matchms package imports, processes, cleans and compares the mass spectrometry data) — github.com/matchms/matchms

## Examples

```
from matchms import Pipeline; pipeline = Pipeline("library_cleaning.yaml"); pipeline.run("massbank_library.msp")
```

## Evaluation signals

- A YAML workflow file was automatically generated containing all settings and filters used, enabling exact replication of the run
- The processing report enumerates the filters applied in the fixed global order with per-filter counts of removed spectra and changed metadata/peaks, comparable to supplementary report tables (Tables S3–S5)
- Spectra whose SMILES, InChI and InChIKey are incomplete or do not match (checked via RDKit) were removed rather than retained with inconsistent annotations
- Repaired parent masses correspond to monoisotopic (not molar) mass and to separate or combined salt ions for salt-containing SMILES
- A sensitivity/error-rate estimate can be obtained by removing SMILES annotations from selected spectra and checking that they are correctly re-derived from compound names via PubChem

## Limitations

- Current filters lack plausibility checks that consider both metadata and measured fragments, so wrong chemical annotations consistent with the measured mass go unnoticed
- Instrument type and collision energies are important fields that might require additional standardization but are not yet cleaned by current matchms filters
- A validator tool for automatic curation during upload to public libraries is proposed as future work, not built
- Filters operate on metadata/annotations, not peaks; running the pipeline on the 500,569-spectra GNPS library took 6 h and 45 min

## Evidence

- [methods] Version and runtime grounding: "Matchms version 0.26.4 was used to run these pipelines."
- [methods] Annotation validation via RDKit: "SMILES, InChI and InChIKey are loaded by RDKit [18] and compared to each other. If one of these is not complete or does not match the spectrum will be removed."
- [methods] Reproducibility via YAML: "it automatically creates a YAML file containing all settings and filters used."
- [discussion] Filter combination rationale: "The pipeline combines various filter functions to harmonize, derive and repair metadataor discard spectra with incorrect annotations."
- [readme] Package scope: "Matchms is a versatile open-source Python package developed for importing, processing, cleaning, and comparing mass spectrometry data (MS/MS)."
- [readme] Pipeline class and YAML support: [evidence span withheld — anchor status: external_doc]
