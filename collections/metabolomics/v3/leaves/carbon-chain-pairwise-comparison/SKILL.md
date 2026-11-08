---
name: carbon-chain-pairwise-comparison
description: Use when you need to quantify how likely two arbitrary carbon chains
  share no double bond at the same position (mismatch probability) — e.g. to validate
  that a lipidomics tool's double bond similarity is meaningful rather than trivially
  satisfied.
when_to_use_negative:
- Input lipid entries are not at the highest structural resolution level (double bond
  positions unspecified, e.g. PC 18:1) — the mismatch computation is undefined for
  chains without explicit positions
- The goal is lipidome-level sample statistics (PCA, Hausdorff distances, feature
  analysis) rather than structural-correctness benchmarking of double bond comparison
- You only need pairwise lipid similarity for exploration — use the full LipidSpace
  MCS/Jaccard workflow instead of the aggregate mismatch statistics
edam_operation: http://edamontology.org/operation_3435
tools:
- name: LipidSpace
  role: Tool whose double bond position comparison logic is being benchmarked; its
    structural comparison ground truth is the LIPID MAPS/SwissLipids high-resolution
    reference set
  repo: https://github.com/lifs-tools/lipidspace
- name: Goslin (cppgoslin)
  role: Parses and standardizes lipid names into structured fatty acyl chains so double
    bond positions can be extracted per counting convention
  repo: https://github.com/lifs-tools/cppgoslin
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c02449
    title: 'LipidSpace: Simple Exploration, Reanalysis, and Quality Control of Large-Scale
      Lipidomics Studies'
  source_content_hash: sha256:8975091890162b64089ed8d4c28fb2c8cdda44c7fea07d65f398c44fc561b180
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/lipidspace
  tools:
  - Goslin (cppgoslin)
  - LipidSpace
  edam_operation: http://edamontology.org/operation_3435
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# carbon-chain-pairwise-comparison

## Summary

Grounds the correctness evaluation of double bond position comparisons in reference lipid structure databases (LIPID MAPS, SwissLipids) by extracting all lipids at the highest structural resolution level, then performing an all-against-all pairwise comparison of fatty acyl carbon chains under two counting conventions (forward from the carbonyl carbon, backward/omega from the methyl end).

## When to use

Use when you need to quantify how likely two arbitrary carbon chains share no double bond at the same position (mismatch probability) — e.g. to validate that a lipidomics tool's double bond similarity is meaningful rather than trivially satisfied.

## When NOT to use

- Input lipid entries are not at the highest structural resolution level (double bond positions unspecified, e.g. PC 18:1) — the mismatch computation is undefined for chains without explicit positions
- The goal is lipidome-level sample statistics (PCA, Hausdorff distances, feature analysis) rather than structural-correctness benchmarking of double bond comparison
- You only need pairwise lipid similarity for exploration — use the full LipidSpace MCS/Jaccard workflow instead of the aggregate mismatch statistics

## Inputs

- SwissLipids database export at the highest structural resolution level with explicit double bond positions on fatty acyl chains
- LIPID MAPS database export at the highest structural resolution level (optional second reference set)
- Structured fatty acyl chain representations (chain length, double bond count, double bond positions) parsed from lipid names

## Outputs

- Set of distinct double bond position sets per reference database (expected: 85 for SwissLipids)
- Forward-counting (carbonyl-based) mismatch probability for the SwissLipids carbon chain ensemble
- Backward/omega-counting (methyl-based) mismatch probability for the SwissLipids carbon chain ensemble
- CSV file tabulating the forward and backward mismatch probabilities

## How to apply

Download the SwissLipids (and optionally LIPID MAPS) content and extract all lipid entries at the highest structural resolution level that carry explicit double bond positions on their carbon chains; verify you recover the 85 distinct double bond position sets reported for SwissLipids. Parse the carbon chains (Goslin/cppgoslin can standardize lipid names into structured fatty acyl chain objects with double bond positions). Then perform a pairwise comparison of all carbon chains twice: once counting double bond positions forward from the carbonyl carbon group, and once counting backward from the methyl group (omega notation), computing for each convention the probability that two arbitrary chains share no overlapping double bond position. Tabulate the forward and backward mismatch probabilities and compare against the reported SwissLipids values (~64% forward, >71% backward), writing results to a CSV file. The asymmetry between the two conventions is the rationale: omega-counted double bonds (e.g. omega-3, omega-6) overlap far less often, so back-counting yields a stronger correctness signal.

## Related tools

- **LipidSpace** (Tool whose double bond position comparison logic is being benchmarked; its structural comparison ground truth is the LIPID MAPS/SwissLipids high-resolution reference set) — https://github.com/lifs-tools/lipidspace
- **Goslin (cppgoslin)** (Parses and standardizes lipid names into structured fatty acyl chains so double bond positions can be extracted per counting convention) — https://github.com/lifs-tools/cppgoslin

## Examples

```
python -c "import pandas as pd; chains = pd.read_csv('swisslipids_db_sets.csv'); n=len(chains); fwd = 1 - (chains.apply(lambda r: any(p in r.db_fwd for _ in [0]), axis=1)).mean()" --out db_mismatch.csv
```

## Evaluation signals

- The count of distinct double bond position sets extracted from SwissLipids reproduces the reported value of 85
- The forward (carbonyl-based) mismatch probability is approximately 64%
- The backward (omega-based) mismatch probability exceeds 71%
- The backward mismatch probability is strictly larger than the forward one, confirming the convention asymmetry
- The output CSV contains both the forward and backward probabilities and matches the 'Correctness of Double Bond Comparisons' section values

## Limitations

- Comparison correctness depends on sn-position specification: when sn-positions are unspecified (e.g. PC 18:0_16:1), all FA chain combinations must be compared to pick the lowest distance, which is more accurate but decreases performance
- Results are specific to the reference database snapshot (SwissLipids/LIPID MAPS at highest structural resolution); other databases or lower resolution levels will shift the mismatch probabilities
- The benchmark addresses only double bond position overlap of fatty acyl chains, not headgroup, abundance, or whole-lipid similarity correctness

## Evidence

- [methods] Reference database extraction at highest resolution: "extracted all lipids from databases such as LIPID MAPS and SwissLipids on the highest structural resolution level"
- [methods] Lipid name parsing standardization: "It utilizes the C++ library of Goslin4,5 to parse and standardize lipid names"
- [abstract] Structural comparison basis: "A graph-based comparison of lipid structures is the basis for calculating structural space models and subsequently computing lipidome similarities."
- [readme] sn-position handling in FA chain comparison: "a mode can be activated to compare all combinations of FA comparisons for both lipids and picking the lowest distance. The results are more accurate, but the performance is decreased"
