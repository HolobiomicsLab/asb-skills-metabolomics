---
name: synthetic-lipidomics-dataset-construction
description: Use when a benchmark or test needs lipid identifier pairs at known annotation
  levels to verify cross-dataset equalization/matching logic, and ground-truth mappings
  must be constructed rather than harvested from real datasets.
when_to_use_negative:
- Input datasets already report all lipid identifiers at a single, uniform annotation
  level — no equalization is needed and the Equalizer matrix would only truncate information
- The analysis targets plant or bacterial lipids — LipidLynxX currently covers mammalian
  lipids from 25 classes across five categories
- The goal is ontology or pathway mapping rather than cross-level identifier matching
  — the Linker module or LION-style mapping tools are the appropriate route
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_0154
tools:
- name: LipidLynxX
  role: Parses synthetic lipid abbreviation lists, equalizes them to a common structural
    annotation level via the Equalizer annotation matrix, and cross-matches the two
    datasets
  repo: https://github.com/SysMedOs/LipidLynxX
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2020.04.09.033894
    title: 'LipidLynxX: a data transfer hub to support integration of large scale
      lipidomics datasets'
  source_content_hash: sha256:8c5d5ddd098f654a3f7b8966fdf052781b0593619506ceb093b99605a97a3118
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
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# synthetic-lipidomics-dataset-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs synthetic lipidomics annotation lists with identifiers deliberately spread across structural levels (species, molecular species, sn-position, DB-position, full structure) so that LipidLynxX Equalizer-based cross-level matching can be exercised and validated end-to-end on controlled ground truth.

## When to use

Use when a benchmark or test needs lipid identifier pairs at known annotation levels to verify cross-dataset equalization/matching logic, and ground-truth mappings must be constructed rather than harvested from real datasets.

## When NOT to use

- Input datasets already report all lipid identifiers at a single, uniform annotation level — no equalization is needed and the Equalizer matrix would only truncate information
- The analysis targets plant or bacterial lipids — LipidLynxX currently covers mammalian lipids from 25 classes across five categories
- The goal is ontology or pathway mapping rather than cross-level identifier matching — the Linker module or LION-style mapping tools are the appropriate route

## Inputs

- Synthetic lipidomics annotation list A (CSV) with identifiers at mixed annotation levels
- Synthetic lipidomics annotation list B (CSV) with identifiers at mixed annotation levels, sharing known overlap with A
- LipidLynxX configuration files (lynx/configurations/, e.g. defined_alias.json) and user-defined target annotation level

## Outputs

- equalizer_crossmatch.csv with columns: original annotation, equalized annotation, assigned level, matched partner
- Per-entry equalization status confirming each input resolves to exactly one entry at the common level

## How to apply

Generate two synthetic lipidomics annotation lists (CSV) containing lipid names from 25 mammalian lipid classes (fatty acids, glycerolipids, glycerophospholipids, sphingolipids, sterols) written in mixed shorthand styles, e.g. 'PC 16:0_18:2' (space shorthand), 'PC(16:0_18:2)' (bracket derivative), and species-level abbreviations, deliberately spanning species, molecular species, sn-position, and DB-position levels, including oxidized/modified lipids. Parse each annotation string with LipidLynxX to decompose it into headgroup, fatty acyl chains, double-bond and sn-position components. Apply the Equalizer annotation-level reduction matrix to truncate every parsed annotation to one common, pre-defined output level (or 'Best match' for automatic level selection). Cross-match equalized identifiers from dataset A against dataset B and write the pairs, together with original annotation, equalized annotation, and assigned level, to equalizer_crossmatch.csv.

## Related tools

- **LipidLynxX** (Parses synthetic lipid abbreviation lists, equalizes them to a common structural annotation level via the Equalizer annotation matrix, and cross-matches the two datasets) — https://github.com/SysMedOs/LipidLynxX

## Examples

```
python cli_lynx.py  # run Equalizer on the two synthetic CSVs and export equalizer_crossmatch.csv; alternatively use examples_notebook.ipynb module calls
```

## Evaluation signals

- Every input annotation resolves to exactly one entry at the chosen common annotation level (no unresolvable or ambiguous equalizations)
- All expected cross-dataset identifier pairs are recovered in equalizer_crossmatch.csv (row-level expert_review of the mapping)
- Output table contains all required fields: original annotation, equalized annotation, assigned level, matched partner (field_present check)
- Script runs to completion without conversion failures on the synthetic input lists (script_runs check)
- Equalized identifiers conform to the unified LipidLynxX ID format with strictly controlled, JSON-schema-validated components

## Limitations

- LipidLynxX currently covers mammalian lipids from 25 classes representing five categories (fatty acids, glycerolipids, glycerophospholipids, sphingolipids, sterols); plant and bacterial lipids are not covered
- Different annotation levels between experimental datasets (e.g. and genome-scale metabolic models) remain a challenge for matching
- Packaging limits: run max 3 tasks at a time in the Windows .exe version; if Linker runs more than 300s or more than 30s per ID, restart the tool

## Evidence

- [readme] LipidLynxX Equalizer: [evidence span withheld — anchor status: external_doc]
- [readme] supported shorthand styles for synthetic inputs: [evidence span withheld — anchor status: not_found]
