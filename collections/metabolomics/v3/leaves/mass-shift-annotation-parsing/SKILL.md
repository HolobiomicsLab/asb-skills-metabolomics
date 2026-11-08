---
name: mass-shift-annotation-parsing
description: Use when lipidomics annotations contain modified-lipid aliases (e.g.
  HETE, PONPC, PAPE, PLPC) that must be resolved to identifiers encoding mass shift,
  elemental composition, modification type, position, and stereochemistry.
when_to_use_negative:
- Input is already in unified LipidLynxX ID format with all five annotation levels
  encoded (re-conversion adds nothing)
- The lipids of interest are plant or bacterial lipids outside the 25 mammalian lipid
  classes across the five categories (fatty acids, glycerolipids, glycerophospholipids,
  sphingolipids, sterols) currently covered
- The abbreviation comes from a database/program not covered by the input rule configurations
  in lynx/configurations/rules/input and no suitable configuration file has been generated
edam_operation: http://edamontology.org/operation_3434
tools:
- name: LipidLynxX
  role: Converter module that maps different lipid abbreviations, including modified-lipid
    aliases, to the uniform LipidLynxX ID; its JSON configuration files supply the
    controlled vocabularies and defined aliases
  repo: https://github.com/SysMedOs/LipidLynxX
- name: Black
  role: Code formatter whose style (Black) must be followed when implementing the
    modified-lipid identifier parsing in Python within the LipidLynxX codebase
  repo: https://github.com/psf/black
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2020.04.09.033894
    title: 'LipidLynxX: a data transfer hub to support integration of large scale
      lipidomics datasets'
  source_content_hash: sha256:e5afd8d1f5730d2b63366c219646288d845c65637996d362e643f1d05bf91d3e
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
  - Black
  - LipidLynxX
  edam_operation: http://edamontology.org/operation_3434
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# mass-shift-annotation-parsing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Parses lipid abbreviations for modified/oxidized lipids (epilipidome) into LipidLynxX identifiers built on ordered controlled vocabularies that encode five annotation levels: modification nominal mass shift, elemental composition, modification type, position, and stereochemistry.

## When to use

Use when lipidomics annotations contain modified-lipid aliases (e.g. HETE, PONPC, PAPE, PLPC) that must be resolved to identifiers encoding mass shift, elemental composition, modification type, position, and stereochemistry.

## When NOT to use

- Input is already in unified LipidLynxX ID format with all five annotation levels encoded (re-conversion adds nothing)
- The lipids of interest are plant or bacterial lipids outside the 25 mammalian lipid classes across the five categories (fatty acids, glycerolipids, glycerophospholipids, sphingolipids, sterols) currently covered
- The abbreviation comes from a database/program not covered by the input rule configurations in lynx/configurations/rules/input and no suitable configuration file has been generated

## Inputs

- Lipid abbreviation lists containing modified-lipid aliases (e.g. HETE, PONPC, PAPE, PLPC)
- Shorthand notations using spaces (e.g. PC 16:0_18:2) or brackets (e.g. PC(16:0_18:2))
- Annotations exported from the 26 supported databases/programs (HMDB, LIPID MAPS LMSD & COMP_DB, LipidHome, RefMet, SwissLipids; ALEX123, MS-DIAL, MZmine2, LipidSearch, LipidBlast, etc.)
- LipidLynxX JSON configuration files (controlled vocabularies and defined aliases, e.g. lynx/configurations/defined_alias.json)

## Outputs

- CSV of converted annotations (e.g. modified_lipid_conversions.csv) with all five annotation-level fields populated per lipid
- Unified LipidLynxX IDs encoding nominal mass shift, elemental composition, modification type, position, and stereochemistry
- Optionally .xlsx export of the conversion table

## How to apply

Load the built-in controlled vocabulary and alias configuration shipped as JSON files with LipidLynxX (e.g. lynx/configurations/defined_alias.json) and map input alias terms to their canonical entries. Implement the modified-lipid identifier scheme in Python (Python 3.8.5, following the Black code style used by all LipidLynxX python codes) so that each annotation is encoded across the five levels: modification nominal mass shift, elemental composition, modification type, position, and stereochemistry. Run the implemented converter on a small demonstration set of modified-lipid annotations through the LipidLynxX conversion logic — the Converter module translates annotations from 26 tested input styles (5 databases such as HMDB, LIPID MAPS, RefMet, SwissLipids, LipidHome, plus 17 programs such as MS-DIAL, LipidSearch, MZmine2). Write the converted annotations to a CSV (e.g. modified_lipid_conversions.csv), retaining all five encoded levels per lipid. Rationale: shorthand notations using spaces (PC 16:0_18:2) or brackets (PC(16:0_18:2)) and common aliases are ambiguous across sources, so a strictly controlled, JSON-schema-validated identifier enables cross-dataset comparison.

## Related tools

- **LipidLynxX** (Converter module that maps different lipid abbreviations, including modified-lipid aliases, to the uniform LipidLynxX ID; its JSON configuration files supply the controlled vocabularies and defined aliases) — https://github.com/SysMedOs/LipidLynxX
- **Black** (Code formatter whose style (Black) must be followed when implementing the modified-lipid identifier parsing in Python within the LipidLynxX codebase) — https://github.com/psf/black

## Examples

```
python cli_lynx.py
```

## Evaluation signals

- Output file exists and is a well-formed CSV
- Converted entries are present for the demonstration aliases (e.g. HETE, PONPC) with all five annotation-level fields populated: nominal mass shift, elemental composition, modification type, position, stereochemistry
- Every input alias resolves through the alias configuration rather than remaining unconverted
- Cross-level consistency: equalizer/compare logic can match converted IDs on shared levels (e.g. nominal mass shift) without contradictions

## Limitations

- Plant and bacterial lipids are not currently covered by LipidLynxX; coverage is limited to mammalian lipids from 25 classes in five categories
- If a source database/program style is not covered by the rule configurations, conversion is not possible until a suitable configuration file is generated (often via an issue report)
- API and code definitions may not be updated in documentation as the code changes rapidly; no changelog is available in the repository snapshot and no specific version/container is cited

## Evidence

- [full_text] LipidLynxX introduces new identifiers for modified lipids based on ordered controlled vocabularies that encode five annotation levels: [evidence span withheld — anchor status: not_found]
- [readme] Unified modification controlled vocabularies; strictly controlled format: [evidence span withheld — anchor status: not_found]
- [readme] Abbreviations such as DHA, PAPE, PLPC, PONPC are included as defined alias: [evidence span withheld — anchor status: external_doc]
