---
name: murcko-scaffold-extraction
description: 'Use when a de novo MS/MS generation pipeline requires per-molecule scaffold
  identities: building an oracle scaffold lookup table, constructing the scaffold
  candidate set for contrastive retrieval, or checking scaffold coverage.'
when_to_use_negative:
- When the scaffold of the query spectrum should be predicted rather than provided—use
  the contrastive spectrum-scaffold retrieval ranker instead of the lookup-table oracle
- When the input is raw vendor or mzML spectral data with no associated canonical
  SMILES, since scaffold extraction operates on the molecular graph derived from SMILES
- When disconnected scaffolds are acceptable outputs—MADGEN's scaffold-coverage check
  requires valid, non-disconnected scaffolds
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3474
tools:
- name: RDKit
  role: Parse canonical SMILES into molecular graphs and extract Murcko scaffolds
    via its scaffold utilities
  repo: https://github.com/HassounLab/MADGEN
- name: wget
  role: Download the processed dataset pickle from Zenodo record 15036069
  repo: https://zenodo.org/records/15036069
- name: conda
  role: Create the pinned MADGEN environment (python=3.9, rdkit=2023.09.5) in which
    scaffold extraction is run
  repo: https://github.com/HassounLab/MADGEN
- name: MADGEN
  role: Consumes the scaffold lookup table as the oracle retriever and the candidate
    scaffold set for contrastive retrieval
  repo: https://github.com/HassounLab/MADGEN
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_content_hash: sha256:360d8a47c1e8d7628b51fac775515cbb6fefefe8a9e3ff83b4373fedb25e43a9
  source_papers:
  - doi: 10.48550/arxiv.2501.01950
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HassounLab/MADGEN
  tools:
  - AdamW
  - MADGEN
  - Python
  - RDKit
  - conda
  - wget
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# murcko-scaffold-extraction

## Summary

Extract the Murcko scaffold from each molecule's canonical SMILES via RDKit and assemble a lookup table keyed by spectrum and chemical formula, providing the correct-scaffold oracle used in MADGEN's oracle retriever setting and the candidate scaffold pool for contrastive retrieval.

## When to use

Use when a de novo MS/MS generation pipeline requires per-molecule scaffold identities: building an oracle scaffold lookup table, constructing the scaffold candidate set for contrastive retrieval, or checking scaffold coverage.

## When NOT to use

- When the scaffold of the query spectrum should be predicted rather than provided—use the contrastive spectrum-scaffold retrieval ranker instead of the lookup-table oracle
- When the input is raw vendor or mzML spectral data with no associated canonical SMILES, since scaffold extraction operates on the molecular graph derived from SMILES
- When disconnected scaffolds are acceptable outputs—MADGEN's scaffold-coverage check requires valid, non-disconnected scaffolds

## Inputs

- Processed dataset pickle (msgym.pkl or canopus.pkl) from Zenodo record 15036069 containing canonical SMILES per molecule
- Per-molecule spectrum identifiers and chemical formulas used as lookup-table keys

## Outputs

- scaffold_lookup_table.pkl mapping each molecule (keyed by spectrum and chemical formula) to its Murcko scaffold
- Scaffold candidate set for retrieval (target molecules removed), used to train the contrastive spectrum-scaffold ranker

## How to apply

Download the preprocessed dataset pickle (e.g. msgym.pkl or canopus.pkl) from Zenodo record 15036069 with wget, then load it and parse each molecule's canonical SMILES into an RDKit molecular graph. Extract each molecule's scaffold from the graph using RDKit's Murcko scaffold utilities (pin rdkit=2023.09.5 as in the MADGEN conda environment, python=3.9). Assemble a lookup table mapping each molecule entry—keyed by its spectrum and chemical formula—to the extracted scaffold, and serialize it as scaffold_lookup_table.pkl for the oracle retriever; for retrieval training, build the candidate scaffold set with target molecules removed from the pool. Validate that every scaffold is a matching, non-disconnected scaffold, consistent with MADGEN's scaffold-coverage criterion, and keep scaffolds disconnected-fragment-free so generation starts from a valid core.

## Related tools

- **RDKit** (Parse canonical SMILES into molecular graphs and extract Murcko scaffolds via its scaffold utilities) — https://github.com/HassounLab/MADGEN
- **wget** (Download the processed dataset pickle from Zenodo record 15036069) — https://zenodo.org/records/15036069
- **conda** (Create the pinned MADGEN environment (python=3.9, rdkit=2023.09.5) in which scaffold extraction is run) — https://github.com/HassounLab/MADGEN
- **MADGEN** (Consumes the scaffold lookup table as the oracle retriever and the candidate scaffold set for contrastive retrieval) — https://github.com/HassounLab/MADGEN

## Examples

```
wget https://zenodo.org/records/15036069/files/msgym.pkl?download=1 -O ./data/msgym/raw/msgym.pkl
```

## Evaluation signals

- Querying scaffold_lookup_table.pkl with each molecule's spectrum and chemical formula returns the correct (matching) scaffold
- Every extracted scaffold is non-disconnected, matching MADGEN's scaffold-coverage criterion
- Scaffold candidates derived from target molecules are excluded from the retrieval candidate pool
- RDKit reports version 2023.09.5 inside the madgen conda environment (python -c "import rdkit; print(rdkit.__version__)")

## Limitations

- Scaffolds must be valid non-disconnected structures; molecules yielding disconnected scaffolds fall outside the scaffold-coverage criterion
- The NIST23 dataset is commercially licensed and not downloadable from the Zenodo record, so the lookup table can only be built directly for CANOPUS and MassSpecGym from the shared pickles
- The oracle lookup table assumes the correct scaffold is known; it does not measure retriever accuracy (that is assessed via scaffold prediction accuracy, SPA, with the predictive retriever)

## Evidence
