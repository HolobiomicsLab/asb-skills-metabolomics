---
name: smiles-morgan-fingerprint-extraction
description: Use when preparing Mol2CCS feature matrices from SMILES strings — i.e.
  when the downstream CCS model consumes a concatenated 7-vector of one-dimensional
  features alongside the molecular graph, as in the enveda/ccs-prediction preprocessing
  pipeline.
when_to_use_negative:
- The downstream model is plain SigmaCCS or GraphCCS, which consume only the molecular
  graph and adduct — no fingerprint/descriptor features are needed
- SMILES are missing or unreadable by RDKit (v2023.09.5) so fingerprints and descriptors
  cannot be computed
- Only 3D coordinate features are required (the train-test.py pipeline with --coordinates-present
  flag)
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3474
tools:
- name: RDKit (v2023.09.5)
  role: Reads SMILES and computes the Morgan fingerprint, molecular weight, and structural
    descriptors used as the one-dimensional features
  repo: https://github.com/enveda/ccs-prediction
- name: Mol2CCS
  role: Downstream GNN that consumes the concatenated additional-feature vector alongside
    the molecular graph and adduct for CCS prediction
  repo: https://github.com/enveda/ccs-prediction
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-024-00899-w
    title: Evaluating the generalizability of graph neural networks for predicting
      collision cross section
  source_content_hash: sha256:fc26abc1a525ad4637b02ebadce17b4b3c6ae1b04f08f8485de2dc0c658ee765
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/enveda/ccs-prediction
  tools:
  - Mol2CCS
  - RDKit (v2023.09.5)
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# smiles-morgan-fingerprint-extraction

## Summary

Computes Morgan fingerprints (256-bit, radius 2), molecular weight, Drug Tax structural class counts, and one-hot encoded adduct/molecule-type/instrument features from SMILES to build the 1-D feature vector Mol2CCS concatenates with the molecular graph and adduct input for CCS prediction.

## When to use

Use when preparing Mol2CCS feature matrices from SMILES strings — i.e. when the downstream CCS model consumes a concatenated 7-vector of one-dimensional features alongside the molecular graph, as in the enveda/ccs-prediction preprocessing pipeline.

## When NOT to use

- The downstream model is plain SigmaCCS or GraphCCS, which consume only the molecular graph and adduct — no fingerprint/descriptor features are needed
- SMILES are missing or unreadable by RDKit (v2023.09.5) so fingerprints and descriptors cannot be computed
- Only 3D coordinate features are required (the train-test.py pipeline with --coordinates-present flag)

## Inputs

- SMILES strings for each molecule (from METLIN-CCS or CCSBase CSV/Excel exports)
- Adduct labels per record (e.g. [M+H]+, [M+Na]+, [M-H]-)
- Instrument type per record
- Monomer/dimer molecule type flag

## Outputs

- Parquet file with a concatenated one-dimensional feature vector per record (256-bit Morgan fingerprint, molecular weight, 35-dim Drug Tax counts, one-hot adduct/molecule-type/instrument) and metadata columns for Mol2CCS input

## How to apply

Read each molecule's SMILES with RDKit v2023.09.5 and compute the 256-bit Morgan fingerprint with radius 2 (rdkit.Chem.AllChem.GetMorganFingerprintAsBitVect). Compute the molecular weight descriptor and the 35-dimensional Drug Tax structural class counts. One-hot encode the specific adduct form, the molecule type (monomer vs dimer, encoded as a binary flag), and the instrument type; concatenate these seven one-dimensional vectors into a single feature vector per record. Write the resulting feature matrix as a parquet file whose columns match the metadata required by Mol2CCS, mirroring the notebooks/data_processing pipeline that formats METLIN-CCS and CCSBase CSV/Excel exports. Rationale: the paper found that extending SigmaCCS with these additional features (Mol2CCS) partially improves generalization to structurally novel chemical spaces.

## Related tools

- **RDKit (v2023.09.5)** (Reads SMILES and computes the Morgan fingerprint, molecular weight, and structural descriptors used as the one-dimensional features) — https://github.com/enveda/ccs-prediction
- **Mol2CCS** (Downstream GNN that consumes the concatenated additional-feature vector alongside the molecular graph and adduct for CCS prediction) — https://github.com/enveda/ccs-prediction

## Examples

```
from rdkit import Chem, DataStructs; from rdkit.Chem import AllChem; fp = AllChem.GetMorganFingerprintAsBitVect(Chem.MolFromSmiles(smiles), 2, nBits=256)
```

## Evaluation signals

- Fingerprint bit vector has exactly 256 bits with radius 2
- Feature matrix contains 35 Drug Tax structural class count columns plus molecular weight
- Adduct, molecule type, and instrument type are one-hot encoded with a binary monomer/dimer flag
- Output parquet opens in the Mol2CCS training pipeline without column-name errors (smiles, adduct, ccs columns present)
- Re-running on the same METLIN-CCS/CCSBase split produces an identical feature matrix

## Limitations

- Evaluation is restricted mainly to small molecules and synthetic structures (METLIN-CCS), so feature generalization to other chemical spaces is untested
- Variations in experimental conditions, such as instrument types, likely introduced inaccuracies in the CCS values used to train the models, which these features partially compensate for but cannot fully resolve
- The paper notes lack of data availability as a major challenge, so the feature matrix quality depends on the underlying database coverage

## Evidence

- [methods] Mol2CCS encodes seven additional one-dimensional features beyond the molecular graph and the adduct, including one-hot encoded specific adduct forms and a one-hot vector differentiating monomeric and dimeric adducts.: [evidence span withheld — anchor status: not_found]
- [methods] Compute the 256-bit Morgan fingerprint (radius 2) for each molecule.: [evidence span withheld — anchor status: not_found]
- [methods] we first read the SMILES representation of each molecule using RDKit (v2023.09.5): "we first read the SMILES representation of each molecule using RDKit (v2023.09.5)"
- [abstract] demonstrates how generalization can be partially improved by extending models to account for additional features such as molecular fingerprints, descriptors, and the molecule types: "demonstrates how generalization can be partially improved by extending models to account for additional features such as molecular fingerprints, descriptors, and the molecule types"
- [methods] Each notebook reads the csv/excel and formats it according to the input of Mol2CCS.: "Each notebook reads the csv/excel and formats it according to the input of Mol2CCS."
- [readme] poetry run python scripts/train-test.py --prefix "train-metlin-test-ccsbase" --train-input-file "ccs-prediction/metlin_train_3d.parquet": "poetry run python scripts/train-test.py --prefix "train-metlin-test-ccsbase" --train-input-file "ccs-prediction/metlin_train_3d.parquet""
