---
name: lipid-identifier-converter-inventory
description: Use when building a converter or mapping table for lipid annotation identifiers
  and you need the definitive list of source software/database notation styles and
  their JSON rule files.
when_to_use_negative:
- Your database or program is already covered — the README instructs testing whether
  an existing configuration file in lynx/configurations/rules/input fits rather than
  rebuilding the inventory.
- The task is converting lipid annotations themselves (LipidLynxX Converter/Equalizer
  runtime conversion) rather than curating the inventory of supported styles.
- Annotations are non-mammalian lipids — LipidLynxX covers mammalian lipids from 25
  classes across five categories (fatty acids, glycerolipids, glycerophospholipids,
  sphingolipids, sterols), not plant or bacterial lipids.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_0091
tools:
- name: LipidLynxX
  role: Source of the JSON input-style configuration files being inventoried; the
    hub that converts annotations from the 26 supported styles into unified identifiers
  repo: https://github.com/SysMedOs/LipidLynxX
- name: RefMet
  role: Lipid annotation/ID converter whose notation style is one of the predefined
    input styles in the inventory
- name: Goslin
  role: Lipid annotation/ID converter whose notation style is one of the predefined
    input styles in the inventory
- name: LipidID mapper
  role: SwissLipids-based ID converter whose notation style is one of the predefined
    input styles in the inventory
- name: Lipid Mini-ON
  role: Lipid ontology converter whose notation style is one of the predefined input
    styles in the inventory
- name: Lion
  role: Tool mapping lipidomics data to structural ontology whose annotation style
    is represented among the input styles
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2020.04.09.033894
    title: 'LipidLynxX: a data transfer hub to support integration of large scale
      lipidomics datasets'
  source_content_hash: sha256:b3f8152f7820c7feaa5853a968d9cbadec647ac845ce78f8e95ad0ae2b681f61
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# lipid-identifier-converter-inventory

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compiles the complete inventory of lipid annotation input styles supported by LipidLynxX — 17 lipid identification programs plus 5 lipid-oriented databases (26 styles total) implemented as JSON configuration files — for curating cross-tool lipid identifier conversion tables.

## When to use

Use when building a converter or mapping table for lipid annotation identifiers and you need the definitive list of source software/database notation styles and their JSON rule files.

## When NOT to use

- Your database or program is already covered — the README instructs testing whether an existing configuration file in lynx/configurations/rules/input fits rather than rebuilding the inventory.
- The task is converting lipid annotations themselves (LipidLynxX Converter/Equalizer runtime conversion) rather than curating the inventory of supported styles.
- Annotations are non-mammalian lipids — LipidLynxX covers mammalian lipids from 25 classes across five categories (fatty acids, glycerolipids, glycerophospholipids, sphingolipids, sterols), not plant or bacterial lipids.

## Inputs

- JSON configuration files from lynx/configurations/rules/input in the SysMedOs/LipidLynxX repository
- Supplementary File 1 reference list of identification tools and lipid-oriented databases
- LipidLynxX README 'Supported lipid notation styles' section

## Outputs

- CSV table with columns: style name, source tool/database, tool category (identification software vs. lipid database)
- Validated inventory of 17 lipid identification programs and 5 lipid-oriented databases (26 input styles)

## How to apply

Load the input-style configuration files from `lynx/configurations/rules/input` in the SysMedOs/LipidLynxX repository and parse each JSON rule set to extract the style name and its source tool or database. Partition entries into two categories: lipid identification programs (17: ALEX123 lipid calculator, Greazy, LDA 2, LipidBlast, LipidCreator, LipiDex, LipidFrag, LipidHunter, LipidMatch, LipidPro, LipidSearch, Lipostar, LIQUID, LPPtiger, MetFrag, MS-DIAL, MZmine2) and lipid-oriented databases (5: HMDB, LIPID MAPS LMSD & COMP_DB, LipidHome, RefMet, SwissLipids), yielding 26 input styles. Emit a CSV with columns for style name, source tool/database, and category. Cross-check the compiled list against Supplementary File 1 and against the README's 'Supported lipid notation styles' section; the validation criterion is that every predefined style appears exactly once and source assignments match the reference list.

## Related tools

- **LipidLynxX** (Source of the JSON input-style configuration files being inventoried; the hub that converts annotations from the 26 supported styles into unified identifiers) — https://github.com/SysMedOs/LipidLynxX
- **RefMet** (Lipid annotation/ID converter whose notation style is one of the predefined input styles in the inventory)
- **Goslin** (Lipid annotation/ID converter whose notation style is one of the predefined input styles in the inventory)
- **LipidID mapper** (SwissLipids-based ID converter whose notation style is one of the predefined input styles in the inventory)
- **Lipid Mini-ON** (Lipid ontology converter whose notation style is one of the predefined input styles in the inventory)
- **Lion** (Tool mapping lipidomics data to structural ontology whose annotation style is represented among the input styles)

## Examples

```
python -c "import json,glob,csv; rows=[(f.split('/')[-1],json.load(open(f))['name']) for f in glob.glob('lynx/configurations/rules/input/*.json')]; w=csv.writer(open('input_styles.csv','w')); w.writerow(['style','source']); w.writerows(rows)"
```

## Evaluation signals

- Every predefined input style present in the JSON configuration files appears exactly once in the output CSV (no duplicates, no omissions).
- The output contains exactly 17 program-derived styles and 5 database-derived styles, totalling 26 input styles.
- Source tool/database assignments for each style agree with the Supplementary File 1 reference list.
- The database list matches the README's 'Databases (5)' set: HMDB, LIPID MAPS LMSD & COMP_DB, LipidHome, RefMet, SwissLipids.
- The program list matches the README's 'Programs (17)' set (ALEX123 lipid calculator through MZmine2).

## Limitations

- LipidLynxX currently covers mammalian lipids from 25 classes in five categories (fatty acids, glycerolipids, glycerophospholipids, sphingolipids, sterols); plant and bacterial lipids are not covered.
- If a database or program is not included in the supported list, conversion may not be possible until a community-contributed configuration file is generated.
- Converter coverage of input/output styles of existing lipid annotation tools remains limited per the article's stated research gap.

## Evidence

- [readme] If your database / program is not included in the list above, you can test if any of the configuration files located in lynx/configurations/rules/input would fit: [evidence span withheld — anchor status: external_doc]
