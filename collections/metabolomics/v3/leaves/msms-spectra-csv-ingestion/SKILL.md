---
name: msms-spectra-csv-ingestion
description: Use when you have four comma-separated CSV peak lists of MS/MS spectra
  — [M+H]+ and [M-H]- at 35% and 130% NCE, with m/z in column 1 and intensity in column
  2 — and need spectra embeddings for downstream structure decoding.
when_to_use_negative:
- Input spectra come from a single collision energy or a single polarity only, so
  the four-file (pos/neg, low/high NCE) interface cannot be filled
- Peak lists are in a vendor or open instrument format (e.g. mzML) rather than the
  required two-column m/z,intensity CSV layout
- The goal is decoding embeddings into SMILES structures rather than producing the
  embeddings themselves
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PyTorch
  role: Implements the Spec2Mol spectra encoder that computes the embeddings from
    the input CSV peak lists
  repo: github.com/KavrakiLab/Spec2Mol
- name: conda
  role: Creates and configures the spec2mol environment (Python 3.7, rdkit, pytorch=1.6.0,
    torchvision) needed to run predict_embs.py
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42004-023-00932-3
    title: An end-to-end deep learning framework for translating mass spectra to de-novo
      molecules
  source_content_hash: sha256:38aad5a634565d3667c5175fee1625e53d2568e8de4f32db2548aff6aa512778
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PyTorch
  - conda
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# msms-spectra-csv-ingestion

## Summary

Ingest four MS/MS peak-list CSV files (positive/negative mode, low/high collision energy) and encode them into spectra embeddings using the Spec2Mol spectra encoder trained on the NIST Tandem Mass Spectral Library 2020. The resulting embeddings are the decoder-facing representation of the input spectra.

## When to use

Use when you have four comma-separated CSV peak lists of MS/MS spectra — [M+H]+ and [M-H]- at 35% and 130% NCE, with m/z in column 1 and intensity in column 2 — and need spectra embeddings for downstream structure decoding.

## When NOT to use

- Input spectra come from a single collision energy or a single polarity only, so the four-file (pos/neg, low/high NCE) interface cannot be filled
- Peak lists are in a vendor or open instrument format (e.g. mzML) rather than the required two-column m/z,intensity CSV layout
- The goal is decoding embeddings into SMILES structures rather than producing the embeddings themselves

## Inputs

- Positive-mode low-energy peak list CSV: precursor [M+H]+, energy 35% NCE (m/z in column 1, intensity in column 2, comma-separated)
- Positive-mode high-energy peak list CSV: precursor [M+H]+, energy 130% NCE
- Negative-mode low-energy peak list CSV: precursor [M-H]-, energy 35% NCE
- Negative-mode high-energy peak list CSV: precursor [M-H]-, energy 130% NCE

## Outputs

- Spectra embeddings computed for all four input spectra sets, suitable as decoder input for SMILES reconstruction

## How to apply

Create and activate a conda environment named spec2mol with Python 3.7, RDKit, and PyTorch 1.6.0 plus torchvision (per the README installation commands). Format the four input spectra as CSV files with m/z values in the first column and intensity values in the second column, comma-separated. Run predict_embs.py, passing the files via -pos_low_file ([M+H]+, 35% NCE), -pos_high_file ([M+H]+, 130% NCE), -neg_low_file ([M-H]-, 35% NCE), and -neg_high_file ([M-H]-, 130% NCE). The spectra encoder — trained on the NIST Tandem Mass Spectral Library 2020 — creates an embedding from each given set of MS/MS spectra, which serves as input to the Spec2Mol decoder.

## Related tools

- **PyTorch** (Implements the Spec2Mol spectra encoder that computes the embeddings from the input CSV peak lists) — github.com/KavrakiLab/Spec2Mol
- **conda** (Creates and configures the spec2mol environment (Python 3.7, rdkit, pytorch=1.6.0, torchvision) needed to run predict_embs.py)

## Examples

```
python predict_embs.py -pos_low_file 'sample_data/[M+H]_low.csv' -pos_high_file 'sample_data/[M+H]_high.csv' -neg_low_file 'sample_data/[M-H]_low.csv' -neg_high_file 'sample_data/[M-H]_high.csv'
```

## Evaluation signals

- Embedding outputs are produced for all four input spectra sets (positive low/high energy, negative low/high energy)
- Input CSVs conform to the expected layout: m/z values in the first column, intensity values in the second, columns separated by commas
- The conda environment resolves with the pinned versions (python=3.7, pytorch=1.6.0, torchvision, rdkit)
- predict_embs.py completes without errors on the sample_data files, indicating correct argument wiring for -pos_low_file, -pos_high_file, -neg_low_file, -neg_high_file

## Limitations

- The spectra encoder has been trained on the NIST Tandem Mass Spectral Library 2020, which is a commercial dataset; applicability to spectra far outside that library's coverage may be limited
- Requires exactly four paired spectra files per analyte with energies specified as 35% and 130% NCE (Normalized Collision Energy); other energy conventions are not described in the README

## Evidence

- [readme] CSV format: "Each csv file has the m/z values in the first column and the intensity values in the second column.
The columns are separated with commas."
- [readme] energy parameters: "`pos_low_file`: precursor [M+H]+, energy 35% NCE (Normalized Collision Energy)

`pos_high_file`: precursor [M+H]+, energy 130\% NCE"
- [intro] encoder role: "The endoder creates an embedding from a given set of MS/MS spectra. The decoder reconstructs the molecular structure, in a SMILES format, given the embedding that the encoder generates."
- [methods] training dataset: "The spectra encoder has been trained on the NIST Tandem Mass Spectral Library 2020"
- [readme] environment setup: "conda create -n spec2mol python=3.7
source activate spec2mol
conda install rdkit -c rdkit
conda install pytorch=1.6.0 torchvision -c pytorch"
