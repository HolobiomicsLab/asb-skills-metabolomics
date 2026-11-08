---
name: lipidomics-cross-reference-count-validation
description: Use when a lipid annotation list must be linked to LION ontology, KEGG
  compound, ChEBI entity, Rhea reaction, or UniProtKB protein identifiers and the
  output needs to be tabulated and validated against reported match counts.
when_to_use_negative:
- The input contains plant or bacterial lipids — LipidLynxX currently covers mammalian
  lipids from 25 classes in five categories (fatty acids, glycerolipids, glycerophospholipids,
  sphingolipids, sterols)
- The goal is only shorthand conversion or level equalization without cross-referencing
  — the Converter or Equalizer modules, not the Linker, are the right tools
- The input format is a database/program not among the 5 databases and 17 programs
  tested — conversion may fail unless a suitable rules file exists in lynx/configurations/rules/input
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3353
- http://edamontology.org/topic_0091
tools:
- name: LipidLynxX
  role: Linker module translates lipid annotations into LION, KEGG, ChEBI, Rhea, and
    UniProtKB identifiers; note the Linker requires network/firewall access to query
    databases
  repo: https://github.com/SysMedOs/LipidLynxX
- name: LION
  role: Target lipid structural ontology receiving the translated lipid identifiers
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2020.04.09.033894
    title: 'LipidLynxX: a data transfer hub to support integration of large scale
      lipidomics datasets'
  source_content_hash: sha256:ed43e78a944236fdb92b4f5a4d1c59c585820eadb8513b074620f8473c413d2e
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
  - LION
  - Lion
  - LipidLynxX
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3353
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# lipidomics-cross-reference-count-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce and validate the per-database match counts produced by the LipidLynxX Linker module when translating a lipid annotation list into ontology, pathway, and network identifiers (LION, KEGG, ChEBI, Rhea, UniProtKB).

## When to use

Use when a lipid annotation list must be linked to LION ontology, KEGG compound, ChEBI entity, Rhea reaction, or UniProtKB protein identifiers and the output needs to be tabulated and validated against reported match counts.

## When NOT to use

- The input contains plant or bacterial lipids — LipidLynxX currently covers mammalian lipids from 25 classes in five categories (fatty acids, glycerolipids, glycerophospholipids, sphingolipids, sterols)
- The goal is only shorthand conversion or level equalization without cross-referencing — the Converter or Equalizer modules, not the Linker, are the right tools
- The input format is a database/program not among the 5 databases and 17 programs tested — conversion may fail unless a suitable rules file exists in lynx/configurations/rules/input

## Inputs

- 131-lipid example annotation list shipped with the LipidLynxX repository (LipidLynxX/doc/sample_data/input)
- Lipid abbreviation list in supported notation styles (e.g. PC 16:0_18:2, PC(16:0_18:2), defined aliases such as DHA, PLPC)
- LipidLynxX configuration files (lynx/config.ini, lynx/configurations/defined_alias.json)

## Outputs

- linker_mapping_counts.csv with per-database match counts (LION, KEGG, ChEBI, Rhea, UniProtKB)
- Per-lipid cross-reference tables mapping annotations to KEGG compound, ChEBI, Rhea reaction, and UniProtKB identifiers

## How to apply

Load the 131-lipid example annotation list shipped with the LipidLynxX repository (sample files live under LipidLynxX/doc/sample_data/input). Invoke the Linker module — via GUI, command line (cli_lynx.py), or the Python API — to translate each annotation into LION ontology IDs, then use the Linker's cross-matching to map lipids to KEGG compound IDs and ChEBI chemical-entity IDs. Run the reaction/protein extraction step to associate matched lipids with Rhea reaction IDs and UniProtKB protein entries. Tabulate per-database counts (LION, KEGG, ChEBI, Rhea, UniProtKB) and save as linker_mapping_counts.csv. Validate against the reported values: 98 of 131 lipids translated to LION IDs, 130 matched to KEGG, 49 to ChEBI, and 125 lipids matched to 249 Rhea IDs and 223 UniProtKB proteins.

## Related tools

- **LipidLynxX** (Linker module translates lipid annotations into LION, KEGG, ChEBI, Rhea, and UniProtKB identifiers; note the Linker requires network/firewall access to query databases) — https://github.com/SysMedOs/LipidLynxX
- **LION** (Target lipid structural ontology receiving the translated lipid identifiers)

## Examples

```
python cli_lynx.py  # run Linker on the 131-lipid sample list from doc/sample_data/input and export cross-reference counts
```

## Evaluation signals

- Exactly 98 of the 131 lipids are translated to LION ontology IDs
- Exactly 130 lipids match KEGG compound IDs and exactly 49 match ChEBI chemical-entity IDs
- 125 lipids link to 249 Rhea reaction IDs and 223 UniProtKB protein entries
- linker_mapping_counts.csv contains all five per-database count columns with no missing values
- Unmatched annotations can be retried via the Binder converter notebook to check whether the Windows .exe version failed where the latest source succeeds

## Limitations

- Plant and bacterial lipids are not currently covered by LipidLynxX
- In the Windows .exe build, if the Linker runs more than 300s or more than 30s per ID it should be restarted, and at most 3 tasks may run concurrently
- Different annotation levels between experimental lipidomics datasets and GSMN contents can challenge downstream integration
- The API and documentation definitions may lag the rapidly changing source code; developers should check for updates before using the API
- No specific version, build, or container image is cited in the discussion; README indicates client release v0.9.24

## Evidence

- [results] The Linker module translated 98 of the 131 example lipids into LION IDs, and matched 49 and 130 lipids to ChEBI and KEGG compound IDs, respectively.: [evidence span withheld — anchor status: not_found]
- [readme] Test input file: `LipidLynxX/doc/sample_data/input`: [evidence span withheld — anchor status: external_doc]
- [readme] Please allow LipidLynxX to pass firewall for the Linker module to access databases: [evidence span withheld — anchor status: external_doc]
