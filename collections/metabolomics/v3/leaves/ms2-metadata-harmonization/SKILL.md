---
name: ms2-metadata-harmonization
description: Use when MS/MS library entries (e.g. GNPS .msp downloads) contain inconsistent
  or missing metadata — mismatched SMILES/InChI/InChIKey, salt-form annotations, molar-mass-vs-monoisotopic
  parent mass errors, or wrong adducts — and must be cleaned reproducibly.
when_to_use_negative:
- Input is already a fully cleaned library with complete, mutually consistent SMILES/InChI/InChIKey
  annotations — re-running repairs may not change anything and the pipeline's Require
  Valid Annotation step could remove entries unnecessarily
- The goal is validating whether measured fragments match the chemical annotation
  — the article explicitly notes such fragment-matching filters are only a possible
  future expansion, not implemented
- The task requires standardizing fields like instrument type or collision energies
  — the article states these are not yet cleaned/standardized by current matchms filters
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: matchms
  role: Python framework providing SpectrumProcessor, the filter presets (basic/default/library
    cleaning), processing report and YAML generation; version 0.26.4 used to run the
    GNPS cleaning pipelines
  repo: https://github.com/matchms/matchms
- name: RDKit
  role: Loads SMILES, InChI and InChIKey annotations and compares them; spectra with
    incomplete or non-matching structure fields are removed by Require Valid Annotation
- name: PubChem
  role: External lookup service from which Derive Annotation From Compound Name retrieves
    canonical SMILES, InChI and InChIKey when only a compound name is present
- name: GNPS
  role: Ecosystem into which the matchms pipeline is integrated; its public library
    is cleaned every 24 h with matchms and made available for download
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-024-00878-1
    title: Reproducible MS/MS library cleaning pipeline in matchms
  source_content_hash: sha256:649264cb03ab9d9d9b31dd3aa0983ed95a9e0f3936ed6efa73a5836aafbefaac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - GNPS
  - PubChem
  - Python
  - RDKit
  - matchms
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 22
---

# ms2-metadata-harmonization

## Summary

A matchms-based skill for harmonizing, deriving, and repairing MS/MS spectrum metadata (SMILES/InChI/InChIKey annotations, adducts, parent masses) so that public spectral libraries like GNPS become computer-interpretable and reproducibly cleaned.

## When to use

Use when MS/MS library entries (e.g. GNPS .msp downloads) contain inconsistent or missing metadata — mismatched SMILES/InChI/InChIKey, salt-form annotations, molar-mass-vs-monoisotopic parent mass errors, or wrong adducts — and must be cleaned reproducibly.

## When NOT to use

- Input is already a fully cleaned library with complete, mutually consistent SMILES/InChI/InChIKey annotations — re-running repairs may not change anything and the pipeline's Require Valid Annotation step could remove entries unnecessarily
- The goal is validating whether measured fragments match the chemical annotation — the article explicitly notes such fragment-matching filters are only a possible future expansion, not implemented
- The task requires standardizing fields like instrument type or collision energies — the article states these are not yet cleaned/standardized by current matchms filters

## Inputs

- Raw MS/MS spectral library in msp, mzML, mzXML, MGF, JSON, or metabolomics-USI format (e.g. GNPS public library downloaded 21-08-2023, 500,569 spectra)
- matchms filter configuration (SpectrumProcessor preset: basic, default, or library cleaning) or a YAML workflow file
- PubChem connectivity for deriving annotations from compound names

## Outputs

- Cleaned spectral library with harmonized metadata and complete valid annotations (final cleaned GNPS library contains 448,485 spectra, shared on Zenodo)
- Automatically generated YAML file with all settings and filters used
- Processing report (e.g. cond_gnps_no_repair_report.txt) giving per-filter counts of spectra removed, metadata changed, and peaks changed

## How to apply

