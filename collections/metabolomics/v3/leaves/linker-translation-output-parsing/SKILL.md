---
name: linker-translation-output-parsing
description: Use when a run of lipid abbreviations through the LipidLynxX Linker yields
  per-lipid cross-reference identifier columns and you must quantify coverage against
  LION, KEGG, ChEBI, Rhea, and UniProtKB, or find which lipid classes fail to map.
when_to_use_negative:
- Input is raw lipid annotation text that has not yet been run through the Linker
  — use the Converter/Equalizer modules or a prior Linker translation step first
- The lipids of interest are plant or bacterial lipids, which are outside LipidLynxX's
  current mammalian coverage of 25 classes in five categories
- The goal is format normalization of annotations (unified LipidLynxX IDs or cross-level
  equalization) rather than mapping to external integration databases
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3512
tools:
- name: LipidLynxX
  role: Linker module produces the per-lipid cross-reference identifier translations
    to LION, KEGG, ChEBI, Rhea, and UniProtKB that are parsed here
  repo: https://github.com/SysMedOs/LipidLynxX
- name: pandas
  role: Builds the binary presence/absence coverage matrix and aggregates unmatched
    identifiers by lipid class and category
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2020.04.09.033894
    title: 'LipidLynxX: a data transfer hub to support integration of large scale
      lipidomics datasets'
  source_content_hash: sha256:3ae87962539d95b396f77042109b8d64b0f0ba1e38a38fb62a3271c6c61f0fc3
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
  - pandas
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# linker-translation-output-parsing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Parse LipidLynxX Linker translation results for a lipid panel into a binary per-lipid identifier coverage matrix across the five data-integration targets (LION, KEGG, ChEBI, Rhea, UniProtKB), then aggregate unmatched identifiers by lipid class and category to reveal systematically unmapped classes per database.

## When to use

Use when a run of lipid abbreviations through the LipidLynxX Linker yields per-lipid cross-reference identifier columns and you must quantify coverage against LION, KEGG, ChEBI, Rhea, and UniProtKB, or find which lipid classes fail to map.

## When NOT to use

- Input is raw lipid annotation text that has not yet been run through the Linker — use the Converter/Equalizer modules or a prior Linker translation step first
- The lipids of interest are plant or bacterial lipids, which are outside LipidLynxX's current mammalian coverage of 25 classes in five categories
- The goal is format normalization of annotations (unified LipidLynxX IDs or cross-level equalization) rather than mapping to external integration databases

## Inputs

- Per-lipid LipidLynxX Linker translation results table with cross-reference identifier columns (LION, KEGG, ChEBI, Rhea, UniProtKB)
- Lipid abbreviation list (e.g. PC 16:0_18:2 shorthand or defined aliases like DHA, PAPE, PLPC) for the ~131 test lipids with class/category annotations

## Outputs

- linker_coverage_table.csv: binary 131-row × 5-column (LION, KEGG, ChEBI, Rhea, UniProtKB) presence/absence coverage matrix
- Per-database unmatched-class summary aggregating NaN identifiers by lipid class and category

## How to apply

Load the per-lipid Linker translation output produced by the LipidLynxX Linker module (the module that links lipid abbreviations to external resources) and parse the cross-reference identifier fields for LION, KEGG, ChEBI, Rhea, and UniProtKB. Build a 131-lipid × 5-database binary presence/absence matrix in pandas, treating NaN/empty cross-reference cells as absence (0) and non-empty IDs as presence (1). Aggregate the unmatched (NaN) identifiers by lipid class and category — fatty acids, glycerolipids, glycerophospholipids, sphingolipids, sterols — the five categories LipidLynxX covers, to detect systematically unmatched classes per database. Write the coverage matrix and per-database unmatched-class summary to linker_coverage_table.csv. Rationale: the Linker is LipidLynxX's integration layer whose purpose is to transfer experimental lipid annotations into systems-biology resources, so its per-database hit/miss pattern directly exposes coverage gaps in downstream integration targets.

## Related tools

- **LipidLynxX** (Linker module produces the per-lipid cross-reference identifier translations to LION, KEGG, ChEBI, Rhea, and UniProtKB that are parsed here) — https://github.com/SysMedOs/LipidLynxX
- **pandas** (Builds the binary presence/absence coverage matrix and aggregates unmatched identifiers by lipid class and category)

## Examples

```
import pandas as pd; m = pd.read_csv('linker_results.csv'); cov = m[['LION','KEGG','ChEBI','Rhea','UniProtKB']].notna().astype(int); cov.to_csv('linker_coverage_table.csv')
```

## Evaluation signals

- Output coverage matrix contains exactly 131 lipid rows and five database coverage columns
- Per-class unmatched summary is internally consistent with the per-lipid binary matrix (row-wise aggregation reconciles)
- Reported coverage counts reproduce the reference result: 98/131 lipids mapped to LION, 49/131 to ChEBI, and 130/131 to KEGG compound IDs
- All NaN cells are consistently encoded as 0 and non-empty identifier strings as 1 (no partial/ambiguous encodings)
- Every lipid row is assigned to one of the five supported categories (fatty acids, glycerolipids, glycerophospholipids, sphingolipids, sterols)

## Limitations

- LipidLynxX currently covers only mammalian lipids from 25 classes across five categories; plant and bacterial lipids are not covered, so unmapped classes outside these categories may reflect scope rather than database gaps
- Known Linker runtime issues: if Linker runs more than 300s or more than 30s per ID, the tool should be restarted and the run retried; max 3 tasks at a time in the Windows build
- The Linker module must access external databases over the network, so firewall permissions are required
- Coverage for Rhea and UniProtKB is not reported in the reference run — treat those columns as potentially sparse and verify emptiness before interpreting as true absence

## Evidence

- [results] Reference coverage counts: [evidence span withheld — anchor status: not_found]
- [readme] Linker runtime limitation: [evidence span withheld — anchor status: external_doc]
