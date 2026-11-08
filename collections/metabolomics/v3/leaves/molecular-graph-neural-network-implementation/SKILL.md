---
name: molecular-graph-neural-network-implementation
description: Use when building a compound-to-spectrum (C2MS) model from a molecular
  structure and a precomputed fragmentation DAG, where per-fragment formula probabilities
  must be learned by message-passing over molecular graphs.
when_to_use_negative:
- The input is an unknown MS/MS spectrum with no candidate molecular structure — the
  Molecule GNN requires a molecular graph G and is for the compound-to-spectrum (C2MS)
  direction, not MS2C retrieval alone
- Fragments require complex chemistry such as cyclizations, which dramatically expands
  the fragment search space beyond the recursive edge-removal DAG and the GNN's support
- The task is a fixed formula-set predictor (e.g. GrAFF-MS-style top-10,000 product/neutral-loss
  formula classification) rather than a per-fragment probabilistic distribution over
  formulae
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Pytorch
  role: Framework in which the FraGNNet model and baselines were implemented (version
    2.1 with CUDA 11.8 for the paper; README quick-start uses torch==2.4.1 cu121)
  repo: github.com/FraGNNet/fragnnet
- name: Pytorch Geometric
  role: Provides the graph neural network modules used to implement the Molecule GNN
    (GINE) and Fragment GNN
  repo: github.com/FraGNNet/fragnnet
- name: Pytorch Lightning
  role: Training/inference module wrapping the two-stage model, with configuration
    and checkpoint-building scripts
  repo: Lightning-AI/pytorch-lightning
- name: RDKit
  role: Data preprocessing and atom/bond featurization of the input molecular graph
    before the Molecule GNN
  repo: github.com/FraGNNet/fragnnet
- name: Cython
  role: Recursive edge-removal fragmentation algorithm that generates the DAG consumed
    by the Fragment GNN
  repo: github.com/FraGNNet/fragnnet
provenance:
  source_task_ids:
  - task_008
  source_content_hash: sha256:49939863fbbfd8618ce5b68070818eedcf633aceee6e100edecd9bad8701cbc2
  source_papers:
  - doi: 10.48550/arxiv.2404.02360
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Cython
  - Python
  - Pytorch
  - Pytorch Geometric
  - Pytorch Lightning
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# molecular-graph-neural-network-implementation

## Summary

Implement FraGNNet's two-stage GNN: a Molecule GNN (GINE in PyTorch Geometric) that embeds the RDKit-featurized input molecular graph, and a Fragment GNN (node-wise MLP variant) over the fragmentation DAG whose per-node output MLPs produce a distribution over formulae, which maps to monoisotopic peak masses.

## When to use

Use when building a compound-to-spectrum (C2MS) model from a molecular structure and a precomputed fragmentation DAG, where per-fragment formula probabilities must be learned by message-passing over molecular graphs.

## When NOT to use

- The input is an unknown MS/MS spectrum with no candidate molecular structure — the Molecule GNN requires a molecular graph G and is for the compound-to-spectrum (C2MS) direction, not MS2C retrieval alone
- Fragments require complex chemistry such as cyclizations, which dramatically expands the fragment search space beyond the recursive edge-removal DAG and the GNN's support
- The task is a fixed formula-set predictor (e.g. GrAFF-MS-style top-10,000 product/neutral-loss formula classification) rather than a per-fragment probabilistic distribution over formulae

## Inputs

- Input molecular graph G with RDKit atom and bond features
- Heavy-atom fragmentation DAG M_hat(G, d, j) generated with fragmentation depth d=4 (FraGNNet-D4) and hydrogen tolerance j=4
- NIST20 MS/MS training spectra with expert-curated formula annotations
- Configuration file modeled on config/template.yml

## Outputs

- Per-DAG-node 9-dimensional logit vector over P(n, f_n^i), i in {-j,...,j}
- Marginal distributions P(n), P(f|n), P(f) and mass distribution P(m) over monoisotopic fragment masses
- Predicted MS/MS spectrum as a table of monoisotopic-mass peaks with intensities and formula/fragment annotations
- Trained PyTorch Lightning checkpoint of the two-stage model

