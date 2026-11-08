---
name: json-configuration-parsing
description: Use when you need to enumerate, audit, or extend the lipid abbreviation
  input styles supported by a converter such as LipidLynxX, starting from its JSON
  rule/configuration files rather than from prose documentation.
when_to_use_negative:
- When the goal is to convert actual lipid abbreviation lists between annotation styles
  — use LipidLynxX's Converter/Equalizer modules directly instead of parsing its configuration
  files.
- When the target database or program is not represented in lynx/configurations/rules/input
  and conversion is required — contact the developers for a suitable configuration
  file rather than hand-crafting a style mapping.
- When surveying general (non-lipid) identifier converters; this skill is grounded
  in the LipidLynxX JSON rule format and its 26 lipid annotation styles.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: LipidLynxX
  role: Source of the JSON configuration files (lynx/configurations/rules/input, defined_alias.json)
    whose input-style definitions are parsed and inventoried
  repo: https://github.com/SysMedOs/LipidLynxX
- name: RefMet
  role: Lipid-oriented database whose annotation format is one of the 5 database input
    styles captured in the JSON configuration files
- name: Goslin
  role: Lipid annotation/ID converter cited as one of the identification tools whose
    input style is represented in the JSON configurations
- name: LipidID mapper
  role: ID converter within SwissLipids cited among the annotation converters whose
    styles appear in the configuration inventory
- name: Lipid Mini-ON
  role: Lipid ontology tool cited among the converters whose annotation styles are
    covered by the JSON rule files
- name: Lion
  role: Lipid ontology mapping tool (LION) cited among the annotation/mapping tools
    referenced when defining input styles
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2020.04.09.033894
    title: 'LipidLynxX: a data transfer hub to support integration of large scale
      lipidomics datasets'
  source_content_hash: sha256:a888a278732151ec68631cd4bc8fa245844715a48670975f7aaa76e58e4b62eb
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
  - Goslin
  - Lion
  - Lipid Mini-ON
  - LipidID mapper
  - LipidLynxX
  - RefMet
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# json-configuration-parsing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Parse the JSON configuration/rule files shipped with LipidLynxX to enumerate the pre-defined lipid annotation input styles (17 identification programs plus 5 lipid databases) and map each style to its source tool or database.

## When to use

Use when you need to enumerate, audit, or extend the lipid abbreviation input styles supported by a converter such as LipidLynxX, starting from its JSON rule/configuration files rather than from prose documentation.

## When NOT to use

- When the goal is to convert actual lipid abbreviation lists between annotation styles — use LipidLynxX's Converter/Equalizer modules directly instead of parsing its configuration files.
- When the target database or program is not represented in lynx/configurations/rules/input and conversion is required — contact the developers for a suitable configuration file rather than hand-crafting a style mapping.
- When surveying general (non-lipid) identifier converters; this skill is grounded in the LipidLynxX JSON rule format and its 26 lipid annotation styles.

## Inputs

- JSON configuration files from the SysMedOs/LipidLynxX GitHub repository (lynx/configurations/rules/input, lynx/configurations/defined_alias.json)
- Supplementary File 1 reference list of identification tools and lipid-oriented databases from the LipidLynxX preprint

## Outputs

- CSV table with columns style name, source tool/database, and tool category (identification software vs. lipid database)
- Counted inventory of 26 pre-defined input styles (17 programs + 5 databases) plus shorthand and alias styles

## How to apply

Load the JSON configuration files under `lynx/configurations/` in the SysMedOs/LipidLynxX repository, especially the input rules in `lynx/configurations/rules/input` and aliases in `lynx/configurations/defined_alias.json`; LipidLynxX enforces a strictly controlled JSON schema, so validate structure against that schema before extraction. Extract every pre-defined input style together with the lipid identification tool or lipid-oriented database whose annotation format it represents (e.g. MS-DIAL, LipidSearch, MZmine2 as programs; HMDB, LIPID MAPS LMSD & COMP_DB, LipidHome, RefMet, SwissLipids as databases). Compile the extracted styles into a single table with columns for style name, source tool/database, and tool category (identification software vs. lipid database). Cross-check the compiled list against the paper's Supplementary File 1 reference list to confirm that all 17 programs and 5 databases are accounted for exactly once. If a target database/program has no configuration file, note it as a gap for community-supplied rule generation rather than inventing a style mapping.

## Related tools

- **LipidLynxX** (Source of the JSON configuration files (lynx/configurations/rules/input, defined_alias.json) whose input-style definitions are parsed and inventoried) — https://github.com/SysMedOs/LipidLynxX
- **RefMet** (Lipid-oriented database whose annotation format is one of the 5 database input styles captured in the JSON configuration files)
- **Goslin** (Lipid annotation/ID converter cited as one of the identification tools whose input style is represented in the JSON configurations)
- **LipidID mapper** (ID converter within SwissLipids cited among the annotation converters whose styles appear in the configuration inventory)
- **Lipid Mini-ON** (Lipid ontology tool cited among the converters whose annotation styles are covered by the JSON rule files)
- **Lion** (Lipid ontology mapping tool (LION) cited among the annotation/mapping tools referenced when defining input styles)

## Examples

```
python -c "import json,glob; styles=[json.load(open(f)) for f in glob.glob('lynx/configurations/rules/input/*.json')]; print(len(styles))"
```

## Evaluation signals

- Every pre-defined input style found in the configuration files appears exactly once in the compiled table (no duplicates, no omissions).
- Source tool/database assignments in the table agree with the Supplementary File 1 reference list.
- Counts reconcile: 17 lipid identification programs and 5 lipid-oriented databases (HMDB, LIPID MAPS LMSD & COMP_DB, LipidHome, RefMet, SwissLipids), totalling 26 input styles.
- Each extracted style's JSON structure conforms to LipidLynxX's strictly controlled JSON schema.
- Custom alias entries (e.g. DHA, PAPE, PLPC, PONPC) trace back to definitions in lynx/configurations/defined_alias.json.

## Limitations

- The configuration snapshot is tied to a specific LipidLynxX release (README references v0.9.24); the code is changing rapidly, so style definitions may be updated without notice.
- If a database/program is not included in the configuration list, conversion is not possible until a suitable configuration file is generated with developer help.
- The curated style list covers mainly mammalian lipids from 25 classes in five categories; plant and bacterial lipids are not currently covered.

## Evidence

- [readme] Databases (5):
  - `HMDB`, `LIPID MAPS LMSD & COMP_DB`, `LipidHome`, `RefMet`, `SwissLipids`

- Programs (17): "Databases (5):
  - `HMDB`, `LIPID MAPS LMSD & COMP_DB`, `LipidHome`, `RefMet`, `SwissLipids`

- Programs (17)"
- [readme] you can test if any of the configuration files located in `lynx/configurations/rules/input` would fit to your database / program.: [evidence span withheld — anchor status: external_doc]
- [readme] Strictly controlled format using JSON schema: [evidence span withheld — anchor status: external_doc]
- [readme] detailed settings can be found in `lynx/configurations/defined_alias.json`: [evidence span withheld — anchor status: external_doc]
- [discussion] Converter and Equalizer modules allow easy translation and cross-matching of lipid annotations into three pre-defined formats: "Converter and Equalizer modules allow easy translation and cross-matching of lipid annotations into three pre-defined formats"
