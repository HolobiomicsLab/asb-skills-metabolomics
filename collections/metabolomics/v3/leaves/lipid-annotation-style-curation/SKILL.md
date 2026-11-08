---
name: lipid-annotation-style-curation
description: Use when a task requires enumerating which lipid identification software
  or lipid databases have predefined annotation input styles, or when building/validating
  a converter's style catalog from JSON rule configurations.
when_to_use_negative:
- The goal is to convert lipid abbreviations between styles (use the LipidLynxX Converter/Equalizer
  modules directly instead of re-curating the style list)
- The target database/program has no configuration file in lynx/configurations/rules/input
  and is not covered by the 26 pre-defined styles
- You need lipid annotation coverage for plant or bacterial lipids, which LipidLynxX
  does not currently cover
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: LipidLynxX
  role: Source of the JSON configuration files that define the 26 pre-defined lipid
    annotation input styles being curated
  repo: https://github.com/SysMedOs/LipidLynxX
- name: RefMet
  role: ID converter whose annotation format is one of the curated input styles
- name: Goslin
  role: Annotation converter whose format is one of the curated input styles
- name: Lipid Mini-ON
  role: Ontology-mapping tool whose format is one of the curated input styles
- name: LION
  role: Lipid ontology tool whose format is one of the curated input styles
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2020.04.09.033894
    title: 'LipidLynxX: a data transfer hub to support integration of large scale
      lipidomics datasets'
  source_content_hash: sha256:82404f75e77a5a339a42219ccc38cb304a98bc5cc93d09af01694e1e8e63e054
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
  - LION
  - Lion
  - Lipid Mini-ON
  - LipidID mapper
  - LipidLynxX
  - RefMet
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# lipid-annotation-style-curation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compiles the full set of predefined lipid annotation input styles (17 lipid identification programs plus 5 lipid-oriented databases = 26 styles) from the LipidLynxX JSON configuration files, producing a cross-checked CSV catalog of style-to-source mappings.

## When to use

Use when a task requires enumerating which lipid identification software or lipid databases have predefined annotation input styles, or when building/validating a converter's style catalog from JSON rule configurations.

## When NOT to use

- The goal is to convert lipid abbreviations between styles (use the LipidLynxX Converter/Equalizer modules directly instead of re-curating the style list)
- The target database/program has no configuration file in lynx/configurations/rules/input and is not covered by the 26 pre-defined styles
- You need lipid annotation coverage for plant or bacterial lipids, which LipidLynxX does not currently cover

## Inputs

- JSON configuration files from lynx/configurations/rules/input in the SysMedOs/LipidLynxX repository
- Supplementary File 1 reference list of identification tools and lipid-oriented databases
- defined alias settings from lynx/configurations/defined_alias.json (optional, for common-abbreviation styles)

## Outputs

- CSV table with columns: style name, source tool/database, tool category (identification software vs. lipid database)
- Counted totals per category (expected: 17 programs, 5 databases, 26 styles overall)

## How to apply

Load the JSON configuration files from the `lynx/configurations/rules/input` directory of the SysMedOs/LipidLynxX repository; each file encodes one predefined input style (e.g. LIPID MAPS LMSD & COMP_DB, RefMet, Goslin, Lipid Mini-ON, LION, SwissLipids, HMDB, LipidBank formats). Parse each JSON structure to extract the style name and the identification program or lipid-oriented database whose annotation format it represents, then bin each source into one of two categories: identification software (17 programs such as ALEX123 lipid calculator, LipidBlast, LipidSearch, MS-DIAL, MZmine2) versus lipid databases (5: HMDB, LIPID MAPS LMSD & COMP_DB, LipidHome, RefMet, SwissLipids). Emit a single CSV table with columns style_name, source_tool_or_database, and category. Cross-check the compiled list against the Supplementary File 1 reference list so the total of 26 styles matches the Converter/Equalizer coverage described in the paper.

## Related tools

- **LipidLynxX** (Source of the JSON configuration files that define the 26 pre-defined lipid annotation input styles being curated) — https://github.com/SysMedOs/LipidLynxX
- **RefMet** (ID converter whose annotation format is one of the curated input styles)
- **Goslin** (Annotation converter whose format is one of the curated input styles)
- **Lipid Mini-ON** (Ontology-mapping tool whose format is one of the curated input styles)
- **LION** (Lipid ontology tool whose format is one of the curated input styles)

## Examples

```
python -c "import json,glob; styles=[json.load(open(f)) for f in glob.glob('lynx/configurations/rules/input/*.json')]; print(len(styles))"
```

## Evaluation signals

- Every style found in the JSON configuration files appears exactly once in the output CSV (no duplicates, no omissions)
- The category totals match the published counts: 17 identification programs and 5 lipid-oriented databases, summing to 26 input styles
- Every source tool/database assignment agrees with the Supplementary File 1 reference list
- Database category contains exactly HMDB, LIPID MAPS LMSD & COMP_DB, LipidHome, RefMet, and SwissLipids
- Each extracted style definition corresponds to an existing configuration file path under lynx/configurations/rules/input

## Limitations

- LipidLynxX coverage is limited to mammalian lipids from 25 classes across five categories (fatty acids, glycerolipids, glycerophospholipids, sphingolipids, sterols); plant and bacterial lipids are not covered
- A database/program not included in the configuration files cannot be curated as a style unless a suitable configuration file is generated with the developers
- The curated list reflects the repository snapshot; no changelog was available, and definitions may change as the rapidly evolving source code is updated

## Evidence

- [readme] Configuration files for unsupported databases/programs: [evidence span withheld — anchor status: external_doc]
