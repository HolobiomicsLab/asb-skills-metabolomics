---
name: tandem-ms-spectra-embedding-consumption
description: Use when you have the four input spectra CSV files ([M+H]+ and [M-H]-
  at 35% and 130% NCE) and need to translate the encoder-generated spectra embeddings
  into de-novo SMILES structures.
when_to_use_negative:
- Input is raw MS/MS spectra (m/z and intensity pairs) rather than already-generated
  spectra embeddings from the Spec2Mol encoder — run predict_embs.py first.
- You need database-matching against reference structures rather than de-novo SMILES
  reconstruction; Spec2Mol recommends molecular structures from MS/MS spectra via
  the decoder, not by lookup against a spectral library.
- The target output format is not a SMILES string (e.g. you need an InChI or a fragmentation
  tree); the decoder reconstructs molecular structures in SMILES format.
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: PyTorch
  role: implements the Spec2Mol architecture; used to run the decoder that autoregressively
    generates SMILES token sequences from each spectra embedding
  repo: https://github.com/KavrakiLab/Spec2Mol
- name: RDKit
  role: processing of the chemical data; used to parse and validate each generated
    SMILES string to confirm chemical validity of the reconstructed molecules
  repo: https://www.rdkit.org/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42004-023-00932-3
    title: An end-to-end deep learning framework for translating mass spectra to de-novo
      molecules
  source_content_hash: sha256:14910a8894a606110e9db42ee2fad12de9e13533ded7f8f763de6c1c257cc89a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KavrakiLab/Spec2Mol
  tools:
  - PyTorch
  - RDKit
  - torchvision
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# tandem-ms-spectra-embedding-consumption

## Summary

Consumes tandem MS/MS spectra embeddings produced by the Spec2Mol encoder and autoregressively reconstructs candidate molecular structures as SMILES sequences using a pre-trained decoder, validating each output with RDKit.

## When to use

Use when you have the four input spectra CSV files ([M+H]+ and [M-H]- at 35% and 130% NCE) and need to translate the encoder-generated spectra embeddings into de-novo SMILES structures.

## When NOT to use

- Input is raw MS/MS spectra (m/z and intensity pairs) rather than already-generated spectra embeddings from the Spec2Mol encoder — run predict_embs.py first.
- You need database-matching against reference structures rather than de-novo SMILES reconstruction; Spec2Mol recommends molecular structures from MS/MS spectra via the decoder, not by lookup against a spectral library.
- The target output format is not a SMILES string (e.g. you need an InChI or a fragmentation tree); the decoder reconstructs molecular structures in SMILES format.

## Inputs

- spectra embeddings produced by the Spec2Mol encoder (from predict_embs.py run on four input spectra CSV files: [M+H]+ and [M-H]- at 35% and 130% NCE)
- pre-trained Spec2Mol decoder checkpoint trained on a massive dataset of chemical structures for translating between different molecular representations

## Outputs

- predicted_smiles.csv containing SMILES sequences of the recommended chemical structures

## How to apply

Load the pre-trained Spec2Mol decoder, which was previously trained on a massive corpus of chemical structures for translating between different molecular representations, and initialize it with the spectra embeddings produced by the encoder (via predict_embs.py). Autoregressively generate SMILES token sequences from each spectra embedding, reconstructing candidate molecular structures de-novo. Parse and validate each generated SMILES string with RDKit to confirm chemical validity of the reconstructed molecules. Collect all predicted SMILES sequences and write them to predicted_smiles.csv, confirming each output row contains a syntactically valid SMILES string parseable by RDKit.

## Related tools

- **PyTorch** (implements the Spec2Mol architecture; used to run the decoder that autoregressively generates SMILES token sequences from each spectra embedding) — https://github.com/KavrakiLab/Spec2Mol
- **RDKit** (processing of the chemical data; used to parse and validate each generated SMILES string to confirm chemical validity of the reconstructed molecules) — https://www.rdkit.org/

## Examples

```
conda install rdkit -c rdkit && conda install pytorch=1.6.0 torchvision -c pytorch && python predict_embs.py -pos_low_file 'sample_data/[M+H]_low.csv' -pos_high_file 'sample_data/[M+H]_high.csv' -neg_low_file 'sample_data/[M-H]_low.csv' -neg_high_file 'sample_data/[M-H]_high.csv'
```

## Evaluation signals

- Each output row in predicted_smiles.csv contains a syntactically valid SMILES string parseable by RDKit, since the decoder's objective is to reconstruct valid SMILES representations of the recommended chemical structures.
- Every spectra embedding fed to the decoder yields exactly one SMILES sequence; rows in predicted_smiles.csv correspond one-to-one with the embeddings generated by predict_embs.py from the four input spectra CSV files.
- All generated SMILES strings round-trip through RDKit without parse errors, confirming the decoder is producing chemically valid molecules rather than invalid token sequences.
- The pre-trained decoder checkpoint is loaded and initialized with the encoder-generated spectra embeddings — no re-training of the decoder is performed during this stage.

## Limitations

- The spectra encoder was trained on the NIST Tandem Mass Spectral Library 2020, a commercial dataset; embeddings for compounds outside this domain may produce unreliable SMILES reconstructions.
- The decoder is pre-trained and is not fine-tuned during this stage, so reconstructions depend entirely on the quality of the encoder's spectra embeddings.
- Spec2Mol shows on-par performance compared to existing fragmentation tree methods particularly when test structure information is not available during training or present in the reference database.

## Evidence

- [abstract] decoder pre-trained on a massive dataset of chemical structures reconstructs SMILES sequences of the recommended chemical structures: "The encoder learns the spectra embeddings, while the decoder, pre-trained on a massive dataset of chemical structures for translating between different molecular representations, reconstructs SMILES"
- [readme] decoder reconstructs molecular structure in SMILES format given the embedding from the encoder: "Spec2Mol is an encoder-decoder architecture: The endoder creates an embedding from a given set of MS/MS spectra. The decoder reconstructs the molecular structure, in a SMILES format, given the"
- [readme] concrete invocation for generating spectra embeddings: "python predict_embs.py -pos_low_file 'sample_data/[M+H]_low.csv' -pos_high_file 'sample_data/[M+H]_high.csv' -neg_low_file 'sample_data/[M-H]_low.csv' -neg_high_file 'sample_data/[M-H]_high.csv'"
- [intro] PyTorch implementation and RDKit chemical processing: "The implementation of the Spec2Mol architecture is based on the Pytorch library. Processing of the chemical data is based on the RDKit software."
