---
name: lipid-annotation-level-equalization
description: Use when two or more lipidomics annotation lists report lipids at mixed
  structural levels (e.g. PC 38:4 vs PC 16:0_20:4 vs PC(16:0/20:4)) and you need to
  cross-match or merge them at a shared level.
when_to_use_negative:
- Input lipids are plant or bacterial lipids — LipidLynxX currently covers mammalian
  lipids from 25 classes across fatty acids, glycerolipids, glycerophospholipids,
  sphingolipids, and sterols
- The goal is identifier conversion to LIPID MAPS, SwissLipids, or PubChem accessions
  rather than cross-level cross-matching — use the LipidLynxX Converter module instead
- Annotations are already reported at a single uniform structural level in both datasets,
  so no level equalization is needed before comparison
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: LipidLynxX
  role: Equalizer module parses lipid shorthand from 26 source styles, decomposes
    identifiers into structural components, and applies the annotation matrix to equalize
    and cross-match at a selected level
  repo: https://github.com/SysMedOs/LipidLynxX
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1101/2020.04.09.033894
    title: 'LipidLynxX: a data transfer hub to support integration of large scale
      lipidomics datasets'
  source_content_hash: sha256:207f94e813ae7ecd3b9d7772e5c84d17771a1e23b43071884aadeb65704d1904
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
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# lipid-annotation-level-equalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

LipidLynxX Equalizer reduces heterogeneous lipid identifiers (species, molecular species, sn-position, DB-position, full-structure) to a single common structural annotation level via an annotation-level matrix, so lipidomics datasets annotated at different levels can be cross-compared and compiled for meta-analysis.

## When to use

Use when two or more lipidomics annotation lists report lipids at mixed structural levels (e.g. PC 38:4 vs PC 16:0_20:4 vs PC(16:0/20:4)) and you need to cross-match or merge them at a shared level.

## When NOT to use

- Input lipids are plant or bacterial lipids — LipidLynxX currently covers mammalian lipids from 25 classes across fatty acids, glycerolipids, glycerophospholipids, sphingolipids, and sterols
- The goal is identifier conversion to LIPID MAPS, SwissLipids, or PubChem accessions rather than cross-level cross-matching — use the LipidLynxX Converter module instead
- Annotations are already reported at a single uniform structural level in both datasets, so no level equalization is needed before comparison

## Inputs

- Lipidomics annotation lists (CSV) containing lipid shorthand names at deliberately mixed levels (species, molecular species, sn-position, DB-position, full structure)
- LipidLynxX Equalizer annotation-level reduction matrix (covering unmodified and modified lipids)
- User-specified target annotation level or 'Best match' mode

## Outputs

- equalizer_crossmatch.csv with columns: original annotation, equalized annotation, assigned level, matched partner
- Per-dataset list of equalized LipidLynxX identifiers at the common level

## How to apply

Parse each annotation string with LipidLynxX to decompose the identifier into headgroup, fatty acyl chains, double-bond and sn-position components; the tool supports unmodified and modified lipids. Apply the Equalizer annotation matrix to truncate every parsed identifier to one pre-defined output level — either a user-selected level or the automatic 'Best match' based on shared levels across the compared datasets. Equalize each dataset independently, then cross-match the equalized identifiers of dataset A against dataset B to record coinciding entries. Write originals, equalized annotations, assigned levels, and matched partners to a single table (e.g. equalizer_crossmatch.csv). Rationale: the paper identifies unequal annotation levels as the core obstacle to cross-dataset lipidomics comparison, and the Equalizer is the module designed to close that gap.

## Related tools

- **LipidLynxX** (Equalizer module parses lipid shorthand from 26 source styles, decomposes identifiers into structural components, and applies the annotation matrix to equalize and cross-match at a selected level) — https://github.com/SysMedOs/LipidLynxX

## Examples

```
python cli_lynx.py
```

## Evaluation signals

- Every input annotation resolves to exactly one entry at the chosen common annotation level (no unresolved or ambiguous equalizations)
- All expected cross-dataset identifier pairs are recovered as matched partners in equalizer_crossmatch.csv
- Row-level expert review confirms the original→equalized→level mapping is biologically correct
- Field-presence check: output table contains all four required columns (original annotation, equalized annotation, assigned level, matched partner)
- Script runs end-to-end on the synthetic mixed-level input lists without conversion failures

## Limitations

- Plant and bacterial lipids are not currently covered; scope is mammalian lipids from 25 classes in five categories
- Matching annotation levels between experimental lipidomics datasets and genome-scale metabolic network models (GSMN) remains a challenge even after equalization
- No specific version, build, or container image is cited in the discussion; the README notes the source code is changing rapidly and API definitions may lag
- README known issues: in the packaged Windows exe, IDs not converted should be retried via the Binder/source-code demo; run max 3 tasks simultaneously and restart if the Linker runs over 300s (30s per ID)

## Evidence

- [readme] Cross compare different level of LipidLynxX ID on selected level: [evidence span withheld — anchor status: external_doc]
- [readme] Suitable for both unmodified lipids and modified lipids: [evidence span withheld — anchor status: external_doc]
