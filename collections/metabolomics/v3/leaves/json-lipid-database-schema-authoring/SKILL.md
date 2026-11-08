---
name: json-lipid-database-schema-authoring
description: Use when a lipidomics input abbreviation style or lipid class (e.g. plant
  or bacterial lipids such as MGDG/DGDG) is unsupported by the shipped converter configurations
  and a new JSON rule file is needed.
when_to_use_negative:
- 'The input notation style is already among the 26 supported styles (5 databases:
  HMDB, LIPID MAPS LMSD & COMP_DB, LipidHome, RefMet, SwissLipids; 17 programs: MS-DIAL,
  LipidSearch, ALEX123, etc.) — test the shipped configurations first'
- The goal is linking lipid annotations to external resources/identifiers (use the
  LipidLynxX Linker module) rather than converting abbreviations
- The change requires modifying converter logic itself rather than adding configuration;
  the article scopes extension to config files without code changes
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: LipidLynxX
  role: Converter whose JSON configuration rules are extended to cover new lipid classes
    and notation styles
  repo: https://github.com/SysMedOs/LipidLynxX
- name: Black
  role: Code style formatter that any accompanying Python changes must follow
  repo: https://github.com/psf/black
- name: Visual Studio Code / PyCharm
  role: Editors used to apply the project's JSON formatting convention to the new
    configuration files
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2020.04.09.033894
    title: 'LipidLynxX: a data transfer hub to support integration of large scale
      lipidomics datasets'
  source_content_hash: sha256:d2cce86305dc843ddbebafa5ebdc1a0e505b163b02c016cac70b4210e28418b0
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
  - PyCharm
  - Visual Studio Code
  - Visual Studio Code / PyCharm
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# JSON lipid database schema authoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Author new JSON configuration files for the LipidLynxX Converter so that lipid annotation styles and lipid classes outside the current mammalian coverage (25 classes, five categories) can be converted without modifying the source code.

## When to use

Use when a lipidomics input abbreviation style or lipid class (e.g. plant or bacterial lipids such as MGDG/DGDG) is unsupported by the shipped converter configurations and a new JSON rule file is needed.

## When NOT to use

- The input notation style is already among the 26 supported styles (5 databases: HMDB, LIPID MAPS LMSD & COMP_DB, LipidHome, RefMet, SwissLipids; 17 programs: MS-DIAL, LipidSearch, ALEX123, etc.) — test the shipped configurations first
- The goal is linking lipid annotations to external resources/identifiers (use the LipidLynxX Linker module) rather than converting abbreviations
- The change requires modifying converter logic itself rather than adding configuration; the article scopes extension to config files without code changes

## Inputs

- Lipid annotation abbreviation strings from unsupported databases/programs (e.g. plant or bacterial lipid classes)
- Existing JSON rule files in lynx/configurations/rules/input
- lynx/configurations/defined_alias.json (customizable aliases such as DHA, PAPE, PLPC, PONPC)

## Outputs

- New JSON lipid class/notation configuration files conforming to the Lynx JSON schema
- Converted lipid annotation table (LipidLynxX IDs, exportable as .csv or .xlsx)
- Feature request issue number referenced in the pull request description

## How to apply

Inspect the existing JSON configuration templates under `lynx/configurations/rules/input` (and the `defined_alias.json` aliases) to match the strictly controlled JSON schema used by the project. Author new entries that map the target database/program notation style to the unified LipidLynxX ID, adding non-mammalian lipid classes beyond fatty acids, glycerolipids, glycerophospholipids, sphingolipids, and sterols. Format the new JSON with the editor formatting convention (Visual Studio Code / PyCharm) and keep any Python changes Black-formatted, since LipidLynxX source code uses the Black code style. Run the Converter on a small test set of plant/bacterial annotation strings sourced from the unsupported software styles and check them against `doc/sample_data/output`. If conversion of a style is impossible, the project convention is to submit a feature request issue via the GitHub issue tracker so a suitable configuration file can be generated collaboratively.

## Related tools

- **LipidLynxX** (Converter whose JSON configuration rules are extended to cover new lipid classes and notation styles) — https://github.com/SysMedOs/LipidLynxX
- **Black** (Code style formatter that any accompanying Python changes must follow) — https://github.com/psf/black
- **Visual Studio Code / PyCharm** (Editors used to apply the project's JSON formatting convention to the new configuration files)

## Examples

```
python cli_lynx.py  # then paste plant/bacterial abbreviation test set into the Converter to verify the new JSON rules
```

## Evaluation signals

- New JSON files pass the strictly controlled JSON schema validation used by LipidLynxX and match the structure of existing files in lynx/configurations/rules/input
- Plant/bacterial test annotation strings convert to unified LipidLynxX IDs via the Converter with no unmapped entries
- Converted output matches the format of doc/sample_data/output test outputs
- Pull request adds substantial functionality (new classes, not cosmetic-only) and cites the corresponding feature request issue number
- Changes are Black-formatted and no source-code edits beyond configuration were required

## Limitations

- Current coverage is mammalian: 25 classes across five categories (fatty acids, glycerolipids, glycerophospholipids, sphingolipids, sterols); plant and bacterial classes are future extensions
- If no existing configuration file fits the target database/program, conversion may be impossible without help from maintainers to generate a suitable configuration file
- A robust converter depends on community-wide collaboration; user issue reports are needed to improve coverage
- No changelog is available in the repository snapshot, and no specific version/build is cited for this extension workflow (latest client release noted as v0.9.24)

## Evidence

- [discussion] Plant/bacterial extension scope: "mammalian lipids from 25 classes representing five categories – fatty acids, glycerolipids, glycerophospholipids, sphingolipids, and sterols, but can be extended in the future for plant and bacterial"
- [readme] Configuration file location for new styles: [evidence span withheld — anchor status: external_doc]
- [methods] Feature request process for new configurations: "Suggest your proposals by submitting a feature request issue in the [issue tracker](https://github.com/SysMedOs/LipidLynxX/issues)"
- [methods] Black code style requirement: "LipidLynxX source code use [code style Black](https://github.com/psf/black)"
- [readme] Customizable alias configuration: [evidence span withheld — anchor status: external_doc]
