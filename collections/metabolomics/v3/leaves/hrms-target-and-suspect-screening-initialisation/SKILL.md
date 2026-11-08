---
name: hrms-target-and-suspect-screening-initialisation
description: Use when starting a Shinyscreen HRMS workflow from a YAML config plus
  a compound or mass list CSV, and a validated compound_table.csv with precursor masses
  is needed before extraction/prescreening.
when_to_use_negative:
- The compound lists are already validated and precursor masses already computed —
  proceed directly to the extraction and prescreening stage instead of re-initialising
- The goal is interactive MS1/MS2 chromatogram visualisation, quality assessment,
  or MetFrag integration — these are later Shinyscreen stages, not initialisation
- The user only needs to inspect or QC raw vendor data without a preselected list
  of compounds or masses of interest, which is Shinyscreen's core scope
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0153
tools:
- name: Shinyscreen
  role: R package / Shiny app whose initialisation stage loads, validates, and mass-corrects
    the compound and mass lists
  repo: https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen
- name: yaml
  role: R package used to parse the Shinyscreen YAML configuration file
- name: enviPat
  role: Computes required precursor ion masses from molecular formulas under the configured
    adduct setting
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-025-01044-x
    title: 'Shinyscreen: mass spectrometry data inspection and quality checking utility'
  source_content_hash: sha256:eceeb0ff2bc2d3984e36712ae9d604b3736cb9b4db6339b912e3beab1c27d967
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Shinyscreen
  - enviPat
  - yaml
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# hrms-target-and-suspect-screening-initialisation

## Summary

Initialises a Shinyscreen target/suspect/non-target HRMS screening run by loading and validating the YAML configuration and compound-list CSV, then computing precursor ion masses under the configured adduct. It converts heterogeneous input lists into a single validated compound table ready for extraction and prescreening.

## When to use

Use when starting a Shinyscreen HRMS workflow from a YAML config plus a compound or mass list CSV, and a validated compound_table.csv with precursor masses is needed before extraction/prescreening.

## When NOT to use

- The compound lists are already validated and precursor masses already computed — proceed directly to the extraction and prescreening stage instead of re-initialising
- The goal is interactive MS1/MS2 chromatogram visualisation, quality assessment, or MetFrag integration — these are later Shinyscreen stages, not initialisation
- The user only needs to inspect or QC raw vendor data without a preselected list of compounds or masses of interest, which is Shinyscreen's core scope

## Inputs

- Shinyscreen YAML configuration file (m/z and RT error tolerances, MS1/MS2 intensity thresholds, RT shifts, output/metadata settings, adduct setting)
- Compound list CSV with target/suspect rows carrying name, formula and/or SMILES, and optional retention time
- Non-target mass list CSV rows carrying explicit precursor m/z and optional retention time
- PubChemLite local CSV database downloaded from its Zenodo deposit

## Outputs

- compound_table.csv containing compound identifiers, name, formula and/or SMILES, optional retention times, and calculated precursor ion masses
- Validation flags for target/suspect rows lacking both formula and SMILES

## How to apply

Read the Shinyscreen YAML configuration with the yaml R package, extracting the experimental m/z and retention-time error tolerances, quality-control parameters (MS1 and MS2 intensity thresholds, retention-time shifts), output/metadata settings, and the adduct setting, falling back to the Orbitrap-based defaults where values are absent. Load the compound-list CSV and split entries into target/suspect rows (which supply a molecular formula or SMILES string plus optional retention time) and non-target rows (which supply precursor m/z directly with optional retention time). Validate each row: flag target/suspect entries that have neither a molecular formula nor a SMILES string, and confirm non-target entries carry an explicit precursor mass. For validated structural entries, calculate the required precursor ion masses for extraction under the configured adduct setting using enviPat, which computes adduct masses rapidly from molecular formulas. Cross-check compound names/identifiers against the PubChemLite local CSV database (downloaded from its Zenodo deposit) to confirm identifier consistency, then write the validated table — compound identifiers, name, formula and/or SMILES, optional retention times, and calculated precursor ion masses — to compound_table.csv.

## Related tools

- **Shinyscreen** (R package / Shiny app whose initialisation stage loads, validates, and mass-corrects the compound and mass lists) — https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen
- **yaml** (R package used to parse the Shinyscreen YAML configuration file)
- **enviPat** (Computes required precursor ion masses from molecular formulas under the configured adduct setting)

## Examples

```
config <- yaml::read_yaml("shinyscreen.yml"); compounds <- utils::read.csv("compound_list.csv")
```

## Evaluation signals

- compound_table.csv exists in CSV format in the configured output location
- The table contains the identifier, name, formula/SMILES and precursor-mass fields
- Every target/suspect row either has a structural field with a derived precursor mass or has structural information supplied directly as precursor m/z
- Rows lacking both molecular formula and SMILES are flagged rather than silently mass-calculated
- Compound names/identifiers cross-check consistently against the PubChemLite local CSV database

## Limitations

- Target/suspect entries with neither a molecular formula nor a SMILES string cannot be used for precursor mass calculation and must be flagged or supplied directly as precursor m/z
- Configuration values absent from the YAML fall back to Orbitrap-based defaults, which may not suit instruments with different m/z or retention-time error behaviour
- Identifier cross-checking depends on the local PubChemLite CSV being present and current (downloaded from its Zenodo deposit)

## Evidence

- [abstract] list-based screening: "Its versatile list-based approach supports the curation of data starting from either known or "suspected" compounds (compound list-based screening) or detected masses (mass list-based screening)"
- [methods] initialisation stage: "The core steps of Shinyscreen include initialisation, extraction and prescreening, visualisation and integration with MetFrag for further analysis"
- [methods] precursor mass calculation: "Shinyscreen will automatically calculate required precursor ion masses for extraction, using the given adduct setting"
- [methods] QC parameters: "quality control parameters such as MS1 and MS2 intensity thresholds and retention time shifts"
- [methods] PubChemLite default: "with the current default use set to PubChemLite [31, 32]"
