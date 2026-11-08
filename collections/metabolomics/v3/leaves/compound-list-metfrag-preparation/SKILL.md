---
name: compound-list-metfrag-preparation
description: Use when a validated Shinyscreen compound list (compound list-based screening
  entries carrying molecular formula and/or SMILES) must be converted into a MetFrag-consumable
  candidate CSV with precursor ion masses for a chosen adduct.
when_to_use_negative:
- The working set comes from mass list-based screening and lacks molecular formula
  or SMILES per entry — without structural information no MetFrag candidate input
  can be assembled.
- The peaks have not yet passed Shinyscreen's prescreening/quality assessment (MS1/MS2
  existence, noise level, retention time shift) — curate the list first.
- The task is running the in silico fragmentation itself (MetFrag configuration and
  execution) rather than preparing its candidate input file.
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3174
tools:
- name: Shinyscreen
  role: Source of the validated compound list after prescreening and quality assessment;
    also auto-calculates precursor ion masses from compound lists using the given
    adduct setting
  repo: https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen
- name: enviPat
  role: 'Adduct-mass calculation: computes precursor ion masses from molecular formulas
    while formula/SMILES fields are kept intact per compound'
- name: MetFrag
  role: Downstream consumer of the assembled candidate input file for in silico fragmentation
    and compound annotation
- name: data.table
  role: R table assembly and CSV export of the per-compound candidate records
- name: R
  role: Implementation environment in which enviPat mass calculation and table assembly
    are executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-025-01044-x
    title: 'Shinyscreen: mass spectrometry data inspection and quality checking utility'
  source_content_hash: sha256:3c9468574567b195e9725e7a5ff1749a99494b15a7de162e490f59391cc35b73
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetFrag
  - MetFrag Command Line
  - MetFragWeb
  - R
  - Shinyscreen
  - data.table
  - enviPat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# compound-list-metfrag-preparation

## Summary

Prepares a MetFrag-ready candidate input table from the compound list validated by Shinyscreen's prescreening stage, keeping each compound's molecular formula and SMILES intact while computing precursor ion masses for the chosen adduct. This bridges Shinyscreen's prescreening/QC output with MetFrag's in silico fragmentation for compound annotation in suspect and non-target screening workflows.

## When to use

Use when a validated Shinyscreen compound list (compound list-based screening entries carrying molecular formula and/or SMILES) must be converted into a MetFrag-consumable candidate CSV with precursor ion masses for a chosen adduct.

## When NOT to use

- The working set comes from mass list-based screening and lacks molecular formula or SMILES per entry — without structural information no MetFrag candidate input can be assembled.
- The peaks have not yet passed Shinyscreen's prescreening/quality assessment (MS1/MS2 existence, noise level, retention time shift) — curate the list first.
- The task is running the in silico fragmentation itself (MetFrag configuration and execution) rather than preparing its candidate input file.

## Inputs

- Validated Shinyscreen prescreening compound list with per-compound molecular formula and/or SMILES
- Adduct setting used during Shinyscreen precursor ion mass calculation (e.g. [M+H]+, [M-H]-)

## Outputs

- metfrag_candidate_input.csv — one row per compound, with non-empty molecular formula and SMILES columns and precursor ion masses computed via enviPat

## How to apply

Read the validated compound list produced by Shinyscreen's prescreening stage, which carries structural information (molecular formula and/or SMILES) for each compound of interest — Shinyscreen normally calculates precursor ion masses automatically 'using the given adduct setting', so reconstruct the equivalent computation with enviPat's adduct-mass capability when exporting externally. Keep the molecular formula and SMILES fields intact per compound; they are the identifiers MetFrag uses to fetch/generate candidate structures. Assemble the records into a single tabular file with one row per compound and explicit molecular formula and SMILES columns, plus the enviPat-computed precursor ion mass consistent with the adduct used during Shinyscreen extraction. Write the table to disk as metfrag_candidate_input.csv so it is directly consumable by MetFrag configuration (Shinyscreen itself hides this complexity by running MetFrag instances in the background via its configuration interface). Finally, validate that the output exists, parses as valid CSV, and has non-empty formula and SMILES in every row.

## Related tools

- **Shinyscreen** (Source of the validated compound list after prescreening and quality assessment; also auto-calculates precursor ion masses from compound lists using the given adduct setting) — https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen
- **enviPat** (Adduct-mass calculation: computes precursor ion masses from molecular formulas while formula/SMILES fields are kept intact per compound)
- **MetFrag** (Downstream consumer of the assembled candidate input file for in silico fragmentation and compound annotation)
- **data.table** (R table assembly and CSV export of the per-compound candidate records)
- **R** (Implementation environment in which enviPat mass calculation and table assembly are executed)

## Evaluation signals

- metfrag_candidate_input.csv exists on disk and parses as valid CSV.
- Every row has non-empty molecular formula and SMILES fields (no missing structural information).
- Row count of the output equals the number of compounds in the validated Shinyscreen prescreening list.
- Precursor ion masses in the output match enviPat computations for the same molecular formulas under the adduct setting used during Shinyscreen extraction.
- The file's column layout (formula + SMILES) is directly consumable by MetFrag configuration without reformatting.

## Limitations

- Compounds lacking molecular formula or SMILES in the prescreening output cannot be prepared for MetFrag — the export requires structural information per entry.
- The precursor masses must be computed under the same adduct setting Shinyscreen used for extraction; a mismatched adduct silently shifts all candidate m/z values.
- The Shinyscreen release states an Apache 2.0 license but no archived code snapshot (Zenodo/OSF) is reported, so this reconstructed export procedure is not backed by a pinned reference implementation.

## Evidence

- [methods] MetFrag config interface and background runs: "Shinyscreen provides a MetFrag configuration interface (Fig. 5) that hides a lot of the complexity and runs several MetFrag instances in background to speed up the analysis"
- [methods] Automatic precursor mass calculation: "Shinyscreen will automatically calculate required precursor ion masses for extraction, using the given adduct setting"
- [abstract] Compound-list-based screening supplies the validated list: "Its versatile list-based approach supports the curation of data starting from either known or "suspected" compounds (compound list-based screening)"
- [methods] Extraction consumes precursor masses from compound lists: "the precursor ion masses from the compound lists, their extracted ion chromatograms (EICs, within the specified m/z tolerance)"
- [discussion] Project repository for Shinyscreen: "Project home page: https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen"
