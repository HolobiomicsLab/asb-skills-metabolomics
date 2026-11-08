---
name: inchi-generation-from-structures
description: Use when raw natural-product structure tables (e.g. smiles.tsv.gz, nominal.tsv.gz)
  hold SMILES identifiers or compound names, and a standardized InChI column is required
  for downstream harmonization or database upload.
when_to_use_negative:
- Input records already carry validated InChI identifiers and only need loading —
  re-running sanitization on correct salts, charged or dimeric molecules will cause
  an unwanted 'sanitization' step that may degrade them
- The goal is structure-organism pair filtering or reference validation (the filter_dirty.R
  stage), not chemical structure standardization
- Names must be mapped to organism taxonomy rather than translated to structures
edam_topics:
- http://edamontology.org/topic_2250
- http://edamontology.org/topic_3314
tools:
- name: RDKit
  role: Parses SMILES strings from smiles.tsv.gz into molecular objects and generates
    the standardized InChI representation for each sanitized structure
- name: MolVS
  role: Sanitizes each molecule in three main stages (standardizing, fragment removal,
    and uncharging) before InChI generation
- name: OPSIN
  role: Translates chemical names to InChI when no structural identifier (InChI or
    SMILES) was available
- name: sanitizing.py
  role: Script in the 3_cleaningAndEnriching stage of lotus-processor that implements
    the MolVS standardizing/fragment-removal/uncharging workflow
  repo: https://github.com/lotusnprod/lotus-processor
- name: smiles.py
  role: lotus-processor script handling SMILES-based structure processing feeding
    InChI generation
  repo: https://github.com/lotusnprod/lotus-processor
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.7554/elife.70780
    title: The LOTUS initiative for open knowledge management in natural products
      research
  source_content_hash: sha256:0dd682bde4d85ef38ba7e6fc2838336abe995f920acbc91a7ef708804f8035a2
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
  - http://edamontology.org/topic_2250
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# inchi-generation-from-structures

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Standardizes heterogeneous natural-product chemical structure inputs (SMILES strings and chemical names) into InChI identifiers using RDKit parsing, MolVS sanitization, and OPSIN name-to-structure translation, as implemented in the LOTUS lotus-processor pipeline.

## When to use

Use when raw natural-product structure tables (e.g. smiles.tsv.gz, nominal.tsv.gz) hold SMILES identifiers or compound names, and a standardized InChI column is required for downstream harmonization or database upload.

## When NOT to use

- Input records already carry validated InChI identifiers and only need loading — re-running sanitization on correct salts, charged or dimeric molecules will cause an unwanted 'sanitization' step that may degrade them
- The goal is structure-organism pair filtering or reference validation (the filter_dirty.R stage), not chemical structure standardization
- Names must be mapped to organism taxonomy rather than translated to structures

## Inputs

- Raw SMILES structure table (smiles.tsv.gz)
- Raw compound-name table (nominal.tsv.gz)
- Individual SMILES strings or chemical names lacking a structural identifier

## Outputs

- Cleaned structure table interim/tables/2_cleaned/structure/cleaned.tsv.gz (gzipped TSV with an InChI column)
- Per-molecule RDKit sanitized molecular objects
- InChI identifiers derived from chemical names via OPSIN when no SMILES/InChI was present

## How to apply

Load the raw structure tables (smiles.tsv.gz for SMILES and nominal.tsv.gz for compound names) with pandas, then parse SMILES strings into RDKit molecular objects. When no structural identifier (InChI or SMILES) is available, translate the chemical name to InChI with OPSIN, the open-source Java-based name-to-structure solution. Sanitize each molecule via MolVS in three main stages — standardizing, fragment removal, and uncharging — implemented through the 3_cleaningAndEnriching/sanitizing.py script from the lotus-processor pipeline; note that remaining dimers (all structures containing a '.' in their SMILES) are discarded. Generate the standardized InChI for each sanitized structure and write the records, including the InChI column, as the cleaned structure table interim/tables/2_cleaned/structure/cleaned.tsv.gz in gzipped TSV format.

## Related tools

- **RDKit** (Parses SMILES strings from smiles.tsv.gz into molecular objects and generates the standardized InChI representation for each sanitized structure)
- **MolVS** (Sanitizes each molecule in three main stages (standardizing, fragment removal, and uncharging) before InChI generation)
- **OPSIN** (Translates chemical names to InChI when no structural identifier (InChI or SMILES) was available)
- **sanitizing.py** (Script in the 3_cleaningAndEnriching stage of lotus-processor that implements the MolVS standardizing/fragment-removal/uncharging workflow) — https://github.com/lotusnprod/lotus-processor
- **smiles.py** (lotus-processor script handling SMILES-based structure processing feeding InChI generation) — https://github.com/lotusnprod/lotus-processor

## Examples

```
from rdkit import Chem; mol = Chem.MolFromSmiles(smi); inchi = Chem.MolToInchi(mol)  # after MolVS standardize/fragment/uncharge in 3_cleaningAndEnriching/sanitizing.py
```

## Evaluation signals

- interim/tables/2_cleaned/structure/cleaned.tsv.gz exists and is a valid gzipped TSV file
- The cleaned table contains an InChI field for each retained record
- No retained record has a '.' in its SMILES (dimer-discard check per the paper's dimer rule)
- Every sanitized molecule passed through all three MolVS stages (standardizing, fragment removal, uncharging)
- Records that originally had only a chemical name yield an InChI traceable to an OPSIN translation rather than a RDKit SMILES parse

## Limitations

- Correct salts, charged or dimeric molecules in the input data will suffer an unwanted 'sanitization' step that can degrade correct inputs
- The current processing pipeline still removes a lot of correct entries and misses or induces some incorrect ones
- The LOTUS processing pipeline is heavy: it includes many dependencies and is convoluted (R, Python 3, and Java >= 17 are required, plus Make on UNIX systems)
- Name-to-structure translation via OPSIN relies on predefined rules that do not cover all cases and can commonly lead to incorrect translations

## Evidence

- [results] Dimer discard rule: [evidence span withheld — anchor status: not_found]