## How to apply

Featurize atoms and bonds of the input molecular graph with RDKit, then implement the Molecule GNN (g_theta^MOL) as a GINE network in PyTorch Geometric that updates atom embeddings for L1 layers via neighborhood aggregation with bond embeddings (Equation 3). Featurize each fragmentation-DAG node by concatenating the average final atom embedding of its subgraph, an embedding of the subgraph formula f_n, and an embedding of the node's DAG depth (Equations 4-5); use the ablated node-wise MLP variant of the Fragment GNN (no neighborhood aggregation term) as in the paper's experiments. Attach a small output MLP to each DAG node emitting a 2j+1 = 9-dimensional logit vector for P(n, f_n^i) over hydrogen-count offsets i in {-j,...,j} (j=4), then derive P(n), P(f|n), P(f), and the mass distribution P(m) by normalization, marginalization, and Bayes' theorem, mapping each formula to its monoisotopic mass (Dirac delta at inference). Assemble g_theta = g_FRAG(g_MOL(G), G_Fd) in PyTorch with a PyTorch Lightning module, a config/template.yml-style configuration, and a checkpoint-building script; train by maximum likelihood estimation with stochastic gradient descent, adding entropy regularization on the latent distributions via Equation 9.

## Related tools

- **Pytorch** (Framework in which the FraGNNet model and baselines were implemented (version 2.1 with CUDA 11.8 for the paper; README quick-start uses torch==2.4.1 cu121)) — github.com/FraGNNet/fragnnet
- **Pytorch Geometric** (Provides the graph neural network modules used to implement the Molecule GNN (GINE) and Fragment GNN) — github.com/FraGNNet/fragnnet
- **Pytorch Lightning** (Training/inference module wrapping the two-stage model, with configuration and checkpoint-building scripts) — Lightning-AI/pytorch-lightning
- **RDKit** (Data preprocessing and atom/bond featurization of the input molecular graph before the Molecule GNN) — github.com/FraGNNet/fragnnet
- **Cython** (Recursive edge-removal fragmentation algorithm that generates the DAG consumed by the Fragment GNN) — github.com/FraGNNet/fragnnet

## Examples

```
bash env/env_setup_gpu_py311_cu121_250602.sh && conda activate FRAGNNET-GPU
```

## Evaluation signals

- The predicted spectrum contains peaks only at monoisotopic fragment masses drawn from the support M_hat(G, d, j) — no masses outside the DAG's formula set
- On NIST20 held-out molecules (InChIKey split), the binned cosine similarity C_BIN reaches the reported FraGNNet-D4 level of approximately 0.736
- Each DAG node's output MLP produces exactly 2j+1 = 9 logits (j=4), and the derived P(n), P(f|n), P(f), and P(m) are valid normalized distributions
- Peak-formula annotations are consistent: annotations with formula masses more than 10 ppm away from the associated peak mass are excluded
- The model is compositional in structure: atom embeddings from the Molecule GNN aggregate into subgraph-average fragment-node features, and the Fragment GNN uses the node-wise MLP variant without neighborhood aggregation

## Limitations

- The fragmentation algorithm's recursive edge-removal operations are not parallelized, limiting runtime; they could be parallelized to accelerate it
- The algorithm fails to account for some fragments, as evidenced by unexplained Outside-of-Support (OS) peaks in the spectra
- Unexplained peaks may arise from complex chemistry such as cyclizations that dramatically expand the fragment search space and are challenging to model
- Performance has not been evaluated on unmerged spectra or under broader experimental conditions (different instrument platforms and precursor adducts); the paper's setup used Orbitrap HCD with [M+H]+ adduct type

## Evidence

- [methods] The first stage g_theta^MOL, called the Molecule GNN, operates on the input molecular graph G: [evidence span withheld — anchor status: not_found]
- [methods] Fragment GNN role: [evidence span withheld — anchor status: not_found]
- [methods] PyG implementation: [evidence span withheld — anchor status: not_found]
- [methods] DAG parameters: [evidence span withheld — anchor status: not_found]
- [readme] README model description: [evidence span withheld — anchor status: external_doc]
