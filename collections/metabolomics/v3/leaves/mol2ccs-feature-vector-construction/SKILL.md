---
name: mol2ccs-feature-vector-construction
description: 'Use when preparing Mol2CCS input records: you have SMILES and an adduct
  label per measurement and must build the additional 1-D feature vectors (fingerprints,
  descriptors, encodings) the model consumes alongside the graph.'
when_to_use_negative:
- You are training SigmaCCS or GraphCCS directly from SMILES and adduct only — those
  models consume the graph (node/adjacency/edge feature matrices from 3D conformers)
  rather than the concatenated 1-D feature vector.
- The input is already a formatted Mol2CCS parquet produced by the data_processing
  notebooks — re-extracting features would duplicate work; instead go straight to
  the training split or prediction step.
- Your molecules have no valid SMILES representation (RDKit cannot parse them) — fingerprint
  and descriptor extraction will fail before the graph is built.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3332
tools:
- name: RDKit (v2023.09.5)
  role: Parses SMILES and computes the Morgan fingerprint, molecular weight descriptor,
    and Drug Tax structural class counts that form part of the feature vector
- name: Mol2CCS
  role: Downstream model that consumes the concatenated feature matrix together with
    the molecular graph and adduct for CCS prediction
  repo: https://github.com/enveda/ccs-prediction
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-024-00899-w
    title: Evaluating the generalizability of graph neural networks for predicting
      collision cross section
  source_content_hash: sha256:547ee7f9ee8be534afe21be42efeeaaacaecab97150500e6062c06974d361310
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/enveda/ccs-prediction
  tools:
  - Mol2CCS
  - RDKit (v2023.09.5)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# mol2ccs-feature-vector-construction

## Summary

Construction of the seven one-dimensional feature vectors Mol2CCS appends to the molecular graph representation for CCS prediction, combining Morgan fingerprints, molecular weight, Drug Tax class counts, adduct/instrument one-hot encodings, and monomer/dimer flags, written as a parquet feature matrix.

## When to use

Use when preparing Mol2CCS input records: you have SMILES and an adduct label per measurement and must build the additional 1-D feature vectors (fingerprints, descriptors, encodings) the model consumes alongside the graph.

## When NOT to use

- You are training SigmaCCS or GraphCCS directly from SMILES and adduct only — those models consume the graph (node/adjacency/edge feature matrices from 3D conformers) rather than the concatenated 1-D feature vector.
- The input is already a formatted Mol2CCS parquet produced by the data_processing notebooks — re-extracting features would duplicate work; instead go straight to the training split or prediction step.
- Your molecules have no valid SMILES representation (RDKit cannot parse them) — fingerprint and descriptor extraction will fail before the graph is built.

## Inputs

- SMILES strings per molecule (parsed with RDKit v2023.09.5)
- Adduct label per measurement (e.g. [M+H]+, [M+Na]+, [M-H]-)
- Molecule type / monomer-dimer association flag
- Instrument type per measurement
- METLIN-CCS or CCSBase records (excel/csv as downloaded from ccsbase.net or metlin.scripps.edu)

## Outputs

- Parquet feature matrix with one concatenated 7-part feature vector per record plus metadata columns (e.g. smiles, adduct, ccs) consumed by Mol2CCS training/prediction scripts

## How to apply

Read each molecule's SMILES with RDKit (v2023.09.5) as in the Mol2CCS preprocessing pipeline, then compute the 256-bit Morgan fingerprint with radius 2 for every structure. Compute the molecular weight descriptor and the 35-dimensional Drug Tax structural class counts, and encode the adduct form, molecule type (monomeric vs dimeric), and instrument type as one-hot vectors, with the monomer/dimer association carried as a binary flag. Concatenate these seven one-dimensional vectors into a single feature vector per record and write the resulting feature matrix as a parquet file containing the metadata columns Mol2CCS expects (e.g. smiles, adduct, ccs columns as used by the train-test script). The rationale is that appending fingerprints, descriptors, and molecule-type information to SigmaCCS's graph-only architecture improves generalizability across chemical spaces.

## Related tools

- **RDKit (v2023.09.5)** (Parses SMILES and computes the Morgan fingerprint, molecular weight descriptor, and Drug Tax structural class counts that form part of the feature vector)
- **Mol2CCS** (Downstream model that consumes the concatenated feature matrix together with the molecular graph and adduct for CCS prediction) — https://github.com/enveda/ccs-prediction

## Examples

```
python scripts/train-test.py --train-input-file "ccs-prediction/metlin_train_3d.parquet" --adduct-column-name "adduct" --ccs-column-name "ccs"
```

## Evaluation signals

- The 256-bit Morgan fingerprint (radius 2) is nonzero-length and derived from RDKit-parsed SMILES for every record.
- The Drug Tax block contributes exactly 35 dimensions per molecule.
- Adduct, molecule type, and instrument one-hot vectors sum to 1 in the categorical blocks; the monomer/dimer flag is a single binary value.
- Output is a valid parquet file whose metadata columns match the column names expected by train-test.py (e.g. --smiles-column-name smiles --adduct-column-name adduct --ccs-column-name ccs).
- Training Mol2CCS on the resulting matrix reproduces the improved generalizability reported for the extended SigmaCCS architecture on held-out chemical spaces.

## Limitations

- Variations in experimental conditions such as instrument types likely introduced inaccuracies in the CCS values used to train the models, which the additional features only partially mitigate.
- The evaluation and feature construction mainly focus on small molecules and specific regions of chemical space (e.g. METLIN-CCS is based on synthetic structures).
- High quality experimental CCS references number only in the low thousands, constraining how much the appended features can improve generalization to structurally novel regions.

## Evidence

- [other] Mol2CCS encodes seven additional one-dimensional features beyond the molecular graph and the adduct, including one-hot encoded specific adduct forms and a one-hot vector differentiating monomeric and dimeric adducts.: [evidence span withheld — anchor status: not_found]
- [intro] we first read the SMILES representation of each molecule using RDKit (v2023.09.5): "we first read the SMILES representation of each molecule using RDKit (v2023.09.5)"
- [abstract] demonstrates how generalization can be partially improved by extending models to account for additional features such as molecular fingerprints, descriptors, and the molecule types: "demonstrates how generalization can be partially improved by extending models to account for additional features such as molecular fingerprints, descriptors, and the molecule types"
- [methods] Each notebook reads the csv/excel and formats it according to the input of Mol2CCS.: "Each notebook reads the csv/excel and formats it according to the input of Mol2CCS."
- [readme] data_processing: notebooks to process the METLIN-CCS and CCSBase and make the data splits.: "data_processing: notebooks to process the METLIN-CCS and CCSBase and make the data splits."
