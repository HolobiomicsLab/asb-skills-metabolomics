---
name: adduct-and-instrument-type-onehot-encoding
description: 'Use when preparing the Mol2CCS feature matrix (parquet) from METLIN-CCS
  or CCSBase records: after computing 256-bit Morgan fingerprints, molecular weight,
  and 35-dimensional Drug Tax counts, encode adduct, molecule type, instrument type
  as one-hot vectors and the monomer/dimer association as a.'
when_to_use_negative:
- Running the original SigmaCCS architecture, which consumes only the molecular graph
  and the adduct without additional descriptor vectors
- The molecular structure lacks a defined adduct or instrument type label (e.g. generic
  neutral-molecule properties unrelated to CCS)
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: RDKit (v2023.09.5)
  role: Read SMILES and compute the Morgan fingerprint (256 bits, radius 2), molecular
    weight, and Drug Tax class counts that are concatenated with the one-hot adduct/instrument
    features
  repo: https://www.rdkit.org/
- name: Mol2CCS
  role: Consumes the concatenated one-hot encoded adduct, instrument type, monomer/dimer
    and descriptor features as additional inputs alongside the molecular graph for
    CCS prediction
  repo: https://github.com/enveda/ccs-prediction
- name: SigmaCCS
  role: Base graph neural network architecture that Mol2CCS extends by adding the
    additional one-dimensional feature vectors
  repo: https://github.com/zmzhang/SigmaCCS
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-024-00899-w
    title: Evaluating the generalizability of graph neural networks for predicting
      collision cross section
  source_content_hash: sha256:5c0c9f82414a3769025654f6e8fd9edf09e45231156ccd912fdadd1386df3a35
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Mol2CCS
  - RDKit (v2023.09.5)
  - SigmaCCS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# Adduct and instrument-type one-hot encoding

## Summary

Mol2CCS augments the SigmaCCS graph neural network with seven additional one-dimensional feature vectors per molecule, including one-hot encodings of the adduct form, instrument type, and a binary monomer/dimer association flag, to improve generalizability of CCS predictions across chemical spaces.

## When to use

Use when preparing the Mol2CCS feature matrix (parquet) from METLIN-CCS or CCSBase records: after computing 256-bit Morgan fingerprints, molecular weight, and 35-dimensional Drug Tax counts, encode adduct, molecule type, instrument type as one-hot vectors and the monomer/dimer association as a binary flag.

## When NOT to use

- Running the original SigmaCCS architecture, which consumes only the molecular graph and the adduct without additional descriptor vectors
- The molecular structure lacks a defined adduct or instrument type label (e.g. generic neutral-molecule properties unrelated to CCS)

## Inputs

- SMILES strings with associated adduct annotation (e.g. [M+H]+, [M+Na]+, [M-H]-) and instrument type from METLIN-CCS or CCSBase (v1.3) export (csv/excel)
- RDKit molecule objects generated with RDKit v2023.09.5
- Morgan fingerprint radius 2, 256 bits per molecule
- Molecular weight and 35-dimensional Drug Tax structural class counts per molecule

## Outputs

- Concatenated seven-dimensional-block feature matrix, one row per molecule record (fingerprint + descriptors + one-hot adduct + one-hot instrument type + monomer/dimer binary flag), stored as parquet
- Feature file formatted per the Mol2CCS input schema, including metadata columns (smiles, adduct, ccs, coordinates) consumed by scripts/train-test.py

## How to apply

For each molecule read from SMILES with RDKit (v2023.09.5), concatenate seven one-dimensional feature vectors: (1) 256-bit Morgan fingerprint (radius 2), (2) molecular weight descriptor, (3) 35-dimensional Drug Tax structural class counts, (4) one-hot encoded specific adduct forms, (5) one-hot vector differentiating monomeric vs dimeric adducts, (6) one-hot encoded instrument type, (7) remaining adduct-derived vector. Write the resulting per-record feature matrix to parquet with the metadata columns required as input by Mol2CCS. Rationale: these additional features extend the SigmaCCS graph-and-adduct architecture, partially recovering generalization lost when models trained on CCSBase are evaluated on structurally novel regions such as METLIN-CCS.

## Related tools

- **RDKit (v2023.09.5)** (Read SMILES and compute the Morgan fingerprint (256 bits, radius 2), molecular weight, and Drug Tax class counts that are concatenated with the one-hot adduct/instrument features) — https://www.rdkit.org/
- **Mol2CCS** (Consumes the concatenated one-hot encoded adduct, instrument type, monomer/dimer and descriptor features as additional inputs alongside the molecular graph for CCS prediction) — https://github.com/enveda/ccs-prediction
- **SigmaCCS** (Base graph neural network architecture that Mol2CCS extends by adding the additional one-dimensional feature vectors) — https://github.com/zmzhang/SigmaCCS

## Examples

```
poetry run python scripts/train-test.py --prefix "train-metlin-test-ccsbase" --train-input-file "ccs-prediction/metlin_train_3d.parquet" --test-input-file "ccs-prediction/ccsbase_3d.parquet" --coordinates-column-name "coordinates" --coordinates-present --smiles-column-name "smiles" --adduct-column-name "adduct" --ccs-column-name "ccs" --dropout-rate 0.1 --epochs 400
```

## Evaluation signals

- Feature vector length per record must include the fixed blocks: 256 (fingerprint) + 1 (molecular weight) + 35 (Drug Tax) + len(adduct set) one-hot + len(instrument set) one-hot + 1 (monomer/dimer binary)
- One-hot blocks must sum to 1 exactly across their category positions for each record (or 0 if category missing)
- Monomer/dimer column must contain only {0,1} values
- Output parquet must retain the metadata columns (smiles, adduct, ccs, coordinates) expected by notebooks/data_processing/2_data_splits.ipynb
- Model accuracy on METLIN-CCS when adding these features should show reduced generalization drop relative to SigmaCCS trained without them

## Limitations

- Generalization gains are only partial: performance still declines on structurally novel regions outside the training chemical space
- Instrument-type one-hot assumes instrument labels are present and consistent in the source database (CCSBase combines data from three different instruments)
- Variations in experimental conditions such as instrument types likely introduced inaccuracies in the CCS training values, which these encodings only partially mitigate
- Evaluation was restricted mainly to small molecules and specific regions of chemical space (METLIN-CCS based on synthetic structures)

## Evidence

- [results] Seven one-dimensional features listed: [evidence span withheld — anchor status: not_found]
- [methods] Workflow for the skill: "Each notebook reads the csv/excel and formats it according to the input of Mol2CCS."
- [readme] Data splits format for parquet: "train-input-file is the training set (see notebooks/data_processing/2_data_splits.ipynb for details on the format)"
- [discussion] Generalization motivation: "we demonstrate how an extension of the architecture of GNN (Mol2CCS) including other additional features improves the generalizability of SigmaCCS"
- [discussion] Instrument variations as limitation: "variations in the experimental conditions, such as instrument types, likely introduced some inaccuracies in the CCS values used to train our models"
