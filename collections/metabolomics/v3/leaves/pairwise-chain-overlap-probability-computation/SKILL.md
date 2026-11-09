---
name: pairwise-chain-overlap-probability-computation
description: 'Use when validating double bond position conventions in a lipid structure
  database: enumerate chains from fully resolved records (explicit double bond positions),
  pairwise-compare forward and omega-direction position sets, and check the complete-mismatch
  probability against the >= 73% threshold.'
when_to_use_negative:
- Records are only structurally resolved at shorthand level (e.g. 'PC 34:1' without
  double bond positions), so per-chain double bond position sets cannot be constructed
- The goal is lipidome-level sample comparison or feature analysis rather than database-wide
  double bond placement statistics (LipidSpace's MCS/Jaccard lipidome similarity workflow
  is the appropriate tool there)
- sn-positions are unspecified and you need exact per-chain correspondence rather
  than a population-level mismatch statistic; LipidSpace's 'Ignore lipid sn-positions'
  mode exists for that case but at reduced performance
edam_operation: http://edamontology.org/operation_0282
edam_topics: []
tools:
- name: LIPID MAPS
  role: Source database from which fully resolved lipid records and their carbon chain
    double bond positions are extracted for the pairwise mismatch computation
- name: cppgoslin (Goslin)
  role: Lipid name parser that standardizes lipid name dialects so chain and double
    bond information can be extracted consistently from database records
  repo: https://github.com/lifs-tools/cppgoslin
- name: LipidSpace
  role: Reference implementation of double-bond-aware fatty acyl chain comparison
    (first-FA vs first-FA, with optional all-combination mode) against which the pairwise
    overlap logic is grounded
  repo: https://github.com/lifs-tools/lipidspace
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c02449
    title: 'LipidSpace: Simple Exploration, Reanalysis, and Quality Control of Large-Scale
      Lipidomics Studies'
  source_content_hash: sha256:707c1cc43afe0a4afa303120a537d5faa4e5b9aabd742563cb89c8753a23122e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/cppgoslin
  tools:
  - LIPID MAPS
  - LipidSpace
  - cppgoslin (Goslin)
  edam_operation: http://edamontology.org/operation_0282
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# pairwise-chain-overlap-probability-computation

## Summary

Computes the probability that pairs of fatty acyl carbon chains share no double bond position in either the carbonyl-to-chain (forward) or methyl-end (omega, backward) counting direction, using all structurally fully resolved lipid records from LIPID MAPS and SwissLipids; the resulting complete-mismatch probability (expected >= 73%) quantifies how non-random double bond placement is across lipid chains.

## When to use

Use when validating double bond position conventions in a lipid structure database: enumerate chains from fully resolved records (explicit double bond positions), pairwise-compare forward and omega-direction position sets, and check the complete-mismatch probability against the >= 73% threshold.

## When NOT to use

- Records are only structurally resolved at shorthand level (e.g. 'PC 34:1' without double bond positions), so per-chain double bond position sets cannot be constructed
- The goal is lipidome-level sample comparison or feature analysis rather than database-wide double bond placement statistics (LipidSpace's MCS/Jaccard lipidome similarity workflow is the appropriate tool there)
- sn-positions are unspecified and you need exact per-chain correspondence rather than a population-level mismatch statistic; LipidSpace's 'Ignore lipid sn-positions' mode exists for that case but at reduced performance

## Inputs

- LIPID MAPS lipid records at the highest structural resolution level (explicit double bond positions on carbon chains)
- SwissLipids records at the highest structural resolution level (optional second database)
- Per-chain double bond position lists in forward (carbonyl) and backward (omega/methyl-end) notation

## Outputs

- lipid_mismatch_stats.csv containing per-pair comparison outcomes (overlap vs. no-overlap per counting direction)
- Direction-wise complete-mismatch probabilities (forward and backward), each expected to be >= 73%

## How to apply

Download LIPID MAPS (and optionally SwissLipids) records at the highest structural resolution level, i.e. only entries with explicit double bond positions on the carbon chains. For each carbon chain build two sets of double bond positions: one counted forward from the carbonyl carbon group and one counted backward from the methyl end (omega-notation). Perform a pairwise comparison over all chain pairs, counting a pair as a complete mismatch when both the forward sets and the backward sets have zero overlapping double bond positions. Compute the complete-mismatch probability as the fraction of chain pairs with no overlap in each direction, and write per-pair comparison outcomes plus direction-wise probabilities to a results table (lipid_mismatch_stats.csv). The validation criterion is that the computed probability reaches >= 73% in both the forward and backward directions, matching the value reported for the LIPID MAPS database. LipidSpace's underlying double bond comparison logic (chain-by-chain FA comparison, with an all-combinations mode when sn-positions are unspecified) shows how such position-set comparisons are operationalized in practice.

## Related tools

- **LIPID MAPS** (Source database from which fully resolved lipid records and their carbon chain double bond positions are extracted for the pairwise mismatch computation)
- **cppgoslin (Goslin)** (Lipid name parser that standardizes lipid name dialects so chain and double bond information can be extracted consistently from database records) — https://github.com/lifs-tools/cppgoslin
- **LipidSpace** (Reference implementation of double-bond-aware fatty acyl chain comparison (first-FA vs first-FA, with optional all-combination mode) against which the pairwise overlap logic is grounded) — https://github.com/lifs-tools/lipidspace

## Evaluation signals

- The computed complete-mismatch probability is >= 73% in the forward (carbonyl-based) counting direction, matching the reported LIPID MAPS value
- The computed complete-mismatch probability is >= 73% in the backward (omega/methyl-end) counting direction as well
- lipid_mismatch_stats.csv exists and contains one row per chain pair with direction-wise overlap/no-overlap outcomes
- Only records with explicit double bond positions (highest structural resolution level) entered the computation; lower-resolution records were excluded
- All chain pairs were considered (full pairwise comparison), not a sampled subset, so the fraction is a well-defined population probability

## Limitations

- The statistic is only meaningful on the highest structural resolution level; shorthand lipid names without explicit double bond positions cannot contribute
- When sn-positions are not specified (e.g. PC 18:0_16:1), pairwise chain correspondence is ambiguous; comparing all FA combinations gives more accurate results but decreased performance
- No changelog/versioning signal was found for the comparison implementation, so reproducibility depends on pinning the tool version (e.g. the Zenodo DOI)

## Evidence

- [methods] Database resolution: "extracted all lipids from databases such as LIPID MAPS and SwissLipids on the highest structural resolution level"
- [methods] Double bond comparison validation: "Correctness of Double Bond Comparisons"
- [readme] Chain pairing default: "In default mode, LipidSpace is comparing the first fatty acyl chain (FA) of the first lipid with the first FA of the second lipid"
- [readme] sn-position ambiguity mode: "when the sn-position is not specified as for instance in PC 18:0_16:1, a mode can be activated to compare all combinations of FA comparisons"
