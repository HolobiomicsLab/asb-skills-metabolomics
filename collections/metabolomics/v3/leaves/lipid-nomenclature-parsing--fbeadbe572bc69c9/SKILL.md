---
name: lipid-nomenclature-parsing
description: Use when lipid identifiers come from mixed sources (e.g. MS-DIAL, LipidSearch,
  LIPID MAPS, SwissLipids) in inconsistent notation styles and must be decomposed
  and cross-compared at a common structural level.
when_to_use_negative:
- The lipids of interest are plant or bacterial lipids — LipidLynxX currently covers
  mammalian lipids from 25 classes across five categories (fatty acids, glycerolipids,
  glycerophospholipids, sphingolipids, sterols)
- The input identifiers are already uniform canonical LipidLynxX IDs at a single level
  and no cross-level comparison is needed
- You need ontology-structured output rather than abbreviation conversion — an ontology
  mapper (e.g. LION/LipidLION) may be more appropriate for that goal
edam_operation: http://edamontology.org/operation_3434
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: LipidLynxX
  role: Converter module parses heterogeneous lipid abbreviations into uniform LipidLynxX
    IDs; Equalizer module cross-compares them at a selected annotation level
  repo: https://github.com/SysMedOs/LipidLynxX
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2020.04.09.033894
    title: 'LipidLynxX: a data transfer hub to support integration of large scale
      lipidomics datasets'
  source_content_hash: sha256:f87c214744040766788faff713a55ce3ec3b54e5afc98cfcf69e0ce2fd113ba3
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
  edam_operation: http://edamontology.org/operation_3434
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# lipid-nomenclature-parsing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Parse heterogeneous lipid abbreviation styles (space shorthand like 'PC 16:0_18:2', bracket derivatives like 'PC(16:0_18:2)', and aliases like DHA/PLPC) into uniform, decomposed LipidLynxX identifiers so lipidomics identifiers from different tools can be compared and equalized to a shared annotation level.

## When to use

Use when lipid identifiers come from mixed sources (e.g. MS-DIAL, LipidSearch, LIPID MAPS, SwissLipids) in inconsistent notation styles and must be decomposed and cross-compared at a common structural level.

## When NOT to use

- The lipids of interest are plant or bacterial lipids — LipidLynxX currently covers mammalian lipids from 25 classes across five categories (fatty acids, glycerolipids, glycerophospholipids, sphingolipids, sterols)
- The input identifiers are already uniform canonical LipidLynxX IDs at a single level and no cross-level comparison is needed
- You need ontology-structured output rather than abbreviation conversion — an ontology mapper (e.g. LION/LipidLION) may be more appropriate for that goal

## Inputs

- CSV/text lists of lipid abbreviations in mixed annotation levels (species, molecular species, sn-position, DB-position)
- Lipid identifier columns exported from lipid identification software (e.g. MS-DIAL, LipidSearch, Lipostar)
- Optional custom alias definitions (lynx/configurations/defined_alias.json)

## Outputs

- Uniformed LipidLynxX IDs decomposed into headgroup, fatty acyl, double-bond and sn-position components
- Equalized annotations at a single pre-defined structural level
- Cross-match table (e.g. equalizer_crossmatch.csv) with original annotation, equalized annotation, assigned level, and matched partner

## How to apply

Feed each annotation string (CSV list of lipid names) through LipidLynxX's Converter, which parses the identifier into structural components — headgroup, fatty acyl chains, double-bond and sn-position details — and emits a uniformed LipidLynxX ID. The parser is tested against abbreviations from 5 databases (HMDB, LIPID MAPS LMSD & COMP_DB, LipidHome, RefMet, SwissLipids) and 17 programs (e.g. MS-DIAL, LipidSearch, LipidBlast, MZmine2), handling space shorthand, bracket shorthand, and defined aliases configured in lynx/configurations/defined_alias.json. If an ID does not convert, check whether a rule file in lynx/configurations/rules/input fits the source, or report an issue. Once parsed, apply the Equalizer annotation-matrix to truncate all identifiers to one user-defined or 'Best match' output level, which is what makes cross-dataset matching and meta-analysis compilation possible. Output is a table (e.g. equalizer_crossmatch.csv) with original annotation, equalized annotation, assigned level, and matched partner.

## Related tools

- **LipidLynxX** (Converter module parses heterogeneous lipid abbreviations into uniform LipidLynxX IDs; Equalizer module cross-compares them at a selected annotation level) — https://github.com/SysMedOs/LipidLynxX

## Examples

```
python LipidLynxX.py  # start GUI; or run the converter notebook converter_notebook.ipynb to paste lipid abbreviations and download the equalized output table as .csv
```

## Evaluation signals

- Every input annotation resolves to exactly one entry at the chosen common output level (no ambiguous or failed parses)
- All expected cross-dataset identifier pairs are recovered in the cross-match output table
- Row-level expert review confirms the original-to-equalized mapping preserves headgroup and fatty acyl composition
- Parsed IDs conform to LipidLynxX's strictly controlled JSON schema format
- Identifiers from different source notations (space shorthand vs bracket shorthand vs alias) collapse to the same equalized ID when they denote the same lipid

## Limitations

- Plant and bacterial lipids are not currently covered — scope is mammalian lipids from 25 classes in five categories
- The Windows .exe build may leave some IDs unconverted; the mybinder converter_notebook demo tracks the latest source and can resolve these
- API and source-code documentation may lag behind rapid code changes, so developers should follow the repository before relying on the API
- Different annotation levels between experimental datasets and downstream models (e.g. genome-scale metabolic networks) can still be a challenge after equalization

## Evidence

- [readme] Abbreviations such as DHA, PAPE, PLPC, PONPC .etc are also included as `defined alias`: [evidence span withheld — anchor status: external_doc]
