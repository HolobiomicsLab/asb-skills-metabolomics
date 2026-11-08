---
name: collision-energy-spectra-pairing
description: Use when raw tandem MS/MS peak lists (m/z and intensity CSVs) must be
  converted into embeddings for molecular structure recommendation with Spec2Mol,
  and the acquisition provides paired [M+H]+/[M-H]- spectra at low (35% NCE) and high
  (130% NCE) collision energies.
when_to_use_negative:
- The task is de-novo structure prediction itself rather than embedding generation
  — the decoder stage (SMILES reconstruction from embeddings) is a separate step
- Input spectra are not paired across the four acquisition conditions (e.g., only
  a single polarity or single collision energy is available), since predict_embs.py
  expects all four spectra files
- Input is already an embedding or a feature table rather than raw peak lists of m/z
  and intensity values
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: PyTorch
  role: Library on which the Spec2Mol spectra-encoder implementation is based; installed
    via conda install pytorch=1.6.0
  repo: github.com/KavrakiLab/Spec2Mol
- name: conda
  role: Environment management to create the spec2mol environment (Python 3.7) with
    RDKit and PyTorch 1.6.0 plus torchvision
- name: RDKit
  role: Software used for chemical data processing in the Spec2Mol pipeline
  repo: https://www.rdkit.org/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42004-023-00932-3
    title: An end-to-end deep learning framework for translating mass spectra to de-novo
      molecules
  source_content_hash: sha256:56b82e060ddb3098b7000ed133c77dfa339dcdb8d6ab1260dc567fad48c3a21c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PyTorch
  - RDKit
  - conda
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# collision-energy-spectra-pairing

## Summary

Encodes MS/MS spectra into spectra embeddings using the Spec2Mol spectra encoder, consuming the four paired spectra files (positive/negative mode at 35% and 130% NCE) that serve as input to the downstream decoder.

## When to use

Use when raw tandem MS/MS peak lists (m/z and intensity CSVs) must be converted into embeddings for molecular structure recommendation with Spec2Mol, and the acquisition provides paired [M+H]+/[M-H]- spectra at low (35% NCE) and high (130% NCE) collision energies.

## When NOT to use

- The task is de-novo structure prediction itself rather than embedding generation — the decoder stage (SMILES reconstruction from embeddings) is a separate step
- Input spectra are not paired across the four acquisition conditions (e.g., only a single polarity or single collision energy is available), since predict_embs.py expects all four spectra files
- Input is already an embedding or a feature table rather than raw peak lists of m/z and intensity values

## Inputs

- CSV file of [M+H]+ spectra at 35% NCE (m/z in column 1, intensity in column 2, comma-separated)
- CSV file of [M+H]+ spectra at 130% NCE (same format)
- CSV file of [M-H]- spectra at 35% NCE (same format)
- CSV file of [M-H]- spectra at 130% NCE (same format)

## Outputs

- Spectra embeddings computed by the Spec2Mol encoder, one set per input spectra file, to be consumed by the decoder for SMILES reconstruction

## How to apply

Set up a conda environment (spec2mol, Python 3.7, RDKit, PyTorch 1.6.0, torchvision) per the repo instructions. Prepare four CSV files with m/z values in the first column and intensities in the second, comma-separated: positive [M+H]+ at 35% NCE, positive [M+H]+ at 130% NCE, negative [M-H]- at 35% NCE, and negative [M-H]- at 130% NCE. Run predict_embs.py passing these files via -pos_low_file, -pos_high_file, -neg_low_file, and -neg_high_file; the spectra encoder, trained on the NIST Tandem Mass Spectral Library 2020, computes the embeddings. Validate that embedding outputs are produced for all four input spectra sets, since the decoder reconstructs SMILES from this embedding.

## Related tools

- **PyTorch** (Library on which the Spec2Mol spectra-encoder implementation is based; installed via conda install pytorch=1.6.0) — github.com/KavrakiLab/Spec2Mol
- **conda** (Environment management to create the spec2mol environment (Python 3.7) with RDKit and PyTorch 1.6.0 plus torchvision)
- **RDKit** (Software used for chemical data processing in the Spec2Mol pipeline) — https://www.rdkit.org/

## Examples

```
python predict_embs.py -pos_low_file 'sample_data/[M+H]_low.csv' -pos_high_file 'sample_data/[M+H]_high.csv' -neg_low_file 'sample_data/[M-H]_low.csv' -neg_high_file 'sample_data/[M-H]_high.csv'
```

## Evaluation signals

- Embedding outputs are produced for all four input spectra sets (pos low, pos high, neg low, neg high)
- Each input CSV parses correctly: two comma-separated columns with m/z in the first column and intensities in the second
- The script runs inside the spec2mol conda environment with Python 3.7, RDKit, PyTorch 1.6.0, and torchvision installed without import errors
- Downstream decoder accepts the embeddings as input, consistent with 'the encoder's operation is to create an embedding from a given set of MS/MS spectra, which serves as input to the decoder'

## Limitations

- The spectra encoder was trained on the NIST Tandem Mass Spectral Library 2020, a commercial dataset; performance on spectra outside that domain may vary
- Exactly four spectra inputs with specific polarities ([M+H]+, [M-H]-) and NCE values (35%, 130%) are required; other acquisition schemes are not supported by the documented interface
- Embedding generation alone does not identify molecules — structure recommendation requires the decoder stage, and Spec2Mol performs on par with fragmentation tree methods particularly when test structure information is unavailable

## Evidence

- [intro] encoder embeds spectra: "The endoder creates an embedding from a given set of MS/MS spectra. The decoder reconstructs the molecular structure, in a SMILES format, given the embedding that the encoder generates."
- [abstract] encoder learns embeddings, decoder reconstructs SMILES: "The encoder learns the spectra embeddings, while the decoder, pre-trained on a massive dataset of chemical structures for translating between different molecular representations, reconstructs SMILES"
- [readme] predict_embs.py invocation: "python predict_embs.py -pos_low_file 'sample_data/[M+H]_low.csv' -pos_high_file 'sample_data/[M+H]_high.csv' -neg_low_file 'sample_data/[M-H]_low.csv' -neg_high_file 'sample_data/[M-H]_high.csv'"
- [readme] input file format: "Each csv file has the m/z values in the first column and the intensity values in the second column. The columns are separated with commas."
- [methods] training dataset: "The spectra encoder has been trained on the NIST Tandem Mass Spectral Library 2020"
