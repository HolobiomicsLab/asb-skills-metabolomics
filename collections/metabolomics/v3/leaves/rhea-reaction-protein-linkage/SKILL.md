---
name: rhea-reaction-protein-linkage
description: Use when an experimentally annotated lipid list (e.g. 131 shorthand lipid
  abbreviations) must be linked to biochemical reactions (Rhea) and protein entries
  (UniProtKB) to enable downstream pathway/network integration.
when_to_use_negative:
- The input annotations describe plant or bacterial lipids, which are not currently
  covered by LipidLynxX (mammalian lipids from 25 classes in five categories only)
- The annotations have not been unified to a common level/format — cross-matching
  between experimental annotation levels and target resources can fail; use the Converter/Equalizer
  first
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3407
tools:
- name: LipidLynxX
  role: Its Linker module performs the reaction and protein extraction, cross-matching
    unified lipid IDs to Rhea reactions and UniProtKB proteins
  repo: https://github.com/SysMedOs/LipidLynxX
- name: Rhea
  role: Target biochemical-reaction resource retrieved by the Linker for matched lipids
- name: UniProtKB
  role: Target protein resource retrieved by the Linker for matched lipids
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2020.04.09.033894
    title: 'LipidLynxX: a data transfer hub to support integration of large scale
      lipidomics datasets'
  source_content_hash: sha256:9f78b25228f1d8a910970f9d1e5570f8bcb06ed92c1bf5a1b0206460e9179dc0
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
  - Rhea
  - UniProtKB
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3407
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# rhea-reaction-protein-linkage

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Uses the LipidLynxX Linker module to extract biochemical reactions (Rhea IDs) and protein entries (UniProtKB) associated with a list of lipid annotations, bridging lipidomics data to systems biology pathway and network analysis.

## When to use

Use when an experimentally annotated lipid list (e.g. 131 shorthand lipid abbreviations) must be linked to biochemical reactions (Rhea) and protein entries (UniProtKB) to enable downstream pathway/network integration.

## When NOT to use

- The input annotations describe plant or bacterial lipids, which are not currently covered by LipidLynxX (mammalian lipids from 25 classes in five categories only)
- The annotations have not been unified to a common level/format — cross-matching between experimental annotation levels and target resources can fail; use the Converter/Equalizer first

## Inputs

- List of lipid annotations in a supported shorthand notation (e.g. PC 16:0_18:2), ideally already converted to LipidLynxX IDs
- LipidLynxX configuration files (lynx/config.ini, lynx/configurations/) with resource-linking settings

## Outputs

- Rhea reaction IDs associated with the matched lipids
- UniProtKB protein entries associated with the matched lipids
- Per-database linkage counts (e.g. linker_mapping_counts.csv)

## How to apply

First convert the input lipid annotations into LipidLynxX IDs using the Converter/Equalizer modules, since the Linker operates on unified LipidLynxX identifiers. Then invoke the Linker, which queries external resources over the network (firewall access must be allowed for this module) to cross-match lipids to KEGG compound IDs and ChEBI chemical-entity IDs, and extract associated Rhea reaction IDs and UniProtKB protein entries. Tabulate the per-database match counts; in the paper's 131-lipid example, 125 lipids matched 249 Rhea IDs and 223 UniProtKB proteins. Note that the Linker covers mammalian lipids from 25 classes in five categories (fatty acids, glycerolipids, glycerophospholipids, sphingolipids, sterols), and that annotations at different levels than the target model (e.g. GSMN) can complicate matching.

## Related tools

- **LipidLynxX** (Its Linker module performs the reaction and protein extraction, cross-matching unified lipid IDs to Rhea reactions and UniProtKB proteins) — https://github.com/SysMedOs/LipidLynxX
- **Rhea** (Target biochemical-reaction resource retrieved by the Linker for matched lipids)
- **UniProtKB** (Target protein resource retrieved by the Linker for matched lipids)

## Examples

```
python cli_lynx.py  # start LipidLynxX terminal tools; allow firewall access so the Linker module can query Rhea/UniProtKB
```

## Evaluation signals

- On the 131-lipid example dataset, the Linker should associate 125 lipids with 249 Rhea reaction IDs
- On the same dataset, the Linker should retrieve 223 UniProtKB protein entries
- Downstream context check: on the same input the Linker should also return 130 KEGG compound and 49 ChEBI matches, indicating the cross-matching step ran correctly before reaction extraction
- Linker outputs must conform to LipidLynxX's strictly controlled JSON schema format
- Linker runtime should stay under ~30 s per ID; if it runs more than 300 s, restart LipidLynxX and retry

## Limitations

- Plant and bacterial lipids are not currently covered — only mammalian lipids from 25 classes across five categories (fatty acids, glycerolipids, glycerophospholipids, sphingolipids, sterols)
- Different levels of annotation between experimental lipidomics datasets and target models (e.g. genome-scale metabolic networks) remain a challenge for linkage
- The Linker module requires network access and firewall permission to query databases; known issues include timeouts if it runs more than 300 s or 30 s per ID
- No specific version, build, or container image is cited; API definitions may lag rapid source-code changes

## Evidence

- [readme] Linker needs network access to databases: [evidence span withheld — anchor status: external_doc]
- [readme] Linker timeout known issue: [evidence span withheld — anchor status: external_doc]
