---
name: spectral-metadata-harmonization
description: Use when a public MS/MS spectral library (e.g. GNPS, MassBank, MoNA in
  msp/MGF) has incomplete or inconsistent metadata — compound names without SMILES,
  mismatched adducts, or wrong parent masses — before library searching or ML training.
when_to_use_negative:
- When the goal is plausibility checking that measured fragments match the annotation
  — the article states such fragment-matching filters are only a possible future expansion,
  not implemented
- When instrument type or collision energies must be standardized — these fields are
  not yet cleaned by current matchms filters
- When spectra already have complete, RDKit-validated annotations that pass the requirement
  filters — the repair filters add runtime (6 h 45 min on 500,569 spectra) with little
  to repair
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: matchms
  role: Python package implementing the Pipeline class, default/repair/requirement
    filters, YAML workflow export, and processing report generation (version 0.26.4
    used)
  repo: https://github.com/matchms/matchms
- name: RDKit
  role: Loads and cross-compares SMILES, InChI and InChIKey to validate annotations
    and remove spectra with incomplete or non-matching structures
- name: PubChem
  role: External lookup from which the Derive Annotation From Compound Name filter
    fetches canonical SMILES, InChI and InChIKey for compound-name-only entries
- name: bioconda
  role: Conda channel providing the installable matchms recipe for reproducible environments
  repo: https://github.com/bioconda/bioconda-recipes
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1186/s13321-024-00878-1
    title: Reproducible MS/MS library cleaning pipeline in matchms
  source_content_hash: sha256:bd7639f85cfd433f554dbaee13d9bc0eed07555afad73cfa081b0f658cf37f84
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - PubChem
  - Python
  - RDKit
  - bioconda
  - matchms
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# spectral-metadata-harmonization

## Summary

Harmonizes, derives, and repairs MS/MS spectral library metadata (SMILES/InChI/InChIKey annotations, adducts, precursor m/z, parent mass) using the matchms library-cleaning pipeline of default and repair filters in a fixed global order.

## When to use

Use when a public MS/MS spectral library (e.g. GNPS, MassBank, MoNA in msp/MGF) has incomplete or inconsistent metadata — compound names without SMILES, mismatched adducts, or wrong parent masses — before library searching or ML training.

## When NOT to use

- When the goal is plausibility checking that measured fragments match the annotation — the article states such fragment-matching filters are only a possible future expansion, not implemented
- When instrument type or collision energies must be standardized — these fields are not yet cleaned by current matchms filters
- When spectra already have complete, RDKit-validated annotations that pass the requirement filters — the repair filters add runtime (6 h 45 min on 500,569 spectra) with little to repair

## Inputs

- Public MS/MS spectral library (e.g. GNPS public library, no-propagated version, downloaded 21-08-2023; also MassBank, MoNA) in msp, mzML, mzXML, MGF, JSON, or metabolomics-USI format
- Spectra with metadata fields: compound name, SMILES, InChI, InChIKey, adduct, precursor m/z, parent mass, ionmode
- GNPS error-estimation subset with modified adduct metadata (for filter error-rate checks)

## Outputs

- Cleaned spectral library (448,485 curated GNPS spectra, deposited on Zenodo)
- Auto-generated YAML file containing all settings and filters used, rerunnable for reproducibility
- Processing report counting per filter how many spectra were removed, had metadata changed, or had peaks changed
- Per-spectrum adduct repair outcomes for error-rate estimation (e.g. repair_adduct_report.json)

## How to apply

