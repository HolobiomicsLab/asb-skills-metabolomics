---
name: lipidomics-data-integration
description: Use when a list of lipid annotations (e.g. shorthand notations like PC
  16:0_18:2) must be mapped to ontology/pathway/reaction IDs (LION, KEGG, ChEBI, Rhea,
  UniProtKB) and coverage gaps per lipid class must be assessed.
when_to_use_negative:
- The lipids of interest are plant or bacterial lipids, which are not currently covered
  by LipidLynxX
- The task is annotation-level normalization or cross-dataset equalization rather
  than linking to external integration resources (use the Converter/Equalizer modules
  instead)
- Input is not lipid annotations but already-mapped database IDs or a feature/abundance
  table
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: LipidLynxX
  role: Linker module translates lipid annotations directly into identifiers for data-integration
    tools; run via GUI (LipidLynxX.exe / python LipidLynxX.py), CLI, or API
  repo: https://github.com/SysMedOs/LipidLynxX
- name: Lion
  role: LIPID structural ontology (e.g. LION22) used as one of the identifier-mapping
    targets for lipid ontology analysis
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2020.04.09.033894
    title: 'LipidLynxX: a data transfer hub to support integration of large scale
      lipidomics datasets'
  source_content_hash: sha256:159dc2fe69614a8a09231d4ccf4afeac165132f5aa024b064d668ed3fe032253
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# lipidomics-data-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Translate lipid abbreviations into cross-reference identifiers for data-integration resources (LION, KEGG, ChEBI, Rhea, UniProtKB) using the LipidLynxX Linker module, then quantify per-database coverage across a lipid test set.

## When to use

Use when a list of lipid annotations (e.g. shorthand notations like PC 16:0_18:2) must be mapped to ontology/pathway/reaction IDs (LION, KEGG, ChEBI, Rhea, UniProtKB) and coverage gaps per lipid class must be assessed.

## When NOT to use

- The lipids of interest are plant or bacterial lipids, which are not currently covered by LipidLynxX
- The task is annotation-level normalization or cross-dataset equalization rather than linking to external integration resources (use the Converter/Equalizer modules instead)
- Input is not lipid annotations but already-mapped database IDs or a feature/abundance table

## Inputs

- Per-lipid Linker translation results from LipidLynxX (cross-reference identifier columns for LION, KEGG, ChEBI, Rhea, UniProtKB)
- List of lipid annotations for major mammalian lipid classes (e.g. 131-lipid test set across 25 classes)
- Lipid class/category labels (fatty acids, glycerolipids, glycerophospholipids, sphingolipids, sterols)

## Outputs

- linker_coverage_table.csv containing a 131-lipid × 5-database binary coverage matrix
- Per-database summary of unmatched identifier classes (e.g. 98/131 LION, 49/131 ChEBI, 130/131 KEGG)

## How to apply

Run the lipid abbreviation list through the LipidLynxX Linker module, which links lipid annotations to available resources and extracts identifiers for biochemical-reaction and ontology databases; allow network access for the Linker to reach databases. Parse the per-lipid Linker translation results and extract the identifier fields for the five target resources (LION, KEGG, ChEBI, Rhea, UniProtKB). Build a binary presence/absence coverage matrix with one row per lipid (e.g. 131 lipids) and one column per target database using pandas, treating unmatched (NaN) identifiers as absent. Aggregate unmatched identifiers by lipid class and category (fatty acids, glycerolipids, glycerophospholipids, sphingolipids, sterols) to detect systematically unmatched classes per database. Write the coverage matrix and per-database unmatched-class summary to linker_coverage_table.csv, and validate that the table has exactly the expected lipid rows and five database coverage columns.

## Related tools

- **LipidLynxX** (Linker module translates lipid annotations directly into identifiers for data-integration tools; run via GUI (LipidLynxX.exe / python LipidLynxX.py), CLI, or API) — https://github.com/SysMedOs/LipidLynxX
- **Lion** (LIPID structural ontology (e.g. LION22) used as one of the identifier-mapping targets for lipid ontology analysis)

## Examples

```
python LipidLynxX.py  # GUI starts backend; use Linker module to link lipid abbreviation list to LION/KEGG/ChEBI/Rhea/UniProtKB resources
```

## Evaluation signals

- Output coverage table contains exactly 131 lipid rows and five database coverage columns (LION, KEGG, ChEBI, Rhea, UniProtKB)
- Per-class unmatched summary is internally consistent with the per-lipid binary presence/absence matrix
- Coverage counts match published expectations: 98 lipids mapped to LION IDs, 49 to ChEBI, and 130 to KEGG compound IDs
- Linker jobs complete within the operational window (restart if the Linker runs more than 300s or more than 30s per ID)
- Every lipid is attributed to one of the five covered mammalian categories (fatty acids, glycerolipids, glycerophospholipids, sphingolipids, sterols)

## Limitations

- LipidLynxX currently covers mammalian lipids from 25 classes representing five categories; plant and bacterial lipids are not covered
- Linker is slow on some IDs: if it runs more than 300s or more than 30s per ID, restart LipidLynxX and retry; the Windows .exe limits users to max 3 concurrent tasks
- Different annotation levels between experimental lipidomics datasets and genome-scale metabolic models (GSMN) can impede downstream integration
- The Linker must be allowed through the firewall to access external databases

## Evidence

- [readme] Linker operational limits from README: [evidence span withheld — anchor status: external_doc]
