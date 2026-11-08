---
name: lipid-ontology-id-mapping
description: Use when a lipid annotation list (e.g. abbreviations like PC 16:0_18:2)
  needs identifiers for downstream ontology (LION), pathway (KEGG, Rhea), or network/interaction
  (UniProtKB, ChEBI) analysis.
when_to_use_negative:
- Input lipids are plant or bacterial — LipidLynxX currently covers mammalian lipids
  from 25 classes in five categories (fatty acids, glycerolipids, glycerophospholipids,
  sphingolipids, sterols)
- The goal is only format normalization between annotation styles (e.g. LIPID MAPS
  to RefMet) — the Converter/Equalizer modules suffice; the Linker is for ontology/pathway/network
  identifier retrieval
- No network access is available, since the Linker must query external databases (ChEBI,
  KEGG, Rhea, UniProtKB) online
edam_operation: http://edamontology.org/operation_0289
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0121
tools:
- name: LipidLynxX
  role: Provides the Linker module that translates lipid annotations into LION, KEGG,
    ChEBI, Rhea, and UniProtKB identifiers; Converter/Equalizer produce the unified
    LipidLynxX IDs the Linker consumes
  repo: https://github.com/SysMedOs/LipidLynxX
- name: Lion
  role: Target lipid structural ontology; LION IDs are one of the identifier outputs
    the Linker maps annotations to
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2020.04.09.033894
    title: 'LipidLynxX: a data transfer hub to support integration of large scale
      lipidomics datasets'
  source_content_hash: sha256:c4e0b61f19acab695b3361e8cafa88b57bda1fc664d508e74445a8adb3e57b45
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
  - Lion
  - LipidLynxX
  edam_operation: http://edamontology.org/operation_0289
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# lipid-ontology-id-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps experimental lipid annotations to ontology, pathway, and network identifiers (LION, KEGG compound, ChEBI, Rhea reaction, UniProtKB protein) using the LipidLynxX Linker module, closing the gap between lipidomics ID lists and systems biology tools.

## When to use

Use when a lipid annotation list (e.g. abbreviations like PC 16:0_18:2) needs identifiers for downstream ontology (LION), pathway (KEGG, Rhea), or network/interaction (UniProtKB, ChEBI) analysis.

## When NOT to use

- Input lipids are plant or bacterial — LipidLynxX currently covers mammalian lipids from 25 classes in five categories (fatty acids, glycerolipids, glycerophospholipids, sphingolipids, sterols)
- The goal is only format normalization between annotation styles (e.g. LIPID MAPS to RefMet) — the Converter/Equalizer modules suffice; the Linker is for ontology/pathway/network identifier retrieval
- No network access is available, since the Linker must query external databases (ChEBI, KEGG, Rhea, UniProtKB) online

## Inputs

- Lipid annotation list / abbreviation table (shorthand notation e.g. PC 16:0_18:2, or tool-specific styles from MS-DIAL, LipidSearch, MZmine2, etc.)
- Example dataset: 131-lipid annotation list from LipidLynxX repository (doc/sample_data/input)

## Outputs

- Per-lipid mapping table with LION ontology IDs, KEGG compound IDs, and ChEBI chemical-entity IDs
- Reaction/protein association table: Rhea reaction IDs and UniProtKB protein entries per matched lipid
- Summary of per-database match counts (e.g. linker_mapping_counts.csv)

## How to apply

Load the lipid abbreviation list (e.g. the 131-lipid example annotation list shipped with the LipidLynxX repository, doc/sample_data/input) and invoke the Linker module, which first converts annotations to unified LipidLynxX IDs via the Converter, then cross-matches them to external resources. The Linker retrieves LION ontology identifiers, KEGG compound IDs, and ChEBI chemical-entity IDs for each lipid, and additionally extracts associated biochemical reactions (Rhea IDs) and proteins (UniProtKB entries). The Linker accesses external databases over the network, so firewall access must be permitted, and the README advises restarting if a run exceeds 300 s total or 30 s per ID. Tabulate per-database match counts; for the 131-lipid benchmark the expected results are 98 LION IDs, 130 KEGG compound IDs, 49 ChEBI IDs, and 125 lipids matched to 249 Rhea reaction IDs and 223 UniProtKB proteins.

## Related tools

- **LipidLynxX** (Provides the Linker module that translates lipid annotations into LION, KEGG, ChEBI, Rhea, and UniProtKB identifiers; Converter/Equalizer produce the unified LipidLynxX IDs the Linker consumes) — https://github.com/SysMedOs/LipidLynxX
- **Lion** (Target lipid structural ontology; LION IDs are one of the identifier outputs the Linker maps annotations to)

## Examples

```
python cli_lynx.py  # run LipidLynxX terminal tools, then select the Linker module on the 131-lipid sample input list
```

## Evaluation signals

- On the 131-lipid example dataset, LION matches = 98, KEGG compound matches = 130, ChEBI matches = 49
- Rhea/protein step: 125 lipids matched to 249 Rhea reaction IDs and 223 UniProtKB protein entries
- Every output identifier maps back to a unified LipidLynxX ID in the input list; unmatched IDs are reported rather than silently dropped
- Linker runtime stays within ~30 s per ID and 300 s per run; restart and re-run if exceeded
- Output table respects the strictly controlled JSON schema / export formats (.csv or .xlsx) defined by LipidLynxX

## Limitations

- Only mammalian lipids from 25 classes across five categories are covered; plant and bacterial lipids are not yet supported
- Linker depends on live access to external databases — firewall blocking or slow responses (>300 s or >30 s per ID) require a restart
- A challenge remains in matching annotation levels between experimental lipidomics datasets and genome-scale metabolic networks (GSMN)
- LipidLynxX is open source but no specific version, build, or container image is pinned in the article; the packaged Windows .exe is for testing only

## Evidence

- [readme] README network access requirement: [evidence span withheld — anchor status: external_doc]
