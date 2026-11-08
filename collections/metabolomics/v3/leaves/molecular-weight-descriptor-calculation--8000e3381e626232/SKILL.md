---
name: molecular-weight-descriptor-calculation
description: Reference for building the seven one-dimensional feature vectors that
  Mol2CCS feeds alongside the 3-D molecular graph, when SMILES, adduct and instrument
  metadata are available per record.
when_to_use_negative:
- The downstream model is vanilla SigmaCCS or GraphCCS, which consumes only the molecular
  graph and adduct — the extra 1-D features are an Mol2CCS extension and are ignored
  by those architectures.
- Input records lack instrument type or molecule-type metadata, so the one-hot encodings
  and monomer/dimer flag cannot be populated without imputation.
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_3316
tools:
- name: RDKit (v2023.09.5)
  role: Parses SMILES and computes the molecular weight descriptor used in the additional
    feature vector
  repo: https://rdkit.org/
- name: Mol2CCS
  role: Downstream GNN whose extended SigmaCCS architecture consumes the concatenated
    1-D feature vector alongside the molecular graph and adduct
  repo: https://github.com/enveda/ccs-prediction
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-024-00899-w
    title: Evaluating the generalizability of graph neural networks for predicting
      collision cross section
  source_content_hash: sha256:c1e80c6c1cbb939ad05ecad9a9c805c3131d75050923f73a83cd45928a98e3fb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Mol2CCS
  - RDKit (v2023.09.5)
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_0153
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# molecular-weight-descriptor-calculation

## Summary

Computes molecular weight and related 1-D molecular descriptors (e.g. 35-dimensional Drug Tax structural class counts, one-hot adduct/instrument/type encodings) from SMILES with RDKit, producing the flat feature vector Mol2CCS concatenates with the molecular graph and Morgan fingerprint for CCS prediction.

## When to use

Reference for building the seven one-dimensional feature vectors that Mol2CCS feeds alongside the 3-D molecular graph, when SMILES, adduct and instrument metadata are available per record.

## When NOT to use

- The downstream model is vanilla SigmaCCS or GraphCCS, which consumes only the molecular graph and adduct — the extra 1-D features are an Mol2CCS extension and are ignored by those architectures.
- Input records lack instrument type or molecule-type metadata, so the one-hot encodings and monomer/dimer flag cannot be populated without imputation.

## Inputs

- CSV/Excel record with molecule SMILES, adduct type, instrument type and monomer/dimer molecule type (e.g. raw METLIN-CCS or CCSBase export)
- SMILES string per molecule (parsed with RDKit v2023.09.5)

## Outputs

- Parquet feature file with concatenated 1-D descriptor vector (molecular weight, Drug Tax counts, one-hot adduct/instrument/type, monomer-dimer flag) plus smiles/adduct/ccs metadata columns consumable by scripts/train-test.py

## How to apply

Read each molecule's SMILES with RDKit (v2023.09.5) as in the Mol2CCS preprocessing pipeline, then compute the molecular weight descriptor and the 35-dimensional Drug Tax structural class counts. Next, one-hot encode the specific adduct form, instrument type, and molecule type, and set a binary flag differentiating monomeric and dimeric adducts. Concatenate the seven one-dimensional feature vectors into a single flat vector per record. This extends the SigmaCCS architecture, which otherwise uses only the molecular graph and adduct, with additional information such as instrument type to improve generalizability. Finally, write the resulting feature matrix as a parquet file whose metadata columns (smiles, adduct, ccs, optional 3-D coordinates) match the schema that scripts/train-test.py expects via --smiles-column-name, --adduct-column-name, --ccs-column-name and --coordinates-column-name.

## Related tools

- **RDKit (v2023.09.5)** (Parses SMILES and computes the molecular weight descriptor used in the additional feature vector) — https://rdkit.org/
- **Mol2CCS** (Downstream GNN whose extended SigmaCCS architecture consumes the concatenated 1-D feature vector alongside the molecular graph and adduct) — https://github.com/enveda/ccs-prediction

## Examples

```
python -c "from rdkit import Chem; m=Chem.MolFromSmiles(smiles); mw=Descriptors.MolWt(m)" && parquet write features
```

## Evaluation signals

- Each record's feature vector must contain exactly the seven one-dimensional components: molecular weight, 35-dim Drug Tax counts, one-hot adduct, one-hot instrument, one-hot molecule type, and the monomer/dimer binary flag
- Parquet output columns must match the names passed to train-test.py (--smiles-column-name, --adduct-column-name, --ccs-column-name, --coordinates-column-name)
- Molecular weight computed with RDKit should match the 'Molecular Weight' field present in SigmaCCS's own per-molecule metadata (PubChem ID, SMILES, InChi, Molecular Weight, Exact Mass, Formula)
- One-hot vectors must have consistent widths across train and test splits so the concatenated matrix has a fixed number of columns

## Limitations

- The extension improves generalizability only partially; performance still declines on structurally novel chemical spaces outside the training data
- Extra features such as instrument type partially mitigate but do not eliminate inaccuracies introduced by variations in experimental conditions
- The pipeline depends on RDKit's SMILES parsing and descriptor availability for Drug Tax classes; unusual valences or unparseable SMILES will fail silently

## Evidence

- [other] Mol2CCS encodes seven additional one-dimensional features beyond the molecular graph and the adduct, including one-hot encoded specific adduct forms and a one-hot vector differentiating monomeric and dimeric adducts.: [evidence span withheld — anchor status: not_found]
- [abstract] demonstrates how generalization can be partially improved by extending models to account for additional features such as molecular fingerprints, descriptors, and the molecule types: "demonstrates how generalization can be partially improved by extending models to account for additional features such as molecular fingerprints, descriptors, and the molecule types"
- [intro] we first read the SMILES representation of each molecule using RDKit (v2023.09.5): "we first read the SMILES representation of each molecule using RDKit (v2023.09.5)"
- [methods] Each notebook reads the csv/excel and formats it according to the input of Mol2CCS.: "Each notebook reads the csv/excel and formats it according to the input of Mol2CCS."
