---
name: sn-position-vs-omega-position-counting
description: Use when comparing fatty acyl chains by double bond position, when numbering
  direction (carbonyl-forward vs ω/methyl-end) affects the comparison outcome, or
  when sn-positions are unspecified in lipid names such as PC 18:0_16:1.
when_to_use_negative:
- Lipid records are not at the highest structural resolution level (no explicit double
  bond positions on the chains), so forward/backward position sets cannot be constructed
- The goal is quantitative lipidome comparison (abundances, study variables, PCA/dendrograms)
  rather than structural double bond correctness checks
- Only a single counting direction is needed and sn-positions are fully specified,
  making the all-combination FA mode an unnecessary performance cost
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: LipidSpace
  role: Performs graph-based structural comparison of lipid species, with a mode to
    ignore lipid sn-positions when comparing fatty acyl chains of sn-unspecified lipids
  repo: https://github.com/lifs-tools/lipidspace
- name: cppgoslin (Goslin)
  role: Parses and standardizes lipid names so fatty acyl chains and double bond positions
    are extracted consistently before pairwise comparison
  repo: https://github.com/lifs-tools/cppgoslin
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c02449
    title: 'LipidSpace: Simple Exploration, Reanalysis, and Quality Control of Large-Scale
      Lipidomics Studies'
  source_content_hash: sha256:19cd9e542e00c55de66a0ffdc712faeb42feae9719b27c9bf3bfbdefc176d085
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/lipidspace
  tools:
  - LipidSpace
  - cppgoslin (Goslin)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# sn-position-vs-omega-position-counting

## Summary

A method for comparing fatty acyl chains by their double bond positions while accounting for the two counting directions — forward from the carbonyl carbon and backward from the methyl end (ω-notation) — and for unspecified sn-positions, used to verify the correctness of structural double bond comparisons against the ≥73% complete-mismatch probability reported for LIPID MAPS.

## When to use

Use when comparing fatty acyl chains by double bond position, when numbering direction (carbonyl-forward vs ω/methyl-end) affects the comparison outcome, or when sn-positions are unspecified in lipid names such as PC 18:0_16:1.

## When NOT to use

- Lipid records are not at the highest structural resolution level (no explicit double bond positions on the chains), so forward/backward position sets cannot be constructed
- The goal is quantitative lipidome comparison (abundances, study variables, PCA/dendrograms) rather than structural double bond correctness checks
- Only a single counting direction is needed and sn-positions are fully specified, making the all-combination FA mode an unnecessary performance cost

## Inputs

- LIPID MAPS lipid records at the highest structural resolution level (explicit double bond positions on carbon chains)
- SwissLipids lipid records at the highest structural resolution level
- Lipid species names parsed and standardized via Goslin/cppgoslin (e.g., PC 18:0_16:1)
- Per-chain double bond position lists for each fatty acyl chain

## Outputs

- Per-pair comparison outcomes (whether forward and backward double bond position sets overlap)
- Direction-wise complete-mismatch probabilities (forward from carbonyl and backward from methyl end)
- Results table of double bond mismatch statistics (e.g., lipid_mismatch_stats.csv)

## How to apply

Extract all lipids from databases such as LIPID MAPS and SwissLipids at the highest structural resolution level, i.e. records with explicit double bond positions on the carbon chains. For each carbon chain, build two sets of double bond positions: one counting forward from the carbonyl carbon group and one counting backward from the methyl end (ω-notation), since a mismatch under one convention can be masked by the other. Pairwise-compare all chains and count each pair as a complete mismatch only if the forward sets AND the backward sets have zero overlapping double bond positions; the complete-mismatch probability is the fraction of such pairs and must reach ≥73% in both directions for LIPID MAPS, matching the reported value. Parse and standardize lipid names with Goslin (cppgoslin) so chain and double bond annotations are extracted consistently. For structural comparisons in LipidSpace, the default mode compares FA1-to-FA1, FA2-to-FA2, etc.; when the sn-position is not specified (e.g., PC 18:0_16:1), activate the 'Ignore lipid sn-positions' mode to compare all combinations of FA pairs and pick the lowest distance, trading runtime for accuracy.

## Related tools

- **LipidSpace** (Performs graph-based structural comparison of lipid species, with a mode to ignore lipid sn-positions when comparing fatty acyl chains of sn-unspecified lipids) — https://github.com/lifs-tools/lipidspace
- **cppgoslin (Goslin)** (Parses and standardizes lipid names so fatty acyl chains and double bond positions are extracted consistently before pairwise comparison) — https://github.com/lifs-tools/cppgoslin

## Evaluation signals

- The complete-mismatch probability is computed in BOTH counting directions and each is >= 73% for LIPID MAPS chains, matching the reported value
- Forward (carbonyl) and backward (ω) probabilities agree with each other for the same chain corpus; a discrepancy indicates a counting-direction error
- For sn-unspecified lipids (e.g., PC 18:0_16:1), per-pair results are checked against the all-combinations FA comparison mode rather than the default FA1-to-FA1 pairing
- Every chain contributing to the statistics carries explicit double bond positions; resolution-level-lower records are excluded before pairwise comparison
- The per-pair outcome table and direction-wise probabilities are both written out and internally consistent (mismatch count / total pairs = reported probability)

## Limitations

- The 'Ignore lipid sn-positions' mode improves accuracy but decreases performance, since all combinations of FA comparisons must be evaluated
- LipidSpace's default mode compares first FA to first FA positionally, which can misjudge structural similarity when sn-positions are unspecified
- Results depend on the structural resolution of the database export; species-level shorthand without double bond positions cannot support forward/backward position-set comparison

## Evidence

- [methods] Extract chains at highest resolution: "extracted all lipids from databases such as LIPID MAPS and SwissLipids on the highest structural resolution level"
- [methods] Double bond comparison correctness: "Correctness of Double Bond Comparisons"
- [readme] sn-unspecified comparison mode: "when the sn-position is not specified as for instance in PC 18:0_16:1, a mode can be activated to compare all combinations of FA comparisons"
- [readme] Default FA pairing mode: "In default mode, LipidSpace is comparing the first fatty acyl chain (FA) of the first lipid with the first FA of the second lipid"
