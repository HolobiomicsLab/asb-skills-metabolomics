---
name: lipid-identifier-cross-matching
description: Use when two or more lipidomics annotation lists report lipids at different
  structural annotation levels (e.g. PC 16:0/18:2 vs PC 16:0_18:2 vs PC 34:2) and
  they must be compiled or compared at a shared level.
when_to_use_negative:
- Annotations are already in one uniform notation and at one level — cross-matching
  adds no information and Equalizer truncation could only discard detail
- Lipids are outside LipidLynxX's covered scope — the paper limits coverage to mammalian
  lipids from 25 classes across fatty acids, glycerolipids, glycerophospholipids,
  sphingolipids, and sterols; plant and bacterial lipids are not supported
- The goal is format conversion between naming styles rather than level comparison
  — use the LipidLynxX Converter module instead of the Equalizer
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3353
- http://edamontology.org/topic_0154
tools:
- name: LipidLynxX
  role: Parses heterogeneous lipid abbreviations, applies the Equalizer annotation-level
    reduction matrix to truncate identifiers to a common level, and cross-matches
    them across datasets; usable via GUI, API, CLI (cli_lynx.py), or as a Python module
  repo: https://github.com/SysMedOs/LipidLynxX
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2020.04.09.033894
    title: 'LipidLynxX: a data transfer hub to support integration of large scale
      lipidomics datasets'
  source_content_hash: sha256:fe1917fac902fa2749d7592a69631e0ef710a09cc42f01517774f45f590fb224
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3353
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# lipid-identifier-cross-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs the LipidLynxX Equalizer workflow: heterogeneous lipid identifiers (species, molecular species, sn-position, DB-position, full-structure) from multiple input styles are parsed, truncated to a common annotation level via the Equalizer annotation matrix, and cross-matched across datasets for meta-analysis.

## When to use

Use when two or more lipidomics annotation lists report lipids at different structural annotation levels (e.g. PC 16:0/18:2 vs PC 16:0_18:2 vs PC 34:2) and they must be compiled or compared at a shared level.

## When NOT to use

- Annotations are already in one uniform notation and at one level — cross-matching adds no information and Equalizer truncation could only discard detail
- Lipids are outside LipidLynxX's covered scope — the paper limits coverage to mammalian lipids from 25 classes across fatty acids, glycerolipids, glycerophospholipids, sphingolipids, and sterols; plant and bacterial lipids are not supported
- The goal is format conversion between naming styles rather than level comparison — use the LipidLynxX Converter module instead of the Equalizer

## Inputs

- CSV or text lipid annotation lists from different sources, at deliberately mixed annotation levels (species, molecular species, sn-position, DB-position, full structure)
- Lipid abbreviations in supported styles (space shorthand 'PC 16:0_18:2', bracket 'PC(16:0_18:2)', database/program notations, defined aliases like DHA or PLPC)

## Outputs

- equalizer_crossmatch.csv with columns for original annotation, equalized annotation, assigned level, and matched partner
- Per-annotation parsed structural components (headgroup, fatty acyl chains, double bonds, sn positions) and the common output level used

## How to apply

First parse each lipid abbreviation with LipidLynxX to decompose identifiers into structural components (headgroup, fatty acyl chains, double-bond and sn-position details); LipidLynxX accepts 5 databases (HMDB, LIPID MAPS, LipidHome, RefMet, SwissLipids) and 17 programs (e.g. MS-DIAL, LipidSearch, MZmine2) plus shorthand and bracket notation. Then apply the Equalizer annotation-level reduction matrix to truncate every parsed annotation to a single user-defined common output level, or use automatic 'Best match' when a level is not pre-specified. Because structural detail cannot be recovered once truncated, choose the coarser of the two datasets' levels to avoid information loss. Cross-match the equalized identifiers between datasets on the shared level, recording coinciding entries as matched pairs, and write original annotation, equalized annotation, assigned level, and matched partner to a single output table. Validation requires that every input annotation resolves to exactly one entry at the common level and that all expected cross-dataset identifier pairs are recovered.

## Related tools

- **LipidLynxX** (Parses heterogeneous lipid abbreviations, applies the Equalizer annotation-level reduction matrix to truncate identifiers to a common level, and cross-matches them across datasets; usable via GUI, API, CLI (cli_lynx.py), or as a Python module) — https://github.com/SysMedOs/LipidLynxX

## Examples

```
python cli_lynx.py  # or: run examples_notebook.ipynb / converter_notebook.ipynb; equalize two annotation CSVs at a chosen level and export equalizer_crossmatch.csv
```

## Evaluation signals

- Every input annotation resolves to exactly one entry at the chosen common annotation level (no nulls or ambiguous multiple matches)
- All expected cross-dataset identifier pairs are recovered in the output match table (row-level expert_review of the mapping)
- The output table contains all required fields: original annotation, equalized annotation, assigned level, and matched partner
- Round-trip check: truncating a more-detailed annotation (e.g. sn-position level) yields the expected coarser form (e.g. PC 34:2) consistent with the reduction matrix
- Script runs end-to-end without errors on both input lists (script_runs check)

## Limitations

- Coverage is limited to mammalian lipids from 25 classes (fatty acids, glycerolipids, glycerophospholipids, sphingolipids, sterols); plant and bacterial lipids are not currently covered
- Different annotation levels between experimental datasets and integration targets (e.g. genome-scale metabolic networks, GSMN) remain a challenge for downstream linking
- If a source database or program is not in the supported list, conversion may require a custom configuration file in lynx/configurations/rules/input
- No specific version, build, or container image is pinned in the source discussion; code is still changing rapidly so API definitions may lag

## Evidence

- [other] Equalizer equalizes heterogeneous lipid identifiers to a common structural annotation level: [evidence span withheld — anchor status: not_found]
