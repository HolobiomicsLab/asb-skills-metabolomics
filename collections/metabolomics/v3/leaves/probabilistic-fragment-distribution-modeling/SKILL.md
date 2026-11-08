---
name: probabilistic-fragment-distribution-modeling
description: Use when predicting an MS/MS spectrum from a known molecular structure
  (C2MS), especially when high mass accuracy, formula annotations, and fragment-level
  interpretability matter for library augmentation.
when_to_use_negative:
- The goal is formula annotation of glycan or peptide spectra, which the paper excluded
  to focus on small molecules
- No candidate molecular structure is available (the skill predicts spectra from a
  known structure G; it does not solve structure elucidation from the spectrum alone)
- Prediction is needed on unmerged spectra or under instrument platforms/adducts beyond
  the Orbitrap HCD [M+H]+ setting validated in the paper
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0153
tools:
- name: RDKit
  role: Data preprocessing and molecule featurization of atoms and bonds before the
    Molecule GNN
- name: Cython
  role: Implements the recursive fragmentation (edge-removal) algorithm that builds
    the fragmentation DAG
- name: PyTorch Geometric
  role: Implements the GNN modules (GINE Molecule GNN and Fragment GNN) operating
    on the molecular graph and DAG
- name: PyTorch
  role: Implements the FraGNNet model, baselines, and the probabilistic output MLPs
- name: PyTorch Lightning
  role: Provides the training/inference LightningModule and Trainer orchestration
    for FraGNNet
  repo: https://github.com/Lightning-AI/pytorch-lightning
- name: FraGNNet
  role: The model itself; official code, configs, preprocessing and checkpoint scripts
    live in this repository
  repo: https://github.com/FraGNNet/fragnnet
provenance:
  source_task_ids:
  - task_008
  source_content_hash: sha256:7e1cfa9b9bfa5700ad47e6973a3cd5013590274a4c6ea3d4bd0d9a44bae41265
  source_papers:
  - doi: 10.48550/arxiv.2404.02360
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Lightning-AI/pytorch-lightning
  tools:
  - Cython
  - FraGNNet
  - PyTorch
  - PyTorch Geometric
  - PyTorch Lightning
  - Python
  - RDKit
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# probabilistic-fragment-distribution-modeling

## Summary

Reconstruct FraGNNet's two-stage probabilistic GNN that formulates C2MS prediction as learning a distribution over molecule fragments: a Molecule GNN embeds the input graph, a Fragment GNN over the fragmentation DAG predicts P(n), P(f|n), P(f), and P(m), which are converted to an MS/MS spectrum via formula masses.

## When to use

Use when predicting an MS/MS spectrum from a known molecular structure (C2MS), especially when high mass accuracy, formula annotations, and fragment-level interpretability matter for library augmentation.

## When NOT to use

- The goal is formula annotation of glycan or peptide spectra, which the paper excluded to focus on small molecules
- No candidate molecular structure is available (the skill predicts spectra from a known structure G; it does not solve structure elucidation from the spectrum alone)
- Prediction is needed on unmerged spectra or under instrument platforms/adducts beyond the Orbitrap HCD [M+H]+ setting validated in the paper

## Inputs

- Input molecular structure (molecular graph G, e.g. SMILES, RDKit-featurized atoms/bonds)
- Approximate fragmentation DAG G_Fd from recursive bond-breaking on the heavy atom skeleton (d=4, j=4)
- Model configuration file (modeled on config/template.yml) and trained checkpoint

## Outputs

- Predicted MS/MS spectrum as a table of monoisotopic-mass peaks with intensities and formula/fragment annotations
- Latent probability distributions P(n), P(f|n), P(f) over fragments and formulae, and mass distribution P(m)
- Per-node 2j+1=9-dimensional logit vector over hydrogen-shift formulae f_n^i, i in {-j,...,j}

## How to apply

Generate the approximate heavy-atom fragmentation DAG on the hydrogen-stripped skeleton with the Cython recursive edge-removal algorithm using depth d=4 (FraGNNet-D4) and hydrogen tolerance j=4, since M_hat(G,d,j) with d in {3,4} and j=4 captures most total peak intensity in real MS/MS data. Featurize atoms and bonds with RDKit and run a GINE Molecule GNN (PyTorch Geometric) for L1 layers of neighborhood aggregation (Equation 3). Featurize each DAG node by concatenating the average final atom embedding of its subgraph, a subgraph-formula embedding f_n, and node DAG depth (Equations 4-5), then apply the ablated node-wise MLP Fragment GNN variant used in the paper. Attach a small output MLP per DAG node emitting a 2j+1=9-dimensional logit vector for P(n, f_n^i), i in {-j,...,j}, then derive P(n), P(f|n), P(f), and P(m) by normalization, marginalization, and Bayes' theorem, mapping each formula to its monoisotopic mass (Dirac delta at inference). Train g_theta = g_FRAG(g_MOL(G), G_Fd) with maximum likelihood via stochastic gradient descent plus normalized-entropy regularization on the latent distributions (Equation 9), orchestrated with PyTorch Lightning and a config/template.yml-style configuration.

## Related tools

- **RDKit** (Data preprocessing and molecule featurization of atoms and bonds before the Molecule GNN)
- **Cython** (Implements the recursive fragmentation (edge-removal) algorithm that builds the fragmentation DAG)
- **PyTorch Geometric** (Implements the GNN modules (GINE Molecule GNN and Fragment GNN) operating on the molecular graph and DAG)
- **PyTorch** (Implements the FraGNNet model, baselines, and the probabilistic output MLPs)
- **PyTorch Lightning** (Provides the training/inference LightningModule and Trainer orchestration for FraGNNet) — https://github.com/Lightning-AI/pytorch-lightning
- **FraGNNet** (The model itself; official code, configs, preprocessing and checkpoint scripts live in this repository) — https://github.com/FraGNNet/fragnnet

## Evaluation signals

- Predicted spectrum contains peaks only at monoisotopic fragment masses from M_hat(G, d, j) — no mass outside the support of the generated fragment set
- On NIST20 held-out molecules, binned cosine similarity C_BIN reaches the reported FraGNNet-D4 level (~0.736 on the InChIKey split)
- Latent distributions P(n), P(f|n), P(f) are properly normalized and each node's formula logits are 2j+1=9-dimensional for j=4
- Formula annotations of predicted peaks satisfy the 10 ppm mass-accuracy criterion used in the paper's formula annotation classification evaluation
- Outside-of-Support (OS) peak fraction stays low, since unexplained OS peaks indicate the fragmentation algorithm missed fragments

## Limitations

- The fragmentation algorithm relies on recursive edge-removal operations that are not parallelized, limiting runtime
- The algorithm fails to account for some fragments, evidenced by unexplained OS peaks in the spectra
- Unexplained peaks may arise from complex chemistry such as cyclizations that dramatically expand the fragment search space
- Performance has not been evaluated on unmerged spectra or under broader experimental conditions (different instrument platforms and precursor adducts); paper experiments used Orbitrap HCD spectra with [M+H]+ adduct

## Evidence

- [methods] Two-stage GNN assembly (Molecule GNN then Fragment GNN over DAG): [evidence span withheld — anchor status: not_found]
- [methods] DAG size parameters grounded in peak-intensity coverage: [evidence span withheld — anchor status: not_found]
- [full_text] Output MLPs on DAG nodes produce the formula distribution: [evidence span withheld — anchor status: not_found]
- [readme] Official repository describes the overall model: [evidence span withheld — anchor status: external_doc]
