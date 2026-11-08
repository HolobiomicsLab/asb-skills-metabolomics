---
name: lipid-database-accession-linking
description: Use when lipid shorthand annotations (e.g. PC 16:0_18:2, MS-DIAL/LipidSearch-style
  abbreviations, DHA-style aliases) must be cross-matched to LIPID MAPS, SwissLipids,
  or PubChem accession IDs for cross-database comparison or systems biology integration.
when_to_use_negative:
- Annotations are plant or bacterial lipids — LipidLynxX currently covers mammalian
  lipids from 25 classes across fatty acids, glycerolipids, glycerophospholipids,
  sphingolipids, and sterols
- Input is already a unified identifier table with LIPID MAPS/SwissLipids accessions
  — re-running conversion adds no annotation-level equalization value
- The goal is metabolic pathway/network analysis rather than identifier cross-matching
  — use the downstream Linker/integration tools (BioPAN, Metabolomics2Networks) directly
  instead
edam_operation: http://edamontology.org/operation_0289
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: LipidLynxX
  role: Converter normalizes abbreviations to uniformed LipidLynxX IDs; Linker links
    abbreviations to external database resources and API identifier mapping
  repo: https://github.com/SysMedOs/LipidLynxX
- name: SwissLipids
  role: Target accession ID mapping service queried via API during cross-mapping
- name: LIPID MAPS
  role: Target accession ID mapping service queried via API during cross-mapping;
    also hosts LipidLynxX on the LIPID MAPS Lipidomics Gateway
- name: PubChem
  role: General chemical-entity accession namespace included in the cross-mapping
    output table
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2020.04.09.033894
    title: 'LipidLynxX: a data transfer hub to support integration of large scale
      lipidomics datasets'
  source_content_hash: sha256:ae739e4fb8e82ca1ec1c18df5743f822cd8e10c6895645b5f16805b355042eec
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
  - LIPID MAPS
  - LipidLynxX
  - PubChem
  - SwissLipids
  edam_operation: http://edamontology.org/operation_0289
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# lipid-database-accession-linking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Cross-maps lipid shorthand annotations into external accession identifiers (LIPID MAPS, SwissLipids, PubChem) via the LipidLynxX Converter and Linker modules, enabling experimental lipidomics tables to be linked to data integration tools. Validated on the paper's 131-lipid example dataset with zero conversion failures.

## When to use

Use when lipid shorthand annotations (e.g. PC 16:0_18:2, MS-DIAL/LipidSearch-style abbreviations, DHA-style aliases) must be cross-matched to LIPID MAPS, SwissLipids, or PubChem accession IDs for cross-database comparison or systems biology integration.

## When NOT to use

- Annotations are plant or bacterial lipids — LipidLynxX currently covers mammalian lipids from 25 classes across fatty acids, glycerolipids, glycerophospholipids, sphingolipids, and sterols
- Input is already a unified identifier table with LIPID MAPS/SwissLipids accessions — re-running conversion adds no annotation-level equalization value
- The goal is metabolic pathway/network analysis rather than identifier cross-matching — use the downstream Linker/integration tools (BioPAN, Metabolomics2Networks) directly instead

## Inputs

- Batch list of lipid shorthand annotations or abbreviations (CSV/text, e.g. 131-lipid Supplementary File 2 of bioRxiv 10.1101/2020.04.09.033894)
- Lipid identifiers from supported sources (HMDB, LIPID MAPS LMSD & COMP_DB, LipidHome, RefMet, SwissLipids; or program outputs from MS-DIAL, MZmine2, LipidSearch, etc.)

## Outputs

- Conversion table (CSV or XLSX) with input shorthand plus shorthand notation, LipidLynxX ID, LIPID MAPS, SwissLipids, and PubChem accession identifiers per lipid
- Extracted biochemical reaction and protein resource links via the Linker module

## How to apply

First normalize input abbreviations to the canonical LipidLynxX shorthand at the equalized structural annotation level using the LipidLynxX Converter, which reads 26 source styles (5 databases: HMDB, LIPID MAPS LMSD & COMP_DB, LipidHome, RefMet, SwissLipids; 17 programs including MS-DIAL, MZmine2, LipidSearch) plus space shorthand (PC 16:0_18:2), bracket shorthand (PC(16:0_18:2)), and defined aliases (DHA, PLAP). Then use the Linker module to query the accession ID mapping API services of SwissLipids and LIPID MAPS, and PubChem for general chemical entities. Batch-process the annotation list and write one combined table with the input shorthand plus shorthand, LipidLynxX, LIPID MAPS, SwissLipids, and PubChem identifiers per lipid (e.g. converted_annotations.csv). If a database/program style is unsupported, check lynx/configurations/rules/input configs before requesting a new configuration file.

## Related tools

- **LipidLynxX** (Converter normalizes abbreviations to uniformed LipidLynxX IDs; Linker links abbreviations to external database resources and API identifier mapping) — https://github.com/SysMedOs/LipidLynxX
- **SwissLipids** (Target accession ID mapping service queried via API during cross-mapping)
- **LIPID MAPS** (Target accession ID mapping service queried via API during cross-mapping; also hosts LipidLynxX on the LIPID MAPS Lipidomics Gateway)
- **PubChem** (General chemical-entity accession namespace included in the cross-mapping output table)

## Examples

```
python cli_lynx.py
```

## Evaluation signals

- All 131 input annotations from the example dataset yield converted identifiers with zero conversion failures
- Output table contains one row per input lipid with all five columns populated: input shorthand, LipidLynxX shorthand, LIPID MAPS, SwissLipids, and PubChem identifiers
- Converted LipidLynxX IDs adhere to the strictly controlled JSON-schema format and unified position-specific annotation rules
- Unconverted IDs (blank identifier cells) are absent, or reproducibly explainable by a missing input rule configuration rather than a conversion error
- Identifiers resolved via Linker map back consistently when re-run (deterministic cross-matching at the equalized structural annotation level)

## Limitations

- Coverage is restricted to mammalian lipids from 25 classes representing five categories (fatty acids, glycerolipids, glycerophospholipids, sphingolipids, sterols); plant and bacterial lipids are not covered
- Different annotation levels between experimental datasets and genome-scale metabolic models (GSMN) remain a matching challenge
- Linker is unstable on long runs: if it runs more than 300 s or more than 30 s per ID, restart LipidLynxX; run at most 3 tasks concurrently in the packaged Windows build and allow firewall access for database queries
- API definitions and documentation in the source code may lag rapid code changes — contact developers or follow the repository before relying on the API

## Evidence

- [readme] Supported shorthand styles (README): [evidence span withheld — anchor status: external_doc]
