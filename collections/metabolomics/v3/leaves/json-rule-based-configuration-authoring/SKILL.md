---
name: json-rule-based-configuration-authoring
description: Use when a lipidomics identifier converter fails to recognize abbreviations
  from a database or program (e.g. oxidized lipid aliases absent from the default
  `lynx/configurations/defined_alias.json`), and the goal is to add new input styles
  via configuration rather than code changes.
when_to_use_negative:
- Input abbreviations already convert under the default configuration for one of the
  5 supported databases or 17 supported programs (e.g. HMDB, LIPID MAPS, MS-DIAL,
  MZmine2) — no new rules are needed
- The abbreviation requires support for an entire unsupported lipid domain (e.g. plant
  or bacterial lipids, which LipidLynxX currently does not cover) rather than a simple
  alias mapping
- The target output is a third-party identifier (SwissLipids, LIPID MAPS accession)
  rather than a unified LipidLynxX shorthand annotation — use the Linker module instead
edam_operation: http://edamontology.org/operation_2409
tools:
- name: LipidLynxX
  role: Target converter whose rule-based JSON alias configuration is extended and
    whose Converter module (API, CLI, GUI, or binder notebook) validates the new rules
  repo: https://github.com/SysMedOs/LipidLynxX
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2020.04.09.033894
    title: 'LipidLynxX: a data transfer hub to support integration of large scale
      lipidomics datasets'
  source_content_hash: sha256:1c2f7e39d372cedd7af7af3af72c72bfb3982ba14f507ccad3b314fbe1315ef5
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# json-rule-based-configuration-authoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Author rule-based JSON configuration files (e.g. extended alias definitions for LipidLynxX's Converter) that map non-standard lipid abbreviations — such as oxidized fatty acid short forms like HETE or PONPC — to unified LipidLynxX identifiers, extending tool coverage without modifying the software source code.

## When to use

Use when a lipidomics identifier converter fails to recognize abbreviations from a database or program (e.g. oxidized lipid aliases absent from the default `lynx/configurations/defined_alias.json`), and the goal is to add new input styles via configuration rather than code changes.

## When NOT to use

- Input abbreviations already convert under the default configuration for one of the 5 supported databases or 17 supported programs (e.g. HMDB, LIPID MAPS, MS-DIAL, MZmine2) — no new rules are needed
- The abbreviation requires support for an entire unsupported lipid domain (e.g. plant or bacterial lipids, which LipidLynxX currently does not cover) rather than a simple alias mapping
- The target output is a third-party identifier (SwissLipids, LIPID MAPS accession) rather than a unified LipidLynxX shorthand annotation — use the Linker module instead

## Inputs

- List of lipid abbreviations that failed conversion (e.g. oxidized fatty acid aliases such as HETE, PONPC)
- Default LipidLynxX alias configuration file (lynx/configurations/defined_alias.json)
- LipidLynxX shorthand annotation targets for each new abbreviation

## Outputs

- Extended rule-based alias configuration JSON file (e.g. extended_aliases.json)
- Conversion output table mapping each new abbreviation to a unified LipidLynxX annotation (csv/xlsx from the converter)

## How to apply

First inspect the default rule structure by loading the built-in alias configuration file shipped in the SysMedOs/LipidLynxX repository (see `lynx/configurations/defined_alias.json`), since LipidLynxX enforces a 'strictly controlled format using JSON schema'. Compile a test list of the abbreviations that failed conversion, then author new rule-based JSON entries mapping each abbreviation to the corresponding LipidLynxX shorthand annotation, keeping the JSON formatted as a standard editor-formatted configuration file and saving it (e.g. as extended_aliases.json). Decision point: prefer configuration extension over code modification — the Converter supports 'common abbreviations (customizable)' as defined aliases precisely so users can add styles for unsupported databases. Finally, run the LipidLynxX Converter via its API, GUI, command line (`python cli_lynx.py`), or the mybinder converter notebook with the extended configuration applied to the test list, and confirm every added abbreviation converts to the expected unified lipid annotation.

## Related tools

- **LipidLynxX** (Target converter whose rule-based JSON alias configuration is extended and whose Converter module (API, CLI, GUI, or binder notebook) validates the new rules) — https://github.com/SysMedOs/LipidLynxX

## Examples

```
python cli_lynx.py
```

## Evaluation signals

- Every abbreviation in the test list converts to the expected unified LipidLynxX shorthand annotation using the extended configuration
- No unmatched/unconverted IDs remain in the converter output table
- The extended JSON parses cleanly and conforms to LipidLynxX's strictly controlled JSON schema format
- Previously failing inputs now convert identically whether run through the Windows .exe, CLI, or binder converter notebook
- Existing default aliases (e.g. DHA, PAPE, PLPC) still convert correctly, confirming no regression in the extended configuration

## Limitations

- LipidLynxX currently covers mammalian lipids from 25 classes in five categories (fatty acids, glycerolipids, glycerophospholipids, sphingolipids, sterols); plant and bacterial lipids are not covered and may need future extension
- If no configuration file in `lynx/configurations/rules/input` fits a database or program, a suitable configuration file may require contacting the developers
- Windows .exe users may see IDs not converted that are fixed in the latest binder demo — the code 'is still changing rapidly', so API definitions and documentation may lag the source
- The README is packaged without a changelog; new features are developed on a separate repository (ZhixuNi/LipidLynxX)

## Evidence

- [readme] Common abbreviations are customizable aliases in JSON: [evidence span withheld — anchor status: not_found]
- [readme] Alias settings live in a JSON configuration file: [evidence span withheld — anchor status: external_doc]
- [abstract] LipidLynxX is customizable without code changes: "LipidLynxX is a flexible, customizable open-access tool freely available for download"
- [discussion] Converter translates annotations into three pre-defined formats: "Converter and Equalizer modules allow easy translation and cross-matching of lipid annotations into three pre-defined formats"
- [readme] Contact developers if no configuration file fits the input style: [evidence span withheld — anchor status: external_doc]
