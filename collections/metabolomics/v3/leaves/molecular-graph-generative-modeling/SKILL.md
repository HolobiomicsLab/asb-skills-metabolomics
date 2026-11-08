---
name: molecular-graph-generative-modeling
description: Use when you need to generate a full molecular graph from an MS/MS spectrum
  given a known or retrieved Murcko scaffold, e.g. as the second stage of MADGEN's
  scaffold-conditioned de novo generation pipeline.
when_to_use_negative:
- You do not have a scaffold — MADGEN's generator requires a Murcko scaffold as the
  starting graph; for scaffold-free generation use a different model class.
- Your spectra are raw, un-normalized, and noisy — MADGEN preprocessing first normalizes
  intensities and removes low-intensity peaks below a predefined threshold; skip that
  preprocessing at your own risk.
- You only need scaffold retrieval/ranking, not molecule generation — use the contrastive
  retrieval module instead of the graph generator.
edam_operation: http://edamontology.org/operation_0364
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0091
tools:
- name: MADGEN
  role: 'Second-stage generator: spectra-conditioned Markov bridge model that produces
    the target molecule starting from the retrieved scaffold'
  repo: https://github.com/HassounLab/MADGEN
- name: RDKit
  role: Derive Murcko scaffolds from molecules to form (scaffold, molecule) training
    pairs and build the fully connected graph representation
- name: PyTorch
  role: Implement the FC-GNN / graph transformer and train the Markov bridge model;
    checkpoint saving and loading
- name: AdamW
  role: Optimizer for training the Markov bridge generator at learning rate 1e-5
- name: wget
  role: Download the processed msgym.pkl training data from Zenodo record 15036069
provenance:
  source_task_ids:
  - task_004
  source_content_hash: sha256:298430d1604468587d0fb0c7788b54ed0510b9dcf06b54d9f04d1f1d7e17ac7b
  source_papers:
  - doi: 10.48550/arxiv.2501.01950
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HassounLab/MADGEN
  tools:
  - AdamW
  - MADGEN
  - PyTorch
  - RDKit
  - wget
  edam_operation: http://edamontology.org/operation_0364
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# molecular-graph-generative-modeling

## Summary

Generate a target molecule as a graph by adapting an absorbing-state Markov bridge model (RetroBridge-style) that starts from a fixed Murcko scaffold and fills in edges connecting scaffold atoms to free atoms, guided by MS/MS spectra via cross-attention.

## When to use

Use when you need to generate a full molecular graph from an MS/MS spectrum given a known or retrieved Murcko scaffold, e.g. as the second stage of MADGEN's scaffold-conditioned de novo generation pipeline.

## When NOT to use

- You do not have a scaffold — MADGEN's generator requires a Murcko scaffold as the starting graph; for scaffold-free generation use a different model class.
- Your spectra are raw, un-normalized, and noisy — MADGEN preprocessing first normalizes intensities and removes low-intensity peaks below a predefined threshold; skip that preprocessing at your own risk.
- You only need scaffold retrieval/ranking, not molecule generation — use the contrastive retrieval module instead of the graph generator.

## Inputs

- Processed dataset pickle (e.g. msgym.pkl from Zenodo record 15036069) containing canonical SMILES, normalized MS/MS spectra, and predefined train/val/test splits
- Murcko scaffold SMILES strings derived from target molecules with RDKit
- Retrieved scaffold candidates (from contrastive scaffold retrieval or an oracle lookup table)

## Outputs

- Trained PyTorch checkpoint of the spectra-conditioned Markov bridge generator
- Generated molecule SMILES strings (graph edges decoded to valid molecular structures)
- Sampling results CSV with true/pred/scaffold columns for downstream evaluation

## How to apply

Represent each (scaffold, molecule) pair as a fully connected graph whose edge entries are D-dimensional one-hot vectors (class 0 = non-edge); keep scaffold edges fixed and generate only edges connecting scaffold atoms to free atoms. Tokenize the normalized MS/MS spectrum by encoding peaks with MLPs over m/z and intensity, refine them with a self-attention module into peak tokens, and inject these tokens into every message-passing layer of a 5-layer FC-GNN via cross-attention (node-only, edge-only, or combined variants). Train the absorbing-state Markov bridge model with endpoint-prediction ELBO loss using 50 diffusion steps and AdamW at learning rate 1e-5; drop the spectrum condition with probability 0.1 during training so classifier-free guidance can later combine logits as (1+lambda_t)*conditioned - lambda_t*unconditioned. Save the trained weights as a PyTorch checkpoint (pre-trained checkpoints available at zenodo.org/records/17115068).

## Related tools

- **MADGEN** (Second-stage generator: spectra-conditioned Markov bridge model that produces the target molecule starting from the retrieved scaffold) — https://github.com/HassounLab/MADGEN
- **RDKit** (Derive Murcko scaffolds from molecules to form (scaffold, molecule) training pairs and build the fully connected graph representation)
- **PyTorch** (Implement the FC-GNN / graph transformer and train the Markov bridge model; checkpoint saving and loading)
- **AdamW** (Optimizer for training the Markov bridge generator at learning rate 1e-5)
- **wget** (Download the processed msgym.pkl training data from Zenodo record 15036069)

## Examples

```
CUDA_VISIBLE_DEVICES=1 python sample.py --config configs/msgym.yaml --checkpoint checkpoints/msgym_best.pt --samples samples --model Madgen --mode test --n_samples 50 --n_steps 100 --table_name msgym_generation --sampling_seed 42
```

## Evaluation signals

- Checkpoint file exists and loads in PyTorch without error
- Training ELBO loss converges without divergence (qualitative acceptance, as the paper reports no numeric training-loss threshold)
- Top-K accuracy: percentage of cases where the true molecule appears in the top-K predictions via InChI key matching
- Maximum Tanimoto similarity between true molecule and top-K predictions using Morgan fingerprints (radius=2, 2048 bits)
- Scaffold coverage: percentage of cases with valid scaffold predictions (non-disconnected scaffolds)

## Limitations

- Scaffold retrieval errors propagate: performance drops when using a predictive retriever instead of an oracle retriever.
- MSNovelist comparison results are limited to accuracy metrics; other measures (e.g. similarity, MCES) are not available.
- NIST23 training data is commercially licensed and requires separate acquisition from NIST.
- Graph generation is restricted to edges involving scaffold atoms and free atoms; disconnections in the scaffold are not reconstructed.

## Evidence
