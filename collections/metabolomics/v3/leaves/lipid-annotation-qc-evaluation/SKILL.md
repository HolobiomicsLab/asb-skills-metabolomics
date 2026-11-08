---
name: lipid-annotation-qc-evaluation
description: Use when lipid structure records (e.g., LIPID MAPS or SwissLipids exports
  at the highest structural resolution level) must be checked for correct double bond
  position encoding, or when the plausibility of double bond annotations in a lipid
  database must be quantified.
when_to_use_negative:
- Input lipid species lack explicit double bond positions (only summary notation like
  PC 38:4 is available) — no positional sets can be computed
- The goal is lipidome-level quantitative comparison or feature analysis rather than
  structural annotation correctness — use LipidSpace's MCS/Jaccard structural comparison
  or statistics module instead
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3353
- http://edamontology.org/topic_0153
tools:
- name: LipidSpace
  role: Lipidome analysis tool that relies on correct double bond comparisons between
    fatty acyl chains (first-FA vs all-combinations FA matching) and provides quality
    control workflows
  repo: https://github.com/lifs-tools/lipidspace
- name: cppgoslin
  role: Parses and standardizes lipid names into structured records from which fatty
    acyl chains and double bond positions can be extracted
  repo: https://github.com/lifs-tools/cppgoslin
- name: LIPID MAPS
  role: Source database of lipid records at the highest structural resolution level
    from which carbon chain double bond positions are extracted
- name: SwissLipids
  role: Secondary source database of high-resolution lipid structure records for cross-database
    validation of the mismatch probability
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c02449
    title: 'LipidSpace: Simple Exploration, Reanalysis, and Quality Control of Large-Scale
      Lipidomics Studies'
  source_content_hash: sha256:e165660c1f3178b5b5f7313255508bf8bdfd87e275cc1ef6e6cbc156f43a3808
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/lipidspace
  tools:
  - LIPID MAPS
  - LipidSpace
  - SwissLipids
  - cppgoslin
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3353
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# lipid-annotation-qc-evaluation

## Summary

Quality-control evaluation of lipid structural annotations by extracting double bond positions from fatty acyl chains (forward from the carbonyl carbon and backward in ω-notation) and computing the probability of complete double bond mismatch between chain pairs, validated against the ~73% threshold reported for LIPID MAPS.

## When to use

Use when lipid structure records (e.g., LIPID MAPS or SwissLipids exports at the highest structural resolution level) must be checked for correct double bond position encoding, or when the plausibility of double bond annotations in a lipid database must be quantified.

## When NOT to use

- Input lipid species lack explicit double bond positions (only summary notation like PC 38:4 is available) — no positional sets can be computed
- The goal is lipidome-level quantitative comparison or feature analysis rather than structural annotation correctness — use LipidSpace's MCS/Jaccard structural comparison or statistics module instead

## Inputs

- LIPID MAPS lipid records at the highest structural resolution level (explicit double bond positions on carbon chains)
- SwissLipids lipid records at the highest structural resolution level
- Parsed fatty acyl chain double bond position lists (forward carbonyl-based and backward ω-notation)

## Outputs

- Per-pair double bond comparison outcome table (lipid_mismatch_stats.csv)
- Direction-wise (forward and backward/ω) complete-mismatch probabilities
- Pass/fail validation against the >= 73% complete-mismatch threshold

## How to apply

Download lipid records at the highest structural resolution level (explicit double bond positions on carbon chains) from LIPID MAPS and SwissLipids. For each fatty acyl chain, list every double bond position and compute two position sets: one counting forward from the carbonyl carbon group and one backward from the methyl end (ω-notation). Perform pairwise comparison of all carbon chains, counting for each pair whether the forward sets and backward sets have zero overlapping double bond positions. Compute the complete-mismatch probability as the fraction of chain pairs with no overlapping double bond in each counting direction, and write per-pair outcomes and direction-wise probabilities to a results table (e.g., lipid_mismatch_stats.csv). The analysis is correct when the computed complete-mismatch probability is >= 73% in both the forward and backward directions, matching the value reported for the LIPID MAPS database. Note that chain-level correctness of double bond comparisons is also a concern in lipidome analysis tools: LipidSpace compares the first FA of one lipid with the first FA of the second lipid, and can be set to compare all FA combinations (picking the lowest distance) when sn-positions are unspecified as in PC 18:0_16:1.

## Related tools

- **LipidSpace** (Lipidome analysis tool that relies on correct double bond comparisons between fatty acyl chains (first-FA vs all-combinations FA matching) and provides quality control workflows) — https://github.com/lifs-tools/lipidspace
- **cppgoslin** (Parses and standardizes lipid names into structured records from which fatty acyl chains and double bond positions can be extracted) — https://github.com/lifs-tools/cppgoslin
- **LIPID MAPS** (Source database of lipid records at the highest structural resolution level from which carbon chain double bond positions are extracted)
- **SwissLipids** (Secondary source database of high-resolution lipid structure records for cross-database validation of the mismatch probability)

## Evaluation signals

- Computed complete-mismatch probability is >= 73% in the forward (carbonyl-based) counting direction
- Computed complete-mismatch probability is >= 73% in the backward (ω-notation) counting direction
- lipid_mismatch_stats.csv contains per-pair outcomes and both direction-wise probabilities with non-empty overlap counts
- Lipid name parsing round-trips through Goslin/cppgoslin without errors, so double bond positions are extracted from every record
- For lipids with unspecified sn-positions (e.g., PC 18:0_16:1), the all-combinations FA comparison mode yields lower distances than the default first-FA-vs-first-FA mode

## Limitations

- The 73% threshold is database-specific; it was reported for LIPID MAPS and may differ for other lipid databases or resolution levels
- LipidSpace cannot save or load a current analysis, so the QC workflow must be re-run end-to-end
- Comparing only positional FA pairs when sn-positions are unspecified (default mode) can give inaccurate structural distances; the all-combinations mode is more accurate but decreases performance

## Evidence

- [methods] extracted all lipids from databases such as LIPID MAPS and SwissLipids on the highest structural resolution level: "extracted all lipids from databases such as LIPID MAPS and SwissLipids on the highest structural resolution level"
- [methods] Correctness of Double Bond Comparisons: "Correctness of Double
Bond Comparisons"
- [readme] In default mode, LipidSpace is comparing the first fatty acyl chain (FA) of the first lipid with the first FA of the second lipid: "In default mode, LipidSpace is comparing the first fatty acyl chain (FA) of the first lipid with the first FA of the second lipid"
- [readme] compare all combinations of FA comparisons for both lipids and picking the lowest distance: "a mode can be activated to compare all combinations of FA comparisons for both lipids and picking the lowest distance"
