---
name: d-mpnn-molecular-graph-encoding
description: Use when SMILES strings of metabolites need to be converted into learned,
  structure-aware molecular vectors (instead of fingerprints/descriptors) to support
  retention-order prediction or other property-learning tasks on molecules in new
  chemical spaces.
when_to_use_negative:
- The task concerns HILIC chromatography — the article states ROASMI currently applies
  only to RPLC systems, so embeddings trained for RPLC retention order should not
  be assumed transferable to HILIC.
- The compounds fall outside the elemental restrictions of C, H, N, O, P, and S, which
  the article flags as a vast chemical space yet to be explored.
- The analysis goal is MS/MS-based annotation where informative spectra are already
  available; this skill targets peaks lacking usable MS/MS data.
tools:
- name: chemprop
  role: Provides the D-MPNN (directed message passing neural network) molecular embedding
    module that converts molecular graphs into continuous vectors; ROASMI's embedding
    module is an extension of chemprop.
  repo: https://github.com/chemprop/chemprop
- name: ROASMI
  role: The host pipeline whose code/ROASMI_train.py drives pair construction, batch
    formation (size 64), and D-MPNN encoding for retraining.
  repo: https://github.com/FangYuan717/ROASMI
- name: PyTorch
  role: Deep-learning backend required for running the D-MPNN encoder; a GPU-enabled
    version may need manual installation.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-025-00968-8
    title: 'ROASMI: accelerating small molecule identification by repurposing retention
      data'
  source_content_hash: sha256:86c7b8ff182793126b9338513fd55c08e2019a924cb5a9ee2d723f103e1b6f11
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chemprop/chemprop
  tools:
  - PyTorch
  - ROASMI
  - chemprop
  - conda
  - cuDNN
  - cuda
  - pip
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# d-mpnn-molecular-graph-encoding

## Summary

Encodes compound structures (isomeric SMILES) into continuous molecular embeddings using a chemprop-based D-MPNN, as the first stage of ROASMI's retention-order model. These embeddings feed a RankNet that learns elution order from retention-time molecule pairs.

## When to use

Use when SMILES strings of metabolites need to be converted into learned, structure-aware molecular vectors (instead of fingerprints/descriptors) to support retention-order prediction or other property-learning tasks on molecules in new chemical spaces.

## When NOT to use

- The task concerns HILIC chromatography — the article states ROASMI currently applies only to RPLC systems, so embeddings trained for RPLC retention order should not be assumed transferable to HILIC.
- The compounds fall outside the elemental restrictions of C, H, N, O, P, and S, which the article flags as a vast chemical space yet to be explored.
- The analysis goal is MS/MS-based annotation where informative spectra are already available; this skill targets peaks lacking usable MS/MS data.

## Inputs

- Retention-time reference records (e.g., dataset_83) with compound identifiers and RT values
- Isomeric SMILES strings retrieved from PubChem for each compound
- Ordered molecule pairs (Ci, Cj) labeled by elution order within a 30-s retention interval

## Outputs

- Batched molecule-pair records saved as 'molecule_pair_batches.pt'
- Continuous molecular embedding vectors saved as 'molecule_embeddings.npy'

## How to apply

Retrieve the isomeric SMILES of each compound from PubChem, starting from the reference retention-time records (e.g., 'dataset_83' as the default retraining reference set in code/ROASMI_train.py). Construct ordered training pairs (Ci, Cj) where molecule Ci elutes before Cj within a 30-s retention interval, then group pairs into batches of 64 (the default batch size in TrainArgs in args.py). Feed each batch's SMILES strings through the directed message passing neural network (D-MPNN), a chemprop extension that iteratively aggregates atom and bond features along directed bonds, and sum the learned atom-level messages into a single molecular embedding per compound. Save the molecule-pair batches and their embeddings to disk (e.g., 'molecule_pair_batches.pt' and 'molecule_embeddings.npy') for downstream use by the RankNet retention prediction module.

## Related tools

- **chemprop** (Provides the D-MPNN (directed message passing neural network) molecular embedding module that converts molecular graphs into continuous vectors; ROASMI's embedding module is an extension of chemprop.) — https://github.com/chemprop/chemprop
- **ROASMI** (The host pipeline whose code/ROASMI_train.py drives pair construction, batch formation (size 64), and D-MPNN encoding for retraining.) — https://github.com/FangYuan717/ROASMI
- **PyTorch** (Deep-learning backend required for running the D-MPNN encoder; a GPU-enabled version may need manual installation.)

## Examples

```
python code/ROASMI_train.py
```

## Evaluation signals

- All generated molecule-pair batches are non-empty after pair construction within the 30-s retention interval.
- The D-MPNN module runs on the batched SMILES strings without error (script_runs on code/ROASMI_train.py).
- The run reproduces the paper's default retraining setup: dataset_83 as reference set, batch size 64 from TrainArgs in args.py.
- Each molecule's atom/bond-level messages are summed into exactly one continuous molecular embedding vector per compound.
- Saved outputs 'molecule_pair_batches.pt' and 'molecule_embeddings.npy' exist on disk and are loadable by the downstream RankNet stage.

## Limitations

- RT outliers resulting from slight changes in experimental conditions are unresolved and require in-depth mechanistic study.
- The model cannot distinguish certain molecular pairs with very close retention (e.g., 4-hydroxybenzoic acid and 2,5-dihydroxybenzoic acid), and such pairs should be excluded from subsequent small molecule identification.
- Applicability is limited to conventional compounds within the elemental restrictions of C, H, N, O, P, and S, and to RPLC chromatographic space.

## Evidence

- [readme] D-MPNN learns directly from structure: "A directed message transfer neural network (D-MPNN) is used to learn directly from the structure of compounds, allowing prediction of compounds in new chemical spaces"
- [methods] Embedding module is a chemprop extension: "This module is an extension of chemprop described in the paper [Analyzing Learned Molecular Representations for Property Prediction]"
- [readme] Two-module architecture of ROASMI: "Designed to predict retention scores for candidate compounds during small molecule identification, ROASMI consists of two main modules."
- [methods] Retraining command: "To retrain ROASMI, run: python code/ROASMI_train.py"
