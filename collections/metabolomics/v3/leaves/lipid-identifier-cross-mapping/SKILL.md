---
name: lipid-identifier-cross-mapping
description: Use when lipid annotation lists from tools like MS-DIAL, LipidSearch,
  or LipidBlast must be converted to a common identifier scheme or cross-mapped to
  LIPID MAPS, SwissLipids, PubChem, KEGG, ChEBI, or LION IDs for dataset compilation
  or meta-analysis.
when_to_use_negative:
- Input lipids are plant or bacterial lipids — LipidLynxX currently covers mammalian
  lipids from 25 classes in five categories only
- The dataset already contains cross-reference accession IDs and only needs pathway
  or network analysis rather than identifier conversion
- The lipid abbreviations come from a database/program outside the 26 supported input
  styles and no matching configuration file fits the export format
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: LipidLynxX
  role: Converter module converts lipid abbreviations to unified LipidLynxX IDs; Linker
    module translates annotations into LION, KEGG, ChEBI, Rhea, UniProtKB identifiers;
    run via GUI, CLI, API, or as Python module
  repo: https://github.com/SysMedOs/LipidLynxX
- name: Lion
  role: Target structural ontology (LION22) receiving LipidLynxX Linker identifier
    translations for lipid ontology analysis
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1101/2020.04.09.033894
    title: 'LipidLynxX: a data transfer hub to support integration of large scale
      lipidomics datasets'
  source_content_hash: sha256:8cfc4fd5d8d873e952a38dcbed83707a41ab6e2d795ab68bd98dbe47929a9256
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
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# lipid-identifier-cross-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps lipid shorthand annotations from heterogeneous identification tools into unified LipidLynxX identifiers and cross-reference accession IDs (LIPID MAPS, SwissLipids, PubChem, KEGG, ChEBI, LION) using the LipidLynxX Converter and Linker modules, closing the data-transfer gap between lipidomics datasets and systems biology integration tools.

## When to use

Use when lipid annotation lists from tools like MS-DIAL, LipidSearch, or LipidBlast must be converted to a common identifier scheme or cross-mapped to LIPID MAPS, SwissLipids, PubChem, KEGG, ChEBI, or LION IDs for dataset compilation or meta-analysis.

## When NOT to use

- Input lipids are plant or bacterial lipids — LipidLynxX currently covers mammalian lipids from 25 classes in five categories only
- The dataset already contains cross-reference accession IDs and only needs pathway or network analysis rather than identifier conversion
- The lipid abbreviations come from a database/program outside the 26 supported input styles and no matching configuration file fits the export format

## Inputs

- List of lipid shorthand annotations / abbreviations (CSV or XLSX; e.g. the 131-lipid Supplementary File 2 of bioRxiv 10.1101/2020.04.09.033894)
- Lipid abbreviations in supported styles: space shorthand (PC 16:0_18:2), bracket derivatives (PC(16:0_18:2)), or defined aliases (DHA, PAPE, PLPC)
- Per-lipid Linker translation results with cross-reference identifier columns

## Outputs

- converted_annotations.csv: input shorthand plus LipidLynxX shorthand, LIPID MAPS, SwissLipids, and PubChem identifiers per lipid
- linker_coverage_table.csv: 131-lipid × 5-database (LION, KEGG, ChEBI, Rhea, UniProtKB) binary coverage matrix and per-class unmatched summary
- Converter output table in .csv or .xlsx with selected export style

## How to apply

Install LipidLynxX from github.com/SysMedOs/LipidLynxX (Python 3.8.5 from source, or the Windows v0.9.24 .exe). Load the lipid abbreviation list (e.g. the 131-lipid Supplementary File 2 dataset, in space-shorthand like 'PC 16:0_18:2' or bracket style 'PC(16:0_18:2)') into the Converter in batch mode, which translates 26 recognized input styles (5 databases, 17 programs) into three canonical formats at the equalized structural annotation level. Cross-map the converted annotations to LIPID MAPS, SwissLipids, and PubChem accession identifiers via the corresponding ID mapping services, then run the Linker module to translate annotations into identifiers for LION, KEGG, ChEBI, Rhea, and UniProtKB. Write per-lipid output tables to CSV/XLSX and build a 131-row × 5-column binary presence/absence coverage matrix (pandas) to quantify per-database mapping coverage, aggregating unmatched (NaN) identifiers by lipid class (fatty acids, glycerolipids, glycerophospholipids, sphingolipids, sterols).

## Related tools

- **LipidLynxX** (Converter module converts lipid abbreviations to unified LipidLynxX IDs; Linker module translates annotations into LION, KEGG, ChEBI, Rhea, UniProtKB identifiers; run via GUI, CLI, API, or as Python module) — https://github.com/SysMedOs/LipidLynxX
- **Lion** (Target structural ontology (LION22) receiving LipidLynxX Linker identifier translations for lipid ontology analysis)

## Examples

```
python cli_lynx.py
```

## Evaluation signals

- All 131 input annotations yield converted identifiers with zero conversion failures
- Coverage matrix contains exactly 131 lipid rows and five database coverage columns
- Per-class unmatched summary is internally consistent with the per-lipid binary presence/absence matrix
- Expected per-database mapping counts for the 131-lipid test set: LION 98, ChEBI 49, KEGG 130
- Output table exports cleanly as .csv or .xlsx with one row per input abbreviation in the selected export style

## Limitations

- Plant and bacterial lipids are not currently covered by LipidLynxX (mammalian lipids from 25 classes, five categories only)
- Known Linker issue: if Linker runs more than 300s or more than 30s per ID, restart LipidLynxX; run max 3 tasks at the same time due to packaging limits
- Linker requires firewall permission to access external ID-mapping databases
- No specific version, build, or container image is cited for the discussion-level claims; source code changes rapidly and API definitions may lag

## Evidence

- [readme] Converter usage and export: [evidence span withheld — anchor status: external_doc]
- [readme] Supported input notation styles: [evidence span withheld — anchor status: external_doc]
