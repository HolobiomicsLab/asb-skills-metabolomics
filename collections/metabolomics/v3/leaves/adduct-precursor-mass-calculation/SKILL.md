---
name: adduct-precursor-mass-calculation
description: Use when a compound list supplies molecular formulas or SMILES strings
  (target/suspect entries) and precursor ion masses must be derived under a chosen
  adduct before peak extraction.
when_to_use_negative:
- The input rows are non-target entries that already supply explicit precursor m/z
  rather than structural information
- The workflow needs peak extraction or prescreening (EIC retrieval, MS2 scan matching)
  rather than deriving precursor masses from structure
- Target/suspect rows lack both a molecular formula and a SMILES string and no precursor
  m/z is given — structural mass calculation cannot proceed
edam_operation: http://edamontology.org/operation_0326
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: Shinyscreen
  role: Hosts the initialisation step that reads the compound list and computes precursor
    ion masses under the configured adduct
  repo: https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen
- name: enviPat
  role: Computes adduct masses rapidly from molecular formulas
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1186/s13321-025-01044-x
    title: 'Shinyscreen: mass spectrometry data inspection and quality checking utility'
  source_content_hash: sha256:bc2abc27f4ccf6a8528b973315f06f7bf833dccdbd13a900c8e994c8ee43b0b5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetFragWeb
  - R
  - Shinyscreen
  - enviPat
  edam_operation: http://edamontology.org/operation_0326
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# adduct-precursor-mass-calculation

## Summary

Compute precursor ion masses from structural information (molecular formula or SMILES) for each compound under a configured adduct setting, so that MS1/MS2 data can be extracted from HRMS runs in Shinyscreen.

## When to use

Use when a compound list supplies molecular formulas or SMILES strings (target/suspect entries) and precursor ion masses must be derived under a chosen adduct before peak extraction.

## When NOT to use

- The input rows are non-target entries that already supply explicit precursor m/z rather than structural information
- The workflow needs peak extraction or prescreening (EIC retrieval, MS2 scan matching) rather than deriving precursor masses from structure
- Target/suspect rows lack both a molecular formula and a SMILES string and no precursor m/z is given — structural mass calculation cannot proceed

## Inputs

- Validated compound list (CSV) with identifier, name, molecular formula and/or SMILES, and optional retention time for target/suspect entries
- Shinyscreen YAML configuration supplying the adduct setting

## Outputs

- compound_table.csv with compound identifiers, structural fields (name, formula and/or SMILES), optional retention times, and calculated precursor ion masses
- metfrag_candidate_input.csv (optional) with one row per compound including molecular formula and SMILES fields

## How to apply

Read the validated compound list and keep target/suspect rows that carry a molecular formula or SMILES (non-target rows already carry explicit precursor m/z and skip this step). Apply the adduct setting from the Shinyscreen YAML configuration to each structural entry, using enviPat to calculate the required precursor ion masses for extraction from the given molecular formula. Keep the identifier, name, formula/SMILES and optional retention-time fields intact per compound while adding the computed precursor masses. Write the enriched table (compound_table.csv) or the MetFrag candidate input (metfrag_candidate_input.csv) with one row per compound. Validate that every target/suspect row either has a structural field with a derived precursor mass or supplies the precursor m/z directly.

## Related tools

- **Shinyscreen** (Hosts the initialisation step that reads the compound list and computes precursor ion masses under the configured adduct) — https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen
- **enviPat** (Computes adduct masses rapidly from molecular formulas)

## Examples

```
Rscript -e 'library(shinyscreen); conf <- read_yaml("config.yaml"); df <- read_csv("compounds.csv"); comp <- shinyscreen::compound_table(df, conf); write.csv(comp, "compound_table.csv", row.names=FALSE)'
```

## Evaluation signals

- compound_table.csv exists and is valid CSV
- The output contains identifier, name, formula/SMILES and precursor-mass fields
- Every target/suspect row either has a structural field with a derived precursor mass or supplies precursor m/z directly
- For the MetFrag candidate input, every row has non-empty molecular formula and SMILES columns

## Limitations

- Entries without molecular formula or SMILES cannot be converted to precursor masses and must be flagged during validation
- The adduct setting must be supplied (or fall back to Orbitrap-based defaults); a mismatched adduct yields masses that will not match the experimental data

## Evidence

- [methods] Shinyscreen will automatically calculate required precursor ion masses for extraction, using the given adduct setting: "Shinyscreen will automatically calculate required precursor ion masses for extraction, using the given adduct setting"
- [abstract] compound list-based screening: "Its versatile list-based approach supports the curation of data starting from either known or "suspected" compounds (compound list-based screening)"
- [methods] Split into target/suspect vs non-target rows: [evidence span withheld — anchor status: not_found]
- [methods] enviPat adduct calculation: [evidence span withheld — anchor status: not_found]
