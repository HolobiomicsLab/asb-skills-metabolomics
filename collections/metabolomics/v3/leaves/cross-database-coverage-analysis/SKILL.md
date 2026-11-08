---
name: cross-database-coverage-analysis
description: Use when a lipidomics dataset has been translated through LipidLynxX
  Linker and you need per-database mapping counts (e.g. LION/KEGG/ChEBI/Rhea/UniProtKB)
  plus class-level summaries of unmatched identifiers.
when_to_use_negative:
- The goal is lipid abbreviation conversion or level equalization (Converter/Equalizer
  modules), not assessing cross-reference coverage of integration databases
- The input lipids are plant or bacterial lipids, which are not covered by LipidLynxX's
  current mammalian-lipid scope
- You need actual identifier values or reaction/protein links per lipid rather than
  presence/absence coverage statistics
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3353
- http://edamontology.org/topic_0089
tools:
- name: LipidLynxX
  role: Linker module translates lipid annotations into identifiers for data integration
    targets; Converter/Equalizer produce the unified LipidLynxX IDs upstream of coverage
    analysis
  repo: https://github.com/SysMedOs/LipidLynxX
- name: LION
  role: Lipid ontology target database whose identifier coverage is quantified in
    the coverage matrix
- name: pandas
  role: Builds the binary presence/absence coverage matrix and aggregates unmatched
    identifiers by lipid class
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2020.04.09.033894
    title: 'LipidLynxX: a data transfer hub to support integration of large scale
      lipidomics datasets'
  source_content_hash: sha256:29867fc27528a6848d075fffd063620d43da5d243823ef30921e9a06c5e6fa1e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/SysMedOs/LipidLynxX
  repo_url: https://github.com/SysMedOs/LipidLynxX
  tools:
  - LION
  - Lion
  - LipidLynxX
  - pandas
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3353
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# cross-database-coverage-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantify and summarize how many lipids in a dataset obtain identifier mappings to each external data-integration resource (LION, KEGG, ChEBI, Rhea, UniProtKB) using LipidLynxX Linker translations, and aggregate unmatched identifiers by lipid class to reveal systematically unmapped classes.

## When to use

Use when a lipidomics dataset has been translated through LipidLynxX Linker and you need per-database mapping counts (e.g. LION/KEGG/ChEBI/Rhea/UniProtKB) plus class-level summaries of unmatched identifiers.

## When NOT to use

- The goal is lipid abbreviation conversion or level equalization (Converter/Equalizer modules), not assessing cross-reference coverage of integration databases
- The input lipids are plant or bacterial lipids, which are not covered by LipidLynxX's current mammalian-lipid scope
- You need actual identifier values or reaction/protein links per lipid rather than presence/absence coverage statistics

## Inputs

- Per-lipid LipidLynxX Linker translation results table (e.g. 131 lipids) with cross-reference identifier columns
- Lipid class/category annotation per lipid (fatty acids, glycerolipids, glycerophospholipids, sphingolipids, sterols)
- List of target integration databases (LION, KEGG, ChEBI, Rhea, UniProtKB)

## Outputs

- Binary lipid × database presence/absence coverage matrix (131 rows × 5 database columns)
- Per-database unmatched-class summary (NaN identifier counts aggregated by lipid class/category)
- linker_coverage_table.csv

## How to apply

Load the per-lipid Linker translation results produced by LipidLynxX (v0.9.24 supports export as .csv/.xlsx) and parse the cross-reference identifier columns for the five target resources: LION, KEGG, ChEBI, Rhea, and UniProtKB. Build a binary presence/absence coverage matrix with one row per lipid (e.g. the 131-lipid test set) and one column per target database using pandas, treating missing (NaN) identifiers as absent. Aggregate unmatched identifiers by lipid class and category — fatty acids, glycerolipids, glycerophospholipids, sphingolipids, and sterols — to detect classes that are systematically unmatched per database. Rationale: LipidLynxX's Linker module directly translates lipid annotations into identifiers for data integration tools, so coverage gaps reflect both the lipid classes present (the paper covers mammalian lipids from 25 classes in five categories) and each target database's annotation depth. Write the coverage matrix and per-database unmatched-class summary to linker_coverage_table.csv and validate that row/column counts and per-class rollups are consistent with the binary matrix.

## Related tools

- **LipidLynxX** (Linker module translates lipid annotations into identifiers for data integration targets; Converter/Equalizer produce the unified LipidLynxX IDs upstream of coverage analysis) — https://github.com/SysMedOs/LipidLynxX
- **LION** (Lipid ontology target database whose identifier coverage is quantified in the coverage matrix)
- **pandas** (Builds the binary presence/absence coverage matrix and aggregates unmatched identifiers by lipid class)

## Examples

```
import pandas as pd; cov = linker_df[['LION_ID','KEGG_ID','ChEBI_ID','Rhea_ID','UniProt_ID']].notna().astype(int); cov.to_csv('linker_coverage_table.csv')
```

## Evaluation signals

- Coverage table contains exactly 131 lipid rows and five database coverage columns
- Per-class unmatched summary is consistent with (recomputable from) the per-lipid binary matrix
- Reported per-database counts reproduce the reference finding: 98/131 mapped to LION, 49/131 to ChEBI, 130/131 to KEGG compound IDs
- No lipid appears with a mapped identifier for a database that its binary matrix row marks absent (NaN vs non-NaN consistency)

## Limitations

- Plant and bacterial lipids are not currently covered by LipidLynxX, which restricts the mammalian 25-class/five-category scope (fatty acids, glycerolipids, glycerophospholipids, sphingolipids, sterols)
- Known Linker runtime issues: if it runs more than 300 s or more than 30 s per ID, restart LipidLynxX; run max 3 tasks concurrently in the Windows .exe version
- Different annotation levels between experimental datasets and target databases (e.g. GSMN) can be a challenge when interpreting coverage gaps
- No specific LipidLynxX version or build is fixed in the article's discussion, so coverage numbers may shift with tool version

## Evidence

- [discussion] LipidLynxX Linker module enables direct translation of lipid annotations into identifiers: "LipidLynxX Linker module enables direct translation of lipid annotations into identifiers"
- [discussion] LipidLynxX Linker additionally allows to extract information on biochemical reactions: "LipidLynxX Linker additionally allows to extract information on biochemical reactions"
- [discussion] mammalian lipids from 25 classes representing five categories – fatty acids, glycerolipids, glycerophospholipids, sphingolipids, and sterols: "mammalian lipids from 25 classes representing five categories – fatty acids, glycerolipids, glycerophospholipids, sphingolipids, and sterols"
- [discussion] LipidLynxX provides multiple opportunities for lipidomics data conversion, cross-matching, and translation: "LipidLynxX provides multiple opportunities for lipidomics data conversion, cross-matching, and translation"
- [readme] select export style, and download the output table as `.csv` or `.xlsx`: [evidence span withheld — anchor status: external_doc]
- [readme] Known issues: if Linker runs more than 300s or more than 30s/per ID, please restart LipidLynxX and try again.: [evidence span withheld — anchor status: external_doc]
