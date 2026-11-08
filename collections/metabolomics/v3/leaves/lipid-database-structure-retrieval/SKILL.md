---
name: lipid-database-structure-retrieval
description: Use when a structural-space or structural-similarity analysis needs molecular
  graphs for named lipid species (e.g., PC 12:0/[12-24,26]:0 series) and only species
  names/SMILES-free identifiers are available.
when_to_use_negative:
- The analysis only needs exact species-name matching across lipidomes and no structural
  comparison — name normalization via Goslin alone suffices
- Lipid names are only at sum-composition level (e.g., PC 36:1) with no chain detail;
  retrieved structures cannot resolve sn-order-aware acyl comparisons accurately
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0153
tools:
- name: LIPID MAPS
  role: Structure database from which lipid molecular structures are extracted at
    the highest structural resolution level
- name: SwissLipids
  role: Complementary structure database from which lipid structures are extracted
- name: cppgoslin
  role: Parses and standardizes lipid names into structured annotations before structure
    retrieval
  repo: https://github.com/lifs-tools/cppgoslin
- name: LipidSpace
  role: Consumes the retrieved structures for graph-based structural distance (MCS
    + Jaccard) computation
  repo: https://github.com/lifs-tools/lipidspace
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.3c02449
    title: 'LipidSpace: Simple Exploration, Reanalysis, and Quality Control of Large-Scale
      Lipidomics Studies'
  source_content_hash: sha256:5f93ab7f33349c68daa04e2de4d02033f8e6a9de93a86a2a73aa784cdcf1e6e1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/cppgoslin
  tools:
  - Goslin
  - LIPID MAPS
  - LipidSpace
  - OpenBLAS
  - SwissLipids
  - cppgoslin
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# lipid-database-structure-retrieval

## Summary

Retrieves lipid molecular structures from structure databases such as LIPID MAPS and SwissLipids at the highest structural resolution level, so that downstream graph-based comparison (e.g., LipidSpace's maximum common subgraph + Jaccard index approach) can operate on actual molecular graphs rather than parsed names alone.

## When to use

Use when a structural-space or structural-similarity analysis needs molecular graphs for named lipid species (e.g., PC 12:0/[12-24,26]:0 series) and only species names/SMILES-free identifiers are available.

## When NOT to use

- The analysis only needs exact species-name matching across lipidomes and no structural comparison — name normalization via Goslin alone suffices
- Lipid names are only at sum-composition level (e.g., PC 36:1) with no chain detail; retrieved structures cannot resolve sn-order-aware acyl comparisons accurately

## Inputs

- Lipid species names (e.g., 'PC 12:0/24:0', 'PC 18:0_16:1') from a lipidomics table or publication
- LIPID MAPS structure database entries for the target species
- SwissLipids database entries (optional complementary source)

## Outputs

- Molecular structures at the highest structural resolution level for each lipid species
- Goslin-standardized lipid names with parsed class and fatty acyl annotations
- Structures suitable for maximum common subgraph (MCS/LaWeCSE) distance computation

## How to apply

Start from standardized lipid names — parse them with Goslin (cppgoslin) to unify dialects and confirm class, fatty acyl composition, and sn-position annotations. Query LIPID MAPS (and optionally SwissLipids) to extract the complete molecular structures of each species, taking the highest structural resolution level available (full structural level, not sum-composition level), so headgroups and individual fatty acyl chains can be decomposed into graphs. Feed the retrieved structures into the pairwise structural distance computation — LipidSpace uses the maximum common subgraph (MCS/LaWeCSE) approach combined with the Jaccard index, comparing headgroups via LaWeCSE and fatty acyls in an sn-order-aware manner. When sn-positions are unspecified in names (e.g., PC 18:0_16:1), enable the all-combinations FA comparison mode ('Ignore lipid sn-positions') so the lowest distance over all FA pairings is picked.

## Related tools

- **LIPID MAPS** (Structure database from which lipid molecular structures are extracted at the highest structural resolution level)
- **SwissLipids** (Complementary structure database from which lipid structures are extracted)
- **cppgoslin** (Parses and standardizes lipid names into structured annotations before structure retrieval) — https://github.com/lifs-tools/cppgoslin
- **LipidSpace** (Consumes the retrieved structures for graph-based structural distance (MCS + Jaccard) computation) — https://github.com/lifs-tools/lipidspace

## Examples

```
cd cppgoslin/examples && make && ./lipid_name_parser  # parse 'PC 12:0/24:0' style names before querying LIPID MAPS structures
```

## Evaluation signals

- Every retrieved species resolves to a full-structural-level entry (distinct headgroup and chain-resolved fatty acyls), not a sum-composition placeholder
- Goslin parse succeeds for all lipid names without leaving unresolved dialect forms
- The number of retrieved structures equals the number of input species (e.g., 14 structures for the 14 PC 12:0/[12-24,26]:0 lipids)
- Downstream pairwise structural distances fall in the [0, 1] range of the bound Jaccard-based metric, with identical lipids giving distance 0
- Embedded vs. isolated structural PCA models (e.g., 500-lipid background) preserve the expected qualitative layout, such as the arc-like sequential ordering of the PC series

## Limitations

- Names lacking sn-position specification (e.g., PC 18:0_16:1) force use of the all-combinations FA comparison mode, which is more accurate but decreases performance
- The default bound distance metric caps distances at 1; the unbound metric (0 to infinity) gives more accurate results but reduces visibility of the lipid spaces
- Quantitative abundance information is not added to structural comparison models, so low-abundant species can influence the structural space without abundance weighting

## Evidence

- [methods] Structures from LIPID MAPS and SwissLipids at highest resolution: "extracted all lipids from databases such as LIPID MAPS and SwissLipids on the highest structural resolution level"
- [intro] MCS + Jaccard basis for structural similarity: "we applied the maximum common subgraph (MCS) approach in combination with the Jaccard index to determine a pairwise similarity of lipid species based on the molecular structure"
- [readme] README: FA comparison mode when sn-position is unspecified: "when the sn-position is not specified as for instance in PC 18:0_16:1, a mode can be activated to compare all combinations of FA comparisons for both lipids and picking the lowest distance"
- [readme] README: Goslin purpose for lipid name parsing: [evidence span withheld — anchor status: external_doc]
