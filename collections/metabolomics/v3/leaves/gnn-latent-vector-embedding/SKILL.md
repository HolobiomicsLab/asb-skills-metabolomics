---
name: gnn-latent-vector-embedding
description: 'Use when you need to validate the chemical rationality of a trained
  GNN model: extract its pooled latent embeddings and check via 2D embedding (e.g.,
  UMAP) whether structurally similar molecules cluster together.'
when_to_use_negative:
- Molecules contain elements outside C, H, O, N, P, S, F, Cl, Br, I, Co, As, Se —
  SigmaCCS cannot encode their graphs
- Input is already a low-dimensional descriptor/feature table — UMAP on raw descriptors
  does not probe the GNN's learned representations
- The goal is CCS value prediction itself rather than interpreting representation
  quality
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: SigmaCCS
  role: Trained GNN whose ECC layers produce the 16-dimensional pooled molecular vectors
    (molecular representations) extracted for embedding analysis
  repo: https://github.com/zmzhang/SigmaCCS
- name: UMAP
  role: Dimension-reduction library (umap-learn 0.5.1, Euclidean metric) used to project
    the 16-dim molecular vectors to 2D for cluster visualization
  repo: https://github.com/lmcinnes/umap
- name: convertToGraph
  role: SigmaCCS GraphData.py utility converting SMILES + conformer coordinates into
    node feature, adjacency, and edge feature matrices fed to the ECC layers
  repo: https://github.com/zmzhang/SigmaCCS
- name: GNN-RT
  role: Prior GNN work whose end-to-end learned molecular-representation approach
    the SigmaCCS visualization strategy follows
  repo: https://github.com/Qiong-Yang/GNN-RT
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42004-023-00939-w
    title: Highly accurate and large-scale collision cross sections prediction with
      graph neural networks
  source_content_hash: sha256:fc477c79a786b50d6e3e36a47bd7f7bb8605e8fbd53b5aeb58828dbe857d941f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zmzhang/SigmaCCS
  tools:
  - GNN-RT
  - SigmaCCS
  - UMAP
  - convertToGraph
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# gnn-latent-vector-embedding

## Summary

Extract the learned 16-dimensional molecular vectors from SigmaCCS's ECC graph convolutional layers (after global sum pooling) and project them with UMAP to test whether the GNN representations are chemically meaningful (clustering by ring content, molecular class, and Morgan-fingerprint Tanimoto similarity).

## When to use

Use when you need to validate the chemical rationality of a trained GNN model: extract its pooled latent embeddings and check via 2D embedding (e.g., UMAP) whether structurally similar molecules cluster together.

## When NOT to use

- Molecules contain elements outside C, H, O, N, P, S, F, Cl, Br, I, Co, As, Se — SigmaCCS cannot encode their graphs
- Input is already a low-dimensional descriptor/feature table — UMAP on raw descriptors does not probe the GNN's learned representations
- The goal is CCS value prediction itself rather than interpreting representation quality

## Inputs

- Trained SigmaCCS model checkpoint (model.h5)
- Molecular datasets: TrainData.csv, TestData.csv
- PubChem 1% sample from Zenodo record 5501674
- SMILES strings with 3D conformer coordinates (ETKDG-generated, MMFF94-minimized)
- Graph matrices per molecule: node feature matrix, adjacency matrix, edge feature matrix

## Outputs

- data_molvec.npy: array of shape (n_molecules, 16) pooled ECC-layer node attributes
- data-UMAP-EUC-60.npy: 2D UMAP embedding, shape (n_molecules, 2)
- UMAP visualization figure colored by ring number / molecular class

## How to apply

Load the trained SigmaCCS model (model.h5) and molecular datasets (TrainData.csv, TestData.csv, and a 1% PubChem sample from Zenodo record 5501674). Convert SMILES to graphs using UMAPDataset.py with the convertToGraph function, producing node feature, adjacency, and edge feature matrices. Forward the graphs through the trained model's ECC layers and collect the 16-dimensional pooled node attributes (molecular vectors), saving them as data_molvec.npy. Compute Euclidean distances between molecular vectors and fit a UMAP projection (umap 0.5.1, Euclidean metric, n_neighbors=60 per the output file data-UMAP-EUC-60.npy) into 2D. Finally, render the 2D embedding colored by ring number or molecular class to visually confirm clustering consistent with the paper's Fig. 4b-i.

## Related tools

- **SigmaCCS** (Trained GNN whose ECC layers produce the 16-dimensional pooled molecular vectors (molecular representations) extracted for embedding analysis) — https://github.com/zmzhang/SigmaCCS
- **UMAP** (Dimension-reduction library (umap-learn 0.5.1, Euclidean metric) used to project the 16-dim molecular vectors to 2D for cluster visualization) — https://github.com/lmcinnes/umap
- **convertToGraph** (SigmaCCS GraphData.py utility converting SMILES + conformer coordinates into node feature, adjacency, and edge feature matrices fed to the ECC layers) — https://github.com/zmzhang/SigmaCCS
- **GNN-RT** (Prior GNN work whose end-to-end learned molecular-representation approach the SigmaCCS visualization strategy follows) — https://github.com/Qiong-Yang/GNN-RT

## Examples

```
import numpy as np, umap; v = np.load('data_molvec.npy'); emb = umap.UMAP(n_neighbors=60, metric='euclidean').fit_transform(v); np.save('data-UMAP-EUC-60.npy', emb)
```

## Evaluation signals

- data_molvec.npy exists with shape (n_molecules, 16)
- data-UMAP-EUC-60.npy exists with shape (n_molecules, 2)
- Rendered UMAP plot exhibits visually separated clusters consistent with Fig. 4b-i (grouping by ring number and molecular class)
- Structurally similar molecules (high Morgan-fingerprint Tanimoto similarity) are placed closer together in the embedding
- All molecules in the input dataset have a corresponding 16-dim vector (no silent drops during graph generation, unlike ETKDG conformer failures noted for PubChem/CCSbase entries)

## Limitations

- SigmaCCS cannot predict molecules containing elements other than C, H, O, N, P, S, F, Cl, Br, I, Co, As, and Se, limiting which molecules can be embedded
- ETKDG conformer generation failed for 33/94,161,896 PubChem molecules and 5/5,645 CCSbase entries, and 662 molecules' conformers could not be MMFF94-minimized — these lack graph inputs and thus embeddings
- UMAP projection is stochastic; cluster appearance depends on n_neighbors, metric, and random state

## Evidence

- [abstract] The model-agnostic interpretation method and the visualization of the learned representations were used to investigate the chemical rationality of SigmaCCS: "The model-agnostic interpretation method and the visualization of the learned representations were used to investigate the chemical rationality of SigmaCCS"
- [discussion] the ECC layers are mainly used for learning multilevel representations from molecular graphs: "the ECC layers are mainly used for learning multilevel representations from molecular graphs. Then, the learned representations are fed to the fully connected layers to predict CCS values."
- [readme] GNN-RT obtains data-driven molecular representations via end-to-end GNN learning: [evidence span withheld — anchor status: external_doc]
