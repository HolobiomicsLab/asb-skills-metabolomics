---
name: lipid-annotation-conversion
description: Use when lipid shorthand annotations from heterogeneous software (e.g.
  MS-DIAL, LipidSearch, MZmine2) or databases (LIPID MAPS, SwissLipids, HMDB, RefMet)
  must be unified, level-equalized, or linked to LION/KEGG/ChEBI/Rhea/UniProtKB identifiers.
when_to_use_negative:
- 'The input lipids are plant or bacterial classes outside the current mammalian coverage
  (25 classes across five categories: fatty acids, glycerolipids, glycerophospholipids,
  sphingolipids, sterols) and no new JSON configuration entry has been added yet'
- The annotations are already in canonical LipidLynxX ID form and only raw identifier
  strings are needed with no cross-mapping or level equalization
- The molecules are non-lipid metabolites needing general ID conversion outside LipidLynxX's
  lipid-focused vocabulary
tools:
- name: LipidLynxX
  role: 'Core tool: Converter module converts abbreviations to uniformed LipidLynxX
    IDs, Equalizer cross-compares annotation levels, Linker translates annotations
    into LION, KEGG, ChEBI, Rhea, and UniProtKB identifiers'
  repo: https://github.com/SysMedOs/LipidLynxX
- name: Lion
  role: Target lipid structural ontology (LION) that the Linker module translates
    lipid annotations into
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_002
  - task_007
  source_papers:
  - doi: 10.1101/2020.04.09.033894
    title: 'LipidLynxX: a data transfer hub to support integration of large scale
      lipidomics datasets'
  source_content_hash: sha256:f50e0ba5a208d69e266de05dca14c03143e6abe8571e10a3086067c77d67848f
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
  - Lion
  - LipidLynxX
  - PyCharm
  - Visual Studio Code
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# lipid-annotation-conversion

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Convert heterogeneous lipid annotations (26 input styles from 5 databases and 17 lipidomics programs) into unified LipidLynxX IDs and cross-mapped LIPID MAPS, SwissLipids, and PubChem identifiers, then translate them into ontology/pathway/network identifiers (LION, KEGG, ChEBI, Rhea, UniProtKB) via LipidLynxX's Converter, Equalizer, and Linker modules.

## When to use

Use when lipid shorthand annotations from heterogeneous software (e.g. MS-DIAL, LipidSearch, MZmine2) or databases (LIPID MAPS, SwissLipids, HMDB, RefMet) must be unified, level-equalized, or linked to LION/KEGG/ChEBI/Rhea/UniProtKB identifiers.

## When NOT to use

- The input lipids are plant or bacterial classes outside the current mammalian coverage (25 classes across five categories: fatty acids, glycerolipids, glycerophospholipids, sphingolipids, sterols) and no new JSON configuration entry has been added yet
- The annotations are already in canonical LipidLynxX ID form and only raw identifier strings are needed with no cross-mapping or level equalization
- The molecules are non-lipid metabolites needing general ID conversion outside LipidLynxX's lipid-focused vocabulary

## Inputs

- Lipid annotation lists in 26 supported styles (5 databases: HMDB, LIPID MAPS LMSD & COMP_DB, LipidHome, RefMet, SwissLipids; 17 programs: ALEX123, Greazy, LDA 2, LipidBlast, LipidCreator, LipiDex, LipidFrag, LipidHunter, LipidMatch, LipidPro, LipidSearch, Lipostar, LIQUID, LPPtiger, MetFrag, MS-DIAL, MZmine2)
- Shorthand notation strings with spaces (e.g. PC 16:0_18:2) or brackets (e.g. PC(16:0_18:2))
- Common abbreviations / defined aliases (e.g. DHA, PAPE, PLPC, PONPC)
- Modified-lipid (epilipidome) annotations encoded across five levels: nominal mass shift, elemental composition, modification type, position, stereochemistry
- JSON configuration files from lynx/configurations/rules/input for custom input styles

## Outputs

- Uniformed LipidLynxX IDs (shorthand notation and LipidLynxX notation at the equalized structural annotation level)
- Cross-mapped LIPID MAPS, SwissLipids, and PubChem accession identifiers
- Combined conversion table converted_annotations.csv (input shorthand plus all converted identifiers per lipid)
- Linker translation table with LION, KEGG compound, ChEBI, Rhea reaction, and UniProtKB identifiers, summarized as linker_mapping_counts.csv
- Per-lipid CSV/XLSX export tables from the converter notebook/GUI

## How to apply

Load the lipid annotation list (e.g. the 131-lipid example dataset, Supplementary File 2) into the LipidLynxX Converter in batch mode; each abbreviation (including space-based shorthand like 'PC 16:0_18:2', bracketed derivatives like 'PC(16:0_18:2)', and defined aliases like DHA, PAPE, PLPC, PONPC from lynx/configurations/defined_alias.json) is converted to the uniformed LipidLynxX ID. Use the Equalizer module to cross-compare IDs at a selected structural annotation level so datasets are equalized for cross-dataset comparison and meta-analysis. Cross-map the converted annotations to LIPID MAPS, SwissLipids, and PubChem accession identifiers via the corresponding ID mapping services, and run the Linker module to translate annotations into LION ontology IDs, KEGG compound IDs, ChEBI IDs, Rhea reaction IDs, and UniProtKB protein entries. If an input style is unsupported, test configuration files in lynx/configurations/rules/input or author new JSON configuration entries (extensible without code changes for non-mammalian classes). Tabulate per-database match counts and write the combined conversion table to CSV (converted_annotations.csv, linker_mapping_counts.csv).

## Related tools

- **LipidLynxX** (Core tool: Converter module converts abbreviations to uniformed LipidLynxX IDs, Equalizer cross-compares annotation levels, Linker translates annotations into LION, KEGG, ChEBI, Rhea, and UniProtKB identifiers) — https://github.com/SysMedOs/LipidLynxX
- **Lion** (Target lipid structural ontology (LION) that the Linker module translates lipid annotations into)

## Examples

```
python cli_lynx.py
```

## Evaluation signals

- On the 131-lipid example dataset (Supplementary File 2), all 131 annotations yield converted identifiers with zero conversion failures
- Linker counts reproduce the reported values: 98 of 131 lipids translated to LION IDs, 130 matched to KEGG compound IDs, 49 to ChEBI, and 125 lipids matched to 249 Rhea IDs and 223 UniProtKB proteins
- For modified lipids, each converted entry retains all five encoded annotation levels (mass shift, elemental composition, modification type, position, stereochemistry) populated
- Output CSV is well-formed and contains one row per input annotation with input shorthand plus converted identifiers
- Conversion of bracketed vs space-separated shorthand variants (PC 16:0_18:2 vs PC(16:0_18:2)) yields the same canonical LipidLynxX ID

## Limitations

- Coverage is mainly mammalian lipids (25 classes, five categories); plant and bacterial lipids are not currently covered and require authoring new JSON configuration template files
- Windows .exe packaging limits users to max 3 tasks at the same time; if the Linker runs more than 300s or more than 30s per ID, LipidLynxX must be restarted
- The Linker module needs firewall permission to access external databases for ID mapping
- API definitions and documentation in the source code may not be updated as the code changes rapidly; contact the developers before using the API
- No changelog is available in the repository snapshot

## Evidence

- [readme] Abbreviations such as DHA, PAPE, PLPC, PONPC .etc are also included as `defined alias`.: [evidence span withheld — anchor status: external_doc]
- [readme] you can test if any of the configuration files located in `lynx/configurations/rules/input` would fit to your database / program: [evidence span withheld — anchor status: external_doc]
