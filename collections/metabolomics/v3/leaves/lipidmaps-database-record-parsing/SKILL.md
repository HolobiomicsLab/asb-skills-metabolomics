---
name: lipidmaps-database-record-parsing
description: Use when input records are lipid database entries with explicit double-bond
  positions on carbon chains and you need to extract per-chain double-bond position
  sets in both carbonyl-forward and methyl-end (ω) counting directions for pairwise
  comparison.
when_to_use_negative:
- Input lipid names lack double-bond positional information (species-level shorthand
  like 'PC 34:1') — the highest-resolution extraction requires explicit positions
- The goal is quantitative lipidomics comparison of sample abundances rather than
  structural chain-level database statistics
- Lipids are already parsed into a standardized feature table; re-parsing with Goslin
  is redundant
edam_topics:
- http://edamontology.org/topic_0153
tools:
- name: cppgoslin
  role: Parses and standardizes lipid names from database records into structured
    chain and double-bond information
  repo: https://github.com/lifs-tools/cppgoslin
- name: LipidSpace
  role: Standalone tool whose double-bond comparison logic (FA chain comparisons,
    sn-position handling) the database-derived chain statistics mirror
  repo: https://github.com/lifs-tools/lipidspace
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c02449
    title: 'LipidSpace: Simple Exploration, Reanalysis, and Quality Control of Large-Scale
      Lipidomics Studies'
  source_content_hash: sha256:f91dee2174f757a497524f786156568dc5ec8efaed5a73a1b6538812fec0c25c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/cppgoslin
  tools:
  - LipidSpace
  - cppgoslin
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# lipidmaps-database-record-parsing

## Summary

Parse lipid database records (e.g., LIPID MAPS, SwissLipids) at the highest structural resolution level and standardize fatty acyl chain notation (forward double-bond positions vs. ω-notation) so chain-level structural comparisons such as double-bond mismatch statistics can be computed.

## When to use

Use when input records are lipid database entries with explicit double-bond positions on carbon chains and you need to extract per-chain double-bond position sets in both carbonyl-forward and methyl-end (ω) counting directions for pairwise comparison.

## When NOT to use

- Input lipid names lack double-bond positional information (species-level shorthand like 'PC 34:1') — the highest-resolution extraction requires explicit positions
- The goal is quantitative lipidomics comparison of sample abundances rather than structural chain-level database statistics
- Lipids are already parsed into a standardized feature table; re-parsing with Goslin is redundant

## Inputs

- LIPID MAPS database lipid records at the highest structural resolution level (explicit double-bond positions)
- SwissLipids database lipid records at the highest structural resolution level (optional additional source)
- Lipid species name strings in heterogeneous dialects (e.g., 'PC 18:0_16:1') to be parsed by Goslin

## Outputs

- Per-chain double-bond position sets in forward (carbonyl) and backward (ω) counting directions
- Per-pair comparison outcomes of complete double-bond mismatch
- Direction-wise complete-mismatch probability table written to lipid_mismatch_stats.csv

## How to apply

Download LIPID MAPS (and optionally SwissLipids) lipid records at the highest structural resolution level, i.e., only entries with explicit double-bond positions on their carbon chains. Parse each lipid name with the Goslin parser (cppgoslin), which unifies existing lipid-name dialects into a standardized structure, and extract every carbon chain with its double-bond position list. For each chain compute two double-bond position sets: one counted forward from the carbonyl carbon group and one counted backward from the methyl end (ω-notation), because database shorthand does not distinguish these counting directions. Compare chain pairs in both directions, counting a pair as a complete mismatch when the forward sets and the backward sets have zero overlapping positions. Write per-pair outcomes and direction-wise mismatch fractions to a results table (e.g., lipid_mismatch_stats.csv) and validate that the complete-mismatch probability is >= 73% in both the forward and backward directions, matching the reported LIPID MAPS value.

## Related tools

- **cppgoslin** (Parses and standardizes lipid names from database records into structured chain and double-bond information) — https://github.com/lifs-tools/cppgoslin
- **LipidSpace** (Standalone tool whose double-bond comparison logic (FA chain comparisons, sn-position handling) the database-derived chain statistics mirror) — https://github.com/lifs-tools/lipidspace

## Examples

```
cd examples && make && ./lipid_name_parser  # parse LIPID MAPS lipid names with cppgoslin, then extract chain double-bond positions
```

## Evaluation signals

- Computed complete double-bond mismatch probability is >= 73% in both the forward and backward counting directions
- Forward-direction and backward-direction mismatch probabilities are both reported and agree with the value reported for LIPID MAPS
- Every parsed record yields double-bond position sets in both counting directions; records without explicit positions are excluded
- A per-pair results table (lipid_mismatch_stats.csv) exists with per-pair outcomes and direction-wise probabilities
- cppgoslin test suite passes after installation (make test / make runtests) to confirm the parser handles the lipid-name dialects

## Limitations

- Only lipids at the highest structural resolution level (explicit double-bond positions) can be analyzed; coarser shorthand records must be dropped
- When the sn-position is unspecified (e.g., PC 18:0_16:1), chain comparisons must consider all combinations of fatty acyl chains rather than positional matching
- cppgoslin is an early development version ('please use at your own risk') and may fail to parse some lipid-name dialects

## Evidence

- [methods] Highest structural resolution extraction from LIPID MAPS and SwissLipids: "extracted all lipids from databases such as LIPID MAPS and SwissLipids on the highest structural resolution level"
- [methods] Goslin used to parse and standardize lipid names: "It utilizes the C++ library of Goslin4,5 to parse and standardize lipid names"
- [readme] cppgoslin's core parsing and printing functions: [evidence span withheld — anchor status: external_doc]
- [readme] cppgoslin unifies lipid-name dialects: [evidence span withheld — anchor status: external_doc]
- [readme] sn-position-agnostic chain comparisons: "a mode can be activated to compare all combinations of FA comparisons for both lipids and picking the lowest distance"
