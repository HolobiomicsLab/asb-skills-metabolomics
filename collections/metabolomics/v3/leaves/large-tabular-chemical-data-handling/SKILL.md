---
name: large-tabular-chemical-data-handling
description: Use when raw chemical tables (e.g. smiles.tsv.gz with SMILES, nominal.tsv.gz
  with compound names) need harmonization into standardized InChI identifiers before
  joining or publishing structure-organism records.
when_to_use_negative:
- Input is already a validated, InChI-bearing structure table — re-running sanitization
  can degrade correct salts, charged or dimeric molecules through an unwanted 'sanitization'
  step
- Records rely solely on chemical-name strings that are ambiguous or organism-tissue
  names — OPSIN name-to-structure translation relies on predefined rules that do not
  cover all cases and can commonly lead to incorrect translations
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: pandas
  role: Loads the raw smiles.tsv.gz and nominal.tsv.gz tables for row-wise processing
  repo: https://github.com/lotusnprod/lotus-processor
- name: RDKit
  role: Parses SMILES strings into molecular objects; dimeric entries (SMILES containing
    '.') are discarded at this step
  repo: https://github.com/lotusnprod/lotus-processor
- name: MolVS
  role: 'Performs the three-stage sanitization: standardizing, fragment removal, and
    uncharging'
  repo: https://github.com/lotusnprod/lotus-processor
- name: OPSIN
  role: Translates chemical names (when no InChI or SMILES was available) into InChI
    identifiers
- name: sanitizing.py
  role: lotus-processor script (3_cleaningAndEnriching/) implementing the standardizing/fragment-removal/uncharging
    stages
  repo: https://github.com/lotusnprod/lotus-processor
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.7554/elife.70780
    title: The LOTUS initiative for open knowledge management in natural products
      research
  source_content_hash: sha256:ababb6fdf90c46c28a18f172a47111e8ef6e141b42fedf8738403722dceeecd8
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/lotusnprod/lotus-processor
  tools:
  - MolVS
  - OPSIN
  - RDKit
  - pandas
  - sanitizing.py
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# large-tabular-chemical-data-handling

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Standardizes heterogeneous chemical structure inputs from large tabular natural-product datasets (SMILES, InChI, or bare compound names) into sanitized, dimer-free InChI representations, as done in the LOTUS curation pipeline.

## When to use

Use when raw chemical tables (e.g. smiles.tsv.gz with SMILES, nominal.tsv.gz with compound names) need harmonization into standardized InChI identifiers before joining or publishing structure-organism records.

## When NOT to use

- Input is already a validated, InChI-bearing structure table — re-running sanitization can degrade correct salts, charged or dimeric molecules through an unwanted 'sanitization' step
- Records rely solely on chemical-name strings that are ambiguous or organism-tissue names — OPSIN name-to-structure translation relies on predefined rules that do not cover all cases and can commonly lead to incorrect translations

## Inputs

- smiles.tsv.gz — gzipped TSV of raw SMILES strings
- nominal.tsv.gz — gzipped TSV of compound names lacking structural identifiers

## Outputs

- interim/tables/2_cleaned/structure/cleaned.tsv.gz — gzipped TSV of sanitized structures with a standardized InChI column, no remaining dimeric '.' SMILES

## How to apply

Load the raw structure and name tables (smiles.tsv.gz for SMILES, nominal.tsv.gz for names) with pandas. Parse SMILES strings into molecular objects with RDKit and discard dimeric entries — all structures containing a '.' in their SMILES — per the paper's dimer-discard rule. Sanitize each molecule via MolVS in three main stages: standardizing, fragment removal, and uncharging, implemented in the lotus-processor script 3_cleaningAndEnriching/sanitizing.py. When no structural identifier (InChI or SMILES) is available, translate chemical names to InChI with OPSIN, the open-source Java-based name-to-structure solution (Java >= 17 is required by lotus-processor). Generate the standardized InChI per sanitized structure and write the cleaned records to interim/tables/2_cleaned/structure/cleaned.tsv.gz.

## Related tools

- **pandas** (Loads the raw smiles.tsv.gz and nominal.tsv.gz tables for row-wise processing) — https://github.com/lotusnprod/lotus-processor
- **RDKit** (Parses SMILES strings into molecular objects; dimeric entries (SMILES containing '.') are discarded at this step) — https://github.com/lotusnprod/lotus-processor
- **MolVS** (Performs the three-stage sanitization: standardizing, fragment removal, and uncharging) — https://github.com/lotusnprod/lotus-processor
- **OPSIN** (Translates chemical names (when no InChI or SMILES was available) into InChI identifiers)
- **sanitizing.py** (lotus-processor script (3_cleaningAndEnriching/) implementing the standardizing/fragment-removal/uncharging stages) — https://github.com/lotusnprod/lotus-processor

## Examples

```
python 3_cleaningAndEnriching/sanitizing.py
```

## Evaluation signals

- interim/tables/2_cleaned/structure/cleaned.tsv.gz exists and is in gzipped TSV format
- cleaned.tsv.gz contains a populated InChI field for retained records
- No retained record has a '.' in its SMILES, confirming the dimer-discard rule was applied
- Every molecule passed MolVS sanitization in the three stages: standardizing, fragment removal, and uncharging
- Chemical names were converted to InChI only for records lacking a structural identifier (InChI or SMILES)

## Limitations

- Correct salts, charged or dimeric molecules in the input data will suffer an unwanted 'sanitization' step
- Name-to-structure translation via OPSIN relies on sets of predefined rules which do not cover all cases and can commonly lead to incorrect translations
- The LOTUS processing pipeline is heavy, includes many dependencies, and is convoluted; it also requires R, Python 3, and Java >= 17

## Evidence

- [results] dimer discard rule: [evidence span withheld — anchor status: not_found]
- [results] OPSIN name translation: [evidence span withheld — anchor status: not_found]
