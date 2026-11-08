---
name: yaml-configuration-parsing-for-ms-workflows
description: Use when starting a Shinyscreen target/suspect/non-target HRMS prescreening
  run and a YAML configuration plus a compound-list CSV must be turned into validated
  inputs with derived precursor ion masses.
when_to_use_negative:
- The input is raw HRMS vendor data needing inspection or plotting rather than configuration-driven
  initialisation — Shinyscreen's later extraction, prescreening and visualisation
  stages, or low-level tools like mzR and MSnbase, are the appropriate choice.
- The workflow has no compound or mass list of interest and aims at untargeted feature
  detection/deconvolution — tools such as xcms, RforMassSpectrometry or patRoon cover
  that niche.
- No YAML configuration or compound-list CSV exists; this skill presupposes both a
  configuration file and a curated list of compounds or masses.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3364
tools:
- name: yaml
  role: R package used to read the Shinyscreen YAML configuration file and extract
    tolerances, thresholds and adduct settings
- name: Shinyscreen
  role: Application whose initialisation stage consumes the parsed configuration and
    compound list to build the validated compound table
  repo: https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen
- name: enviPat
  role: Computes precursor ion masses from molecular formulas under the adduct setting
    read from the configuration
- name: R
  role: Language in which the configuration parsing, validation and mass calculation
    are implemented
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-025-01044-x
    title: 'Shinyscreen: mass spectrometry data inspection and quality checking utility'
  source_content_hash: sha256:a8ba8cb3868e89c96c4d921a6c1903063e2ff20fbd858a7ca876bd107f1e1eb6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Shinyscreen
  - enviPat
  - yaml
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3364
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# yaml-configuration-parsing-for-ms-workflows

## Summary

Parse a Shinyscreen YAML configuration file to extract experimental m/z and retention-time error tolerances, MS1/MS2 intensity thresholds, retention-time shifts, output/metadata settings, and the adduct setting, falling back to Orbitrap-based defaults where values are absent, as the initialisation step of a list-based HRMS screening workflow.

## When to use

Use when starting a Shinyscreen target/suspect/non-target HRMS prescreening run and a YAML configuration plus a compound-list CSV must be turned into validated inputs with derived precursor ion masses.

## When NOT to use

- The input is raw HRMS vendor data needing inspection or plotting rather than configuration-driven initialisation — Shinyscreen's later extraction, prescreening and visualisation stages, or low-level tools like mzR and MSnbase, are the appropriate choice.
- The workflow has no compound or mass list of interest and aims at untargeted feature detection/deconvolution — tools such as xcms, RforMassSpectrometry or patRoon cover that niche.
- No YAML configuration or compound-list CSV exists; this skill presupposes both a configuration file and a curated list of compounds or masses.

## Inputs

- Shinyscreen YAML configuration file (m/z and RT error tolerances, MS1/MS2 intensity thresholds, retention-time shifts, output/metadata settings, adduct setting)
- Compound list CSV with target/suspect rows (name, molecular formula and/or SMILES, optional retention time) and non-target rows (precursor m/z, optional retention time)
- PubChemLite local CSV database downloaded from its Zenodo deposit

## Outputs

- compound_table.csv containing compound identifiers, name, formula and/or SMILES, optional retention times, and calculated precursor ion masses
- Validation flags for target/suspect rows missing structural fields and non-target rows missing precursor mass

## How to apply

Read the Shinyscreen YAML configuration file using the yaml R package, extracting (a) experimental m/z and retention time errors, (b) quality-control parameters such as MS1 and MS2 intensity thresholds and retention time shifts, (c) output/metadata settings, and (d) the adduct setting, applying Orbitrap-based defaults where values are absent. Load the compound list CSV and split rows into target/suspect entries (molecular formula or SMILES plus optional retention time) and non-target entries (direct precursor m/z plus optional retention time). Validate rows: flag target/suspect entries lacking both a molecular formula and a SMILES string, and confirm non-target entries carry an explicit precursor mass. For validated structural entries, use enviPat to compute precursor ion masses from molecular formulas under the configured adduct setting, cross-check identifiers against the PubChemLite local CSV (downloaded from its Zenodo deposit), and write compound_table.csv. This parsing step is the Shinyscreen 'initialisation' stage, before extraction and prescreening, visualisation, and MetFrag integration.

## Related tools

- **yaml** (R package used to read the Shinyscreen YAML configuration file and extract tolerances, thresholds and adduct settings)
- **Shinyscreen** (Application whose initialisation stage consumes the parsed configuration and compound list to build the validated compound table) — https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen
- **enviPat** (Computes precursor ion masses from molecular formulas under the adduct setting read from the configuration)
- **R** (Language in which the configuration parsing, validation and mass calculation are implemented)

## Examples

```
library(yaml); conf <- yaml::read_yaml("shinyscreen_conf.yml"); mz_tol <- conf$tolerances$mz; rt_shift <- conf$qc$rt_shift
```

## Evaluation signals

- compound_table.csv exists and is valid CSV
- The table contains identifier, name, formula/SMILES and precursor-mass fields
- Every target/suspect row either has a structural field with a derived precursor mass or supplies structural information directly as precursor m/z
- Rows with neither a molecular formula nor a SMILES string are flagged rather than silently passed through
- Non-target entries carry an explicit precursor mass; configured adduct setting is honoured in the enviPat-computed masses

## Limitations

- Quality-control parameters fall back to Orbitrap-based defaults where values are absent, so non-Orbitrap instruments require explicit configuration
- Cross-checking depends on the PubChemLite local CSV being current; the paper notes the default is downloaded from its Zenodo deposit and license (Apache 2.0) but reports no archived code snapshot
- Shinyscreen's focus is exclusively on data exploration, visualisation and automated quality control combined with manual overrides, centered on a preselected list of compounds or masses of interest

## Evidence

- [abstract] Shinyscreen is an R package and Shiny-based web application designed for the exploration, visualization, and quality assessment of raw data from high resolution mass spectrometry instruments.: "Shinyscreen is an R package and Shiny-based web application designed for the exploration, visualization, and quality assessment of raw data from high resolution mass spectrometry instruments."
- [methods] The core steps of Shinyscreen include initialisation, extraction and prescreening, visualisation and integration with MetFrag for further analysis: "The core steps of Shinyscreen include initialisation, extraction and prescreening, visualisation and integration with MetFrag for further analysis"
- [methods] quality control parameters such as MS1 and MS2 intensity thresholds and retention time shifts: "quality control parameters such as MS1 and MS2 intensity thresholds and retention time shifts"
- [methods] Shinyscreen will automatically calculate required precursor ion masses for extraction, using the given adduct setting: "Shinyscreen will automatically calculate required precursor ion masses for extraction, using the given adduct setting"
- [methods] with the current default use set to PubChemLite [31, 32]: "with the current default use set to PubChemLite [31, 32]"
