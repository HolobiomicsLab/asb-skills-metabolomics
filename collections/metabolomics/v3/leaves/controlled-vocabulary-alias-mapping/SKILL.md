---
name: controlled-vocabulary-alias-mapping
description: Use when lipid annotations arrive as heterogeneous shorthand or alias
  terms (e.g. PC(16:0_18:2), PONPC, HETE) from different databases/programs and must
  be normalized to canonical, computer-processable identifiers.
when_to_use_negative:
- When the input is already in canonical LipidLynxX ID format or another already-unified
  identifier format — conversion adds no information.
- When the annotation comes from a database/program style not covered by the 5 databases,
  17 programs, or rules in lynx/configurations/rules/input — contact maintainers for
  a suitable configuration file instead of forcing a mapping.
- When the target vocabulary must cover plant or bacterial lipids — mammalian lipids
  from 25 classes across five categories are the covered scope.
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0089
tools:
- name: LipidLynxX
  role: Converter module that turns heterogeneous lipid abbreviations into uniform
    LipidLynxX IDs using controlled vocabularies and alias JSON configurations
  repo: https://github.com/SysMedOs/LipidLynxX
- name: Black
  role: Code style formatter that all LipidLynxX python code, including alias/converter
    implementations, must follow
  repo: https://github.com/psf/black
- name: RefMet
  role: One of the lipid annotation/ID source styles supported as converter input
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2020.04.09.033894
    title: 'LipidLynxX: a data transfer hub to support integration of large scale
      lipidomics datasets'
  source_content_hash: sha256:2e28155156393b5e30aa51113cf86918b1c1f0c71de61c1dddcb908da1415d6f
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
  - RefMet
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# controlled-vocabulary-alias-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps free-form lipid abbreviations and alias terms (e.g. DHA, PAPE, PLPC, PONPC) to canonical LipidLynxX identifiers using the ordered controlled vocabularies and JSON configuration files shipped with LipidLynxX, encoding five annotation levels for modified lipids.

## When to use

Use when lipid annotations arrive as heterogeneous shorthand or alias terms (e.g. PC(16:0_18:2), PONPC, HETE) from different databases/programs and must be normalized to canonical, computer-processable identifiers.

## When NOT to use

- When the input is already in canonical LipidLynxX ID format or another already-unified identifier format — conversion adds no information.
- When the annotation comes from a database/program style not covered by the 5 databases, 17 programs, or rules in lynx/configurations/rules/input — contact maintainers for a suitable configuration file instead of forcing a mapping.
- When the target vocabulary must cover plant or bacterial lipids — mammalian lipids from 25 classes across five categories are the covered scope.

## Inputs

- Lipid abbreviation lists from supported databases (HMDB, LIPID MAPS LMSD & COMP_DB, LipidHome, RefMet, SwissLipids)
- Lipid shorthand notations with spaces or brackets, e.g. PC 16:0_18:2 or PC(16:0_18:2)
- Common alias abbreviations (e.g. DHA, PAPE, PLPC, PONPC, HETE) resolvable via lynx/configurations/defined_alias.json
- Controlled vocabulary and alias JSON configuration files from the LipidLynxX package

## Outputs

- Canonical LipidLynxX IDs per input annotation
- Converted annotation table (e.g. modified_lipid_conversions.csv) with all five annotation-level fields populated (nominal mass shift, elemental composition, modification type, position, stereochemistry)
- Optionally .csv or .xlsx export tables as produced by the converter notebook/GUI

## How to apply

Load the built-in controlled vocabulary and alias configuration shipped as JSON files with the LipidLynxX package, notably `lynx/configurations/defined_alias.json` for common abbreviations such as DHA, PAPE, PLPC, and PONPC. Map each input alias term to its canonical entry via the alias configuration, then emit the LipidLynxX ID, which for modified lipids encodes five ordered annotation levels: nominal mass shift, elemental composition, modification type, position, and stereochemistry. Implement any new converter code in Python following the Black code style used by all LipidLynxX python codes. Run the converter on the demonstration set through the LipidLynxX conversion logic (CLI, GUI, API, or as a Python module per `examples_notebook.ipynb`) and write results retaining all five encoded levels per lipid.

## Related tools

- **LipidLynxX** (Converter module that turns heterogeneous lipid abbreviations into uniform LipidLynxX IDs using controlled vocabularies and alias JSON configurations) — https://github.com/SysMedOs/LipidLynxX
- **Black** (Code style formatter that all LipidLynxX python code, including alias/converter implementations, must follow) — https://github.com/psf/black
- **RefMet** (One of the lipid annotation/ID source styles supported as converter input)

## Examples

```
python cli_lynx.py
```

## Evaluation signals

- Output CSV is well-formed and contains a converted entry for every demonstration alias (e.g. HETE, PONPC).
- All five annotation-level fields (nominal mass shift, elemental composition, modification type, position, stereochemistry) are populated for modified-lipid identifiers.
- Each alias resolves to exactly one canonical entry in defined_alias.json — no unmatched or ambiguous outputs.
- Converted IDs pass the strictly controlled JSON schema format used by LipidLynxX.
- New converter code passes Black formatting checks consistent with the LipidLynxX codebase style.

## Limitations

- If a database or program style is not covered by existing input rule configurations, conversion is not possible until a suitable configuration file is generated, potentially requiring maintainer collaboration.
- LipidLynxX currently covers mammalian lipids from 25 classes representing five categories (fatty acids, glycerolipids, glycerophospholipids, sphingolipids, sterols); plant and bacterial lipids are not yet covered.
- The Windows .exe version may fail to convert some IDs that are already fixed in the latest source; the mybinder converter demo tracks the master branch and can be used to check.
- API definitions and documentation in the source code may lag the rapidly changing code; API users are advised to contact developers or follow the repository for changes.

## Evidence

- [readme] Common abbreviations (customizable): Abbreviations such as DHA, PAPE, PLPC, PONPC .etc are also included as `defined alias`. detailed settings can be found in `lynx/configurations/defined_alias.json`: [evidence span withheld — anchor status: external_doc]
- [readme] Shorthand notations supported: [evidence span withheld — anchor status: external_doc]
- [abstract] LipidLynxX introduces new identifiers for modified lipids based on ordered controlled vocabularies encoding five annotation levels: [evidence span withheld — anchor status: not_found]
- [readme] Supported database/program styles: [evidence span withheld — anchor status: external_doc]
- [readme] Converter module purpose: [evidence span withheld — anchor status: external_doc]
- [methods] Black code style requirement: "LipidLynxX source code use [code style Black]"
