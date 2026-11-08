---
name: lipidomics-database-coverage-mapping
description: Use when auditing which lipid annotation formats / source tools a converter
  supports, or when compiling a reference table of lipid identification software vs.
  lipid database notation styles.
when_to_use_negative:
- The goal is converting lipid annotations themselves rather than inventorying which
  annotation styles are supported — use the converter's Converter/Equalizer modules
  directly instead
- You need to add support for a database/program not in the list and conversion is
  not possible with existing configuration files — contact the developers for a new
  configuration file rather than force-mapping
- The lipid domain is outside LipidLynxX's covered scope (plant or bacterial lipids,
  which are not currently covered)
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: LipidLynxX
  role: Converter hub whose JSON configuration files define the 26 input styles being
    enumerated; source of the style inventory
  repo: https://github.com/SysMedOs/LipidLynxX
- name: RefMet
  role: Lipid-oriented ID converter whose annotation format is one of the pre-defined
    input styles
- name: Goslin
  role: Lipid annotation/ID converter whose input style is among the supported sources
- name: LipidID mapper
  role: ID converter within SwissLipids whose annotation style is one of the mapped
    inputs
- name: Lipid Mini-ON
  role: Lipid ontology tool whose annotation style is among the supported sources
- name: Lion
  role: Lipid structural ontology mapping tool included in the converter-style landscape
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2020.04.09.033894
    title: 'LipidLynxX: a data transfer hub to support integration of large scale
      lipidomics datasets'
  source_content_hash: sha256:1722b0484711b7d33ea8c36e172f8f2f9d135b9e7ab065fdf8c8d09e99902f0d
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
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# lipidomics-database-coverage-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compiles the full set of lipid annotation input styles supported by a converter by enumerating the identification programs and lipid-oriented databases whose shorthand notations it maps, as defined in JSON configuration files (here: 17 programs + 5 databases = 26 styles in LipidLynxX).

## When to use

Use when auditing which lipid annotation formats / source tools a converter supports, or when compiling a reference table of lipid identification software vs. lipid database notation styles.

## When NOT to use

- The goal is converting lipid annotations themselves rather than inventorying which annotation styles are supported — use the converter's Converter/Equalizer modules directly instead
- You need to add support for a database/program not in the list and conversion is not possible with existing configuration files — contact the developers for a new configuration file rather than force-mapping
- The lipid domain is outside LipidLynxX's covered scope (plant or bacterial lipids, which are not currently covered)

## Inputs

- JSON input-style configuration files from lynx/configurations/rules/input in the SysMedOs/LipidLynxX repository
- defined_alias.json custom-abbreviation settings
- Supplementary File 1 reference list of identification tools and lipid-oriented databases

## Outputs

- CSV table with columns: style name, source tool/database, tool category (identification software vs. lipid database)
- Enumerated inventory of 26 input styles (17 programs + 5 databases)

## How to apply

Load the JSON configuration files that define predefined input styles from the converter's repository (for LipidLynxX these live under lynx/configurations/rules/input, with alias definitions in lynx/configurations/defined_alias.json). Parse each style definition and record the style name plus the identification program or lipid-oriented database it represents. Partition entries into two categories — lipid identification software (e.g. MS-DIAL, MZmine2, LipidSearch, LipidBlast, LipiDex) versus lipid-oriented databases (e.g. HMDB, LIPID MAPS LMSD & COMP_DB, LipidHome, RefMet, SwissLipids) — and compile into a table with columns for style name, source tool/database, and category. Cross-check the count against the publication's stated coverage (17 programs and 5 databases, i.e. 26 sources translating into three pre-defined output formats) and against Supplementary File 1. Validate by confirming every predefined input style appears exactly once with source assignments matching the reference list.

## Related tools

- **LipidLynxX** (Converter hub whose JSON configuration files define the 26 input styles being enumerated; source of the style inventory) — https://github.com/SysMedOs/LipidLynxX
- **RefMet** (Lipid-oriented ID converter whose annotation format is one of the pre-defined input styles)
- **Goslin** (Lipid annotation/ID converter whose input style is among the supported sources)
- **LipidID mapper** (ID converter within SwissLipids whose annotation style is one of the mapped inputs)
- **Lipid Mini-ON** (Lipid ontology tool whose annotation style is among the supported sources)
- **Lion** (Lipid structural ontology mapping tool included in the converter-style landscape)

## Examples

```
parse lynx/configurations/rules/input/*.json; compile styles.csv with columns style_name,source_tool,category  # 17 programs + 5 databases = 26 input styles
```

## Evaluation signals

- Every predefined input style present in the JSON configuration files appears exactly once in the compiled table
- Count of program-derived styles equals 17 and count of database-derived styles equals 5 (26 total)
- Source tool/database assignments agree with the reference list in Supplementary File 1
- Database category contains exactly HMDB, LIPID MAPS LMSD & COMP_DB, LipidHome, RefMet, and SwissLipids
- Each style definition parsed from lynx/configurations/rules/input maps to a named tool or database with a valid category assignment

## Limitations

- LipidLynxX coverage is limited to mammalian lipids from 25 classes in five categories (fatty acids, glycerolipids, glycerophospholipids, sphingolipids, sterols); plant and bacterial lipids are not currently covered
- Databases/programs not in the list require testing existing configuration files or requesting a new one; a robust converter depends on community-wide collaboration and issue reports
- If a target database/program's notation does not fit any configuration file in lynx/configurations/rules/input, conversion is not possible out of the box

## Evidence

- [readme] The current LipidLynxX source code was tested using our collection of lipid abbreviations for major lipid classes from following databases and programs: [evidence span withheld — anchor status: external_doc]