Build a matchms (version 0.26.4) workflow via the create_workflow function running all default filters plus the repair filters (Derive Annotation From Compound Name via PubChem, Repair Smiles Of Salts, Repair Parent Mass Is Molar Mass, Repair Adduct And Parent Mass Based On Smiles, Repair Not Matching Annotation) followed by requirement filters (Require Valid Annotation via RDKit, Require Parent Mass Match Smiles, Require Matching Adduct/Precursor m/z/Parent Mass, Require Matching Adduct And Ionmode, Require Minimum Number Of High Peaks). The rationale is to run dependent filters after their prerequisites in a predefined global order: names are first resolved to canonical SMILES via PubChem (only if the derived structure matches the given parent mass), then salts, molar-vs-monoisotopic parent mass errors, and adduct/parent-mass inconsistencies are repaired using SMILES plus precursor m/z, and finally spectra whose annotations still fail RDKit validation are removed. Collect the auto-generated YAML file (all settings and filters, rerunnable for reproducibility) and the processing report that counts, per filter, how many spectra were removed, had metadata changed, or had peaks changed. On the 21-08-2023 GNPS no-propagated library of 500,569 spectra this ran in 6 h 45 min, removing 31,758 spectra and repairing metadata for 52,084 spectra.

## Related tools

- **matchms** (Python package implementing the Pipeline class, default/repair/requirement filters, YAML workflow export, and processing report generation (version 0.26.4 used)) — https://github.com/matchms/matchms
- **RDKit** (Loads and cross-compares SMILES, InChI and InChIKey to validate annotations and remove spectra with incomplete or non-matching structures)
- **PubChem** (External lookup from which the Derive Annotation From Compound Name filter fetches canonical SMILES, InChI and InChIKey for compound-name-only entries)
- **bioconda** (Conda channel providing the installable matchms recipe for reproducible environments) — https://github.com/bioconda/bioconda-recipes

## Examples

```
from matchms.importing import load_from_msp; from matchms.filtering import repair_adduct_based_on_smiles; spectra = load_from_msp("gnps_library.msp"); repaired = [repair_adduct_based_on_smiles(s) for s in spectra]
```

## Evaluation signals

- Processing report counts match the reported flow: 500,569 input spectra and exactly 31,758 removed with repair functions enabled (vs 83,843 without repair), i.e. 52,084 repaired, 448,485 curated output
- For repair_adduct_based_on_smiles, ≥99.98% of spectra get a derived/confirmed adduct and residual incorrect adducts stay at ≤0.024%
- Auto-generated YAML file contains the full filter list and settings and can be rerun to reproduce the pipeline
- Every retained spectrum passes Require Valid Annotation: SMILES, InChI and InChIKey loaded by RDKit are complete and mutually consistent
- PubChem-derived SMILES are accepted only when they match the given parent mass

## Limitations

- Current filters lack plausibility checks combining metadata and measured fragments — wrong chemical annotations consistent with the measured mass go unnoticed
- Instrument type and collision energies are not yet standardized by matchms filters
- Pipeline is computationally heavy: 6 h 45 min on 500,569 GNPS spectra
- Name-to-structure derivation via PubChem is conditional on the derived structure matching the given parent mass, so ambiguous or wrong names may not resolve
- A validator tool for automatic curation during upload to public libraries is proposed as future work, not built

## Evidence

- [methods] Matchms version 0.26.4 was used to run these pipelines.: "Matchms version 0.26.4 was used to run these pipelines."
- [methods] Many annotations in the GNPS library are only compound names without SMILES. This filter derives the canonical SMILES, InChI and InChIKey from PubChem.: "Many annotations in the GNPS library are only compound names without SMILES. This filter derives the canonical SMILES, InChI and InChIKey from PubChem."
- [results] Before cleaning, the GNPS library contained 500,569 spectra.: "Before cleaning, the GNPS library contained 500,569 spectra."
- [discussion] the instrument type or collision energies could be important fields that might require additional standardization: "the instrument type or collision energies could be important fields that might require additional standardization"
- [readme] Matchms is a versatile open-source Python package developed for importing, processing, cleaning, and comparing mass spectrometry data (MS/MS).: "Matchms is a versatile open-source Python package developed for importing, processing, cleaning, and comparing mass spectrometry data (MS/MS)."
