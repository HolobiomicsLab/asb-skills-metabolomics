---
name: smiles-formula-structural-field-curation
description: Use when a validated Shinyscreen compound list (rows carrying molecular
  formula and/or SMILES for suspects/targets) must be converted into a MetFrag candidate
  input file ahead of in silico fragmentation screening.
when_to_use_negative:
- Input is raw HRMS data (e.g. vendor files or mzR/MSnbase-accessible scans) rather
  than a curated compound list — Shinyscreen's extraction and prescreening stages
  must run first.
- Compounds lack molecular formula and SMILES entirely (mass list-based screening
  with only detected m/z values) — there are no structural fields to curate for MetFrag.
- The identification strategy is spectral library matching against resources like
  MassBank rather than in silico fragmentation — MetFrag candidate generation is unnecessary.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: Shinyscreen
  role: Source of the validated compound list and of the MetFrag configuration interface
    that runs MetFrag instances in background
  repo: https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen
- name: enviPat
  role: Adduct-mass calculation from molecular formula to obtain precursor ion masses
    for the MetFrag candidate input
- name: MetFrag
  role: Downstream in silico fragmentation consumer of the generated candidate input
    CSV
- name: data.table
  role: Tabular assembly and CSV export of the candidate records
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-025-01044-x
    title: 'Shinyscreen: mass spectrometry data inspection and quality checking utility'
  source_content_hash: sha256:f52ae691a8fbdb31bac1b59f76fd377947ecdeb02e282389394c903a1311bbf5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetFrag
  - MetFragWeb
  - R
  - Shinyscreen
  - data.table
  - enviPat
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# smiles-formula-structural-field-curation

## Summary

Assembles a MetFrag-ready candidate input table from a Shinyscreen-validated compound list, preserving each entry's molecular formula and SMILES and computing precursor ion masses (e.g. via enviPat adduct-mass calculation) so the CSV can be consumed directly by a MetFrag configuration for in silico fragmentation-based compound annotation.

## When to use

Use when a validated Shinyscreen compound list (rows carrying molecular formula and/or SMILES for suspects/targets) must be converted into a MetFrag candidate input file ahead of in silico fragmentation screening.

## When NOT to use

- Input is raw HRMS data (e.g. vendor files or mzR/MSnbase-accessible scans) rather than a curated compound list — Shinyscreen's extraction and prescreening stages must run first.
- Compounds lack molecular formula and SMILES entirely (mass list-based screening with only detected m/z values) — there are no structural fields to curate for MetFrag.
- The identification strategy is spectral library matching against resources like MassBank rather than in silico fragmentation — MetFrag candidate generation is unnecessary.

## Inputs

- Validated compound list from Shinyscreen prescreening (tabular, one row per compound, with molecular formula and/or SMILES)
- Adduct specification (e.g. [M+H]+, [M-H]-) matching the HRMS acquisition mode

## Outputs

- metfrag_candidate_input.csv — one row per compound with non-empty molecular formula and SMILES fields plus computed precursor ion masses
- MetFrag configuration consumable by MetFrag Command Line, MetFragWeb, or Shinyscreen's built-in MetFrag interface

## How to apply

Read the validated compound list emitted by Shinyscreen's prescreening stage, which carries structural identifiers (molecular formula and/or SMILES) for each compound of interest. For each record, compute the required precursor ion mass from the formula using the adduct setting (e.g. [M+H]+ via enviPat's adduct-mass calculation), keeping the formula and SMILES fields intact — Shinyscreen itself performs this calculation automatically for extraction, 'using the given adduct setting'. Assemble one row per compound into a table with explicit molecular formula and SMILES columns and write it as metfrag_candidate_input.csv so a MetFrag configuration (command line or web) can ingest it. Validate that the file exists, parses as CSV, and that no row is empty in the formula or SMILES columns; Shinyscreen can then launch several MetFrag instances in background for the fragmentation search.

## Related tools

- **Shinyscreen** (Source of the validated compound list and of the MetFrag configuration interface that runs MetFrag instances in background) — https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen
- **enviPat** (Adduct-mass calculation from molecular formula to obtain precursor ion masses for the MetFrag candidate input)
- **MetFrag** (Downstream in silico fragmentation consumer of the generated candidate input CSV)
- **data.table** (Tabular assembly and CSV export of the candidate records)

## Examples

```
write.csv(data.table(formula=compounds$formula, smiles=compounds$smiles, precursor_mz=enviPat_adduct_mass(compounds$formula, '[M+H]+')), 'metfrag_candidate_input.csv')
```

## Evaluation signals

- metfrag_candidate_input.csv exists on disk and parses as valid CSV.
- Every row has non-empty molecular formula and SMILES fields — no empty cells in those columns.
- One row per compound from the validated Shinyscreen list; no duplicates or dropped entries.
- Computed precursor ion masses match the chosen adduct setting applied to each molecular formula (spot-check against expected [M+H]+/[M-H]- monoisotopic masses).
- The file loads into a MetFrag configuration without schema errors (column names for formula and SMILES present).

## Limitations

- Requires structural information (formula/SMILES) per compound; masses-only entries from mass list-based screening cannot populate the MetFrag candidate fields.
- Precursor mass correctness depends on the adduct setting matching the experiment; a wrong adduct silently shifts all candidate m/z values.
- Shinyscreen's MetFrag integration is a configuration convenience layer — 'hides a lot of the complexity' — so users relying on it may not control all MetFrag parameters directly.
- No archived code snapshot (Zenodo/OSF) is reported for the described Shinyscreen version; only the GitLab repository under Apache 2.0.

## Evidence

- [methods] Integration with MetFrag: "Shinyscreen provides a MetFrag configuration interface (Fig. 5) that hides a lot of the complexity and runs several MetFrag instances in background to speed up the analysis"
- [methods] Precursor ion mass calculation: "Shinyscreen will automatically calculate required precursor ion masses for extraction, using the given adduct setting"
- [methods] Core pipeline: "The core steps of Shinyscreen include initialisation, extraction and prescreening, visualisation and integration with MetFrag for further analysis"
- [abstract] Positioning in workflow: "Shinyscreen supports quality control of data for further identification or upload of spectra to public data resources"
