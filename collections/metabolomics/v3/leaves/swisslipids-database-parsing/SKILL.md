---
name: swisslipids-database-parsing
description: Use when a lipidomics tool's structural similarity or double bond position
  comparison must be validated against curated reference lipid structure databases
  rather than against measured data.
when_to_use_negative:
- The input is experimental lipidomics intensity data (e.g. csv/xlsx or mzTab-M sample
  tables) rather than a curated structural reference database — use LipidSpace's data
  import and structural comparison workflow instead.
- Lipid names in the analysis are not resolved to the highest structural resolution
  level (no explicit double bond positions), since the reference comparison requires
  fully specified DB positions.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_0153
tools:
- name: LipidSpace
  role: The standalone tool whose double bond comparison correctness is grounded in
    the reference database extraction; its graph-based lipid structure comparison
    consumes the parsed chain structures
  repo: https://github.com/lifs-tools/lipidspace
- name: SwissLipids
  role: Reference structural database providing the highest-resolution lipid entries
    with explicit double bond positions used as structural ground truth
- name: LIPID MAPS
  role: Additional reference structural database from which lipids are extracted at
    the highest structural resolution level
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c02449
    title: 'LipidSpace: Simple Exploration, Reanalysis, and Quality Control of Large-Scale
      Lipidomics Studies'
  source_content_hash: sha256:ea1f12520dba2179c0f5a16a3a4bcc5776d0a6fc1bbec859d389def4d72852b8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/lipidspace
  tools:
  - LIPID MAPS
  - LipidSpace
  - SwissLipids
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# swisslipids-database-parsing

## Summary

Extracts all lipids from reference structure databases such as SwissLipids and LIPID MAPS at the highest structural resolution level to establish the structural ground truth against which double bond position comparisons are verified.

## When to use

Use when a lipidomics tool's structural similarity or double bond position comparison must be validated against curated reference lipid structure databases rather than against measured data.

## When NOT to use

- The input is experimental lipidomics intensity data (e.g. csv/xlsx or mzTab-M sample tables) rather than a curated structural reference database — use LipidSpace's data import and structural comparison workflow instead.
- Lipid names in the analysis are not resolved to the highest structural resolution level (no explicit double bond positions), since the reference comparison requires fully specified DB positions.

## Inputs

- SwissLipids database content (lipid entries with explicit double bond positions at the highest structural resolution level)
- LIPID MAPS database content (alternative/additional reference structural database)
- Carbon chain / fatty acyl structure records parsed from the reference entries

## Outputs

- Set of distinct double bond position sets per reference database (85 for SwissLipids)
- Forward (carbonyl-anchored) and backward (omega) double bond mismatch probabilities per database (~64% and >71% for SwissLipids)
- CSV file tabulating forward and backward mismatch probabilities

## How to apply

Download the SwissLipids (and optionally LIPID MAPS) database content and extract every lipid entry at the highest structural resolution level, i.e. entries with explicit double bond positions on their carbon chains. Parse the carbon chain (fatty acyl) information and collect the distinct double bond position sets — for SwissLipids this should yield 85 distinct DB position sets, which serves as a count-level sanity check. Then, for each of the two counting conventions — forward counting from the carbonyl carbon group and backward (omega) counting from the methyl end — perform a pairwise comparison over all carbon chains to compute the probability that two arbitrary chains share no overlapping double bond at the same position. Tabulate the forward and backward mismatch probabilities (expected approximately 64% forward and over 71% backward for SwissLipids) and write the results to a CSV file for comparison against the values reported in the article's 'Correctness of Double Bond Comparisons' section.

## Related tools

- **LipidSpace** (The standalone tool whose double bond comparison correctness is grounded in the reference database extraction; its graph-based lipid structure comparison consumes the parsed chain structures) — https://github.com/lifs-tools/lipidspace
- **SwissLipids** (Reference structural database providing the highest-resolution lipid entries with explicit double bond positions used as structural ground truth)
- **LIPID MAPS** (Additional reference structural database from which lipids are extracted at the highest structural resolution level)

## Evaluation signals

- The number of distinct double bond position sets recovered from SwissLipids equals 85.
- The computed forward-counting mismatch probability matches the reported ~64% for SwissLipids.
- The computed backward (omega) counting mismatch probability exceeds 71%, matching the reported value.
- Results are written to a CSV file with separate forward and backward mismatch probability columns per database.
- Only entries at the highest structural resolution level (explicit DB positions on carbon chains) contribute to the statistics.

## Limitations

- LipidSpace's default FA comparison pairs chains by sn-position (first FA with first FA); a combinatorial 'ignore sn-positions' mode is more accurate but decreases performance.
- The mismatch probability estimates depend on the content and resolution snapshot of SwissLipids/LIPID MAPS at the time of extraction; database updates can shift the counts and probabilities.

## Evidence

- [methods] extracted all lipids from databases such as LIPID MAPS and SwissLipids on the highest structural resolution level: "extracted all lipids from databases such as LIPID MAPS and SwissLipids on the highest structural resolution level"
- [methods] Correctness of Double Bond Comparisons: "Correctness of Double Bond Comparisons"
- [readme] graph-based comparison: "A graph-based comparison of lipid structures allows to calculate distances between lipids and to determine similarities across lipidomes."
- [readme] sn-position comparison mode: "a mode can be activated to compare all combinations of FA comparisons for both lipids and picking the lowest distance. The results are more accurate, but the performance is decreased."
