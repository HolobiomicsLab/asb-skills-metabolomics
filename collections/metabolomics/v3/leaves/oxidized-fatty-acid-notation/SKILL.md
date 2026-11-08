---
name: oxidized-fatty-acid-notation
description: Use when lipid abbreviation inputs (e.g. oxidized fatty-acid shorthand
  like HETE, PONPC) fail to convert to LipidLynxX IDs and you need to add a new input
  style via a rule-based JSON alias configuration file.
when_to_use_negative:
- 'The target abbreviation is already supported: LipidLynxX already converts 5 databases
  and 17 programs plus defined aliases such as DHA, PAPE, PLPC, PONPC — check lynx/configurations/rules/input
  before authoring new rules.'
- 'The lipid classes are plant or bacterial: the tool currently covers mammalian lipids
  from 25 classes across five categories (fatty acids, glycerolipids, glycerophospholipids,
  sphingolipids, sterols) and other lipid kingdoms are only a possible future extension.'
- The goal is leveling/ cross-matching annotation levels rather than abbreviation
  conversion — use the Equalizer module instead of extending alias rules.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: LipidLynxX
  role: Converter module whose alias (defined_alias.json) configuration is extended
    and then run via API, CLI, GUI, or notebook to validate conversion of new abbreviations
    to LipidLynxX IDs
  repo: https://github.com/SysMedOs/LipidLynxX
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2020.04.09.033894
    title: 'LipidLynxX: a data transfer hub to support integration of large scale
      lipidomics datasets'
  source_content_hash: sha256:a2164adf854b4def6613a390aaa1116468f497d12ab2de2735156eb41756b276
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
  - LipidLynxX
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# oxidized-fatty-acid-notation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extend LipidLynxX's built-in alias configuration (JSON rule files such as lynx/configurations/defined_alias.json) so abbreviations of modified lipids — e.g. oxidized fatty acids like HETE or PONPC — can be converted to unified LipidLynxX IDs without modifying the software source code.

## When to use

Use when lipid abbreviation inputs (e.g. oxidized fatty-acid shorthand like HETE, PONPC) fail to convert to LipidLynxX IDs and you need to add a new input style via a rule-based JSON alias configuration file.

## When NOT to use

- The target abbreviation is already supported: LipidLynxX already converts 5 databases and 17 programs plus defined aliases such as DHA, PAPE, PLPC, PONPC — check lynx/configurations/rules/input before authoring new rules.
- The lipid classes are plant or bacterial: the tool currently covers mammalian lipids from 25 classes across five categories (fatty acids, glycerolipids, glycerophospholipids, sphingolipids, sterols) and other lipid kingdoms are only a possible future extension.
- The goal is leveling/ cross-matching annotation levels rather than abbreviation conversion — use the Equalizer module instead of extending alias rules.

## Inputs

- List of lipid abbreviations for modified lipids not covered by the default alias file (e.g. oxidized fatty acids: HETE, PONPC)
- Default LipidLynxX alias configuration file (lynx/configurations/defined_alias.json, JSON)

## Outputs

- extended_aliases.json — a rule-based JSON configuration mapping new abbreviations to LipidLynxX shorthand annotations
- Conversion table mapping each added abbreviation to its unified LipidLynxX ID (CSV/XLSX or text list)

## How to apply

Load the default LipidLynxX alias configuration file shipped in the SysMedOs/LipidLynxX repository (lynx/configurations/defined_alias.json) and inspect its rule structure for existing defined aliases such as DHA, PAPE, PLPC and PONPC. Compile a test list of oxidized fatty-acid abbreviations absent from the default configuration, then author new rule-based JSON entries mapping each abbreviation to the corresponding LipidLynxX shorthand annotation. Save the result as extended_aliases.json in standard editor-formatted JSON — the format is strictly controlled by JSON schema, and the configuration is customizable without any software update. Run the LipidLynxX converter via its API, command line tools (cli_lynx.py), GUI, or the converter_notebook.ipynb Binder demo on the test list with the extended configuration. Validate that every added abbreviation converts to the expected unified LipidLynxX ID and that no unmatched annotations remain.

## Related tools

- **LipidLynxX** (Converter module whose alias (defined_alias.json) configuration is extended and then run via API, CLI, GUI, or notebook to validate conversion of new abbreviations to LipidLynxX IDs) — https://github.com/SysMedOs/LipidLynxX

## Examples

```
python cli_lynx.py
```

## Evaluation signals

- Every added oxidized fatty-acid abbreviation in the test list converts to the expected unified LipidLynxX ID when run with extended_aliases.json.
- No unmatched (unconverted) annotations remain in the converter output after applying the extended configuration.
- extended_aliases.json is a valid, editor-formatted JSON document consistent with LipidLynxX's strictly controlled JSON-schema format.
- The conversion works without modifying LipidLynxX source code or reinstalling/updating the software.
- Behavior is identical whether run via the API, command line, GUI, or the converter_notebook.ipynb demo.

## Limitations

- LipidLynxX currently covers mammalian lipids from 25 classes representing five categories; plant and bacterial lipids are not covered, so new alias rules for those domains may not resolve to IDs.
- If a database or program is not in the supported list, its rule file in lynx/configurations/rules/input must be tested first; conversion may require contacting the developers to generate a suitable configuration file.
- API definitions and source-code documentation may lag the rapidly changing code; contact the developers or follow the repository before relying on API access for alias conversion.

## Evidence

- [readme] Common abbreviations (customizable) with defined_alias.json location: [evidence span withheld — anchor status: external_doc]
- [readme] Strictly controlled JSON format for configurations: [evidence span withheld — anchor status: external_doc]
- [readme] Converter module converts different abbreviations to uniformed LipidLynxX ID: [evidence span withheld — anchor status: external_doc]
