---
name: python-module-implementation
description: Use when a task requires translating a published identifier scheme (e.g.
  LipidLynxX modified-lipid IDs) into runnable Python code that maps alias abbreviations
  to structured, multi-level annotations.
when_to_use_negative:
- The lipids of interest are plant or bacterial lipids, which are not currently covered
  by LipidLynxX's mammalian-lipid scope (25 classes across five categories).
- The input is already in unified LipidLynxX identifier form and only needs cross-level
  matching at a selected level — use the Equalizer module instead of re-implementing
  conversion.
- You need accession ID mapping to external databases (LIPID MAPS, SwissLipids) rather
  than alias-to-LipidLynxX-ID conversion.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3353
tools:
- name: LipidLynxX
  role: Source of the controlled-vocabulary and alias JSON configuration files, the
    conversion logic the Python module implements against, and the Converter module
    whose identifier scheme is being reproduced
  repo: https://github.com/SysMedOs/LipidLynxX
- name: Black
  role: Code formatter whose style all LipidLynxX python codes follow; the implemented
    module must conform to it
  repo: https://github.com/psf/black
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2020.04.09.033894
    title: 'LipidLynxX: a data transfer hub to support integration of large scale
      lipidomics datasets'
  source_content_hash: sha256:1d1daae9cfca3c6d0a2c87a2c2132deb668870846f1d147e71518019bb9f263b
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
  - Black
  - LipidLynxX
  - Python
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3353
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# python-module-implementation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implement a scientific data-conversion module in Python, following the Black code style used by LipidLynxX, that encodes modified-lipid (epilipidome) identifiers across five ordered controlled-vocabulary annotation levels and converts alias terms to canonical LipidLynxX identifiers.

## When to use

Use when a task requires translating a published identifier scheme (e.g. LipidLynxX modified-lipid IDs) into runnable Python code that maps alias abbreviations to structured, multi-level annotations.

## When NOT to use

- The lipids of interest are plant or bacterial lipids, which are not currently covered by LipidLynxX's mammalian-lipid scope (25 classes across five categories).
- The input is already in unified LipidLynxX identifier form and only needs cross-level matching at a selected level — use the Equalizer module instead of re-implementing conversion.
- You need accession ID mapping to external databases (LIPID MAPS, SwissLipids) rather than alias-to-LipidLynxX-ID conversion.

## Inputs

- Modified-lipid alias abbreviations (e.g. HETE, PONPC) from databases or programs such as HMDB, LIPID MAPS LMSD & COMP_DB, RefMet, SwissLipids, MS-DIAL, MZmine2
- LipidLynxX JSON configuration files for controlled vocabularies and aliases (lynx/configurations/, including defined_alias.json)

## Outputs

- modified_lipid_conversions.csv — a well-formed CSV with converted entries containing all five annotation-level fields (nominal mass shift, elemental composition, modification type, position, stereochemistry) per lipid

## How to apply

Load the built-in controlled vocabulary and alias configuration shipped as JSON configuration files with the LipidLynxX package (e.g. lynx/configurations/defined_alias.json). Implement the modified-lipid identifier scheme in Python — written in the Black code style used by all LipidLynxX python codes — so each annotation is encoded across five levels: modification nominal mass shift, elemental composition, modification type, position, and stereochemistry. Map input alias terms (e.g. HETE, PONPC) to their canonical entries via the alias configuration, then run the converter on a small demonstration set through the LipidLynxX conversion logic. Retain all five encoded levels per lipid when writing converted annotations so downstream validation can check every level is populated. Because LipidLynxX definitions are still changing rapidly, verify the implementation against the sample data in LipidLynxX/doc/sample_data/input and output before extending it.

## Related tools

- **LipidLynxX** (Source of the controlled-vocabulary and alias JSON configuration files, the conversion logic the Python module implements against, and the Converter module whose identifier scheme is being reproduced) — https://github.com/SysMedOs/LipidLynxX
- **Black** (Code formatter whose style all LipidLynxX python codes follow; the implemented module must conform to it) — https://github.com/psf/black

## Examples

```
python LipidLynxX.py  # or run from source: pip install -r requirements.txt; python cli_lynx.py to convert alias annotations to LipidLynxX IDs
```

## Evaluation signals

- modified_lipid_conversions.csv exists and is a well-formed CSV
- Every converted entry has all five annotation-level fields populated: nominal mass shift, elemental composition, modification type, position, stereochemistry
- Demonstration aliases (e.g. HETE, PONPC) resolve to their canonical entries through the alias configuration without unmatched IDs
- Output conversions match the expected converted annotations in LipidLynxX/doc/sample_data/output for the corresponding sample inputs
- The implemented Python module passes Black formatting (code style Black) consistent with LipidLynxX source conventions

## Limitations

- LipidLynxX currently covers mammalian lipids from 25 classes representing five categories — fatty acids, glycerolipids, glycerophospholipids, sphingolipids, and sterols — not plant or bacterial lipids.
- API and source-code documentation may lag the rapidly changing code; any API-based implementation should be validated against the current repository state.
- If a database or program notation style is not among the 5 databases and 17 programs tested, no input-rules configuration may exist and a new one must be generated with the maintainers.

## Evidence

- [results] LipidLynxX introduces new identifiers for modified lipids based on ordered controlled vocabularies: [evidence span withheld — anchor status: not_found]
- [readme] Since the code is still changing rapidly, the definitions of API and documentations in the source code may not be updated accordingly.: [evidence span withheld — anchor status: external_doc]
