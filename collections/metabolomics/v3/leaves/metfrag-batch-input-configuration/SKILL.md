---
name: metfrag-batch-input-configuration
description: Use when a validated compound list from HRMS suspect/target screening
  contains molecular formula and/or SMILES entries that must be converted into a MetFrag-consumable
  candidate file with computed precursor ion masses.
when_to_use_negative:
- Input is a raw HRMS file or mass list without structural information — Shinyscreen's
  mass list-based screening path has no molecular formula/SMILES to populate the MetFrag
  candidate columns
- Compounds have not passed Shinyscreen prescreening/quality assessment (MS1/MS2 existence,
  noise level, retention time shift checks) — candidate lists would include sub-quality
  entries
- You need full MetFrag parameterisation beyond candidate lists — Shinyscreen's interface
  intentionally 'hides a lot of the complexity' rather than exposing it
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_3272
tools:
- name: Shinyscreen
  role: Produces the validated compound list and provides the MetFrag configuration
    interface that assembles/launches the batch MetFrag runs
  repo: https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen
- name: enviPat
  role: Computes precursor ion masses from the molecular formula under the given adduct
    setting while formula/SMILES fields are kept intact
- name: MetFrag
  role: Downstream consumer of metfrag_candidate_input.csv for in silico fragmentation
    and candidate annotation
- name: R
  role: Implementation language for the compound-list to MetFrag candidate-file assembly
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-025-01044-x
    title: 'Shinyscreen: mass spectrometry data inspection and quality checking utility'
  source_content_hash: sha256:8331c82fc798e44081611706833e4252a0be90a2d7b46b2b9412db327e804b2b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetFrag
  - MetFrag Command Line
  - MetFragWeb
  - R
  - Shinyscreen
  - enviPat
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# metfrag-batch-input-configuration

## Summary

Generates a MetFrag-ready candidate input table from the validated compound list produced by Shinyscreen prescreening: precursor ion masses are computed from molecular formulae/SMILES with the given adduct setting (enviPat), and each record is written with explicit molecular-formula and SMILES columns to metfrag_candidate_input.csv for downstream in silico fragmentation.

## When to use

Use when a validated compound list from HRMS suspect/target screening contains molecular formula and/or SMILES entries that must be converted into a MetFrag-consumable candidate file with computed precursor ion masses.

## When NOT to use

- Input is a raw HRMS file or mass list without structural information — Shinyscreen's mass list-based screening path has no molecular formula/SMILES to populate the MetFrag candidate columns
- Compounds have not passed Shinyscreen prescreening/quality assessment (MS1/MS2 existence, noise level, retention time shift checks) — candidate lists would include sub-quality entries
- You need full MetFrag parameterisation beyond candidate lists — Shinyscreen's interface intentionally 'hides a lot of the complexity' rather than exposing it

## Inputs

- Validated Shinyscreen compound list (compound-list-based screening output) with molecular formula and/or SMILES per compound
- Adduct setting used during precursor extraction (e.g. [M+H]+, [M-H]-)
- Specified m/z tolerance context from the Shinyscreen extraction step

## Outputs

- metfrag_candidate_input.csv — one row per compound with non-empty molecular formula and SMILES columns plus computed precursor ion mass
- MetFrag configuration input consumable by MetFrag Command Line or MetFragWeb

## How to apply

Read the validated compound list produced by the Shinyscreen prescreening stage (which holds molecular formula and/or SMILES per compound of interest) and, for each entry, compute the precursor ion mass using enviPat's adduct-mass calculation under the adduct setting used for extraction — Shinyscreen calculates these automatically 'using the given adduct setting'. Keep the molecular formula and SMILES fields intact per compound rather than converting between identifiers. Assemble one row per compound into a single tabular file that explicitly carries molecular-formula and SMILES columns so MetFrag configuration can consume it directly, then write it to metfrag_candidate_input.csv. In the article's pipeline this step corresponds to Shinyscreen's 'Integration with MetFrag' stage, where a configuration interface runs several MetFrag instances in background to speed up the analysis.

## Related tools

- **Shinyscreen** (Produces the validated compound list and provides the MetFrag configuration interface that assembles/launches the batch MetFrag runs) — https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen
- **enviPat** (Computes precursor ion masses from the molecular formula under the given adduct setting while formula/SMILES fields are kept intact)
- **MetFrag** (Downstream consumer of metfrag_candidate_input.csv for in silico fragmentation and candidate annotation)
- **R** (Implementation language for the compound-list to MetFrag candidate-file assembly)

## Evaluation signals

- metfrag_candidate_input.csv exists on disk and parses as valid CSV
- Every row has non-empty molecular formula and SMILES fields — no partially populated candidates
- One output row per compound in the validated Shinyscreen compound list (row-count invariant)
- Computed precursor ion masses match the molecular formulae under the specified adduct setting (enviPat adduct-mass check)
- File loads directly into MetFrag configuration without column renaming or reformatting

## Limitations

- Requires compounds with structural information; entries lacking both molecular formula and SMILES cannot yield valid MetFrag candidate rows
- Precursor mass correctness depends on choosing the adduct setting consistently with the Shinyscreen extraction stage
- Shinyscreen's MetFrag interface hides configuration complexity, which limits fine-grained control over MetFrag parameters
- The Shinyscreen project is Apache 2.0 licensed but no archived code snapshot (Zenodo/OSF) or dataset accession is reported in the paper

## Evidence

- [methods] MetFrag config via Shinyscreen: "Shinyscreen provides a MetFrag configuration interface (Fig. 5) that hides a lot of the complexity and runs several MetFrag instances in background to speed up the analysis"
- [methods] Precursor mass calculation: "Shinyscreen will automatically calculate required precursor ion masses for extraction, using the given adduct setting"
- [methods] Pipeline stage placement: "The core steps of Shinyscreen include initialisation, extraction and prescreening, visualisation and integration with MetFrag for further analysis"
- [abstract] Compound list provenance: "Its versatile list-based approach supports the curation of data starting from either known or "suspected" compounds (compound list-based screening)"
