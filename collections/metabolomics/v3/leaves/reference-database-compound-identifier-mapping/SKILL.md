---
name: reference-database-compound-identifier-mapping
description: 'Use when constructing or validating the compound input for HRMS prescreening:
  mapping compound names, formulas or SMILES (or precursor m/z for non-targets) to
  adduct-specific precursor ion masses and confirming identifiers against PubChemLite.'
when_to_use_negative:
- Input is already a validated compound_table.csv with identifier, structural fields
  and precursor masses present.
- The analysis goal is de novo annotation of unknowns without any preselected compound
  or mass list — Shinyscreen's focus is exclusively on a preselected list of compounds
  or masses of interest.
- Raw vendor files (e.g. Thermo Xcalibur data) need conversion or feature detection
  rather than compound-table preparation; Shinyscreen starts from compounds/masses,
  not vendor acquisition files.
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: Shinyscreen
  role: Package whose initialisation routine validates the compound list, computes
    precursor masses and writes compound_table.csv
  repo: https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen
- name: yaml
  role: R package used to read the Shinyscreen YAML configuration supplying tolerances,
    QC thresholds and the adduct setting
- name: enviPat
  role: Computes adduct-specific precursor ion masses from molecular formulas for
    validated target/suspect entries
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-025-01044-x
    title: 'Shinyscreen: mass spectrometry data inspection and quality checking utility'
  source_content_hash: sha256:1a12ec7c6015bf3b749dcbff078bdd4b32fafbb181c877e51ec63813bacb7d49
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Shinyscreen
  - enviPat
  - yaml
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# reference-database-compound-identifier-mapping

## Summary

Builds the validated compound table that drives Shinyscreen's list-based HRMS prescreening: parsing the YAML config and compound-list CSV, splitting target/suspect vs non-target entries, computing adduct-specific precursor masses with enviPat, and cross-checking compound identifiers against a local reference database (PubChemLite).

## When to use

Use when constructing or validating the compound input for HRMS prescreening: mapping compound names, formulas or SMILES (or precursor m/z for non-targets) to adduct-specific precursor ion masses and confirming identifiers against PubChemLite.

## When NOT to use

- Input is already a validated compound_table.csv with identifier, structural fields and precursor masses present.
- The analysis goal is de novo annotation of unknowns without any preselected compound or mass list — Shinyscreen's focus is exclusively on a preselected list of compounds or masses of interest.
- Raw vendor files (e.g. Thermo Xcalibur data) need conversion or feature detection rather than compound-table preparation; Shinyscreen starts from compounds/masses, not vendor acquisition files.

## Inputs

- Shinyscreen YAML configuration file (m/z and RT error tolerances, MS1/MS2 intensity thresholds, RT shifts, adduct setting)
- Compound list CSV with target/suspect rows (name plus molecular formula or SMILES, optional retention time) and non-target rows (precursor m/z, optional retention time)
- PubChemLite local CSV reference database (downloaded from its Zenodo deposit)

## Outputs

- compound_table.csv containing compound identifiers, name, formula and/or SMILES, optional retention times, and calculated precursor ion masses

## How to apply

Read the Shinyscreen YAML configuration with the yaml R package, extracting the experimental m/z and retention-time error tolerances, the QC parameters (MS1 and MS2 intensity thresholds, retention-time shifts), output/metadata settings and the adduct setting, falling back to Orbitrap-based defaults where values are absent. Load the compound list CSV and split it into target/suspect rows — which must supply a molecular formula or SMILES string plus optional retention time — and non-target rows, which supply precursor m/z directly with optional retention time. Validate each row, flagging target/suspect entries that have neither a formula nor a SMILES, and confirm non-target entries carry an explicit precursor mass. For validated structural entries, compute the required precursor ion masses for extraction under the configured adduct setting using enviPat, which derives adduct masses rapidly from molecular formulas. Cross-check compound names/identifiers against the PubChemLite local CSV database (the current default, downloaded from its Zenodo deposit) to confirm identifier consistency. Write the validated compound table, with identifiers, structural fields, optional retention times and calculated precursor ion masses, to compound_table.csv.

## Related tools

- **Shinyscreen** (Package whose initialisation routine validates the compound list, computes precursor masses and writes compound_table.csv) — https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen
- **yaml** (R package used to read the Shinyscreen YAML configuration supplying tolerances, QC thresholds and the adduct setting)
- **enviPat** (Computes adduct-specific precursor ion masses from molecular formulas for validated target/suspect entries)

## Evaluation signals

- compound_table.csv exists and is valid CSV.
- The table contains the identifier, name, formula/SMILES and precursor-mass fields.
- Every target/suspect row either has a structural field with a derived precursor mass, or supplies precursor m/z directly as structural information.
- Every non-target row carries an explicit precursor mass.
- Compound names/identifiers are consistent with the PubChemLite local CSV database.

## Limitations

- Target/suspect entries lacking both a molecular formula and a SMILES string cannot be mapped to precursor masses and are flagged as invalid.
- Computed precursor masses depend entirely on the configured adduct setting; an incorrect adduct yields wrong extraction m/z values.
- PubChemLite is the current default reference and is consumed as a local CSV from its Zenodo deposit, so database version/currency depends on the downloaded snapshot.

## Evidence

- [abstract] list-based screening modes: "Its versatile list-based approach supports the curation of data starting from either known or "suspected" compounds (compound list-based screening) or detected masses (mass list-based screening)"
- [methods] PubChemLite as default reference database: "with the current default use set to PubChemLite [31, 32]"
- [methods] precursor mass calculation from structures: "Shinyscreen will automatically calculate required precursor ion masses for extraction, using the given adduct setting"
- [methods] configuration parameters extracted at initialisation: "parametrise (a) experimental m/z and retention time errors"
