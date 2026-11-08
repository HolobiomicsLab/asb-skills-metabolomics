---
name: lipid-double-bond-position-extraction
description: Use when lipid structure records carry explicit double bond positions
  on fatty acyl chains and you must quantify how often two arbitrary chains share
  no double bond at the same position under forward (carbonyl) and backward (omega)
  numbering, or need a reference-database ground truth for.
when_to_use_negative:
- Lipid entries are only at partial structural resolution (e.g., sum-composition or
  DB-position-free notations) — the mismatch statistics are undefined without explicit
  double bond positions
- The goal is lipidome-level abundance analysis or PCA/dendrogram exploration in LipidSpace
  rather than validating structural double bond comparison correctness
- Input is already a lipidomics intensity table (csv/xlsx/mzTab-M) for import; this
  skill operates on reference structure databases, not measured abundance data
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3368
tools:
- name: LipidSpace
  role: Tool whose double bond position comparison correctness is grounded by the
    extracted reference statistics
  repo: https://github.com/lifs-tools/lipidspace
- name: Goslin (cppgoslin)
  role: Parses and standardizes lipid names so chain and double bond position notation
    is unified before extraction
  repo: https://github.com/lifs-tools/cppgoslin
- name: LIPID MAPS
  role: Reference structure database supplying high-resolution lipid records for chain-pair
    mismatch computation
- name: SwissLipids
  role: Reference structure database supplying the 85 distinct double bond position
    sets used as ground truth
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c02449
    title: 'LipidSpace: Simple Exploration, Reanalysis, and Quality Control of Large-Scale
      Lipidomics Studies'
  source_content_hash: sha256:d5ae61becfa67018acaeb37e5a866b1bbf0c75a3989dd8cb8123bf40925fc7d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/lipidspace
  tools:
  - Goslin (cppgoslin)
  - LIPID MAPS
  - LipidSpace
  - SwissLipids
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3368
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# lipid-double-bond-position-extraction

## Summary

Extracts carbon chains with explicit double bond positions from high-resolution lipid structure records in reference databases (LIPID MAPS, SwissLipids), then quantifies directional double bond mismatch probabilities between chain pairs under carbonyl-forward versus omega-backward counting conventions to ground correctness evaluation of lipid structural comparisons.

## When to use

Use when lipid structure records carry explicit double bond positions on fatty acyl chains and you must quantify how often two arbitrary chains share no double bond at the same position under forward (carbonyl) and backward (omega) numbering, or need a reference-database ground truth for validating structural comparison correctness.

## When NOT to use

- Lipid entries are only at partial structural resolution (e.g., sum-composition or DB-position-free notations) — the mismatch statistics are undefined without explicit double bond positions
- The goal is lipidome-level abundance analysis or PCA/dendrogram exploration in LipidSpace rather than validating structural double bond comparison correctness
- Input is already a lipidomics intensity table (csv/xlsx/mzTab-M) for import; this skill operates on reference structure databases, not measured abundance data

## Inputs

- LIPID MAPS lipid records at highest structural resolution level with explicit double bond positions on carbon chains
- SwissLipids lipid entries at highest structural resolution level with explicit double bond positions on carbon chains
- Parsed fatty acyl chain definitions (head-to-tail carbon numbering) per lipid structure

## Outputs

- Set of distinct double bond position sets per database (85 for SwissLipids)
- Pairwise chain-comparison table of forward-counting and backward-counting zero-overlap outcomes
- Direction-wise complete-mismatch probability values (CSV, e.g., lipid_mismatch_stats.csv)

## How to apply

Download lipid records from LIPID MAPS and/or SwissLipids at the highest structural resolution level, keeping only entries whose carbon chains carry explicit double bond positions. Parse each chain (Goslin/cppgoslin can standardize the lipid names feeding this step) and collect the distinct double bond position sets — for SwissLipids the extraction should yield 85 distinct DB position sets. For every chain derive two position sets: forward counting from the carbonyl carbon group and backward counting from the methyl end (omega notation). Perform a full pairwise comparison of all chains, flagging pairs whose forward sets and backward sets have zero overlap, and compute the complete-mismatch probability as the fraction of zero-overlap pairs per counting direction. Write per-pair outcomes and direction-wise probabilities to a CSV table (e.g., lipid_mismatch_stats.csv) and validate against reported reference values (~64% forward, >71% backward for SwissLipids; >=73% for LIPID MAPS).

## Related tools

- **LipidSpace** (Tool whose double bond position comparison correctness is grounded by the extracted reference statistics) — https://github.com/lifs-tools/lipidspace
- **Goslin (cppgoslin)** (Parses and standardizes lipid names so chain and double bond position notation is unified before extraction) — https://github.com/lifs-tools/cppgoslin
- **LIPID MAPS** (Reference structure database supplying high-resolution lipid records for chain-pair mismatch computation)
- **SwissLipids** (Reference structure database supplying the 85 distinct double bond position sets used as ground truth)

## Evaluation signals

- SwissLipids extraction must recover exactly 85 distinct double bond position sets
- SwissLipids mismatch probability must approximate ~64% in the forward (carbonyl) direction and exceed 71% in the backward (omega) direction
- LIPID MAPS computed complete-mismatch probability must be >= 73% in both forward and backward directions
- Output table lipid_mismatch_stats.csv contains per-pair zero-overlap outcomes plus direction-wise probability summary rows
- Each chain yields exactly two position sets (forward from carbonyl carbon, backward from methyl end), and pairwise comparisons cover all chain pairs

## Limitations

- Results depend on counting convention: probabilities differ substantially between forward (carbonyl) and backward (omega) numbering, so convention must be stated when comparing values
- Only entries at the highest structural resolution level with explicit double bond positions contribute; coarser entries are excluded and can bias coverage
- The reported values are database-specific — LIPID MAPS and SwissLipids thresholds differ and are not interchangeable

## Evidence

- [methods] Reference database extraction at highest resolution: "extracted all lipids from databases such as LIPID MAPS and SwissLipids on the highest structural resolution level"
- [methods] Article section for this validation analysis: "Correctness of Double
Bond Comparisons"
- [other] Source card finding on grounding double bond comparison correctness: [evidence span withheld — anchor status: not_found]
- [readme] README on how fatty acyl chains are paired in comparisons: "In default mode, LipidSpace is comparing the first fatty acyl chain (FA) of the first lipid with the first FA of the second lipid"
- [readme] README on lipid name parsing standardization: "It utilizes the C++ library of Goslin4,5 to parse and standardize lipid names"
