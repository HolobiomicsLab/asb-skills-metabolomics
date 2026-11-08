---
name: smiles-sequence-tokenization-for-transformers
description: Use when molecules are given as SMILES strings and a Transformer-based
  retention-time predictor (RT-Transformer) needs sequence inputs for its 1D-Transformer
  branch alongside graph inputs.
when_to_use_negative:
- Retention-time prediction on datasets where only molecular graphs or fingerprints
  are available and no SMILES sequences can be formed
- Predicting retention times for gas-chromatography or non-LC methods outside the
  liquid-chromatography scope RT-Transformer was built for
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: RT-Transformer
  role: 'Target model: tokenized SMILES sequences feed its 1D-Transformer branch while
    the paired molecular graph feeds the graph attention network; the full pipeline
    outputs retention-time predictions.'
  repo: https://github.com/01dadada/RT-Transformer
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae084
    title: 'RT-Transformer: retention time prediction for metabolite annotation to
      assist in metabolite identification'
  source_content_hash: sha256:793f76eb4057b68c04f990ee46dfb0bbb9d451490fbfe9e7c625f280e6c129f2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/01dadada/RT-Transformer
  tools:
  - RT-Transformer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# smiles-sequence-tokenization-for-transformers

## Summary

Converts molecule SMILES strings into tokenized 1D sequences for the RT-Transformer's 1D-Transformer branch, pairing with a molecular-graph representation to predict liquid-chromatography retention times on the 80,038-molecule SMRT dataset.

## When to use

Use when molecules are given as SMILES strings and a Transformer-based retention-time predictor (RT-Transformer) needs sequence inputs for its 1D-Transformer branch alongside graph inputs.

## When NOT to use

- Retention-time prediction on datasets where only molecular graphs or fingerprints are available and no SMILES sequences can be formed
- Predicting retention times for gas-chromatography or non-LC methods outside the liquid-chromatography scope RT-Transformer was built for

## Inputs

- CSV of molecules with SMILES/InChI structures and retention times (e.g. SMRT dataset, 80,038 molecules)

## Outputs

- Tokenized SMILES sequences (1D-Transformer input)
- Per-molecule predicted retention times in seconds
- Test-set MAE and MedAE metrics

## How to apply

Take the SMILES strings from the SMRT small-molecule retention-time dataset (80,038 molecules, with an 'InChI'/'RT' style CSV layout per the README) and tokenize each into the 1D sequence representation that feeds the RT-Transformer's 1D-Transformer branch. Generate the paired molecular graph for the graph attention network branch so both representations are produced from the same molecule, then train or fine-tune the network with PyTorch (torch, torch_geometric, rdkit-pypi). Evaluate on the held-out test split by computing mean absolute error and median absolute error in seconds; the full-data (no samples removed) configuration should land near MAE 33.41 s and MedAE 13.05 s, which is the expected behavior when tokenization and graph construction are correct.

## Related tools

- **RT-Transformer** (Target model: tokenized SMILES sequences feed its 1D-Transformer branch while the paired molecular graph feeds the graph attention network; the full pipeline outputs retention-time predictions.) — https://github.com/01dadada/RT-Transformer

## Examples

```
python ./train.py
```

## Evaluation signals

- Test-set MAE on the all-molecules (no samples removed) SMRT configuration is within a small tolerance of 33.41 s
- Test-set MedAE matches the reported 13.05 s
- Every molecule in the SMRT dataset yields both a tokenized SMILES sequence and a molecular graph representation
- Predicted retention times are in seconds and are finite (no NaN/inf) across the test split

## Limitations

- Different chromatographic conditions produce different retention times for the same metabolite, so a single tokenized-representation model must be transferred/fine-tuned per condition
- Removal of not-retained molecules changes the error (27.30 vs 33.41 average absolute error), so the dataset-filtering choice strongly affects results

## Evidence

- [methods] tokenized SMILES sequence for the 1D-Transformer: [evidence span withheld — anchor status: not_found]
- [results] MAE 33.41 s MedAE 13.05 s: [evidence span withheld — anchor status: not_found]
- [readme] model description: [evidence span withheld — anchor status: external_doc]
- [readme] input format: [evidence span withheld — anchor status: external_doc]
- [readme] motivation: "different chromatographic conditions may result in different retention times for the same metabolite"