Load the raw spectra (e.g. the GNPS public mass spectral library of 500,569 spectra) with matchms 0.26.4 into a SpectrumProcessor and apply the layered filter stacks: 'Basic filters' runs basic metadata harmonization, 'Default filters' additionally derives missing metadata from other fields, and the 'Library cleaning' preset runs all default filters plus the repair functions (Repair Smiles Of Salts, Repair Parent Mass Is Molar Mass, Repair Adduct And Parent Mass Based On Smiles, Repair Not Matching Annotation, Derive Annotation From Compound Name) before Require Valid Annotation. Structure fields are loaded and cross-checked by RDKit, and Derive Annotation From Compound Name fetches canonical SMILES/InChI/InChIKey from PubChem, deriving the SMILES only if it matches the given parent mass. The pipeline automatically writes a YAML file with all settings and a processing report counting spectra changed, metadata changed, and peaks changed per filter; spectra failing validation (e.g. annotation fields not matching per RDKit) are removed. To quantify the repair functions' benefit, run the same pipeline once with and once without the repair filters and compare the spectra-removed counts in the two reports (31,758 with repairs vs 83,843 without on the 21-08-2023 GNPS download).

## Related tools

- **matchms** (Python framework providing SpectrumProcessor, the filter presets (basic/default/library cleaning), processing report and YAML generation; version 0.26.4 used to run the GNPS cleaning pipelines) — https://github.com/matchms/matchms
- **RDKit** (Loads SMILES, InChI and InChIKey annotations and compares them; spectra with incomplete or non-matching structure fields are removed by Require Valid Annotation)
- **PubChem** (External lookup service from which Derive Annotation From Compound Name retrieves canonical SMILES, InChI and InChIKey when only a compound name is present)
- **GNPS** (Ecosystem into which the matchms pipeline is integrated; its public library is cleaned every 24 h with matchms and made available for download)

## Examples

```
from matchms import SpectrumProcessor; processor = SpectrumProcessor("library_cleaning"); cleaned, report = processor.run(spectra); report.save("cond_gnps_no_repair_report.txt")
```

## Evaluation signals

- The processing report's spectra-removed count in the no-repair run (83,843 on GNPS) must be strictly greater than the with-repair reference count (31,758), matching the paper's claim that repair functions reduce removals
- Total library size after cleaning: 500,569 input spectra should yield 448,485 curated spectra with the full library-cleaning pipeline
- The run must auto-generate a YAML file whose settings/filters reproduce the pipeline, enabling byte-level re-execution and reproducibility checks
- Every spectrum retained by Require Valid Annotation must have SMILES, InChI and InChIKey that RDKit parses and that agree with each other
- The processing report must contain per-filter counts of spectra changed, metadata changed, and peaks changed — absence of the report indicates the pipeline was not run as specified

## Limitations

- Wrong chemical annotations that are consistent with the measured mass go unnoticed because no plausibility checks consider both metadata and measured fragments
- Instrument type and collision-energy metadata fields are not yet standardized by current matchms filters
- A validator tool for automatic curation during upload to public libraries is only proposed as future work; processing-report examples are only in supplementary tables (S1/S2)

## Evidence

- [methods] Library cleaning. Runs all default filters, but in addition repairs errors in the annotations and requires complete annotations: "Library cleaning. Runs all default filters, but in addition repairs errors in the annotations and requires complete annotations"
- [methods] Matchms version 0.26.4 was used to run these pipelines.: "Matchms version 0.26.4 was used to run these pipelines."
- [methods] SMILES, InChI and InChIKey are loaded by RDKit [18] and compared to each other. If one of these is not complete or does not match the spectrum will be removed.: "SMILES, InChI and InChIKey are loaded by RDKit [18] and compared to each other. If one of these is not complete or does not match the spectrum will be removed."
- [methods] it automatically creates a YAML file containing all settings and filters used.: "it automatically creates a YAML file containing all settings and filters used."
- [readme] The matchms package provides functionality to import, process, clean, and compare mass spectrometry data.: "Matchms is a versatile open-source Python package developed for importing, processing, cleaning, and comparing mass spectrometry data (MS/MS)."
