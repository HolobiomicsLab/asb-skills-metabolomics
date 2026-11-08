---
name: drug-tax-structural-class-encoding
description: 'Use when building CCS prediction inputs for Mol2CCS: molecules are given
  as SMILES and you must assemble the seven 1-D feature vectors (including the 35-bit
  Drug Tax structural class encoding) that augment the molecular graph and adduct
  representation.'
when_to_use_negative:
- Running SigmaCCS or GraphCCS unmodified — those models consume only the molecular
  graph and adduct and do not accept the seven additional 1-D features
- When no SMILES/adduct/molecule-type metadata is available for the records (RDKit
  cannot compute the Drug Tax classes or Morgan fingerprint)
- Tasks where molecular identity is already summarized in an existing feature table
  — this skill recomputes descriptors from raw SMILES
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_0082
- http://edamontology.org/topic_3474
tools:
- name: RDKit (v2023.09.5)
  role: Reads the SMILES representation of each molecule and computes the Morgan fingerprint
    (radius 2, 256 bits), molecular weight, and Drug Tax structural class counts used
    as the additional 1-D features
- name: Mol2CCS
  role: Downstream model that consumes the concatenated seven-feature vector alongside
    the molecular graph and adduct for CCS prediction
  repo: https://github.com/enveda/ccs-prediction
- name: Python
  role: Execution environment used to run the notebooks/scripts (via poetry) that
    generate the feature matrix and parquet outputs
  repo: https://github.com/enveda/ccs-prediction
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-024-00899-w
    title: Evaluating the generalizability of graph neural networks for predicting
      collision cross section
  source_content_hash: sha256:b84852f5c074d623f0d8122e3135c804d5a712581a8f154f897eaf40514fadc6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/enveda/ccs-prediction
  tools:
  - Mol2CCS
  - Python
  - RDKit (v2023.09.5)
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_0082
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# drug-tax-structural-class-encoding

## Summary

Computes a 35-dimensional one-hot vector of Drug Tax (drug-type) structural class counts for each molecule with RDKit, and concatenates it with the Morgan fingerprint, molecular weight, adduct, molecule type, instrument type, and monomer/dimer flags into the seven additional one-dimensional features that Mol2CCS feeds alongside its graph representation.

## When to use

Use when building CCS prediction inputs for Mol2CCS: molecules are given as SMILES and you must assemble the seven 1-D feature vectors (including the 35-bit Drug Tax structural class encoding) that augment the molecular graph and adduct representation.

## When NOT to use

- Running SigmaCCS or GraphCCS unmodified — those models consume only the molecular graph and adduct and do not accept the seven additional 1-D features
- When no SMILES/adduct/molecule-type metadata is available for the records (RDKit cannot compute the Drug Tax classes or Morgan fingerprint)
- Tasks where molecular identity is already summarized in an existing feature table — this skill recomputes descriptors from raw SMILES

## Inputs

- SMILES strings for each molecule (e.g. from CCSBase v1.3 or METLIN-CCS records)
- Adduct label per record (e.g. [M+H]+, [M+Na]+, [M-H]-)
- Molecule type (monomer vs dimer) and instrument type metadata

## Outputs

- Per-record concatenated 1-D feature vector (Morgan fingerprint, molecular weight, 35-bit Drug Tax structural class counts, adduct/molecule-type/instrument-type one-hots, monomer/dimer flag)
- Parquet feature-matrix file (e.g. metlin_train_3d.parquet / ccsbase_3d.parquet) with smiles, adduct, ccs, and coordinates columns consumable by train-test.py

## How to apply

Read each molecule's SMILES with RDKit (v2023.09.5) and compute its 256-bit Morgan fingerprint (radius 2), molecular weight, and the 35-dimensional Drug Tax structural class counts; one-hot encode the adduct form, molecule type, and instrument type, and flag the monomer/dimer association as a binary value. Concatenate these seven one-dimensional feature vectors into a single vector per record, pairing it with the molecule's 3D graph representation and adduct. Write the resulting feature matrix as a parquet file with the metadata columns (e.g. smiles, adduct, ccs, coordinates) required by Mol2CCS's train-test.py, using the data_splits notebook conventions in the enveda/ccs-prediction repo. The rationale, per the article, is that extending SigmaCCS with these additional features improves generalization to structurally novel chemical spaces such as METLIN-CCS.

## Related tools

- **RDKit (v2023.09.5)** (Reads the SMILES representation of each molecule and computes the Morgan fingerprint (radius 2, 256 bits), molecular weight, and Drug Tax structural class counts used as the additional 1-D features)
- **Mol2CCS** (Downstream model that consumes the concatenated seven-feature vector alongside the molecular graph and adduct for CCS prediction) — https://github.com/enveda/ccs-prediction
- **Python** (Execution environment used to run the notebooks/scripts (via poetry) that generate the feature matrix and parquet outputs) — https://github.com/enveda/ccs-prediction

## Examples

```
poetry run python scripts/train-test.py --prefix "train-metlin-test-ccsbase" --train-input-file "ccs-prediction/metlin_train_3d.parquet" --test-input-file "ccs-prediction/ccsbase_3d.parquet" --coordinates-present --smiles-column-name "smiles" --adduct-column-name "adduct" --ccs-column-name "ccs" --dropout-rate 0.1 --epochs 400
```

## Evaluation signals

- The per-molecule feature vector concatenates exactly seven one-dimensional components: Morgan fingerprint, molecular weight, 35-bit Drug Tax class counts, adduct one-hot, molecule-type one-hot, instrument-type one-hot, and monomer/dimer flag
- The Drug Tax component is a 35-dimensional vector of structural class counts derived from RDKit on the input SMILES
- Output parquet files parse correctly with the expected columns (smiles, adduct, ccs, coordinates) and load into train-test.py without schema errors
- Records retain their adduct labels consistent with the CCSBase/METLIN-CCS source (e.g. [M+H]+, [M+Na]+, [M-H]-) so the adduct one-hot matches training conventions

## Limitations

- Variations in experimental conditions, such as instrument types, likely introduced inaccuracies in the CCS training values, motivating the instrument-type feature but also limiting confidence in absolute CCS accuracy
- Applicability is restricted to chemical spaces similar to training data; performance declines on structurally novel regions even with the additional features
- The additional features only partially improve generalization — they do not eliminate the train/test chemical-space mismatch observed when benchmarking METLIN-CCS vs CCSBase

## Evidence

- [methods] RDKit encodes seven 1-D features for Mol2CCS: "we first read the SMILES representation of each molecule using RDKit (v2023.09.5)"
- [discussion] Mol2CCS extends SigmaCCS with additional features for generalization: "we demonstrate how an extension of the architecture of GNN (Mol2CCS) including other additional features improves the generalizability of SigmaCCS"
- [abstract] Additional features improve generalizability to structurally novel regions: "demonstrates how generalization can be partially improved by extending models to account for additional features such as molecular fingerprints, descriptors, and the molecule types"
- [readme] train-test.py consumes parquet files with named columns: "poetry run python scripts/train-test.py \
	--prefix "train-metlin-test-ccsbase" \
	--train-input-file "ccs-prediction/metlin_train_3d.parquet" \
	--test-input-file "ccs-prediction/ccsbase_3d.parquet""
- [intro] Instrument type included as an additional feature in Mol2CCS: "We also demonstrate increased generalizability using an extension of SigmaCCS, which we call Mol2CCS, that incorporates additional information such as instrument type."
