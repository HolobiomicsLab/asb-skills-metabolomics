---
name: directional-db-mismatch-probability-computation
description: Use when validating whether a structural-comparison tool's double bond
  position matching is correct, by quantifying mismatch probabilities over high-resolution
  reference lipid entries from SwissLipids or LIPID MAPS.
when_to_use_negative:
- When the input is user lipidomics sample data rather than a reference database of
  fully resolved lipid structures — the mismatch statistic requires structural ground
  truth, not measured abundances
- When lipid identifiers lack explicit double bond positions (e.g., species-level
  names like PC 34:1); the comparison is undefined until names are parsed to a higher
  structural resolution
- When the goal is lipidome similarity or maximum-common-subgraph distance computation
  rather than assessing double bond position comparison correctness
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: LipidSpace
  role: Tool whose double bond position comparison correctness is grounded and validated
    by the computed mismatch probabilities
  repo: https://github.com/lifs-tools/lipidspace
- name: SwissLipids
  role: Reference lipid structure database providing the ground-truth double bond
    position entries for the pairwise mismatch computation
- name: LIPID MAPS
  role: Alternative reference database of high-resolution lipid structures usable
    alongside SwissLipids
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c02449
    title: 'LipidSpace: Simple Exploration, Reanalysis, and Quality Control of Large-Scale
      Lipidomics Studies'
  source_content_hash: sha256:dfd2eceea421b46f2dd32255582967c9842601ae52058dcdb9c866b510907781
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/lipidspace
  tools:
  - LIPID MAPS
  - LipidSpace
  - SwissLipids
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# directional-db-mismatch-probability-computation

## Summary

Computes, over a reference lipid structure database (SwissLipids), the probability that two arbitrary fatty-acyl carbon chains share no double bond at the same position, under both counting conventions (forward from the carbonyl carbon and backward/omega from the methyl end), to ground the correctness of double bond position comparisons in LipidSpace.

## When to use

Use when validating whether a structural-comparison tool's double bond position matching is correct, by quantifying mismatch probabilities over high-resolution reference lipid entries from SwissLipids or LIPID MAPS.

## When NOT to use

- When the input is user lipidomics sample data rather than a reference database of fully resolved lipid structures — the mismatch statistic requires structural ground truth, not measured abundances
- When lipid identifiers lack explicit double bond positions (e.g., species-level names like PC 34:1); the comparison is undefined until names are parsed to a higher structural resolution
- When the goal is lipidome similarity or maximum-common-subgraph distance computation rather than assessing double bond position comparison correctness

## Inputs

- SwissLipids database export containing lipid entries at the highest structural resolution level with explicit double bond positions on fatty-acyl carbon chains
- Optionally LIPID MAPS entries at the same structural resolution level as an additional reference set

## Outputs

- Table (CSV) of forward (carbonyl-based) and backward (omega-based) double bond mismatch probabilities
- Verified collection of distinct double bond position sets extracted from the reference database (expected 85 for SwissLipids)

## How to apply

Download the SwissLipids database content and extract all lipid entries at the highest structural resolution level whose carbon chains carry explicit double bond positions. Parse the fatty-acyl chain information and collect the distinct double bond position sets, verifying the count of 85 distinct DB position sets reported for SwissLipids. Perform a pairwise comparison over all carbon chains and, for each of the two counting conventions — forward counting from the carbonyl carbon group and backward (omega) counting from the methyl group — compute the probability that two arbitrary chains share no overlapping double bond at the same position. Tabulate the forward and backward mismatch probabilities side by side and write them to a CSV file for comparison with published values.

## Related tools

- **LipidSpace** (Tool whose double bond position comparison correctness is grounded and validated by the computed mismatch probabilities) — https://github.com/lifs-tools/lipidspace
- **SwissLipids** (Reference lipid structure database providing the ground-truth double bond position entries for the pairwise mismatch computation)
- **LIPID MAPS** (Alternative reference database of high-resolution lipid structures usable alongside SwissLipids)

## Evaluation signals

- The number of distinct double bond position sets extracted from SwissLipids carbon chains matches the reported 85
- The forward-counting (carbonyl-based) mismatch probability reproduces the reported value of approximately 64%
- The backward (omega-counting) mismatch probability exceeds 71%, as stated in the article's 'Correctness of Double Bond Comparisons' section
- Both counting conventions are computed on the same parsed chain set, so the directional probabilities differ only by the counting origin
- Results are tabulated and written to a CSV file with separate forward and backward probability entries

## Limitations

- Results depend on extracting entries at the highest structural resolution level; coarser-resolution entries without explicit double bond positions cannot contribute to the pairwise comparison
- The reported benchmark values (85 distinct DB position sets, ~64% forward, >71% backward) are specific to SwissLipids content and will differ for other reference databases such as LIPID MAPS
- LipidSpace comparisons themselves are chain-bound by default (first FA to first FA); sn-position-unspecified pairs like PC 18:0_16:1 require the all-combinations mode, which is more accurate but reduces performance

## Evidence

- [methods] extracted all lipids from databases such as LIPID MAPS and SwissLipids on the highest structural resolution level: "extracted all lipids from databases such as LIPID MAPS and SwissLipids on the highest structural resolution level"
- [methods] Correctness of Double Bond Comparisons: "Correctness of Double
Bond Comparisons"
- [readme] sn-position not specified: "when the sn-position is not specified as for instance in PC 18:0_16:1, a mode can be activated to compare all combinations of FA comparisons for both lipids and picking the lowest distance"
