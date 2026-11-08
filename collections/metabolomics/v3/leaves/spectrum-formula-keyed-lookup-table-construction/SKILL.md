---
name: spectrum-formula-keyed-lookup-table-construction
description: 'Use when constructing MADGEN''s oracle retrieval setting: you have processed
  spectra datasets (e.g. msgym.pkl or canopus.pkl) with canonical SMILES and chemical
  formulas and need a per-entry correct-scaffold table keyed by spectrum and formula.'
when_to_use_negative:
- You need a predictive retriever that scores candidate scaffolds from the spectrum
  via contrastive learning — this table is the oracle, not the ranking model
- The dataset already ships a pre-computed retrieval results file (e.g. ranks_msgym_pred.pkl),
  which serves the predictive-retrieval path instead
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: RDKit
  role: Parses canonical SMILES into molecular graphs and extracts the Murcko scaffold
    for each molecule to populate the lookup table
  repo: https://github.com/HassounLab/MADGEN
- name: wget
  role: Downloads the processed dataset pickles (msgym.pkl, canopus.pkl) from Zenodo
    record 15036069
- name: Python
  role: Loads the dataset pickle, iterates entries, builds and serializes the spectrum/formula-keyed
    scaffold table
  repo: https://github.com/HassounLab/MADGEN
provenance:
  source_task_ids:
  - task_006
  source_content_hash: sha256:acc825a0284b9ba1b8896d6e74e33090583dc432877a8ba6350bb295d0cf1753
  source_papers:
  - doi: 10.48550/arxiv.2501.01950
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HassounLab/MADGEN
  tools:
  - MADGEN
  - Python
  - RDKit
  - wget
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# Spectrum- and formula-keyed scaffold lookup table construction

## Summary

Builds the MADGEN oracle-retriever artifact: a serialized lookup table (scaffold_lookup_table.pkl) that maps each MS/MS spectrum–chemical formula pair to the molecule's Murcko scaffold, extracted from canonical SMILES via RDKit. This table supplies the correct scaffold that MADGEN's oracle retriever requires for scaffold-conditioned generation.

## When to use

Use when constructing MADGEN's oracle retrieval setting: you have processed spectra datasets (e.g. msgym.pkl or canopus.pkl) with canonical SMILES and chemical formulas and need a per-entry correct-scaffold table keyed by spectrum and formula.

## When NOT to use

- You need a predictive retriever that scores candidate scaffolds from the spectrum via contrastive learning — this table is the oracle, not the ranking model
- The dataset already ships a pre-computed retrieval results file (e.g. ranks_msgym_pred.pkl), which serves the predictive-retrieval path instead

## Inputs

- Processed dataset pickle (msgym.pkl or canopus.pkl) from Zenodo record 15036069 containing canonical SMILES, MS/MS spectra, molecular properties, and train/validation/test splits
- Per-molecule chemical formula associated with each spectrum

## Outputs

- scaffold_lookup_table.pkl — lookup table keyed by spectrum and chemical formula yielding the correct Murcko scaffold per molecule entry

## How to apply

Download the processed dataset pickle (msgym.pkl or canopus.pkl) from Zenodo record 15036069 with wget, then load it and parse each molecule's canonical SMILES into an RDKit molecular graph. Extract each molecule's Murcko scaffold from the graph representation using RDKit's scaffold utilities. Assemble a lookup table mapping each molecule entry — keyed by its spectrum and chemical formula — to the extracted scaffold, and serialize it as scaffold_lookup_table.pkl. This implements the oracle setting in which the model has direct access to the correct scaffold, in contrast to the predictive retriever that scores candidate scaffolds via contrastive ranking. Validate that every queried entry returns a matching, non-disconnected scaffold.

## Related tools

- **RDKit** (Parses canonical SMILES into molecular graphs and extracts the Murcko scaffold for each molecule to populate the lookup table) — https://github.com/HassounLab/MADGEN
- **wget** (Downloads the processed dataset pickles (msgym.pkl, canopus.pkl) from Zenodo record 15036069)
- **Python** (Loads the dataset pickle, iterates entries, builds and serializes the spectrum/formula-keyed scaffold table) — https://github.com/HassounLab/MADGEN

## Examples

```
wget https://zenodo.org/records/15036069/files/msgym.pkl?download=1 -O ./data/msgym/raw/msgym.pkl
```

## Evaluation signals

- Querying the table with each entry's spectrum and chemical formula returns the correct (matching) scaffold for that molecule
- Every returned scaffold is a valid, non-disconnected scaffold, consistent with MADGEN's scaffold-coverage check (percentage of cases with valid non-disconnected scaffolds)
- Key coverage: every molecule entry in the source pickle (msgym.pkl / canopus.pkl) has a table entry; no spectrum/formula key resolves to a missing value
- Serialization check: scaffold_lookup_table.pkl loads cleanly and preserves the SMILES-derived Murcko scaffold strings

## Limitations

- The oracle table assumes the correct scaffold is knowable per spectrum/formula pair; it does not generalize to novel spectra and is an evaluation upper-bound, not a deployed retriever
- Scaffolds must be non-disconnected to count as valid in MADGEN's scaffold-coverage metric; molecules whose Murcko scaffold extraction yields disconnected fragments fail this check

## Evidence
