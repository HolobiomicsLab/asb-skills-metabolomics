---
name: compound-list-structural-field-validation
description: 'Use when preparing a compound list (CSV) for Shinyscreen screening:
  target/suspect rows must carry a molecular formula or SMILES, non-target rows must
  carry a precursor m/z.'
when_to_use_negative:
- Input is already a feature table or peak list from untargeted feature detection
  (e.g. xcms output) — Shinyscreen's compound list-based screening expects a preselected
  list of compounds or masses, not a feature table
- Rows carry a structural identifier but no precursor mass and no SMILES/formula to
  derive one (e.g. a compound name alone) — Shinyscreen will flag these as invalid
  for precursor mass calculation
- The analysis goal is de novo spectral identification from raw vendor files without
  a compound list — Shinyscreen is a prescreening/QC tool, not an identification engine
edam_operation: http://edamontology.org/operation_3283
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: Shinyscreen
  role: implements the compound list validation, precursor mass calculation and compound
    table output
  repo: https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen
- name: yaml
  role: reads the Shinyscreen YAML configuration to obtain the adduct setting and
    QC thresholds
- name: enviPat
  role: calculates precursor ion masses from molecular formulas under the configured
    adduct setting
- name: PubChemLite
  role: local CSV database used to cross-check compound names/identifiers for consistency
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-025-01044-x
    title: 'Shinyscreen: mass spectrometry data inspection and quality checking utility'
  source_content_hash: sha256:575387b0f1ae9acd279c2b4aeb62c50a1640728f7ae3bcbdc3f5c5284bb8dd18
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PubChemLite
  - R
  - Shinyscreen
  - enviPat
  - yaml
  edam_operation: http://edamontology.org/operation_3283
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# compound-list-structural-field-validation

## Summary

Validation step in Shinyscreen that checks each row of a compound list CSV for a usable structural identifier (molecular formula or SMILES for target/suspect entries; explicit precursor m/z for non-target entries) before precursor ion masses are calculated.

## When to use

Use when preparing a compound list (CSV) for Shinyscreen screening: target/suspect rows must carry a molecular formula or SMILES, non-target rows must carry a precursor m/z.

## When NOT to use

- Input is already a feature table or peak list from untargeted feature detection (e.g. xcms output) — Shinyscreen's compound list-based screening expects a preselected list of compounds or masses, not a feature table
- Rows carry a structural identifier but no precursor mass and no SMILES/formula to derive one (e.g. a compound name alone) — Shinyscreen will flag these as invalid for precursor mass calculation
- The analysis goal is de novo spectral identification from raw vendor files without a compound list — Shinyscreen is a prescreening/QC tool, not an identification engine

## Inputs

- Shinyscreen YAML configuration file (experimental m/z and retention-time error tolerances, quality-control parameters such as MS1 and MS2 intensity thresholds, retention time shifts, adduct setting)
- Compound list CSV with compound identifier, name, molecular formula and/or SMILES, optional retention time (target/suspect rows) or precursor m/z (non-target rows)

## Outputs

- compound_table.csv containing compound identifiers, structural fields (name, formula and/or SMILES), optional retention times, and calculated precursor ion masses

## How to apply

Load the Shinyscreen YAML configuration alongside the compound list CSV and split entries into target/suspect rows (which supply structural data such as a molecular formula or SMILES plus optional retention time) and non-target rows (which supply precursor m/z directly with optional retention time). Validate each row: flag target/suspect entries that have neither a molecular formula nor a SMILES string, and confirm non-target entries carry an explicit precursor mass. For validated structural entries, calculate the required precursor ion masses for extraction under the configured adduct setting using enviPat. Cross-check compound names/identifiers against the PubChemLite local CSV database (the current default, downloaded from its Zenodo deposit) to confirm identifier consistency. Write the validated compound table with compound identifiers, structural fields (name, formula and/or SMILES), optional retention times, and calculated precursor ion masses to compound_table.csv.

## Related tools

- **Shinyscreen** (implements the compound list validation, precursor mass calculation and compound table output) — https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen
- **yaml** (reads the Shinyscreen YAML configuration to obtain the adduct setting and QC thresholds)
- **enviPat** (calculates precursor ion masses from molecular formulas under the configured adduct setting)
- **PubChemLite** (local CSV database used to cross-check compound names/identifiers for consistency)

## Examples

```
Rscript shinyscreen_init.R --config config.yaml --compound_list compounds.csv --adduct [M+H]+
```

## Evaluation signals

- compound_table.csv exists in CSV format in the working directory after the validation step
- compound_table.csv contains the identifier, name, formula/SMILES and precursor-mass fields
- every target/suspect row either has a structural field (molecular formula or SMILES) with a derived precursor mass or has structural information supplied directly as precursor m/z
- non-target rows carry an explicit precursor mass
- compound names/identifiers are consistent with entries in the PubChemLite local CSV database

## Limitations

- Target/suspect entries that have neither a molecular formula nor a SMILES string are flagged as invalid and cannot be used for precursor mass calculation
- Non-target entries without an explicit precursor mass are not validated
- Identifier cross-checks rely on the PubChemLite local CSV database; entries not present there are not verified against an external reference

## Evidence

- [abstract] versatile list-based approach: "Its versatile list-based approach supports the curation of data starting from either known or "suspected" compounds (compound list-based screening) or detected masses (mass list-based screening)"
- [methods] compound list structural validation: "Shinyscreen will automatically calculate required precursor ion masses for extraction, using the given adduct setting"
- [methods] PubChemLite default: "with the current default use set to PubChemLite [31, 32]"
- [methods] quality assessment stage: "the data is subjected to a quality assessment stage. The quality of each MS1 parent data point is assessed based on the existence of MS1 and MS2 data, the noise level and the retention time shift"
