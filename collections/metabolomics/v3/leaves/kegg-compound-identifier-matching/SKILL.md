---
name: kegg-compound-identifier-matching
description: Use when a list of lipid annotations (e.g. shorthand notations like PC
  16:0_18:2) must be matched to KEGG compound IDs — or ChEBI/LION identifiers — as
  a prerequisite for pathway (KEGG, Reactome) or ontology-based lipidome analysis.
when_to_use_negative:
- The input annotations cover plant or bacterial lipids — LipidLynxX currently covers
  mammalian lipids from 25 classes (fatty acids, glycerolipids, glycerophospholipids,
  sphingolipids, sterols)
- The input is raw mass-spectrometry data (mzML, spectra) rather than already-identified
  lipid annotations requiring identifier cross-matching
- Only a unified shorthand format is needed with no downstream database linkage —
  the Converter or Equalizer module suffices without invoking the Linker
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: LipidLynxX
  role: Provides the Linker module that cross-matches lipid annotations to KEGG compound,
    ChEBI, LION, Rhea, and UniProtKB identifiers; run via GUI, API, CLI (cli_lynx.py),
    or as a Python module
  repo: https://github.com/SysMedOs/LipidLynxX
- name: LION
  role: Lipid ontology target database; the Linker translates lipid annotations into
    LION ontology identifiers alongside KEGG matches
- name: KEGG
  role: Pathway database providing the compound IDs that matched lipids are mapped
    onto (130/131 on the example dataset)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2020.04.09.033894
    title: 'LipidLynxX: a data transfer hub to support integration of large scale
      lipidomics datasets'
  source_content_hash: sha256:86f1ac565d3729240fedc0b802a16a894d20cab218755c6e96f798164951a8e5
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
  - KEGG
  - LION
  - Lion
  - LipidLynxX
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# kegg-compound-identifier-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Uses the LipidLynxX Linker module to cross-match mammalian lipid abbreviations against KEGG compound identifiers (with parallel ChEBI and LION mappings), enabling transfer of experimental lipidomics annotations into pathway and network analysis resources.

## When to use

Use when a list of lipid annotations (e.g. shorthand notations like PC 16:0_18:2) must be matched to KEGG compound IDs — or ChEBI/LION identifiers — as a prerequisite for pathway (KEGG, Reactome) or ontology-based lipidome analysis.

## When NOT to use

- The input annotations cover plant or bacterial lipids — LipidLynxX currently covers mammalian lipids from 25 classes (fatty acids, glycerolipids, glycerophospholipids, sphingolipids, sterols)
- The input is raw mass-spectrometry data (mzML, spectra) rather than already-identified lipid annotations requiring identifier cross-matching
- Only a unified shorthand format is needed with no downstream database linkage — the Converter or Equalizer module suffices without invoking the Linker

## Inputs

- Lipid annotation list (e.g. the 131-lipid example file shipped with the LipidLynxX repository, in doc/sample_data/input)
- Lipid abbreviations in any of the supported styles: database exports (HMDB, LIPID MAPS, SwissLipids, RefMet, LipidHome) or program outputs (MS-DIAL, MZmine2, LipidSearch, ALEX123, etc.)
- Defined aliases (e.g. DHA, PAPE, PLPC) resolvable via lynx/configurations/defined_alias.json

## Outputs

- lipid-to-KEGG compound ID mapping table
- Cross-match tables to ChEBI and LION identifiers
- Rhea reaction ID and UniProtKB protein associations for matched lipids
- Per-database match-count summary (linker_mapping_counts.csv)

## How to apply

Load the lipid annotation list into LipidLynxX (GUI, API, CLI via cli_lynx.py, or Python module) and invoke the Linker module, which links lipid abbreviations to available external resources and may require network/firewall access to query remote databases. The Linker first normalizes annotations to unified LipidLynxX IDs (the Converter/Equalizer modules support 26 input styles including HMDB, LIPID MAPS LMSD & COMP_DB, SwissLipids, RefMet, MS-DIAL, MZmine2), then cross-matches each lipid to KEGG compound identifiers, ChEBI chemical-entity IDs, and LION ontology IDs. A reaction/protein extraction step then associates matched lipids with Rhea reaction IDs and UniProtKB protein entries. Tabulate per-database match counts (e.g. as linker_mapping_counts.csv); on the paper's 131-lipid example dataset, correct execution yields 130/131 KEGG compound matches, 49/131 ChEBI matches, 98/131 LION translations, and 125 lipids linked to 249 Rhea IDs and 223 UniProtKB proteins.

## Related tools

- **LipidLynxX** (Provides the Linker module that cross-matches lipid annotations to KEGG compound, ChEBI, LION, Rhea, and UniProtKB identifiers; run via GUI, API, CLI (cli_lynx.py), or as a Python module) — https://github.com/SysMedOs/LipidLynxX
- **LION** (Lipid ontology target database; the Linker translates lipid annotations into LION ontology identifiers alongside KEGG matches)
- **KEGG** (Pathway database providing the compound IDs that matched lipids are mapped onto (130/131 on the example dataset))

## Examples

```
python cli_lynx.py
```

## Evaluation signals

- On the 131-lipid example dataset, the KEGG compound match count must equal 130 of 131 lipids
- Parallel counts must reproduce: 98/131 LION ID translations and 49/131 ChEBI matches
- Reaction/protein extraction must yield 125 lipids matched to 249 Rhea IDs and 223 UniProtKB proteins
- Every output mapping should trace back to a valid unified LipidLynxX ID produced by the Converter (strictly controlled JSON schema format)
- Per-database counts written to linker_mapping_counts.csv should be complete (one row per target database: LION, KEGG, ChEBI, Rhea, UniProtKB)

## Limitations

- Lipid annotations at structural levels differing from those expected by downstream tools can challenge matching; the Linker relies on LipidLynxX's annotation-level equalization
- The Linker requires database access over the network — firewall permissions are needed, and known instability exists if Linker runs longer than 300s or 30s per ID (restart required)
- Tool coverage is limited to mammalian lipids (25 classes, five categories); plant and bacterial lipids are not covered
- API definitions in the source code may lag behind rapid changes; specific version (v0.9.24 client) should be noted for reproducibility

## Evidence

- [abstract] LipidLynxX provides the possibility to convert, cross-match, and link various lipid annotations to the tools supporting lipid ontology, pathway, and network analysis: "LipidLynxX provides the possibility to convert, cross-match, and link various lipid annotations to the tools supporting lipid ontology, pathway, and network analysis"
- [discussion] LipidLynxX Linker enables direct translation of lipid annotations into identifiers: "LipidLynxX Linker module enables direct translation of lipid annotations into identifiers"
- [intro] pathways (e.g. KEGG32, Reactome33): "pathways (e.g. KEGG32, Reactome33)"
- [readme] Please allow LipidLynxX to pass firewall for the Linker module to access databases: [evidence span withheld — anchor status: external_doc]
