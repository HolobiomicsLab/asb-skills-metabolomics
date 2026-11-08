---
name: smiles-structure-sanitization
description: Use when a natural-products or compound table mixes SMILES strings, InChIs,
  and compound names, and you need sanitized structures and standardized InChIs before
  joining or publishing structure-organism pairs.
when_to_use_negative:
- Input structures are already canonical, salt-stripped InChIs or standardized SMILES
  that you must preserve verbatim — the systematic sanitization step can degrade correct
  salts, charged, or dimeric molecules ('correct salts, charged or dimeric molecules
  in the input data will suffer an unwanted sanitization step').
- The goal is name-to-structure translation only, with structures already present
  — OPSIN name translation is only invoked when no InChI or SMILES is available.
- You need structural or spectral search over the cleaned structures — the LOTUS pipeline
  itself delegates indexing/search to Wikidata SPARQL, IDSM, and the Sachem PostgreSQL
  cartridge rather than this cleaning step.
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: MolVS
  role: Performs the three-stage sanitization (standardizing, fragment removal, uncharging)
    on each parsed molecule
- name: RDKit
  role: Parses SMILES strings into molecular objects prior to MolVS sanitization
- name: sanitizing.py
  role: lotus-processor script (3_cleaningAndEnriching) that implements the sanitization
    workflow end-to-end
  repo: https://github.com/lotusnprod/lotus-processor
- name: OPSIN
  role: Open-source Java name-to-structure converter used to translate chemical names
    to InChI when no structural identifier is available
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.7554/elife.70780
    title: The LOTUS initiative for open knowledge management in natural products
      research
  source_content_hash: sha256:a2ed2f74d6c225fbe0d75d0ef5b8c88e79694dc89ee6807d4ff0c7f08776b4ba
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/lotusnprod/lotus-processor
  tools:
  - MolVS
  - OPSIN
  - RDKit
  - sanitizing.py
  - smiles.py
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# smiles-structure-sanitization

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Standardize heterogeneous chemical structure inputs from natural-products tables into sanitized, non-dimeric SMILES with a standardized InChI representation, using RDKit parsing and the MolVS package as implemented in the lotus-processor pipeline.

## When to use

Use when a natural-products or compound table mixes SMILES strings, InChIs, and compound names, and you need sanitized structures and standardized InChIs before joining or publishing structure-organism pairs.

## When NOT to use

- Input structures are already canonical, salt-stripped InChIs or standardized SMILES that you must preserve verbatim — the systematic sanitization step can degrade correct salts, charged, or dimeric molecules ('correct salts, charged or dimeric molecules in the input data will suffer an unwanted sanitization step').
- The goal is name-to-structure translation only, with structures already present — OPSIN name translation is only invoked when no InChI or SMILES is available.
- You need structural or spectral search over the cleaned structures — the LOTUS pipeline itself delegates indexing/search to Wikidata SPARQL, IDSM, and the Sachem PostgreSQL cartridge rather than this cleaning step.

## Inputs

- smiles.tsv.gz — raw table of compound SMILES strings
- nominal.tsv.gz — raw table of compound names (fallback when no InChI/SMILES is available)

## Outputs

- interim/tables/2_cleaned/structure/cleaned.tsv.gz — gzipped TSV of sanitized records with an InChI column
- Sanitized, non-dimeric SMILES per retained compound

## How to apply

Load the raw structure tables (smiles.tsv.gz for SMILES and nominal.tsv.gz for compound names) with pandas. Parse the SMILES strings into RDKit molecular objects and discard dimeric entries — all structures containing a '.' in their SMILES. Sanitize each molecule via MolVS in three main stages: standardizing, fragment removal, and uncharging, as implemented in the 3_cleaningAndEnriching/sanitizing.py script of the lotus-processor pipeline. When no structural identifier (InChI or SMILES) is available, translate chemical names to structures using OPSIN before sanitization. Generate a standardized InChI for each sanitized structure and write the cleaned records as a gzipped TSV (interim/tables/2_cleaned/structure/cleaned.tsv.gz). Rationale: salts, charged, and dimeric molecules in natural-products data must be systematically normalized so that the same metabolite from over 40 electronic resources maps to one structure identifier.

## Related tools

- **MolVS** (Performs the three-stage sanitization (standardizing, fragment removal, uncharging) on each parsed molecule)
- **RDKit** (Parses SMILES strings into molecular objects prior to MolVS sanitization)
- **sanitizing.py** (lotus-processor script (3_cleaningAndEnriching) that implements the sanitization workflow end-to-end) — https://github.com/lotusnprod/lotus-processor
- **OPSIN** (Open-source Java name-to-structure converter used to translate chemical names to InChI when no structural identifier is available)

## Examples

```
python 3_cleaningAndEnriching/sanitizing.py  # runs within lotus-processor: RDKit parse -> MolVS standardize/uncharge -> InChI -> interim/tables/2_cleaned/structure/cleaned.tsv.gz
```

## Evaluation signals

- interim/tables/2_cleaned/structure/cleaned.tsv.gz exists and is in gzipped TSV format
- Every retained record has a non-empty InChI field
- No retained record's SMILES contains a '.' (dimer-discard rule satisfied)
- Spot-check that salts/charged molecules were uncharged and fragments removed by comparing a sample of input vs. cleaned SMILES
- Records whose only input was a compound name are traced back to an OPSIN translation rather than dropped silently

## Limitations

- Systematic sanitization can degrade correct inputs: 'correct salts, charged or dimeric molecules in the input data will suffer an unwanted sanitization step'
- Name-to-structure translation relies on predefined rules that do not cover all cases and 'can commonly lead to incorrect translations'
- The overall LOTUS processing pipeline 'still removes a lot of correct entries and misses or induces some incorrect ones'
- The pipeline is heavy: 'It includes many dependencies and is convoluted' (requires R, Python 3, and Java >= 17, the latter for OPSIN)

## Evidence

- [results] Dimeric structures discarded by dot-in-SMILES rule: [evidence span withheld — anchor status: not_found]
- [results] OPSIN fallback for name-to-InChI: [evidence span withheld — anchor status: not_found]
